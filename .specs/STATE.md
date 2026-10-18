# STATE

## Decisions

### AD-001 — Ruby 4.0.7 e Rails 8.1.4 fixados no projeto
- **Decisão:** Ruby 4.0.7 fixado em `.tool-versions` (asdf) e `.ruby-version`; Rails 8.1.4 fixado em `Gemfile`/`Gemfile.lock`. Nada global.
- **Motivo:** versões mais recentes disponíveis em 2026-10-18; isoladas do resto da máquina.

### AD-002 — PostgreSQL como único banco
- **Decisão:** PostgreSQL (ActiveRecord) para todo o domínio e para a infraestrutura do Rails 8 (Solid Queue/Cache/Cable, Active Storage, autenticação gerada). Substitui a escolha inicial por MongoDB e o SQLite gerado pelo `rails new`.
- **Motivo:** escolha do usuário em 2026-10-18; evita banco híbrido e mantém o stack padrão do Rails 8.
- **Ambiente:** PostgreSQL 14.22 local. Credenciais vêm de variáveis de ambiente, nunca versionadas no repositório.

### AD-003 — Modelo totalmente relacional
- **Decisão:** todo o domínio é relacional, com tabelas associativas para relações N:N (ex.: `card_discipline_requirements (card_id, discipline_id, group)`, onde linhas do mesmo grupo são "e" e grupos distintos são "ou"). Sem colunas `jsonb` até F7.
- **Motivo:** escolha do usuário em 2026-10-18; integridade referencial, joins e consultas de sinergia em SQL.
- **Revisitar:** registro de eventos de partida (F8/F9), cujo conteúdo varia por tipo de jogada.

### AD-004 — Catálogo a partir do CSV VEKN; TWDA a partir da KRCG
- **Decisão:** CSV oficial VEKN é a fonte canônica de cartas; TWDA importada do JSON da KRCG.
- **Motivo:** confirmado pelo usuário; URLs respondendo em 2026-10-18.

### AD-005 — Coleção por impressão
- **Decisão:** item de coleção referencia a impressão (carta × expansão), não a carta.
- **Motivo:** confirmado pelo usuário; preserva expansão e arte.

### AD-006 — Motor de regras adiado
- **Decisão:** o simulador terá motor que aplica as regras, mas só na última fase (F9). Até lá nada de modelos de partida.
- **Motivo:** prioridade é o deckbuilder inteligente.

### AD-007 — Código em inglês, interface em pt-BR
- **Decisão:** código, rotas, tabelas, colunas e chaves I18n em inglês; todo texto exibido vem do locale `pt-BR`.
- **Motivo:** decisão do usuário em 2026-10-18.

## Handoff

- **Fase atual:** F0 `foundation` — spec aprovada; criando tarefas.
- **Próximo passo:** aprovar `.specs/features/foundation/tasks.md` e executar.
- **Git:** esqueleto Rails gerado e ainda não commitado.
