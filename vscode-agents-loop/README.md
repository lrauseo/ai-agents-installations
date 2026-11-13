# Loop de agentes para VS Code Agents Window (Copilot harness)

Este pacote configura quatro custom agents: Orchestrator -> Planner -> Implementer -> Reviewer -> (Implementer/Reviewer ate 2 rodadas) -> resultado.

## Instalar por projeto
Copie a pasta `.github/agents` para a raiz do repositorio. Abra o repositorio no VS Code.

## Instalar globalmente (Windows)
Copie os quatro arquivos `*.agent.md` para `%USERPROFILE%\.copilot\agents\` (crie a pasta caso necessario). A instalacao global dispensa copia em cada projeto.

## Usar
1. No VS Code, abra Command Palette > `Chat: Open Agents window` ou execute `code --agents`.
2. Crie sessao em `This Computer`, escolha o repositorio e use o harness `Copilot`.
3. Confira em `Customizations > Agents` se os quatro agentes foram detectados.
4. Selecione `Orchestrator` e envie uma tarefa com objetivo e criterios de aceite. Use permissao manual e, se aplicavel, worktree nova.
5. Acompanhe o historico de delegacao (Planner, Implementer, Reviewer), veja o diff e valide os testes antes de aprovar merge.

Prompt de teste:
> Use obrigatoriamente o Planner para inspecionar os arquivos relevantes antes de implementar. Depois use Implementer e Reviewer. Corrija apenas achados comprovados; no maximo 2 rodadas de correcao. Ao final, detalhe diffs, comandos executados e resultados. Tarefa: [descrever tarefa concreta].

## Modelo por agente
- Orchestrator: `gpt-6-luna` (coordenacao)
- Planner: `claude-opus-5.5` (planejamento/arquitetura)
- Implementer: `gpt-5.3-codex` (edicao e testes)
- Reviewer: `claude-sonnet-5.5` (revisao independente)

Esses identificadores sao referencias de modelos documentados no Copilot CLI em novembro de 2026; a disponibilidade depende do plano, regiao, habilitacao e harness. Se o editor marcar um modelo como indisponivel, substitua `model:` pelo ID indicado no autocomplete do arquivo `.agent.md` ou selecione um modelo disponivel pela interface. `model-policy: preferred` permite substituicao manual; altere para `required` somente se quiser bloquear o agente quando o modelo configurado estiver ausente.

## Limites importantes
- `agents:` e `handoffs:` do harness Local nao devem ser usados neste pacote Copilot: sao atributos nao suportados pelo Copilot no schema atual. O Orchestrator delega com `tools: ['agent']` e instrucao em linguagem natural; a execucao deve ser verificada.
- Os agentes sao templates de workflow, nao um scheduler deterministico: delegacao e repeticao sao guiadas pelo modelo, com limite de duas rodadas nas instrucoes.
- O campo `model` define preferencia conforme versao/servidor; verifique o modelo executado na atividade da sessao. Um bug reportado em setembro/outubro de 2026 foi corrigido no VS Code Insiders em 3/nov/2026, com milestone 1.146.0 para a correcao.
- Os arquivos foram validados estaticamente como YAML, mas nao foram executados no seu VS Code (nem acessam seu repositorio).

## Fontes oficiais
- https://code.visualstudio.com/docs/agents/run/agents-window
- https://code.visualstudio.com/docs/agents/reference/custom-agent-file-reference
- https://code.visualstudio.com/docs/agents/run/subagents
- https://code.visualstudio.com/docs/agent-customization/language-models
- https://github.com/microsoft/vscode/issues/338949
