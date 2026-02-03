# Fork Manager Skill

Skill para gerenciar forks de repositórios onde você contribui com PRs mas também usa as melhorias antes de serem mergeadas no upstream.

## Estrutura

```
fork-manager/
├── .git/                 # Repositório Git
├── .gitignore            # Ignora configs locais
├── SKILL.md              # Instruções completas para o agente
├── README.md             # Este arquivo
├── ARCHITECTURE.md       # Decisões de design e arquitetura
└── repos/
    ├── openclaw/         # Config do fork OpenClaw
    │   ├── config.json             (local only - não versionado)
    │   └── config.example.json     (template versionado)
    └── claude-mem/       # Config do fork Claude-mem
        ├── config.json             (local only - não versionado)
        └── config.example.json     (template versionado)
```

## Instalação

```bash
# 1. Clonar o repositório
git clone https://github.com/Glucksberg/fork-manager-skill.git /path/to/fork-manager

# 2. Criar configs locais a partir dos exemplos
cd /path/to/fork-manager
cp repos/openclaw/config.example.json repos/openclaw/config.json
cp repos/claude-mem/config.example.json repos/claude-mem/config.json

# 3. Editar com suas informações
vim repos/openclaw/config.json

# 4. Configurar para OpenClaw (global para todos os agentes)
openclaw config set skills.load.extraDirs '["/path/to/parent/directory"]'
# Exemplo: se fork-manager está em /home/dev/agents/fork-manager
# Adicione: ["/home/dev/agents"]

# 5. Configurar para Claude Code CLI
ln -s /path/to/fork-manager ~/.claude/skills/fork-manager
```

## Integração com CLIs

### OpenClaw
A skill é carregada via `extraDirs` no config global:
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

### Claude Code CLI
A skill é carregada via symlink:
```bash
~/.claude/skills/fork-manager → /path/to/fork-manager
```

**Importante:** Ambos os CLIs leem/escrevem no mesmo local, evitando duplicação.

## Comandos Disponíveis

- `status` - Verificar estado atual do fork
- `sync` - Sincronizar main com upstream
- `rebase <branch>` - Rebase de uma branch específica
- `rebase-all` - Rebase de todas as branches de PR
- `update-config` - Atualizar config com PRs atuais
- `build-production` - Criar branch de produção com todos os PRs
- `full-sync` - Sincronização completa (sync + rebase-all + build-production)

## Projetos Gerenciados

### OpenClaw
- **Upstream:** openclaw/openclaw
- **Fork:** Glucksberg/clawdbot
- **Local:** /home/dev/clawdbot
- **Branch de produção:** main-with-all-prs

### Claude-mem
- **Upstream:** anthropics/claude-mem
- **Fork:** Glucksberg/claude-mem
- **Local:** /home/dev/claude-mem
- **Branch de produção:** (conforme configurado)

## Uso

### Via OpenClaw
```bash
cd /tmp  # Qualquer diretório
openclaw
# "Use fork-manager skill para fazer full-sync do openclaw"
```

### Via Claude Code CLI
```bash
cd /tmp  # Qualquer diretório
claude
# "Use fork-manager skill para fazer full-sync do openclaw"
```

Leia o `SKILL.md` para instruções completas dos comandos disponíveis.

## Documentação

- **[README.md](README.md)** - Este arquivo (visão geral e instalação)
- **[SKILL.md](SKILL.md)** - Instruções completas para o agente
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Decisões de design e arquitetura

## Nomenclatura

- **OpenClaw** = CLI multi-provider (não é fork do Claude Code)
- **Claude Code CLI** = CLI oficial da Anthropic
- **moltbot** = nome antigo do projeto OpenClaw
- **clawdbot** = nome do fork local do OpenClaw (Glucksberg/clawdbot)
