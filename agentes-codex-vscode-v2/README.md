# Agentes Codex no VS Code

Selecione **Orquestrador Codex** no **Agents Window** do VS Code para iniciar o fluxo. A sessão principal coordena subagentes nativos e encerra após a primeira aprovação ou, se houver correções, após a segunda revisão.

## Instalar no projeto

1. Extraia o ZIP em uma pasta à parte. Copie as pastas `.codex` e `.github` para a raiz do repositório. Se já instalou a primeira versão do pacote, basta adicionar `.github/agents/orquestrador-codex.agent.md` nessa mesma raiz. O VS Code usa o Markdown para mostrar o orquestrador. O Codex usa os TOMLs para carregar os papéis delegados. [Descoberta de agentes no VS Code](https://github.com/microsoft/vscode/blob/main/src/vs/platform/agentHost/node/codex/codexCustomizations.ts#L43-L69) · [Agentes personalizados no Codex](https://learn.chatgpt.com/docs/agent-configuration/subagents#custom-agents)
2. Se já houver `.github` ou `.codex`, mescle as pastas e preserve os outros arquivos do projeto. Em um `.codex/config.toml` existente, ajuste `enabled`, `max_concurrent_threads_per_session` e `max_depth` na tabela `[agents]` existente. Se ela usar `max_threads`, troque essa chave por `max_concurrent_threads_per_session`. Os padrões `model` e `model_reasoning_effort` são chaves de nível superior. Você também pode manter os padrões existentes e definir o modelo da sessão principal pelo seletor.
3. No **Agents Window**, crie uma sessão em **This Computer** com a raiz do repositório como workspace. Selecione o harness **Codex** e o modo **Interactive**, quando disponível. Se aparecer uma solicitação de confiança, revise os arquivos e marque como confiável apenas o projeto que você reconhece. A configuração do projeto depende da confiança do Codex. [Configuração por projeto](https://learn.chatgpt.com/docs/config-file/config-advanced#project-config-files-codexconfigtoml) · [Harness Codex no VS Code](https://code.visualstudio.com/docs/agents/run/agent-harnesses#codex)
4. No seletor **Agent**, escolha **Orquestrador Codex**. Se sua interface colocar a opção dentro do menu de anexos, abra **Add Context (+) → Agent...**. No seletor de modelo, escolha **GPT-6.1 Sol** no grupo **ChatGPT**, com esforço **Medium**. Você também pode conferir a entrada em **Customizations → Agents** com a sessão do projeto selecionada. [Editor de personalizações do VS Code](https://code.visualstudio.com/docs/agent-customization/overview#agent-customizations-editor)

A escolha no grupo **ChatGPT** usa o acesso que você já conectou. Para testar com arquivos ainda fora do Git, selecione **Branch** e use a pasta atual. Para usar **New Worktree**, inclua os arquivos do pacote no branch que serve de base. [Seleção de conta](https://code.visualstudio.com/docs/agents/run/agent-harnesses#codex) · [Escolha da pasta de trabalho](https://code.visualstudio.com/docs/agents/run/agents-window#choose-where-an-agent-makes-changes)

## Iniciar uma tarefa

Com **Orquestrador Codex** selecionado, envie a tarefa, o escopo e os critérios de aceite. O orquestrador lê o TOML de instruções do projeto e chama os especialistas pelo nome. Você pode acompanhar `planner`, `implementer` e `reviewer` na atividade da sessão à medida que forem acionados. [Subagentes no VS Code](https://code.visualstudio.com/docs/agents/run/subagents)

Se o seletor de agente estiver indisponível, cole o prompt abaixo no chat principal de uma sessão Codex do projeto. Ele também serve para a extensão oficial do Codex e o Codex CLI.

```text
Atue como orquestrador na sessão principal. Leia
.codex/agents/orquestrador.toml e siga o campo developer_instructions.
Use os subagentes nativos planner, implementer e reviewer pelos nomes
configurados. Aguarde cada etapa e respeite o limite de duas revisões.

Tarefa: [descreva a mudança]
Escopo: [área ou arquivos que podem mudar]
Critérios de aceite: [resultados que devem ser verificados]
```

### Conferir a instalação

Se **Orquestrador Codex** estiver ausente do seletor, abra uma sessão nova na raiz do repositório e confirme que `.github/agents/orquestrador-codex.agent.md` está na pasta em que essa sessão trabalha. Abra **Customizations → Agents** para conferir se o VS Code encontrou o arquivo. Você pode iniciar o fluxo pelo prompt acima enquanto verifica o seletor.

Para testar só a delegação, abra uma sessão Codex normal na mesma pasta e peça `Acione apenas o subagente nativo planner para indicar os comandos de teste do repositório. Aguarde o resultado e preserve os arquivos.` Confira se a atividade identifica o papel `planner`. Se a interface mostrar o modelo e o esforço, confira também `gpt-6.1-sol` e `high`. Se `planner` estiver indisponível, confira a confiança do projeto, a presença de `.codex/agents/planner.toml` na pasta em uso e `agents.enabled`. [Subagentes no Codex](https://learn.chatgpt.com/docs/agent-configuration/subagents)

## Modelos e papéis

| Papel | Modelo | Esforço | Responsabilidade |
| --- | --- | --- | --- |
| `orquestrador` | `gpt-6.1-sol` | `medium` | Mantém o contexto, delega as etapas e decide quando encerrar. |
| `planner` | `gpt-6.1-sol` | `high` | Examina contratos e dependências e propõe verificações para os critérios de aceite. |
| `implementer` | `gpt-6.1-sol` | `medium` | Altera os arquivos do projeto e executa as verificações. |
| `reviewer` | `gpt-6-astra` | `medium` | Usa um modelo diferente para julgar a mudança e as evidências de forma independente. |

Sol é o padrão do pacote para trabalho de código com várias etapas. Astra fica reservado ao parecer. Se Astra estiver indisponível no grupo ChatGPT da sua sessão, ajuste `reviewer.toml` para `model = "gpt-6.1-sol"` e `model_reasoning_effort = "high"`. Para mudar os demais papéis delegados, edite os mesmos campos no TOML correspondente. Para mudar o padrão do orquestrador, ajuste o seletor da sessão e mantenha `.codex/config.toml` e `orquestrador.toml` alinhados com sua escolha. Os subagentes herdam o provedor da sessão. O modelo e o esforço definidos no arquivo do papel têm precedência para aquele subagente. [Modelos Codex](https://learn.chatgpt.com/docs/models) · [Configuração dos subagentes](https://learn.chatgpt.com/docs/agent-configuration/subagents#custom-agents)

## Ciclo e limites

```text
orquestrador
  planner → implementer → reviewer (1)
                              └─ se pedir ajustes → implementer → reviewer (2)
```

`APROVADO` encerra com as evidências disponíveis. `AJUSTES_NECESSÁRIOS` na primeira revisão autoriza uma rodada de correção. Na segunda, encerra com a lista de pendências. `BLOQUEADO` encerra a execução quando falta acesso, uma decisão ou evidência essencial. O fluxo deixa as alterações na pasta de trabalho para inspeção.

- `max_concurrent_threads_per_session = 5` limita a cinco as conversas de subagentes abertas simultaneamente por sessão. A conversa principal fica fora dessa contagem. O valor acomoda as cinco etapas possíveis mesmo quando conversas concluídas permanecem abertas. O pacote executa uma etapa por vez. `max_threads` é o nome legado. Com a chave omitida, o Codex usa o padrão do mecanismo ativo. [Referência de configuração](https://learn.chatgpt.com/docs/config-file/config-reference)
- `max_depth = 1` limita a profundidade no mecanismo V1. O V2 ignora essa chave. Os três papéis delegados recebem instruções para trabalhar sem novas delegações. [Esquema oficial de configuração](https://learn.chatgpt.com/docs/config-schema.json)
- O máximo de dois pareceres e cinco acionamentos por execução vem das instruções do orquestrador. O limite do TOML mede conversas abertas, enquanto o orquestrador conta etapas iniciadas ao longo da tarefa.
- `planner` e `reviewer` seguem instruções de somente leitura. As permissões efetivas são herdadas da sessão principal, incluindo a permissão de escrita usada pelo `implementer`. [Permissões dos subagentes](https://learn.chatgpt.com/docs/agent-configuration/subagents#approvals-and-sandbox-controls)

Formato e limites conferidos na documentação oficial em 13 de novembro de 2026.

## Git Manager
Selecione Git Manager diretamente no VS Code ou delegue pelo orquestrador. No Codex o papel nativo é `git_manager`; no Copilot, `Git Manager`. Instale os novos TOML/Markdown do pacote e abra nova sessão. O fluxo pergunta sempre no início sobre branch, nome/origem, PR/base, conflitos e commit/push. Merge efetivo exige autorização separada após revisão e checks. O Git Manager não substitui Implementer ou Reviewer.

### Checklist de validação manual
- Inicie uma orquestração e confira perguntas antes do Planner; negue branch e PR e confira ausência de escrita Git.
- Em repositório descartável, autorize branch com nome delegado e confira origem e preservação de alterações.
- Autorize PR sem merge; confira URL, base/head e PR aberto.
- Simule conflito: negue autorização e confira ausência de edição; autorize e confira diff, testes e nova revisão antes de publicar.
- Selecione Git Manager diretamente; teste falta de terminal/autenticação e confirme BLOCKED.
- Teste múltiplos repositórios e retomada sem repetir decisões inalteradas.

A execução depende do harness e das permissões locais. Os gates são instruções orientadas por modelo, não garantias determinísticas.
