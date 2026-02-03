# Fork Manager Skill

Skill para gerenciar forks de repositórios onde você contribui com PRs mas também usa as melhorias antes de serem mergeadas no upstream.

## Estrutura

```
fork-manager/
├── SKILL.md              # Instruções completas para o agente
├── README.md             # Este arquivo
└── repos/
    ├── openclaw/         # Config do fork OpenClaw
    │   └── config.json
    └── claude-mem/       # Config do fork Claude-mem
        └── config.json
```

## Locais Sincronizados

Esta skill está disponível em 3 locais via symlink:

1. **Origem (standalone):** `/home/dev/agents/fork-manager/`
2. **OpenClaw (clawdbot):** `/home/dev/clawdbot/skills/fork-manager/` → symlink
3. **Claude Code CLI:** Quando rodado dentro de clawdbot, enxerga automaticamente

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

Leia o `SKILL.md` para instruções completas de como usar esta skill.

## Nomenclatura

- **OpenClaw** = nome atual do projeto
- **moltbot** = nome antigo do OpenClaw
- **clawdbot** = nome do fork local (Glucksberg/clawdbot)
