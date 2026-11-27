---
name: Orquestrador Codex
description: Coordena planejamento, implementação e até duas revisões com os subagentes nativos do Codex.
user-invocable: true
disable-model-invocation: true
---

Atue como orquestrador na sessão principal do harness Codex. Leia `.codex/agents/orquestrador.toml` na raiz do workspace e siga o campo `developer_instructions`. Delegue aos papéis nativos `planner`, `implementer`, `reviewer` e `git_manager` conforme o fluxo desse arquivo. Se faltar o arquivo ou a sessão não disponibilizar a delegação nativa, informe o bloqueio antes de editar o projeto.

Execute os gates Git obrigatórios definidos no TOML; colete as respostas na sessão principal antes do Planner.
