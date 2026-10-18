# Foundation Specification

## Problem Statement

O esqueleto Rails gerado usa SQLite e não tem contas de usuário. Todas as fases
seguintes (coleção, decks, sugestões por estilo) dependem de PostgreSQL (AD-002,
AD-003) e de usuários autenticados com identidade pública. A F0 entrega essa base
e um gate de qualidade local que as próximas fases vão usar.

## Goals

- [ ] Aplicação rodando em PostgreSQL em desenvolvimento e teste, sem credenciais no repositório.
- [ ] Visitante consegue criar conta, confirmar o e-mail, entrar, sair e redefinir a senha.
- [ ] `bin/ci` passa localmente: estilo, auditorias de segurança e testes.

## Out of Scope

| Feature | Reason |
| ------- | ------ |
| CI remoto (GitHub Actions/GitLab CI) | Decisão do usuário: sem CI até existir remoto |
| Login social (OAuth), 2FA | Não pedidos para o MVP |
| Edição de perfil, troca de e-mail, exclusão de conta | Ficam para quando houver perfil público (F3+) |
| Papéis e área administrativa | Nenhuma fase atual precisa |
| SMTP de produção e deploy | Sem ambiente de produção nesta fase |
| Limpeza automática de contas não confirmadas | Volume irrelevante no MVP |

---

## Assumptions & Open Questions

| Assumption / decision | Chosen default | Rationale | Confirmed? |
| --------------------- | -------------- | --------- | ---------- |
| Framework de testes | Minitest (padrão Rails) | Já gerado; sem custo de troca | n |
| Idioma da interface | pt-BR padrão via I18n, com `rails-i18n` para mensagens de validação | Usuário escreve em pt-BR; textos de carta seguem em inglês | y |
| Idioma do código | Código, rotas, nomes de tabela e chaves I18n em inglês; textos exibidos só via locale `pt-BR` | Decisão do usuário em 2026-10-18 | y |
| Identidade da conta | Login por e-mail; `username` único e público | Decks públicos precisam de autor visível sem expor e-mail | n |
| Credenciais do banco | Variáveis padrão da libpq (`PGHOST`, `PGUSER`, `PGPASSWORD`) lidas pelo adapter `pg`; nada em `database.yml` | Zero gem extra; nada sensível versionado | n |
| Bancos de produção | `primary`, `cache`, `queue`, `cable` em PostgreSQL, espelhando a estrutura gerada | Mantém Solid Cache/Queue/Cable do Rails 8 | n |
| Entrega de e-mail em dev | Gem `letter_opener` (abre o e-mail no navegador) | Confirmação e reset testáveis sem SMTP | n |
| Validade do link de confirmação | 24 horas, uso único | Prazo usual; uso único impede reaproveitar link | n |
| Validade do link de reset de senha | 15 minutos (padrão do gerador Rails 8) | Mantém o padrão seguro do gerador | n |
| Regras de senha | Mínimo 8 caracteres, máximo 72 bytes (limite do bcrypt) | Equilíbrio entre segurança e atrito | n |
| Regras de username | 5–30 caracteres, `a-z 0-9 _ -`, salvo em minúsculas, único sem diferenciar maiúsculas | Mínimo de 5 definido pelo usuário; URL amigável e sem homônimos | y |
| Limite de tentativas | 10 tentativas por IP a cada 3 minutos em login, cadastro, reenvio de confirmação e reset (padrão do gerador) | Proteção contra força bruta com o mecanismo nativo `rate_limit` | n |
| Páginas protegidas na F0 | Início público; `/account` exige login e mostra e-mail e username | Uma rota protegida mínima para validar o bloqueio | n |
| GitHub workflow e dependabot gerados | Removidos | Usuário escolheu sem CI; evita arquivos sem uso | n |

**Open questions:** none - all resolved or logged above.

---

## User Stories

### P1: Banco PostgreSQL ⭐ MVP

**User Story**: Como desenvolvedor, quero a aplicação em PostgreSQL para que todas as fases usem o mesmo banco relacional.

**Why P1**: Todas as features dependem do banco.

**Acceptance Criteria**:

1. The system SHALL usar o adapter `postgresql` nos ambientes development, test e production (DB-01)
2. WHEN `bin/rails db:prepare` roda com `PGHOST`, `PGUSER` e `PGPASSWORD` definidos THEN the system SHALL criar os bancos `vtes_development` e `vtes_test` (DB-02)
3. The system SHALL manter `config/database.yml` sem usuário e senha literais (DB-03)
4. The system SHALL definir os bancos de produção `primary`, `cache`, `queue` e `cable` em PostgreSQL (DB-04)

**Independent Test**: `bin/rails db:prepare` cria os bancos; `bin/rails test` roda contra PostgreSQL; `grep` em `config/database.yml` não encontra senha.

---

### P1: Cadastro com confirmação de e-mail ⭐ MVP

**User Story**: Como visitante, quero criar uma conta e confirmar meu e-mail para depois guardar minha coleção e meus decks.

**Why P1**: Coleção e decks pertencem a um usuário.

**Acceptance Criteria**:

1. WHEN o visitante envia cadastro com e-mail, username, senha e confirmação válidos THEN the system SHALL criar o usuário com `confirmed_at` nulo e enviar um e-mail com link de confirmação (AUTH-01)
2. IF o cadastro tem campo inválido THEN the system SHALL responder 422 com as mensagens de erro em pt-BR, sem criar usuário e sem enviar e-mail (AUTH-02)
3. IF o e-mail já existe, com qualquer combinação de maiúsculas ou espaços nas pontas THEN the system SHALL recusar o cadastro com o erro "E-mail já está em uso" (AUTH-03)
4. IF o username já existe, com qualquer combinação de maiúsculas THEN the system SHALL recusar o cadastro com o erro "Username já está em uso" (AUTH-04)
5. The system SHALL garantir unicidade de e-mail e de username por índice único no banco (AUTH-05)
6. WHEN o usuário abre um link de confirmação válido THEN the system SHALL preencher `confirmed_at`, iniciar a sessão e redirecionar para a página inicial com o aviso "E-mail confirmado" (AUTH-06)
7. IF o link de confirmação está expirado, foi adulterado ou já foi usado THEN the system SHALL não alterar o usuário e mostrar "Link inválido ou expirado" com opção de reenviar (AUTH-07)
8. WHEN alguém pede reenvio de confirmação THEN the system SHALL mostrar a mesma mensagem neutra exista ou não o e-mail, e enviar o e-mail apenas se a conta existir e não estiver confirmada (AUTH-08)

**Independent Test**: Cadastrar, abrir o e-mail no letter_opener, clicar no link e cair logado na página inicial.

---

### P1: Entrar e sair ⭐ MVP

**User Story**: Como usuário confirmado, quero entrar e sair da minha conta.

**Why P1**: Base de todas as áreas pessoais.

**Acceptance Criteria**:

1. WHEN um usuário confirmado envia e-mail e senha corretos THEN the system SHALL criar a sessão e redirecionar para a página inicial (AUTH-09)
2. IF o e-mail não existe ou a senha está errada THEN the system SHALL não criar sessão e mostrar "E-mail ou senha inválidos" (AUTH-10)
3. IF um usuário não confirmado envia credenciais corretas THEN the system SHALL não criar sessão e mostrar "Confirme seu e-mail para entrar" com opção de reenviar (AUTH-11)
4. IF o mesmo IP passa de 10 tentativas em 3 minutos em login, cadastro, reenvio ou reset THEN the system SHALL recusar a tentativa e mostrar "Muitas tentativas. Tente novamente mais tarde." (AUTH-12)
5. WHEN o usuário logado clica em sair THEN the system SHALL destruir a sessão atual e redirecionar para a página de login (AUTH-13)
6. WHILE o visitante não está logado the system SHALL redirecionar qualquer acesso a `/account` para a página de login (AUTH-14)
7. WHEN o usuário logado acessa `/account` THEN the system SHALL mostrar seu e-mail e seu username (AUTH-15)

**Independent Test**: Entrar, ver `/account`, sair, e confirmar que `/account` redireciona para o login.

---

### P1: Redefinir senha ⭐ MVP

**User Story**: Como usuário que esqueceu a senha, quero redefini-la por e-mail.

**Why P1**: Sem isso, quem esquece a senha perde a conta.

**Acceptance Criteria**:

1. WHEN alguém pede redefinição de senha THEN the system SHALL mostrar a mesma mensagem neutra exista ou não o e-mail, e enviar o link apenas se a conta existir (AUTH-16)
2. WHEN o usuário envia nova senha válida por um link válido THEN the system SHALL trocar a senha, encerrar todas as sessões existentes desse usuário e redirecionar para o login (AUTH-17)
3. IF o link de redefinição está expirado ou adulterado THEN the system SHALL não alterar a senha e mostrar "Link inválido ou expirado" (AUTH-18)

**Independent Test**: Pedir reset, abrir o e-mail, trocar a senha, entrar com a nova senha; a sessão antiga em outro navegador cai.

---

### P2: Gate de qualidade local

**User Story**: Como desenvolvedor, quero um comando único que valide o projeto antes de cada commit.

**Why P2**: Substitui o CI remoto enquanto não existe remoto.

**Acceptance Criteria**:

1. WHEN `bin/ci` roda num repositório saudável THEN the system SHALL executar rubocop, bundler-audit, importmap audit, brakeman e `bin/rails test` e terminar com código 0 (QA-01)
2. IF qualquer etapa de `bin/ci` falha THEN the system SHALL terminar com código diferente de 0 (QA-02)
3. The system SHALL não conter `.github/workflows/` nem `.github/dependabot.yml` (QA-03)
4. The system SHALL usar pt-BR como locale padrão (QA-04)

**Independent Test**: `bin/ci` verde; introduzir uma ofensa de rubocop faz `bin/ci` falhar.

---

## Edge Cases

- IF dois cadastros simultâneos usam o mesmo e-mail THEN the system SHALL aceitar no máximo um, pelo índice único (AUTH-05)
- IF a senha tem mais de 72 bytes THEN the system SHALL recusar com erro de tamanho (AUTH-02)
- WHEN o usuário já logado abre a página de login ou cadastro THEN the system SHALL redirecionar para a página inicial (AUTH-19)

### Implicit-requirement sweep

| Dimensão | Cobertura |
| -------- | --------- |
| Validação e limites | AUTH-02..04, regras de senha e username |
| Falha parcial | N/A porque cadastro e confirmação são uma única escrita; e-mail é enviado após o commit |
| Idempotência / duplicados | AUTH-05, AUTH-07 (link de uso único) |
| Autenticação e rate limit | AUTH-09..14, AUTH-12 |
| Concorrência | AUTH-05 |
| Ciclo de vida dos dados | Validade dos links (24 h, 15 min); limpeza de não confirmados fora do escopo |
| Observabilidade | N/A além do log padrão; senhas e tokens já filtrados por `filter_parameter_logging` |
| Falha de dependência externa | N/A porque em dev o e-mail vai para o letter_opener; SMTP de produção fora do escopo |
| Integridade de estado | Não confirmado → confirmado é a única transição (AUTH-06, AUTH-11) |

---

## Requirement Traceability

| Requirement ID | Story | Phase | Status |
| -------------- | ----- | ----- | ------ |
| DB-01 | P1: Banco PostgreSQL | Tasks | Pending |
| DB-02 | P1: Banco PostgreSQL | Tasks | Pending |
| DB-03 | P1: Banco PostgreSQL | Tasks | Pending |
| DB-04 | P1: Banco PostgreSQL | Tasks | Pending |
| AUTH-01 | P1: Cadastro | Tasks | Pending |
| AUTH-02 | P1: Cadastro | Tasks | Pending |
| AUTH-03 | P1: Cadastro | Tasks | Pending |
| AUTH-04 | P1: Cadastro | Tasks | Pending |
| AUTH-05 | P1: Cadastro | Tasks | Pending |
| AUTH-06 | P1: Cadastro | Tasks | Pending |
| AUTH-07 | P1: Cadastro | Tasks | Pending |
| AUTH-08 | P1: Cadastro | Tasks | Pending |
| AUTH-09 | P1: Entrar e sair | Tasks | Pending |
| AUTH-10 | P1: Entrar e sair | Tasks | Pending |
| AUTH-11 | P1: Entrar e sair | Tasks | Pending |
| AUTH-12 | P1: Entrar e sair | Tasks | Pending |
| AUTH-13 | P1: Entrar e sair | Tasks | Pending |
| AUTH-14 | P1: Entrar e sair | Tasks | Pending |
| AUTH-15 | P1: Entrar e sair | Tasks | Pending |
| AUTH-16 | P1: Redefinir senha | Tasks | Pending |
| AUTH-17 | P1: Redefinir senha | Tasks | Pending |
| AUTH-18 | P1: Redefinir senha | Tasks | Pending |
| AUTH-19 | P1: Entrar e sair | Tasks | Pending |
| QA-01 | P2: Gate de qualidade | Tasks | Pending |
| QA-02 | P2: Gate de qualidade | Tasks | Pending |
| QA-03 | P2: Gate de qualidade | Tasks | Pending |
| QA-04 | P2: Gate de qualidade | Tasks | Pending |

**Coverage:** 27 total, 0 mapped to tasks, 27 unmapped ⚠️ (tarefas criadas após aprovação da spec)

---

## Success Criteria

- [ ] Fluxo completo cadastro → confirmação → login → `/account` → logout funciona no navegador.
- [ ] `bin/ci` termina com código 0.
- [ ] Nenhuma credencial de banco versionada.
