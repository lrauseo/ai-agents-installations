# Agentes Codex no VS Code · instalação global

Instale os agentes uma vez no Windows e use o fluxo em sessões locais do harness **Codex** no **Agents Window**.

## Instalar os agentes nativos

1. Extraia o ZIP em uma pasta temporária.
2. Copie a pasta `agents` para `%USERPROFILE%\.codex`, deixando os arquivos TOML em `%USERPROFILE%\.codex\agents`. Se o Agent Host usa um `CODEX_HOME` personalizado, copie para `agents` dentro dessa pasta. Preserve os arquivos existentes ao mesclar. [Agentes pessoais do Codex](https://learn.chatgpt.com/docs/agent-configuration/subagents#custom-agents)
3. Para usar os limites do pacote globalmente, adicione as chaves abaixo ao `config.toml` da pasta do Codex. Se já houver uma tabela `[agents]`, ajuste as chaves nela. Se existir `max_threads`, substitua essa chave pela atual. O limite de conversas abertas é aplicado às suas sessões Codex e o de profundidade vale para o mecanismo V1. Projetos confiáveis podem ter seus próprios ajustes. [Referência de configuração](https://learn.chatgpt.com/docs/config-file/config-reference) · [Esquema oficial](https://learn.chatgpt.com/docs/config-schema.json)

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 5
max_depth = 1
```

## Exibir o Orquestrador no seletor do VS Code

1. Copie a pasta `vscode-agents` do ZIP para `%USERPROFILE%\.codex`. O arquivo ficará em `%USERPROFILE%\.codex\vscode-agents\orquestrador-codex.agent.md`. Esse destino é lido pelo VS Code por meio da configuração a seguir. Mesmo com `CODEX_HOME` personalizado, você pode manter o Markdown nesse caminho.
2. No VS Code, abra **Preferences: Open User Settings (JSON)** pela paleta de comandos. Adicione esta propriedade ou mescle a entrada com a propriedade existente. O VS Code usa essa configuração de usuário para descobrir o Markdown e encaminhá-lo ao Agent Host. [Locais de agentes no VS Code](https://code.visualstudio.com/docs/agents/reference/ai-settings#custom-agents-settings)

```json
{
  "chat.agentFilesLocations": {
    "~/.codex/vscode-agents": true
  }
}
```

3. Abra uma sessão nova em **This Computer** com o repositório desejado e selecione **Codex**. No seletor **Agent** ou em **+ → Agent...**, escolha **Orquestrador Codex**. Selecione **GPT-6.1 Sol** no grupo **ChatGPT**, com esforço **Medium**, e envie a tarefa, o escopo e os critérios de aceite. [Agents Window](https://code.visualstudio.com/docs/agents/run/agents-window#start-an-agent-session)

## Uso e diagnóstico

O fluxo usa `planner` (GPT-6.1 Sol, alto), `implementer` (GPT-6.1 Sol, médio) e `reviewer` (GPT-6 Astra, médio). Se o Astra estiver indisponível na sessão, ajuste `agents/reviewer.toml` para `gpt-6.1-sol` com esforço `high`. O orquestrador encerra após uma aprovação ou após a segunda revisão, com uma única rodada de correção. [Modelos Codex](https://learn.chatgpt.com/docs/models)

- Para conferir a entrada do seletor, abra **Customizations → Agents** com a sessão Codex selecionada. Se a entrada estiver ausente, confira a configuração de **usuário** e se o caminho contém o `.agent.md`. [Editor de personalizações](https://code.visualstudio.com/docs/agent-customization/overview#agent-customizations-editor)
- Para testar só a delegação, peça no chat principal `Acione apenas o subagente nativo planner para indicar os comandos de teste do repositório. Aguarde o resultado e preserve os arquivos.` Confira a atividade da sessão. Se o papel estiver indisponível, confira a pasta do Codex usada pelo Agent Host e abra uma sessão nova. [Subagentes no VS Code](https://code.visualstudio.com/docs/agents/run/subagents)
- Se o seletor estiver indisponível, você pode iniciar o fluxo pelo chat. Peça `Atue como orquestrador na sessão principal. Leia o campo developer_instructions do arquivo agents/orquestrador.toml da minha pasta global do Codex e siga o fluxo. Use os subagentes nativos planner, implementer e reviewer.` Informe o caminho completo do TOML se estiver usando um `CODEX_HOME` personalizado.

Se o repositório já tiver o orquestrador da edição por projeto, podem aparecer duas entradas com o mesmo nome. Você pode conferir a origem em **Customizations → Agents**. A edição global inclui as instruções completas no Markdown para funcionar em repositórios sem o TOML do orquestrador na raiz. Ao alterar o fluxo global, mantenha o Markdown e `agents/orquestrador.toml` alinhados.

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
