# Foundation Context

**Gathered:** 2026-10-18
**Spec:** `.specs/features/foundation/spec.md`
**Status:** Ready for tasks

---

## Feature Boundary

PostgreSQL como único banco, contas de usuário (cadastro com confirmação, login,
logout, reset de senha) e gate de qualidade local via `bin/ci`.

---

## Implementation Decisions

### Cadastro

- Cadastro aberto; a conta só entra depois de confirmar o e-mail.

### CI

- Sem CI remoto por enquanto; o gate é `bin/ci` local.

### Banco

- PostgreSQL local (14.22) para tudo, inclusive Solid Cache/Queue/Cable em produção.

### Agent's Discretion

- Minitest, pt-BR via I18n, username público, credenciais por variáveis da libpq,
  letter_opener em dev (registrado como assumptions na spec).

### Declined / Undiscussed Gray Areas → Assumptions

- Validade dos links, regras de senha e username, limite de tentativas: ver
  Assumptions & Open Questions da spec.

---

## Specific References

- Gerador `bin/rails generate authentication` do Rails 8.1 como base.

---

## Deferred Ideas

- CI remoto quando houver repositório remoto.
- Limpeza de contas não confirmadas.
