# Roadmap — Plataforma VTES

## Visão

Plataforma para jogadores de Vampire: The Eternal Struggle, na linha do VDB e do
vtesdecks: catálogo de cartas, coleção pessoal e deckbuilder. O diferencial é um
**deckbuilder inteligente** que sugere cartas por sinergia e por uso em decks
vencedores de torneio (TWDA), ajustado ao estilo de cada jogador.

Futuramente: jogos online com motor de regras e estatísticas no estilo dotabuff.

## Fontes de dados

| Dado | Fonte | Papel |
| ---- | ----- | ----- |
| Cartas, expansões, impressões | CSV oficial VEKN (`vtescsv_utf8.zip`) | Fonte canônica do catálogo |
| Decks vencedores de torneio | TWDA via KRCG (`static.krcg.org/data/twda.json`) | Base de sinergia e popularidade |
| Regras | Rulebook V5 1.1 (out/2023) | Validação de deck e, no futuro, motor |

## Fases

Cada fase é uma feature com spec própria em `.specs/features/<feature>/`,
especificada só quando chegar a vez dela.

| # | Feature | Entrega | Depende de |
| - | ------- | ------- | ---------- |
| F0 | `foundation` | Troca SQLite → PostgreSQL, contas de usuário (cadastro/login), suíte de testes e CI | — |
| F1 | `card-catalog` | Importação do CSV VEKN; busca e filtros de cripta e biblioteca (nome, texto, clã, disciplina e nível, tipo, custos, capacidade, grupo, seita, título, traits, expansão); página da carta com impressões | F0 |
| F2 | `collection` | Coleção por impressão (quantidade, condição, idioma); importação/exportação | F1 |
| F3 | `deck-builder` | CRUD de decks; validação de legalidade (cripta ≥ 12, biblioteca 60–90, grupos consecutivos); estatísticas do deck; importação/exportação em texto; decks públicos; "tenho × falta" com a coleção | F1, F2 |
| F4 | `twda-ingest` | Importação da TWDA ligada ao catálogo; navegação e busca nos decks vencedores; arquétipos | F1 |
| F5 | `suggestions` | Sugestões por coocorrência e sinergia na TWDA, condicionadas ao deck em edição | F3, F4 |
| F6 | `player-style` | Perfil de estilo do jogador derivado dos seus decks; sugestões ponderadas por esse perfil | F5 |

### Futuro (fora do escopo atual)

| # | Feature | Observação |
| - | ------- | ---------- |
| F7 | `stats` | Estatísticas estilo dotabuff; depende de uma fonte de resultados de partidas |
| F8 | `online-play` | Lobby, mesa e partidas em tempo real |
| F9 | `rules-engine` | Motor que aplica as regras do rulebook e o texto das cartas |

O modelo de mesa (Game, GamePlayer, GameCard, GameEvent) levantado na análise do
rulebook fica reservado para F8/F9.
