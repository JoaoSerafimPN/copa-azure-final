# Guia do Aluno — A Grande Final (F5/F6: Chatbot MCP + Flow Visualizer) do zero

> **O que você vai construir nesta aula:** as duas últimas fases do FIFA 2026 Tickets, criando **do zero** os recursos novos e plugando-os ao ambiente das Quartas:
> - **F5 — a Voz:** um **McpServer** (7 ferramentas read-only) atrás do gateway YARP + um **chatbot Gemini** que consulta o estado REAL da Copa por conversa natural. A regra de ouro — "o chatbot nunca escreve no banco" — vale **por construção**, não por roteamento.
> - **F6 — a Visão:** o serviço **FlowEvents** (SignalR + Log Analytics) + o **Flow Visualizer** do frontend, onde uma compra real acende **5 nós** animados, rastreados de ponta a ponta por `correlationId`.
>
> **Importante (leia antes de começar):**
> - **Este lab ASSUME as Quartas no ar** (gateway YARP, identidade CIAM + admin workforce, backend v1, SQL). A Final **ADICIONA** dois microsserviços ao MESMO ambiente — **não** recria o gateway nem a identidade.
> - **Cada aluno cria TUDO no próprio Azure / GitHub**: seus recursos, com **seus próprios nomes**. Os valores deste guia são **genéricos** (`<sufixo>`, `<seu-rg>`, `<gateway-fqdn>`) — preencha os seus na tabela de convenção.
> - **O fork NÃO é o passo zero.** A infra dos serviços novos é criada **à mão** no Portal (Fases 1–7); o **fork + GitHub Actions é o ÚLTIMO passo de deploy** ([Fase 9](#fase-9--pr-do-lab--rodar-os-acao-na-ordem)).

> ⚠️ **A regra de ouro do dia:** no F5 o chatbot só tem **sentidos** (7 tools de leitura). Ele **não consegue** executar nenhuma ação — não existe uma ferramenta de escrita para o LLM chamar. Você vai **ver isso ao vivo** na [Fase 10](#fase-10--smokes-e-validação-o-coração-do-lab).

> **Referências:** Story [3.3](../stories/3.3.story.md) · [3.4](../stories/3.4.story.md) · [ADE-008 (re-arquitetura da Final)](../architecture/) · [ADE-004 (gateway issuer-agnóstico)](../architecture/) · Guia das Quartas [`quartas-f2-portal-guide.md`](./quartas-f2-portal-guide.md) · Workflow [`lab-a-final.yml`](../../.github/workflows/lab-a-final.yml)

---

## Como as peças se encaixam

Há **duas divisões de trabalho** bem distintas — a mesma lógica das Oitavas/Quartas:

| O quê | Como é feito | Onde |
|---|---|---|
| **INFRA nova** (Container Apps McpServer/FlowEvents, SignalR, Managed Identity, App Settings novas do gateway) | **À mão, no Portal do Azure** | Portal (Fases 1–7) |
| **CÓDIGO + FRONTEND** (imagens dos serviços, rebuild do gateway, bundle do front) | **GitHub Actions** (workflow único `Lab A Final`) | Seu fork (Fases 8–9) |

O que muda em relação às Quartas:

| | **Quartas (F2/F3)** | **A Final (F5/F6)** |
|---|---|---|
| Gateway YARP | você criou | **reusado sem mudança de infra** (só rebuild do código p/ pegar o hardening) |
| Identidade CIAM + admin | você criou | reusada |
| Backend v1 / SQL | reusado | reusado |
| **McpServer** (7 tools read-only) | — | **NOVO** — Container App **interno**, atrás do gateway |
| **Chatbot Gemini** | — | **NOVO** — no frontend, chave no **proxy server-side** |
| **FlowEvents** (SignalR + Kusto) | — | **NOVO** — Container App + Azure SignalR + Managed Identity |
| **Flow Visualizer** (`/flow`) | — | **NOVO** — 5 nós animados por `correlationId` |

```
        VOCÊ (Portal Azure — à mão)                       SEU FORK (GitHub Actions)
        ───────────────────────────                       ─────────────────────────
  ┌──────────────────────────────────────────────┐
  │  Reusa das Quartas:                           │
  │    ACR · Container Apps Environment (CAE)      │
  │    Gateway YARP · SQL · Frontend Web App       │
  │                                                │
  │  CRIA novo para a Final:                       │
  │    F5 → Container App McpServer (INTERNO)       │ ──┐  (você cria vazio)
  │    F6 → Container App FlowEvents (externo)      │   │
  │         + Azure SignalR (Free) + Managed Ident.│   │
  │  + App Settings novas no gateway               │   │
  └──────────────────────────────────────────────┘   │
                                                       ▼
                     ┌───────────────────────────────────────────────┐
                     │  Workflow ÚNICO "Lab A Final" (lab-a-final.yml) │
                     │  input `acao`:                                 │
                     │    mcp-server → build+deploy do McpServer       │
                     │    gateway    → rebuild do gateway (hardening)   │
                     │    flow-events→ build+deploy do FlowEvents        │
                     │    frontend   → chatbot + /flow embutidos        │
                     │    tudo       → mcp-server → gateway →           │
                     │                 flow-events → frontend           │
                     └───────────────────────────────────────────────┘
```

A regra de ouro da arquitetura: **o Portal cria os recursos vazios; os Actions só publicam código.** Nenhum recurso Azure é criado pelo workflow.

> 🟢 **Retro-compatibilidade:** nada das Quartas deixa de funcionar. A compra continua a mesma; a Final só **acrescenta** observação (chatbot que lê + visualizador que mostra). A notificação pós-compra passou a ser **inline** (dentro da Function Consumer), sem orquestração externa.

> 🔵 **Fluxo em runtime (F5):** front → `POST {gateway}/mcp` (Bearer CIAM) → gateway injeta `X-Entra-OID` + `X-Gateway-Key` → **McpServer** (`tools/list`, `tools/call`) → `SELECT` no SQL. A chave Gemini fica no **proxy** (`{gateway}/llm/gemini/...`), nunca no browser.
> 🔵 **Fluxo em runtime (F6):** compra atravessa Gateway YARP → Function Entry → Service Bus → Function Consumer → SQL; cada hop emite um trace com `correlationId`; o **FlowEvents** lê os traces (Kusto) e empurra por **SignalR** para a rota `/flow`, acendendo os 5 nós.

---

## Convenção de nomes (preencha a SUA)

Reuse os recursos das Quartas e crie os **novos** da Final. Anote os **seus** valores — todas as fases referenciam estes placeholders.

| Recurso | Convenção sugerida | Seu valor |
|---|---|---|
| Resource Group | `<seu-rg>` (reuse das Quartas) | ____________ |
| Container Registry (ACR) | `cr<sufixo>.azurecr.io` (reuse) | ____________ |
| Container Apps Environment | `cae-<sufixo>` (reuse) | ____________ |
| Container App (gateway) | `ca-gateway-<sufixo>` (reuse) | ____________ |
| FQDN do gateway | `<gateway-fqdn>` (das Quartas) | ____________ |
| Frontend Web App | `<seu-frontend>` → `https://<seu-frontend>.azurewebsites.net` (reuse) | ____________ |
| SQL Server / DB | `<seu-sql-server>` / `FIFA2026Tickets` (reuse) | ____________ |
| **Container App (McpServer)** | `ca-mcp-<sufixo>` — **NOVO, ingress interno** | ____________ |
| FQDN interno do McpServer | `<mcp-fqdn>` (gerado; termina em `.internal.<domínio-do-cae>`) | ____________ |
| **Container App (FlowEvents)** | `ca-flow-<sufixo>` — **NOVO** | ____________ |
| FQDN do FlowEvents | `<flow-fqdn>` (gerado) | ____________ |
| **Azure SignalR** | `signalr-<sufixo>` — **NOVO, tier Free** | ____________ |
| **Log Analytics Workspace** | reuse o das fases anteriores (App Insights) | ____________ |
| Workspace ID (GUID) do Log Analytics | `<workspace-id>` | ____________ |

> 💡 **Um único segredo de gateway (`X-Gateway-Key`):** você já gerou um `Gateway__AdminSharedSecret` nas Quartas. **Reuse exatamente o mesmo valor** aqui — ele agora também protege o hop gateway→McpServer (ver [Fase 3](#fase-3--app-settings-do-gateway-mcpserverurl--trava-x-gateway-key)). Se não tiver anotado, gere um novo (`openssl rand -hex 24`) e reaplique em TODOS os serviços confiáveis.

---

## Pré-requisitos (checklist de entrada)

- [ ] Ambiente das **Quartas no ar**: gateway YARP responde `GET /health` = 200; login CIAM funciona; compra v2 grava em `purchases`.
- [ ] ACR (`cr<sufixo>`) e o Container Apps Environment (`cae-<sufixo>`) existentes.
- [ ] **Chave Gemini** pronta (`GEMINI_API_KEY`) — você a gera na [Fase 0](#fase-0--conta-google--chave-gemini-ai-studio) (conta Google dedicada + AI Studio). Modelo do lab: **`gemini-2.5-flash`** (ver [Apêndice B](#apêndice-b--modelo-gemini-real-vs-comentário)).
- [ ] O valor do `Gateway__AdminSharedSecret` das Quartas anotado (ou um novo gerado).
- [ ] Fork NOVO do repo do evento com **TODAS as branches** (a branch do lab é `lab-a-final`; ver [Fase 9](#fase-9--pr-do-lab--rodar-os-acao-na-ordem)).

---

## Fase 0 — Conta Google + chave Gemini (AI Studio)

O chatbot da Final (F5) usa o **Google Gemini** para decidir qual das 7 tools chamar. A chave (`GEMINI_API_KEY`) é **parte do provisionamento** — você a gera **agora**, antes de subir qualquer serviço. Ela **nunca** entra no código: vai como **Secret do fork** na [Fase 8](#fase-8--fork-novo--variablessecrets-consolidados) e é usada **só pelo proxy server-side** (McpServer).

### 0.1 — Criar a conta Google dedicada ao lab

1. Abra uma **janela anônima/privada** do navegador (para não colidir com sua conta Google pessoal já logada).
2. Acesse **https://accounts.google.com/signup**.
3. Crie uma **conta Google nova, dedicada ao lab** — ex.: `copa.azure.lab.<suas-iniciais>@gmail.com`.

> 💡 **Por que uma conta dedicada?** para **isolar a cota e o faturamento** do free tier do Gemini — a chave fica presa a essa conta e a um **projeto novo**, sem misturar com sua conta pessoal. (Se o facilitador já mantém um Gmail do lab, dá para usar um alias `+` no e-mail de cadastro, ex.: `gmail-do-lab+final@gmail.com`; mas a isolação de cota que importa vem da **conta/projeto novo** — na dúvida, crie a conta dedicada.)

### 0.2 — Gerar a chave no Google AI Studio

1. Ainda logado **nessa conta**, acesse **https://aistudio.google.com/apikey**.
2. **Aceite os termos** do AI Studio.
3. Clique em **Create API key** → **Create API key in new project**.
4. **Copie** a chave e guarde num lugar seguro (gerenciador de senhas / bloco de notas local). Ela **não** vai para o código.

> 🔒 **A chave é server-side:** o `GEMINI_API_KEY` será usado **apenas pelo PROXY** (McpServer, `/llm/gemini/...`) — **nunca** no browser. O frontend só conhece a URL do proxy (`VITE_LLM_PROXY_URL` = o gateway).

✅ **Checkpoint:** você tem um `GEMINI_API_KEY` guardado **fora do código** (vai como Secret do fork na [Fase 8](#fase-8--fork-novo--variablessecrets-consolidados)) e sabe que o modelo do lab é **`gemini-2.5-flash`** (ver [Apêndice B](#apêndice-b--modelo-gemini-real-vs-comentário)).

---

## Fase 1 — Container App do McpServer (ingress INTERNO)

O McpServer é um microsserviço .NET 8 que expõe o endpoint **`/mcp`** (Streamable HTTP, JSON-RPC 2.0 pelo SDK oficial). Ele fica **atrás do gateway** — o browser **nunca** o chama direto. O gateway valida o Bearer Entra, injeta `X-Entra-OID` (identidade) e `X-Gateway-Key` (prova de origem), e roteia `/mcp` e `/llm/**` para ele.

Nesta fase você cria o Container App **vazio** (imagem placeholder). A imagem real vem pelo Actions na [Fase 9](#fase-9--pr-do-lab--rodar-os-acao-na-ordem).

### 1.1 Criar o Container App (Basics → Container → Ingress)

Tudo no **[portal.azure.com](https://portal.azure.com)**, na `<sua-subscription>` / `<seu-rg>`.

1. Busca do topo → **Container Apps → `+ Create`**.
2. **Basics:**

   | Campo | Valor | Por quê |
   |---|---|---|
   | **Container app name** | `ca-mcp-<sufixo>` | nome do McpServer (vira a Variable `PHASE05_MCP_APP_NAME`) |
   | **Environment** | `cae-<sufixo>` | o **MESMO** CAE do gateway (só quem está no mesmo CAE alcança um ingress interno) |

   → **Next: Container**.
3. **Container:**

   | Campo | Valor | Por quê |
   |---|---|---|
   | **Use quickstart image** | marcado | o ACR real vem pelo Actions; agora é só um placeholder |
   | **CPU / Memory** | menor preset | suficiente para o workshop |

   → **Next: Ingress**.
4. **Ingress:**

   | Campo | Valor | Por quê |
   |---|---|---|
   | **Ingress** | **Enabled** | o gateway precisa alcançá-lo |
   | **Ingress traffic** | **`Limited to Container Apps Environment`** | ⚠️ **INTERNO** — só o gateway, dentro do mesmo CAE, alcança; **sem endereço público** |
   | **Target port** | **`8080`** | obrigatório (`Dockerfile`: `EXPOSE 8080` + `ASPNETCORE_URLS=http://+:8080`); qualquer outra porta = **502** |

5. **Review + create → Create → Go to resource**.
6. Na **Overview**, copie a **Application Url** — é o seu `<mcp-fqdn>` (um host `*.internal.<região>.azurecontainerapps.io`). É o valor da App Setting `McpServerUrl` do gateway ([Fase 3](#fase-3--app-settings-do-gateway-mcpserverurl--trava-x-gateway-key)).

> 🔒 **Ingress INTERNO é o ponto de segurança do bloco:** o McpServer não tem endereço público. Só o gateway (mesmo CAE) fala com ele — e só com o `X-Gateway-Key` correto. Um `curl` externo direto no McpServer nem chega.

✅ **Checkpoint:** Container App `ca-mcp-<sufixo>` rodando (placeholder), **ingress interno** (`Limited to Container Apps Environment`) na **porta 8080**, e a **Application Url** (`<mcp-fqdn>`, host `.internal…`) anotada.

---

## Fase 2 — Conectar o ACR + App Settings do McpServer

### 2.1 Conectar o ACR

1. No Container App `ca-mcp-<sufixo>` → **Settings → Registries → `+ Add`**.
2. **Registry** = `cr<sufixo>.azurecr.io` · **Authentication** = **Admin Credentials** → **Save**.

### 2.2 App Settings do McpServer

O McpServer precisa da connection string do SQL, da chave Gemini (que o **proxy** injeta server-side) e do segredo do gateway. No Container App: **Application → Containers → `Edit and deploy`** → selecione o container → aba **Environment variables** → adicione (Source = **Manual entry**, ou **Reference a secret** para os sensíveis) → **Save → Create**:

| App Setting | Valor | Papel |
|---|---|---|
| `SqlConnectionString` | connection string ADO.NET do `FIFA2026Tickets` | as 7 tools fazem `SELECT` parametrizado (Dapper) |
| `GEMINI_API_KEY` | sua chave Gemini | injetada pelo **proxy** (`/llm/gemini/...`) — NUNCA no bundle |
| `GATEWAY_SHARED_SECRET` | **mesmo** valor do `Gateway__AdminSharedSecret` das Quartas | trava `X-Gateway-Key`: só aceita requests que passaram pelo gateway |

> ⚠️ **Dois caminhos para o `GATEWAY_SHARED_SECRET` — escolha UM:** ou você define aqui **manual** (App Setting no Portal), ou deixa o job `mcp-server` do `lab-a-final.yml` aplicá-lo como *secretref* a partir do Secret `GATEWAY_SHARED_SECRET` do fork ([Fase 8](#fase-8--fork-novo--variablessecrets-consolidados)). Não faça os dois pela metade — use **um** dos dois.

> 🔒 **Chave Gemini no server-side:** o frontend só conhece a URL do **proxy** (`VITE_LLM_PROXY_URL` = o gateway). O McpServer expõe `/llm/{provider}/{*path}`, injeta a `GEMINI_API_KEY` como header e encaminha ao endpoint oficial. Assim a key **nunca** vai para o browser — o próprio workflow tem um guard que falha se qualquer key vazar no bundle.
> 🟢 **Opcionais (fallback/portabilidade):** se quiser oferecer outros provedores, o McpServer também lê `GROQ_API_KEY` e `MISTRAL_API_KEY`. Para o lab, só a Gemini basta.

✅ **Checkpoint:** ACR conectado em **Registries**; McpServer com `SqlConnectionString`, `GEMINI_API_KEY` e (se optou pelo caminho manual) `GATEWAY_SHARED_SECRET` nas Environment variables.

---

## Fase 3 — App Settings do gateway (`McpServerUrl` + trava `X-Gateway-Key`)

O gateway já roteia para o McpServer — o `McpServerDestinationConfigFilter` **já existe** no `Program.cs` (Story 2.5, reusado sem mudança). Você só precisa dar a URL real e garantir o segredo. No Container App do **gateway** (`ca-gateway-<sufixo>`) → **Application → Containers → `Edit and deploy` → Environment variables**:

| App Setting | Valor | Papel |
|---|---|---|
| `McpServerUrl` | `https://<mcp-fqdn>` (Application Url da [Fase 1.1](#11-criar-o-container-app-basics--container--ingress)) | o filtro sobrescreve a destination do cluster `mcp-server` |
| `Gateway__AdminSharedSecret` | **mesmo** valor da [Fase 2.2](#22-app-settings-do-mcpserver) (já configurado nas Quartas) | injetado como `X-Gateway-Key` nos clusters confiáveis (`backend-v1`, `functions-f1`, **`mcp-server`**) |

> 🔒 **O P0 que a Final fecha:** a partir do hardening (Story 4.2 / ADE-009), o gateway injeta `X-Gateway-Key` também no cluster `mcp-server`. Um `curl` forjando `X-Entra-OID` direto no McpServer **não tem** o segredo e é rejeitado (401); via gateway, a request carrega o segredo real. Por isso é preciso **rebuildar o gateway** a partir da branch `lab-a-final` (`acao=gateway`, [Fase 9](#fase-9--pr-do-lab--rodar-os-acao-na-ordem)) — a imagem das Quartas ainda não tinha o `mcp-server` no conjunto confiável.
> 🔒 **Duplo underscore:** `Gateway:AdminSharedSecret` na config .NET vira `Gateway__AdminSharedSecret` em env var. Vazio no repo = injeção desligada (retro-compat com labs sem gateway).

> 🔵 **Roteamento e cache (só entendimento, nada a configurar):** o gateway roteia `/mcp` → cluster `mcp-server` e `/llm/{**}` → cluster `mcp-server` (o proxy de LLM que injeta a chave). Requisições `POST` **não são cacheadas** (o fix de cache separou o cache de `GET` das chamadas MCP/LLM). O cache de borda (30s) roda **pós-autenticação** (hardening da Story 4.4): um HIT só é servido depois que o JWT é validado.
> 🔵 **Identidade propagada (idem):** o gateway extrai o claim `oid` do token CIAM e injeta `X-Entra-OID` na request ao McpServer. As tools **leem** esse header apenas para **logging mascarado** — **nunca** revalidam o JWT (o gateway é o guardião único). É por isso que o McpServer pode ficar atrás do gateway sem reimplementar autenticação.

✅ **Checkpoint:** gateway com `McpServerUrl = https://<mcp-fqdn>` e `Gateway__AdminSharedSecret` presente (mesmo valor do McpServer). *(A trava só fica ativa depois do rebuild `acao=gateway` na Fase 9.)*

---

## Fase 4 — Azure SignalR (Free, Service Mode Default)

O FlowEvents empurra os eventos dos 5 nós para o browser via WebSocket, hospedando um **FlowHub** SignalR. Crie o serviço SignalR primeiro — a connection string dele alimenta o FlowEvents.

1. Portal → **SignalR → `+ Create`**.
2. **Basics:**

   | Campo | Valor | Por quê |
   |---|---|---|
   | **Resource name** | `signalr-<sufixo>` | vira o segredo `PHASE06_SIGNALR_CONNECTION_STRING` do fork |
   | **Region** | a **mesma** do CAE | proximidade com o FlowEvents |
   | **Pricing tier** | **Free** (Free_F1) | 20 conexões simultâneas — suficiente para o workshop |

3. **Review + create → Create → Go to resource**.
4. Em **Settings → Service Mode**, confirme **`Default`** (⚠️ **NÃO** `Serverless`) — o `FlowHub` é hospedado pelo próprio serviço FlowEvents (.NET, `AddAzureSignalR`), que exige o modo **Default**.
5. Em **Settings → CORS**, garanta que o **origin do frontend** (`https://<seu-frontend>.azurewebsites.net`) está permitido (o WebSocket do SignalR usa credentials — **não** pode ser `*`).
6. Em **Keys**, copie a **Connection String** — vira o segredo `PHASE06_SIGNALR_CONNECTION_STRING` do fork ([Fase 8](#fase-8--fork-novo--variablessecrets-consolidados)).

> 💡 IaC de referência (não obrigatório aplicar): [`infra/phase-06/signalr.bicep`](../../infra/phase-06/signalr.bicep) declara exatamente esse recurso (Free_F1, ServiceMode=Default, CORS restrito).

✅ **Checkpoint:** SignalR `signalr-<sufixo>` criado, **tier Free**, **Service Mode = Default**, **CORS** com o origin do front, e a **Connection String** anotada.

---

## Fase 5 — Container App do FlowEvents (ingress externo + WebSocket) + ACR

O FlowEvents é um microsserviço .NET 8 que consulta os traces via Kusto (Log Analytics) e empurra os eventos por SignalR. Diferente do McpServer, ele é **externo** — o front conecta o WebSocket a ele (via gateway). Crie o Container App vazio; a imagem real vem pelo Actions.

1. Portal → **Container Apps → `+ Create`**.
2. **Basics:**

   | Campo | Valor | Por quê |
   |---|---|---|
   | **Container app name** | `ca-flow-<sufixo>` | vira a Variable `PHASE06_FLOW_APP_NAME` |
   | **Environment** | `cae-<sufixo>` | o **MESMO** CAE |

   → **Next: Container**.
3. **Container:** mantenha **Use quickstart image** (a real vem pelo Actions) → **Next: Ingress**.
4. **Ingress:**

   | Campo | Valor | Por quê |
   |---|---|---|
   | **Ingress** | **Enabled** | o front conecta o WebSocket |
   | **Ingress traffic** | **`Accepting traffic from anywhere`** | **externo** (diferente do McpServer) |
   | **Transport** | **`Auto`** | habilita **WebSocket** para o SignalR |
   | **Target port** | **`8080`** | obrigatório (`Dockerfile`: `EXPOSE 8080`); outra porta = 502 |

5. **Review + create → Create → Go to resource**. Anote a **Application Url** = `<flow-fqdn>`.
6. **Conectar o ACR:** **Settings → Registries → `+ Add`** → `cr<sufixo>.azurecr.io` → **Authentication** = **Admin Credentials** → **Save**.

> 💡 IaC de referência: [`infra/phase-06/flow-events-containerapp.yaml`](../../infra/phase-06/flow-events-containerapp.yaml) (ingress external, transport auto, target port 8080, Managed Identity SystemAssigned, scale 0→2).

✅ **Checkpoint:** Container App `ca-flow-<sufixo>` rodando (placeholder), **ingress externo**, **Transport = Auto**, **porta 8080**, **ACR conectado** e a **Application Url** (`<flow-fqdn>`) anotada.

---

## Fase 6 — Managed Identity + Log Analytics Reader + App Settings do FlowEvents

O FlowEvents consulta os traces via Kusto usando uma **Managed Identity** — sem chave, sem senha.

### 6.1 Ligar a Managed Identity (System assigned)

1. No `ca-flow-<sufixo>` → **Settings → Identity → System assigned** → **Status = On** → **Save**.

### 6.2 Dar a role Log Analytics Reader (IAM)

1. Vá ao **Log Analytics Workspace** (o que recebe a telemetria do App Insights) → **Access control (IAM) → `+ Add → Add role assignment`**.
2. **Role** = **`Log Analytics Reader`** → **Next**.
3. **Assign access to** = **Managed identity** → **`+ Select members`** → selecione a identidade do `ca-flow-<sufixo>` → **Select** → **Review + assign**.
4. Anote o **Workspace ID** (GUID) do Log Analytics (**Overview** do workspace) → vira `PHASE06_LOG_ANALYTICS_WORKSPACE_ID` ([Fase 8](#fase-8--fork-novo--variablessecrets-consolidados)).

> ⚠️ Sem o papel **Log Analytics Reader**, o `LogsQueryClient` recebe **403** e os nós nunca acendem.

### 6.3 App Settings do FlowEvents

No `ca-flow-<sufixo>` → **Application → Containers → `Edit and deploy` → Environment variables**:

| App Setting | Valor | Papel |
|---|---|---|
| `AzureSignalRConnectionString` | connection string do SignalR ([Fase 4](#fase-4--azure-signalr-free-service-mode-default)) | hospeda o FlowHub (secretref) |
| `LogAnalyticsWorkspaceId` | `<workspace-id>` (Fase 6.2) | qual workspace consultar (Kusto) |
| `FrontendOrigin` | `https://<seu-frontend>.azurewebsites.net` | CORS do SignalR (credentials → não pode ser `*`) |
| `APPLICATIONINSIGHTS_CONNECTION_STRING` *(opcional)* | connection string do App Insights | telemetria de borda (no-op se ausente) |

> 🔒 **Nota de escopo (não confundir com o F5):** o cluster `flow-events` **NÃO** recebe o `X-Gateway-Key` (fica fora do escopo da ADE-009 Inv 1). Portanto **não** configure `GATEWAY_SHARED_SECRET` no FlowEvents — diferente do McpServer, que precisa dele.

✅ **Checkpoint:** MI **System assigned = On** no `ca-flow-<sufixo>`, role **Log Analytics Reader** atribuída no workspace, **Workspace ID** anotado e as App Settings `AzureSignalRConnectionString` + `LogAnalyticsWorkspaceId` + `FrontendOrigin` presentes.

---

## Fase 7 — App Setting do gateway (`FlowEventsUrl`)

O gateway já roteia FlowEvents — o `FlowEventsDestinationConfigFilter` **já existe** (Story 2.6, reusado). Só falta a URL real. No gateway `ca-gateway-<sufixo>` → **Environment variables**:

| App Setting | Valor | Papel |
|---|---|---|
| `FlowEventsUrl` | `https://<flow-fqdn>` ([Fase 5](#fase-5--container-app-do-flowevents-ingress-externo--websocket--acr)) | o filtro sobrescreve a destination do cluster `flow-events` |

O gateway expõe duas rotas para o front:
- `/flow-events/api/{**}` → API do FlowEvents (`/api/flow/recent`, `/{id}`, `/{id}/replay`);
- `/flow-events/hubs/{**}` → o Hub SignalR (WebSocket).

> 🔵 O gateway continua o **NÓ ZERO**: injeta `X-Correlation-ID` (transform global) também nas requests ao FlowEvents.

✅ **Checkpoint:** gateway com `FlowEventsUrl = https://<flow-fqdn>`. *(Assim como o `McpServerUrl`, passa a ser lido pela imagem após o rebuild `acao=gateway` da Fase 9.)*

---

## Fase 8 — Fork novo + Variables/Secrets consolidados

Toda a infra acima foi criada **à mão**. Agora vem a parte do fork. No **seu fork** → **Settings → Secrets and variables → Actions**. Os **nomes** são **fixos** (iguais para todos); os **valores** são os **seus** (placeholders da convenção).

### Variables

| Nome EXATO | Valor (seu) | Usada em (ação) |
|---|---|---|
| `ACR_LOGIN_SERVER` | `cr<sufixo>.azurecr.io` | mcp-server · gateway · flow-events |
| `PHASE02_RESOURCE_GROUP` | `<seu-rg>` | todos os deploys *(fallback interno `rg-hml-tik-cin-001` no YAML)* |
| `PHASE02_CONTAINERAPP_NAME` | `ca-gateway-<sufixo>` (o Container App do gateway das Quartas) | gateway (rebuild) |
| `PHASE05_MCP_APP_NAME` | `ca-mcp-<sufixo>` | mcp-server |
| `PHASE06_FLOW_APP_NAME` | `ca-flow-<sufixo>` | flow-events |
| `PHASE06_LOG_ANALYTICS_WORKSPACE_ID` | `<workspace-id>` | flow-events |
| `PHASE06_FRONTEND_ORIGIN` | `https://<seu-frontend>.azurewebsites.net` | flow-events |
| `FRONTEND_APP_NAME` | `<seu-frontend>` | frontend |
| `VITE_GATEWAY_V2_URL` | `https://<gateway-fqdn>` | frontend (base das rotas `/mcp`, `/llm`) |
| `VITE_LLM_PROXY_URL` | `https://<gateway-fqdn>` | frontend (proxy de LLM = o gateway) |
| `VITE_LLM_PROVIDER` | `gemini` | frontend (provider ativo do chatbot) |
| `VITE_GEMINI_MODEL` *(opcional)* | `gemini-2.5-flash` | frontend (override; default do código já é `gemini-2.5-flash`) |
| `VITE_FLOW_EVENTS_BASE_URL` | `https://<gateway-fqdn>/flow-events` | frontend (rota `/flow`) |

> 📌 **Modelo real:** o runtime do `gemini.ts` usa **`gemini-2.5-flash`** (o comentário de cabeçalho do arquivo ainda cita `2.0-flash` — inconsistência conhecida e inofensiva; ver [Apêndice B](#apêndice-b--modelo-gemini-real-vs-comentário)). Não precisa mexer no código.

### Secrets

| Nome EXATO | Conteúdo | Usada em (ação) |
|---|---|---|
| `AZURE_CREDENTIALS` | JSON do Service Principal com acesso ao RG | mcp-server · gateway · flow-events |
| `PHASE05_SQL_CONNECTION_STRING` | connection string ADO.NET do `FIFA2026Tickets` | mcp-server |
| `GEMINI_API_KEY` | sua chave Gemini | mcp-server (proxy server-side) |
| `GROQ_API_KEY` / `MISTRAL_API_KEY` *(opcionais)* | chaves de fallback | mcp-server |
| `GATEWAY_SHARED_SECRET` | **mesmo** valor do `Gateway__AdminSharedSecret` (gateway + McpServer) — o job `mcp-server` aplica como *secretref* no McpServer (equivale ao App Setting manual da [Fase 2.2](#22-app-settings-do-mcpserver)) | mcp-server (trava X-Gateway-Key) |
| `PHASE06_SIGNALR_CONNECTION_STRING` | connection string do Azure SignalR | flow-events |
| `AZURE_FRONTEND_PUBLISH_PROFILE` | publish profile do `<seu-frontend>` (SCM Basic Auth On **antes** de capturar) | frontend |

> ⚠️ **Pré-requisito — recrie as Variables/Secrets das Quartas NESTE fork novo (o build do frontend reusa o MESMO Web App).** A Final **acrescenta** o chatbot e a rota `/flow` ao mesmo bundle das Quartas — ela **não** recria o front. Como a [Fase 9](#fase-9--pr-do-lab--rodar-os-acao-na-ordem) manda **criar um fork NOVO** e as Variables/Secrets **não migram entre forks**, você precisa **criar também neste fork novo** — copiando os valores do seu fork das Quartas — as seguintes Variables que o job `frontend` injeta além das listadas acima: `VITE_CIAM_AUTHORITY`, `VITE_CIAM_CLIENT_ID`, `VITE_ADMIN_TENANT_ID`, `VITE_ADMIN_CLIENT_ID`, `VITE_ADMIN_SCOPE` (login CIAM + admin workforce), `GATEWAY_V2_URL`, `BACKEND_URL`, `FUNCTION_V2_URL` (gateway/backend/compra v2). **Se você não recriá-las aqui, o build passa verde mas publica um bundle com login CIAM e compra v2 mortos.** (O workflow aceita tanto o nome das Quartas quanto o prefixado da Final, ex.: `GATEWAY_V2_URL` **ou** `VITE_GATEWAY_V2_URL`; `FUNCTION_V2_URL` **ou** `VITE_FUNCTION_V2_URL`.)

✅ **Checkpoint:** as 13 Variables da Final + as 8 Variables herdadas das Quartas e os 8 Secrets criados no fork, com os nomes EXATOS acima. *(O job `frontend` tem um fail-fast que aborta se `VITE_CIAM_CLIENT_ID` ou `VITE_FUNCTION_V2_URL` estiverem vazios.)*

---

## Fase 9 — PR do lab + rodar os `acao` na ordem

Este é o **último bloco de deploy**: o Actions só **constrói e publica** imagens/código. A infra já existe (Fases 1–7).

### 9.1 Preparar o fork (tudo pela web do GitHub)

A branch do lab no repositório do evento (org **TFTEC**) chama-se **`lab-a-final`** — traz o workflow `lab-a-final.yml` + o código do F5/F6 (McpServer só-sentidos, FlowEvents 5 nós).

1. **Fork NOVO** do repo do evento, **com TODAS as branches** — na tela de fork, **desmarque** *Copy the `main` branch only* → **Create fork**. (⚠️ **Não reuse** o fork das Quartas: **Sync fork** só atualiza a `main` e **não traz branches novas**.)
2. **Habilite o workflow na `main` do seu fork:** abra um **Pull Request `lab-a-final` → `main`** (base = `main`, compare = `lab-a-final`) **no próprio fork** e faça o **merge**. Esse PR é o "exercício" da aula — ele faz o `lab-a-final.yml` aparecer no Actions. (Você nunca dá PR no repo da TFTEC.)

> 🖱️ **Disparo manual apenas:** o workflow só tem `workflow_dispatch` — nada roda até você clicar em **Run workflow** e escolher a ação. Antes do `frontend`, garanta **SCM Basic Auth `On`** no Web App do front e capture o publish profile **depois** disso.

### 9.2 Rodar o workflow — nesta ordem

Sempre em **Actions → "Lab A Final" → Run workflow → branch `main`** (já com o workflow após o merge da 9.1), variando o `acao`. A ordem (a mesma do `tudo`) é **`mcp-server` → `gateway` → `flow-events` → `frontend`**:

1. **`acao = mcp-server`** — `dotnet build/test` do McpServer, build & push da imagem no ACR (`cr<sufixo>.azurecr.io/mcp-server:<sha>`), `az containerapp update --image` (troca o placeholder) e aplica os App Settings sensíveis como secrets (`SqlConnectionString`, `GEMINI_API_KEY` e — se você optou pelo caminho do fork — `GATEWAY_SHARED_SECRET`; `GROQ`/`MISTRAL` só se presentes).
   > **O que esperar no log:** como o ingress do McpServer é **interno** (sem endereço público), o workflow **não** faz `curl /health` — ele confirma via `az` que a revisão ativa provisionou (`provisioningState = Provisioned`, `runningState = Running`). O smoke funcional (`tools/list` = 7 via gateway) é o passo manual da [Fase 10](#fase-10--smokes-e-validação-o-coração-do-lab).
2. **`acao = gateway`** — **rebuild do gateway** a partir de `lab-a-final` para pegar o hardening (`X-Gateway-Key` no cluster `mcp-server` + leitura de `FlowEventsUrl`). Troca a imagem do gateway; suas App Settings das Fases 3 e 7 permanecem.
   > **O que esperar no log:** step **"[gateway] Smoke test"** → `POST /purchase` sem token = **401** (fail-closed) + `GET /health` = **200**.
3. **`acao = flow-events`** — `dotnet build/test` do FlowEvents, build & push da imagem (`cr<sufixo>.azurecr.io/flow-events:<sha>`), `az containerapp update --image` + aplica `AzureSignalRConnectionString` (secretref), `LogAnalyticsWorkspaceId`, `FrontendOrigin`.
   > **O que esperar no log:** step **"[flow-events] Smoke test"** → `GET /health` com `.status == "healthy"` (ingress externo, então há `curl` público).
4. **`acao = frontend`** — `npm ci` + `npm run lint` + `vite build` (chatbot **e** rota `/flow` embutidos, com todas as `VITE_*`) + deploy no Web App.
   > **O que esperar no log:** step **"[frontend] Guard"** → `Guard OK — nenhuma key de LLM no bundle`. Se alguma key de LLM aparecer no bundle, o job **falha** de propósito (a key deve ficar só no proxy server-side).

> 🧩 **Origem dos blocos (reuso, não invenção):** `mcp-server` ← `deploy-phase-05.yml`; `gateway` ← `deploy-phase-02.yml` (é onde vive o deploy do Gateway YARP) + smoke fail-closed do `lab-quartas-de-final.yml`; `flow-events` ← `deploy-phase-06.yml`; `frontend` ← fusão dos jobs de front do phase-05 (chatbot + guard) e phase-06 (rota `/flow`); seletor `acao` ← `lab-quartas-de-final.yml`.

✅ **Checkpoint:** quatro jobs verdes na ordem `mcp-server → gateway → flow-events → frontend` (ou um `tudo`); revisões ativas apontando para as imagens `:<sha>` (não mais o placeholder); frontend publicado com chatbot + `/flow`.

---

## Fase 10 — Smokes e validação (o coração do lab)

Com tudo no ar, prove que o lab funciona — e viva o momento didático central (a regra de ouro ao vivo + os 5 nós).

### 10.1 Smoke do McpServer (tools/list = 7)

```bash
GW="<gateway-fqdn>"
TOKEN="<access-token-CIAM>"   # cole um Bearer CIAM válido (login no front → DevTools)

# tools/list via gateway → tem de listar EXATAMENTE 7 tools, todas readOnly
curl -s -X POST "https://${GW}/mcp" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' \
  -i | tee mcp-tools.txt
# Espere: 7 tools em result.tools[]; NENHUM cabeçalho X-Cache: HIT (POST /mcp não é cacheado)
```

As **7 tools** que devem aparecer (todas read-only):

| Tool | O que consulta |
|---|---|
| `consultar_disponibilidade` | disponibilidade e preços de ingressos de uma partida |
| `verificar_ingresso` | se um ingresso/ID é válido + dados da compra |
| `consultar_bracket` | jogos de uma fase do mata-mata (oitavas…final) |
| `consultar_partidas` | partidas com filtros (time, fase, estádio, grupo, data) |
| `consultar_classificacao` | tabela de pontos de um grupo |
| `consultar_time` | dados de uma seleção (grupo, ranking FIFA, código) |
| `consultar_estadio` | dados de um estádio/sede (cidade, capacidade) |

✅ **Checkpoint (AC-2/AC-8):** `tools/list` = **7 tools, todas `readOnly: true`**; `POST /mcp` sem `X-Cache: HIT`; McpServer com ingress **interno** (não responde por URL pública).

### 10.2 Chatbot: 3 perguntas em linguagem natural

Abra o portal e o **chatbot**. Ele descobre as 7 tools via `tools/list` e deixa o **Gemini** decidir qual chamar (function calling, modo `AUTO`). Faça pelo menos **3** perguntas e observe a tool escolhida:

| Você pergunta | Tool que o Gemini chama | Dado real retornado |
|---|---|---|
| *"Quando o Brasil joga?"* | `consultar_partidas` | jogos do Brasil (com placar se já disputado) |
| *"Como está o grupo A?"* | `consultar_classificacao` | tabela de pontos do grupo A |
| *"Me fala do Maracanã"* | `consultar_estadio` | cidade, capacidade, descrição do estádio |

> 🔎 Cada resposta vem do **SQL real** (via `FifaQueryRepository`, só `SELECT`). O chatbot não inventa: ele lê o banco através das tools.

✅ **Checkpoint (AC-3):** ≥3 das 7 tools demonstradas em conversa natural, com dados reais do SQL.

### 10.3 A regra de ouro AO VIVO (o momento central do F5)

Este é o clímax didático do F5. O facilitador pede à turma que tente uma pergunta de **AÇÃO**:

> *"Cria um alerta pra mim quando abrir ingresso VIP."*

E a turma observa, junto: **o chatbot não tem essa ferramenta.** O `tools/list` só expõe 7 tools de **leitura** — não existe nenhuma tool de escrita para o Gemini chamar. Não há vetor de escrita **por construção**.

Pontos a reforçar em sala:
- A "mão" de ação (uma antiga tool de criar alerta) **foi removida** — o McpServer é só "sentidos".
- Não é preciso explicar roteamento por fila/webhook para provar a segurança: **basta olhar a lista de ferramentas**. O que não existe não pode ser chamado.
- ⚠️ **Nuance honesta:** o LLM pode até *responder em texto* algo como "ok, criei o alerta". Isso é **alucinação de texto**, não uma tool call real — **nenhuma escrita ocorre** no banco. Deixe isso explícito: a "promessa" no texto não é uma ação; o único jeito de escrever seria uma tool call, e ela não existe.

✅ **Checkpoint (AC-4/AC-9):** a turma vê que o chatbot não executa ações; o material não menciona nenhuma "mão"/tool de escrita.

### 10.4 A bolinha atravessa 5 nós (o smoke central do F6)

1. Faça uma **compra v2** no portal (login CIAM → comprar um ingresso).
2. Navegue para **`/flow`**.
3. Observe a "bolinha" atravessar **exatamente 5 nós, em < 30s**, com o **mesmo `correlationId`** em cada hop:

| # | Nó | O que acontece |
|---|---|---|
| 0 | **Gateway YARP** | recebe a request, injeta `X-Correlation-ID` (nó zero do tracing) |
| 1 | **Function Entry** | `PurchaseEntryFunction` valida e publica no Service Bus |
| 2 | **Service Bus** | fila `tickets-purchase` (desacopla entrada e processamento) |
| 3 | **Function Consumer** | `PurchaseConsumerFunction` grava no SQL (idempotente) **e emite a notificação pós-compra INLINE** |
| 4 | **SQL** | linha gravada em `purchases.correlation_id` — fim do fluxo |

4. Abra o **Sheet de inspeção** de cada nó e confira o payload / `correlationId`.

### 10.5 A notificação inline (trade-off dos 5 nós)

No nó **Function Consumer** (nó 3), inspecione o payload e localize a **notificação pós-compra**: ela acontece **inline** (log estruturado correlacionado), **dentro** desse nó — **não tem nó próprio**.

> 🔵 **Por que 5 nós e não 6?** A re-arquitetura da Final **removeu a orquestração externa** de pós-compra: a notificação virou uma etapa **inline** da própria Function Consumer. Ganhamos simplicidade (menos peças, menos falhas, menos custo) ao preço de uma perda visual — a notificação não aparece como uma "bolinha" separada. É um trade-off consciente: a observabilidade da notificação vive no log correlacionado do nó Consumer.

✅ **Checkpoint (AC-4/AC-5/AC-8):** 5 nós exatos, `correlationId` ponta-a-ponta em < 30s; a notificação é encontrada **dentro** do nó Function Consumer; **zero** referência a um 6º nó ou a orquestração externa.

---

## Retrospectiva — o que você construiu (e por quê)

| Missão | O que provou |
|---|---|
| **Voz** (F5, McpServer) | uma IA pode consultar dados reais com segurança — a regra de ouro vale **por construção** (só 7 sentidos, zero escrita) |
| **Visão** (F6, Flow Visualizer) | observabilidade distribuída: uma compra rastreável ponta-a-ponta por `correlationId`, animada em 5 nós |
| **Blindar** (hardening) | o gateway é o guardião único: `X-Gateway-Key` fecha o bypass direto ao McpServer; cache pós-auth; chave Gemini nunca no bundle |
| **Simplificar** (re-arquitetura) | menos peças (notificação inline), menos custo, mesma funcionalidade — retro-compatível com Oitavas/Quartas |

## Perguntas para fechar (discussão em turma)

- Por que o McpServer tem **ingress interno** e o FlowEvents **externo**? (guardião único vs. serviço de leitura de telemetria consumido pelo front via gateway)
- Se alguém tentar `curl` direto no McpServer forjando `X-Entra-OID`, o que acontece? (401 — falta o `X-Gateway-Key`)
- Onde está a chave do Gemini? (no proxy server-side; o front só conhece a URL do proxy)
- Por que a notificação pós-compra não tem nó próprio? (trade-off da re-arquitetura: inline no Consumer)

## Quiz de encerramento

Feche a aula com o **quiz** (Google Forms — link fornecido pelo facilitador na sala): 8 perguntas rápidas sobre o que você construiu — MCP, RAG por tool-use, a regra de ouro por construção, `correlationId`/observabilidade, os 5 nós e a lição de simplificação (por que removemos a orquestração externa). Conteúdo-fonte das perguntas: [`docs/workshops/final/QUIZ.md`](../workshops/final/QUIZ.md).

> 🔗 **Link do quiz:** `<informado pelo facilitador>` (o Forms é criado fora do repositório, padrão das Quartas).

---

## Resumo do que você criou nesta aula

| Camada | Recursos / artefatos |
|---|---|
| F5 — Voz | Container App **McpServer** (ingress interno, 7 tools read-only) + chatbot Gemini (chave no proxy server-side) |
| F5 — Gateway | App Settings `McpServerUrl` + `Gateway__AdminSharedSecret` (X-Gateway-Key no cluster `mcp-server`) |
| F6 — Visão | Container App **FlowEvents** + **Azure SignalR** (Free/Default) + **Managed Identity** (Log Analytics Reader) |
| F6 — Gateway/Front | App Setting `FlowEventsUrl` + rota `/flow` (`VITE_FLOW_EVENTS_BASE_URL`) |
| Automação | Fork: Variables + Secrets + workflow único **Lab A Final** (`mcp-server`/`gateway`/`flow-events`/`frontend`/`tudo`) |
| Segurança | McpServer só-leitura por construção · chave Gemini nunca no bundle · X-Gateway-Key fecha o bypass · cache pós-auth |

---

## Apêndice A — Chave Gemini (AI Studio)

> ➡️ **Movido para a [Fase 0 — Conta Google + chave Gemini (AI Studio)](#fase-0--conta-google--chave-gemini-ai-studio)**, agora parte do provisionamento (antes da Fase 1). O passo a passo de criar a conta Google dedicada e gerar a chave está lá.

## Apêndice B — Modelo Gemini: real vs. comentário

- O **runtime** do `gemini.ts` usa `import.meta.env.VITE_GEMINI_MODEL ?? 'gemini-2.5-flash'` — ou seja, **`gemini-2.5-flash`** por default (sobrescrevível pela Variable `VITE_GEMINI_MODEL`).
- O **comentário de cabeçalho** do arquivo ainda menciona `models/gemini-2.0-flash` (o `2.0-flash` saiu do free tier). É uma **inconsistência de documentação pré-existente**, **inofensiva** e **fora do escopo** deste lab corrigir. Para o aluno, o que vale é o modelo real: **`gemini-2.5-flash`**.

## Apêndice C — Troubleshooting F5 (McpServer + chatbot)

| Sintoma | Causa provável | Mitigação |
|---|---|---|
| `tools/list` retorna **8** (não 7) | branch não parte do estado pós-Story 3.1 (McpServer só-sentidos) | confirme que `lab-a-final` está baseada em pós-3.1; deve haver **7** `[McpServerTool(..., ReadOnly = true)]` |
| **401** no `POST /mcp` mesmo com Bearer válido | `Gateway__AdminSharedSecret` (gateway) ≠ `GATEWAY_SHARED_SECRET` (McpServer), ou gateway não rebuildado | use o **mesmo** segredo nos dois e rode `acao=gateway` (o `mcp-server` só entrou no conjunto confiável no hardening) |
| **502** em `/mcp` | `McpServerUrl` ausente/errado no gateway, ou target port do McpServer ≠ 8080 | `McpServerUrl = https://<mcp-fqdn>` (Fase 3); ingress target port = **8080** |
| McpServer responde por **URL pública** | ingress criado como **External** (deveria ser interno) | recriar/ajustar ingress = **Limited to Container Apps Environment** (Fase 1.1) |
| Chatbot diz "chat indisponível" | `VITE_LLM_PROXY_URL` não setado no build | definir a Variable (= gateway) e re-rodar `acao=frontend` |
| Chatbot **inventa** uma resposta de ação | alucinação de texto do LLM (function calling não é 100% infalível) | reforçar: a "promessa" no texto **não** é uma tool call; nenhuma escrita ocorre — não há tool de escrita |
| `POST /mcp` retorna `X-Cache: HIT` | regressão do fix de cache do gateway | confirmar que a branch inclui o fix (POST não é cacheado) |
| Build do frontend falha no **guard de key** | uma key de LLM apareceu no bundle | a key deve ficar **só** no proxy server-side; remover qualquer uso direto no front |
| Chatbot responde mas sem dados reais | `SqlConnectionString` ausente/errada no McpServer | conferir o App Setting (Fase 2.2) |

## Apêndice D — Troubleshooting F6 (FlowEvents + Flow Visualizer)

| Sintoma | Causa provável | Mitigação |
|---|---|---|
| Diagrama mostra **6 nós** ou falta o "Gateway YARP" | branch não parte do estado pós-Story 3.1 (5 nós) | confirmar `flowNodes.ts` com **5** entradas; reconstruir `lab-a-final` do commit correto |
| Nós **nunca acendem** / erro 403 nos traces | Managed Identity sem **Log Analytics Reader** | conceder o papel à MI do `ca-flow-<sufixo>` no workspace (Fase 6.2) |
| Bolinha **para no nó 2** (Service Bus) | Consumer com backlog ou atraso de ingestão do Kusto (segundos) | aguardar; confirmar Function Consumer rodando |
| `correlationId` não aparece em nenhum nó | SignalR desconectado ou `VITE_FLOW_EVENTS_BASE_URL` incorreto | conferir a Variable (= `{gateway}/flow-events`) e a rota `/flow` conectando ao Hub |
| SignalR não conecta (WebSocket) | ingress do FlowEvents sem transport **Auto**, ou CORS sem o origin do front | ingress transport = **Auto** (Fase 5); CORS do SignalR + `FrontendOrigin` com o origin exato |
| **502** em `/flow-events/**` | `FlowEventsUrl` ausente no gateway | definir `FlowEventsUrl = https://<flow-fqdn>` (Fase 7) |
| SignalR recusa por tier | recurso criado em modo **Serverless** | recriar SignalR em **Service Mode Default** (Fase 4) |
| Aluno procura um **nó de notificação** dedicado | trade-off aceito (5 nós, notificação inline no Consumer) | reforçar didaticamente (Fase 10.5): a notificação está **dentro** do nó Function Consumer |
