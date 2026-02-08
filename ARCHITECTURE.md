# Fork Manager - Arquitetura e Decisões de Design

Este documento explica a arquitetura do fork-manager e as decisões tomadas durante seu desenvolvimento.

## Visão Geral

Fork Manager é uma skill reutilizável que funciona em múltiplos ambientes:

- **Claude Code CLI** (Anthropic)
- **OpenClaw** (multi-provider LLM CLI)

A skill não é um script executável, mas sim um **documento de instruções** (SKILL.md) que agentes LLM leem e executam.

## Estrutura de Diretórios

```
fork-manager/
├── .git/                           # Repositório Git
├── .gitignore                      # Ignora configs locais
├── README.md                       # Documentação do projeto
├── SKILL.md                        # Instruções para o agente
├── ARCHITECTURE.md                 # Este arquivo
└── repos/
    ├── openclaw/
    │   ├── config.json             # Local only (gitignored)
    │   └── config.example.json     # Template versionado
    └── claude-mem/
        ├── config.json             # Local only (gitignored)
        └── config.example.json     # Template versionado
```

## Decisões Arquiteturais

### 1. Configs são Local-Only

**Decisão:** Arquivos `config.json` não são versionados no Git.

**Razão:**

- Contêm informações específicas do ambiente de cada usuário
- Paths locais (`/home/dev/clawdbot`)
- Lista de PRs específicos do fork do usuário
- Histórico de sincronização pessoal

**Implementação:**

- `.gitignore` contém `repos/*/config.json`
- Templates `config.example.json` são versionados como documentação

### 2. Skill vs Plugin

**Decisão:** Fork-manager é uma **skill**, não um plugin.

**Diferença:**

- **Skill:** Documento MD que o agente lê e executa (sem código)
- **Plugin:** Adiciona novas ferramentas/capacidades ao CLI (com código)

**Razão:**

- Fork-manager apenas orquestra ferramentas existentes (git, gh)
- Não precisa de nova funcionalidade no CLI
- Mais simples e portátil entre diferentes CLIs

### 3. Estrutura Multi-Repo

**Decisão:** Um diretório por repositório em `repos/`.

**Razão:**

- Usuários podem gerenciar múltiplos forks
- Cada repo tem suas próprias configurações
- Escalável para adicionar novos repos facilmente

**Exemplos de uso:**

- `repos/openclaw/` - Fork do OpenClaw
- `repos/claude-mem/` - Fork do Claude-mem
- `repos/your-project/` - Futuros projetos

### 4. Distribuição via GitHub

**Decisão:** Repositório público no GitHub.

**Razão:**

- Facilita instalação via `git clone`
- Permite contribuições da comunidade
- Templates `.example.json` servem como documentação

**Uso:**

```bash
git clone https://github.com/Glucksberg/fork-manager-skill.git
cp repos/openclaw/config.example.json repos/openclaw/config.json
# Editar config.json com suas informações
```

## Integração com CLIs

### OpenClaw

**Método:** Config global via `extraDirs`

**Configuração:**

```json
// ~/.openclaw/openclaw.json
{
  "skills": {
    "load": {
      "extraDirs": ["/home/dev/agents"]
    }
  }
}
```

**Razão para extraDirs vs Workspace:**

- Fork-manager gerencia **múltiplos projetos** (openclaw + claude-mem)
- Não é específico de um único workspace
- Disponível para todos os agentes OpenClaw
- Evita redundância de symlinks

**Precedência no OpenClaw:**

1. `<workspace>/skills/` (highest)
2. `~/.openclaw/skills/`
3. Bundled skills
4. `extraDirs` (lowest)

### Claude Code CLI

**Método:** Symlink em `~/.claude/skills/`

**Implementação:**

```bash
ln -s /home/dev/agents/fork-manager ~/.claude/skills/fork-manager
```

**Razão:**

- Claude Code CLI não tem conceito de `extraDirs`
- Skills vêm de plugins ou `~/.claude/skills/`
- Symlink evita duplicação de código

## Fluxo de Dados

```
┌─────────────────────────────────────────┐
│  Origem (Única Fonte da Verdade)       │
│  /home/dev/agents/fork-manager/         │
│  - Git repository                       │
│  - Configs locais (não versionados)     │
└─────────────────────────────────────────┘
                  │
      ┌───────────┴───────────┐
      │                       │
      ▼                       ▼
┌──────────────┐      ┌──────────────────┐
│  OpenClaw    │      │  Claude Code CLI │
│  (extraDirs) │      │  (symlink)       │
└──────────────┘      └──────────────────┘
```

**Importante:** Ambos os CLIs leem/escrevem no **mesmo local**:

- Não há duplicação de dados
- Mudanças em qualquer CLI afetam a origem
- Git rastreia mudanças em um único lugar

## Casos de Uso

### Sync do OpenClaw

```bash
cd /tmp  # Qualquer diretório
openclaw
# "Use fork-manager skill para fazer full-sync do openclaw"
```

### Sync via Claude Code CLI

```bash
cd /tmp  # Qualquer diretório
claude
# "Use fork-manager skill para fazer full-sync do openclaw"
```

### Uso Standalone

```bash
cd /home/dev/agents/fork-manager
vim repos/openclaw/config.json
git add repos/openclaw/config.json
git commit -m "update: sync openclaw PRs"
```

## Evolução Futura

### Possíveis Melhorias

- [ ] Script auxiliar para automatizar operações comuns
- [ ] Templates para outros repos populares
- [ ] Integração com GitHub Actions para sync automático
- [ ] Plugin do Claude Code para comandos diretos

### Não Planejado

- ❌ Transformar em plugin com código executável
  - Mantém simplicidade como skill baseada em instruções
- ❌ Configurações versionadas
  - Configs permanecem locais e específicos do usuário

## Contribuindo

Para adicionar suporte a um novo repositório:

1. Criar diretório em `repos/novo-repo/`
2. Adicionar `config.example.json` com template
3. Documentar no README.md
4. Commit e PR

## Referências

- [AgentSkills Spec](https://agentskills.io) - Padrão de skills
- [OpenClaw Skills Docs](https://github.com/openclaw/openclaw/blob/main/docs/tools/skills.md)
- [GitHub Fork Manager](https://github.com/Glucksberg/fork-manager-skill)
