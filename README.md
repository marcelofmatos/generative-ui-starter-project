# generative-ui-starter-project (Docker)

Empacotamento em container do **Generative UI Starter** — um app de demonstração/starter que mostra
**UI generativa dirigida por agente**: o assistente monta e atualiza componentes de UI (kanban de
tarefas, cards de voo, dashboards com métricas/gráficos/tabelas) diretamente na tela, além do chat.
Front **Next.js** com **CopilotKit** + **AG-UI** e agente **LangGraph** (Python, OpenAI).

- **Upstream:** [Shubhamsaboo/awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps) — `generative_ui_agents/generative-ui-starter-project`
- **Imagem:** `ghcr.io/marcelofmatos/generative-ui-starter-project`
- **Stack Portainer:** [`marcelofmatos/portainer-stacks`](https://github.com/marcelofmatos/portainer-stacks) → `gen-ui-starter/`

## Arquitetura da imagem

Imagem **combinada** (um único container roda os dois processos):

| Processo | Porta | Papel |
|---|---|---|
| Next.js (`node server.js`, standalone) | `3000` | UI web (exposta) — proxia o chat para o agente via `AGENT_URL` |
| Agente (`python serve.py` / uvicorn) | `8123` | LangGraph servido via AG-UI; detém a chave de API; `GET /health` |

O `entrypoint.sh` sobe o agente em background e serve o front em foreground; se qualquer um dos dois
cair, o container encerra (o orquestrador reinicia). O `serve.py` embrulha o grafo LangGraph original
e o serve via protocolo AG-UI (sem `langgraph-cli`/Docker-in-Docker). Em Docker, a rota
`/api/copilotkit` é substituída por uma versão que usa o `HttpAgent` do AG-UI (`docker-route-override.ts`).

## Variáveis de ambiente

| Variável | Obrigatória | Default | Descrição |
|---|:---:|---|---|
| `OPENAI_API_KEY` | ✅ | — | Chave OpenAI usada pelo agente LangGraph |
| `AGENT_URL` | ❌ | `http://localhost:8123` | URL front→agente |

## Uso rápido (Docker)

```bash
docker run --rm -p 3000:3000 \
  -e OPENAI_API_KEY=sk-... \
  ghcr.io/marcelofmatos/generative-ui-starter-project:latest
# abra http://localhost:3000
```

Estado (kanban, cards) vive **em memória** por sessão no agente — não há banco nem volume.

## Build e release

Imagem publicada pelo workflow **🚀 Release and build** (`workflow_dispatch`): calcula a próxima
versão SemVer, cria tag + release e publica em `ghcr.io/marcelofmatos/generative-ui-starter-project`
com as tags `x.y.z`, `x.y`, `x` e `latest`. Build local:

```bash
docker build -t ghcr.io/marcelofmatos/generative-ui-starter-project:dev .
```

## Licença

Código sob **MIT** (licença própria do app; a coleção awesome-llm-apps é Apache-2.0). Ver [LICENSE](LICENSE).
