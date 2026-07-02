# Storyboard da Apresentação — A Grande Final (F5/F6)

> **O que é este arquivo:** um **prompt pronto para colar no Claude for PowerPoint**. Ele gera o deck da Grande Final (`.pptx`) no **mesmo DNA visual do deck real das Quartas** ("Quartas de final - Apresentação.pptx"): **11 slides**, tema escuro, uma tecnologia por slide, selos "TECNOLOGIA N DE 4", footer fixo.
>
> **Como usar:**
> 1. Abra o Claude for PowerPoint (ou Claude com a skill de PowerPoint).
> 2. **Cole todo o bloco abaixo** (da linha `═══ INÍCIO DO PROMPT ═══` até `═══ FIM DO PROMPT ═══`). O prompt é **auto-contido** — todo o texto de cada slide já está escrito por extenso; não é preciso dar acesso ao repositório.
> 3. Gere o `.pptx`. Ele **não** entra no git versionado (mesmo padrão de Oitavas/Quartas: o repo guarda o storyboard + o deck reveal `slides.md`; o binário é exportado à parte).
>
> **Fidelidade técnica (Art. IV — No Invention):** todo número, nome de tool, nó e componente já está fixado no prompt e bate com o código/guia reais: **7 tools read-only**, **5 nós**, notificação pós-compra **inline** na Function Consumer (**n8n removido** — aparece só como "o que removemos"), **`gemini-2.5-flash`**, chave Gemini **só no proxy server-side**, `X-Gateway-Key` fechando o bypass ao McpServer. Não altere esses números ao gerar o deck.

---

```
═══ INÍCIO DO PROMPT (cole tudo a partir daqui no Claude for PowerPoint) ═══

Gere uma apresentação PowerPoint (.pptx) de EXATAMENTE 11 SLIDES para o encerramento
de um workshop técnico ("A Grande Final" da "Copa do Mundo Azure 2026"). Siga com
precisão o layout e o DNA descritos abaixo — este deck é o gêmeo, na fase seguinte,
de um deck já existente das "Quartas de Final".

────────────────────────────────────────────────────────────────────────
DIRETRIZES VISUAIS GLOBAIS (valem para TODOS os slides)
────────────────────────────────────────────────────────────────────────
• Tema ESCURO (fundo quase preto), tipografia sem serifa, tom "produto de nuvem".
• Cores de acento:
   - VERDE-AZULADO (teal) → tudo de F5 / voz / leitura / read-only.
   - ROXO → tudo de F6 / visão / observabilidade.
   - VERMELHO → usar SÓ no realce de segurança (X-Gateway-Key / "por construção").
• FOOTER FIXO em todos os 11 slides (rodapé, uma linha, discreto):
   COPA DO MUNDO AZURE 2026   |   A GRANDE FINAL · F5/F6 — VOZ & VISÃO
• Nos slides de tecnologia (4, 5, 6, 7): um SELO no canto — "TECNOLOGIA N DE 4".
• Diagramas SEMPRE como caixas + setas simples (estilo "COMO FUNCIONA"), nunca
   screenshots. Sem blocos de código longos (o código vive no runbook do aluno).
• UMA tecnologia por slide. Densidade moderada: título forte, um diagrama, quatro
   recursos, uma caixa "▸ NESTA ETAPA". Frases curtas.
• Slides de CONCEITO-CHAVE (3, 8, 9): destaque visual — uma frase de efeito grande,
   entre aspas, como âncora do slide.

════════════════════════════════════════════════════════════════════════
SLIDE 1 — CAPA
════════════════════════════════════════════════════════════════════════
• Etiqueta (topo): A GRANDE FINAL · COPA DO MUNDO AZURE
• Título grande: Voz & Visão — a Copa que te responde
• Subtítulo narrativo (UMA frase): "Um chatbot que lê o estado real da Copa e uma
   tela onde cada compra se acende em cinco nós — com segurança por construção."
• Faixa de jornada (chips na base): Oitavas (F1) → Quartas (F2/F3) → **Final (F5/F6)**
• Footer padrão.

════════════════════════════════════════════════════════════════════════
SLIDE 2 — STACK DA FASE · "As tecnologias que vamos usar"
════════════════════════════════════════════════════════════════════════
• Rótulo grande: STACK DA FASE
• Subtítulo: As tecnologias que vamos usar (só o que é NOVO nesta fase)
• Cinco linhas (ícone + nome em destaque + uma frase cada):
   1. MCP (McpServer .NET) — protocolo que dá "ferramentas" ao LLM
      (tools/list / tools/call); server .NET, ingress INTERNO, atrás do gateway.
   2. RAG por tool-use (Gemini + 7 tools) — o chatbot RECUPERA o fato real via
      tool antes de responder; não inventa.
   3. Google Gemini (gemini-2.5-flash) — o LLM que decide QUAL tool chamar
      (function calling AUTO); chave server-side.
   4. Managed Identity (+ Key Vault como direção) — o FlowEvents lê a telemetria
      SEM segredo (role Log Analytics Reader).
   5. Azure SignalR (Free / Service Mode Default) — empurra eventos ao browser em
      tempo real (WebSocket) → o Flow Visualizer (5 nós).
• Nota de rodapé do slide (pequena): as fases anteriores já deram compra async,
   gateway, identidade, Container Apps e SQL — este deck NÃO os reexplica.
• Footer padrão.

════════════════════════════════════════════════════════════════════════
SLIDE 3 — CONCEITO-CHAVE · A regra de ouro (agora trivial)
════════════════════════════════════════════════════════════════════════
• Rótulo: CONCEITO-CHAVE
• Título: A regra de ouro, agora por CONSTRUÇÃO — o chatbot nunca escreve no banco
• Corpo (duas ideias curtas, contrastadas):
   - ANTES: a segurança de uma ação dependia de ROTEAR a ação por um caminho seguro
      (fila, orquestração, validação).
   - AGORA: o McpServer é SÓ SENTIDOS — não existe um vetor de escrita para o LLM
      chamar. A regra vale por construção, não por roteamento.
• Mini-visual (faixa horizontal): os 7 sentidos (todos de LEITURA) de um lado —
   consultar_disponibilidade · verificar_ingresso · consultar_bracket ·
   consultar_partidas · consultar_classificacao · consultar_time · consultar_estadio —
   e, do outro lado, um "✗ nenhuma tool de escrita" em vermelho.
• FRASE DE EFEITO (grande, entre aspas, âncora do slide):
   "O que não existe não pode ser chamado."
• Nuance honesta (linha pequena, importante): o LLM pode DIZER em texto "criei o
   alerta" — isso é alucinação de TEXTO, não uma tool call; nada é gravado.
• Footer padrão.

════════════════════════════════════════════════════════════════════════
SLIDE 4 — TECNOLOGIA 1 DE 4 · MCP (Model Context Protocol)
════════════════════════════════════════════════════════════════════════
• Título da tecnologia: MCP · MODEL CONTEXT PROTOCOL
• "O que é MCP?" (2-3 frases): Um protocolo padrão para expor "ferramentas" (tools)
   que um LLM DESCOBRE e CHAMA em runtime — o "USB-C das integrações de IA". Aqui, um
   McpServer .NET fica ATRÁS do gateway, com ingress interno; o browser nunca o alcança.
• Bloco COMO FUNCIONA (mini-fluxo com setas, caixas):
   Chatbot → [tools/list] descobre → [tools/call] executa (JSON-RPC 2.0) →
   McpServer (interno, atrás do gateway) → SELECT no Azure SQL
• Bloco PRINCIPAIS RECURSOS (4 itens, nome + frase):
   - tools/list — o LLM descobre, em runtime, as 7 ferramentas disponíveis.
   - tools/call (JSON-RPC 2.0) — a chamada tipada de uma ferramenta.
   - McpServer .NET (SDK oficial) — Container App de ingress INTERNO; sem URL pública.
   - 7 tools read-only — todas [McpServerTool(ReadOnly = true)]; SELECT parametrizado
      (Dapper). Nenhuma escreve.
• Caixa "▸ NESTA ETAPA": subir o McpServer (Container App interno, porta 8080) e ver
   tools/list retornar EXATAMENTE 7 sentidos, todos readOnly: true.
• Selo: TECNOLOGIA 1 DE 4. Acento teal. Footer padrão.

════════════════════════════════════════════════════════════════════════
SLIDE 5 — TECNOLOGIA 2 DE 4 · RAG por tool-use (Gemini + 7 tools)
════════════════════════════════════════════════════════════════════════
• Título da tecnologia: RAG POR TOOL-USE
• "O que é RAG?" (2-3 frases): Retrieval-Augmented Generation — o modelo RECUPERA um
   fato de uma fonte externa ANTES de responder, em vez de "lembrar" (e inventar). Aqui
   NÃO é vector store: é grounding via MCP (SELECT no banco, não embeddings). Mesmo
   princípio — recuperar antes de gerar — implementação diferente.
• Bloco COMO FUNCIONA (mini-fluxo):
   "Quando o Brasil joga?" → Gemini decide a tool (function calling AUTO) →
   consultar_partidas → SQL real → resposta FUNDAMENTADA
• Bloco PRINCIPAIS RECURSOS (4 itens):
   - Function calling (AUTO) — o Gemini decide QUAL das 7 tools chamar.
   - Grounding — a resposta vem do BANCO, não do conhecimento paramétrico do modelo.
   - Chave server-side — a GEMINI_API_KEY fica no PROXY /llm, nunca no browser; um
      guard falha o build se a key vazar no bundle.
   - gemini-2.5-flash — o modelo do lab.
• Caixa "▸ NESTA ETAPA": fazer ≥3 perguntas ao chatbot e ver, no painel, QUAL tool o
   Gemini chamou em cada uma.
• Selo: TECNOLOGIA 2 DE 4. Acento teal. Footer padrão.

════════════════════════════════════════════════════════════════════════
SLIDE 6 — TECNOLOGIA 3 DE 4 · Managed Identity
════════════════════════════════════════════════════════════════════════
• Título da tecnologia: MANAGED IDENTITY
• "O que é Managed Identity?" (2-3 frases): Uma identidade do Azure AD gerenciada pela
   plataforma — o serviço se autentica SEM senha e SEM segredo no código. Aqui, o
   FlowEvents usa a MID para ler a telemetria da compra.
• Bloco COMO FUNCIONA (mini-fluxo):
   FlowEvents (System-assigned MI) → role Log Analytics Reader no workspace →
   LogsQueryClient consulta os traces (Kusto)
• Bloco PRINCIPAIS RECURSOS (4 itens):
   - System-assigned — a identidade nasce e morre com o Container App.
   - RBAC — recebe o papel Log Analytics Reader (o mínimo necessário).
   - Sem credencial — nenhuma connection string de telemetria a guardar/rotacionar.
   - Fail-visível — sem o papel, o LogsQueryClient toma 403 e os nós NUNCA acendem.
• Caixa "▸ NESTA ETAPA": ligar a Managed Identity do ca-flow (FlowEvents) e conceder
   Log Analytics Reader no workspace.
• Selo: TECNOLOGIA 3 DE 4. Acento roxo. Footer padrão.

════════════════════════════════════════════════════════════════════════
SLIDE 7 — TECNOLOGIA 4 DE 4 · Azure SignalR + observabilidade (Flow Visualizer)
════════════════════════════════════════════════════════════════════════
• Título da tecnologia: AZURE SIGNALR · OBSERVABILIDADE AO VIVO
• "O que é?" (2-3 frases): Um serviço de leitura de telemetria (FlowEvents) + Azure
   SignalR que empurra eventos ao browser em tempo real (WebSocket) → o Flow Visualizer,
   onde uma "bolinha" atravessa CINCO nós animados pela mesma compra.
• Bloco COMO FUNCIONA (mini-fluxo):
   traces (correlationId) → FlowEvents lê via Kusto → TraceEventMapper classifica cada
   trace num nó → SignalR → a rota /flow ACENDE os 5 nós
• Bloco PRINCIPAIS RECURSOS (4 itens):
   - Trace-driven — o motor lê traces CORRELACIONADOS; é agnóstico a quem os emitiu.
   - Azure SignalR (Free_F1) — Service Mode DEFAULT (⚠ não Serverless — o FlowHub é
      hospedado pelo serviço).
   - CORS restrito — o WebSocket usa credentials → origin EXATO do front, nunca "*".
   - 5 nós — a bolinha atravessa a jornada em < 30s, pelo mesmo correlationId.
• Caixa "▸ NESTA ETAPA": criar o SignalR (Free/Default) + o FlowEvents (Container App
   EXTERNO, transport Auto), fazer uma compra real e ver /flow acender os 5 nós.
• Selo: TECNOLOGIA 4 DE 4. Acento roxo. Footer padrão.

════════════════════════════════════════════════════════════════════════
SLIDE 8 — CONCEITO-CHAVE · Key Vault, o destino de produção
════════════════════════════════════════════════════════════════════════
• Rótulo: CONCEITO-CHAVE
• Título: A mesma identidade que lê a telemetria elimina o segredo em claro
• Corpo (contraste hoje → amanhã):
   - HOJE, no lab: os segredos (SQL, Gemini, SignalR) são App Settings / secretref do
      Container App.
   - EM PRODUÇÃO (EPIC-004 — próximo passo, honestamente NÃO cabeado neste lab): esses
      segredos deveriam virar Key Vault references resolvidas pela Managed Identity, e o
      SQL usar Authentication=Active Directory Managed Identity.
• Mini-visual (seta): [Managed Identity] —(hoje)→ lê a telemetria · —(amanhã)→ resolve
   os segredos do Key Vault.
• FRASE DE EFEITO (grande, entre aspas):
   "A mesma Managed Identity que lê a telemetria é a que, amanhã, apaga o segredo em claro."
• Honestidade arquitetural (linha pequena): registrado como débito conhecido em
   docs/security/final-security-debt.md. O lab ENSINA a Managed Identity; o Key Vault é
   a DIREÇÃO, não um passo entregue aqui.
• Acento roxo/vermelho. Footer padrão.

════════════════════════════════════════════════════════════════════════
SLIDE 9 — CONCEITO-CHAVE · Onde foi o n8n? (simplificar > substituir)
════════════════════════════════════════════════════════════════════════
• Rótulo: CONCEITO-CHAVE
• Título: Removemos um componente — NÃO o substituímos
• Corpo: no desenho original, um 6º nó faria a orquestração da notificação pós-compra.
   Ele NÃO existe mais: a notificação virou uma etapa INLINE dentro da Function Consumer
   (o nó 3).
• Mini-tabela (antes → agora):
   | | Antes | Agora (Final) |
   | Pós-compra | orquestração externa (Container App + Postgres) | inline na Function Consumer |
   | Nós no visualizer | 6 | 5 |
   | Rastreabilidade | trace correlacionado | trace correlacionado (igual) |
• FRASE DE EFEITO (grande, entre aspas):
   "É a Function que orquestra o pós-compra."
• Trade-off honesto (linha pequena): a notificação fica INVISÍVEL na animação (dobrada
   no nó 3); a observabilidade dela vive no log correlacionado. Ganhamos simplicidade —
   menos peças, menos falhas, menos custo. Cinco nós, não seis.
• ⚠ Regra de linguagem para quem gera/apresenta: NUNCA escrever "automação no-code" nem
   citar orquestração externa como algo PRESENTE — na Final ela não existe.
• Acento teal/roxo. Footer padrão.

════════════════════════════════════════════════════════════════════════
SLIDE 10 — ARQUITETURA · A foto completa da Final
════════════════════════════════════════════════════════════════════════
• Rótulo: ARQUITETURA
• Título: A foto completa da Grande Final (pós-ADE-008)
• DIAGRAMA (caixas + setas, três trilhas partindo do mesmo gateway):
   [Browser SPA]
        │
        ▼
   [Gateway YARP]  — NÓ 0 · guardião único · X-Entra-OID · X-Gateway-Key
        ├── F5 VOZ:  → [McpServer] (interno · 7 sentidos) → [Azure SQL] (SELECT)
        │            → proxy /llm → [Gemini] (chave server-side)
        ├── COMPRA (5 nós): [Gateway 0] → [Function Entry 1] → [Service Bus 2]
        │                    → [Function Consumer 3 · notificação INLINE] → [Azure SQL 4]
        └── F6 VISÃO: [FlowEvents] (Managed Identity → Kusto por correlationId)
                       → [Azure SignalR] → rota /flow
• LEGENDA NUMERADA do fluxo (abaixo do diagrama):
   ① a VOZ — a pergunta vira uma tool call; o McpServer só LÊ (7 sentidos).
   ② a COMPRA — 5 nós, um correlationId; a notificação é inline no nó 3.
   ③ a VISÃO — traces correlacionados → SignalR → /flow acende os 5 nós.
• Linha-resumo (destaque): ZERO n8n, ZERO PostgreSQL. O gateway é o nó 0 e o guardião
   único. Tudo retro-compatível com Oitavas/Quartas — nada quebrou.
• Footer padrão.

════════════════════════════════════════════════════════════════════════
SLIDE 11 — ENCERRAMENTO DA JORNADA
════════════════════════════════════════════════════════════════════════
• Rótulo: ENCERRAMENTO DA JORNADA
• Título: Você concluiu a Copa do Mundo Azure
• Quatro bullets (o que foi construído — cada um com um ícone/acento):
   • VOZ (F5) — um chatbot MCP + RAG que consulta o estado real da Copa: 7 sentidos,
      zero escrita, segurança por construção.
   • VISÃO (F6) — observabilidade ao vivo: uma compra animada em 5 nós por correlationId
      (Azure SignalR + Managed Identity).
   • BLINDAR — o gateway é o guardião único: X-Gateway-Key fecha o bypass direto ao
      McpServer; a chave do Gemini nunca vai no bundle.
   • SIMPLIFICAR — menos peças (notificação inline, n8n removido), mesma função,
      retro-compatível com as fases anteriores.
• Fala de fechamento (uma frase, destaque): "Você começou com uma compra de ingresso e
   terminou com um sistema Azure-native completo — construído do zero, com as próprias
   mãos. Isso é uma Grande Final."
• Footer padrão.

════════════════════════════════════════════════════════════════════════
LEMBRETES FINAIS PARA A GERAÇÃO
════════════════════════════════════════════════════════════════════════
• São 11 slides — nem mais, nem menos.
• Slides 4, 5, 6, 7 são as tecnologias, cada um com o selo "TECNOLOGIA N DE 4",
   diagrama "COMO FUNCIONA", 4 recursos e a caixa "▸ NESTA ETAPA".
• Slides 3, 8, 9 são os CONCEITOS-CHAVE — dê o maior destaque visual às frases de efeito.
• Slide 10 é a arquitetura (diagrama + legenda numerada); slide 11 é o encerramento
   celebrativo da jornada INTEIRA (não só da fase).
• NÃO invente tools, nós ou números: são 7 tools read-only, 5 nós, notificação inline,
   n8n removido, gemini-2.5-flash, chave Gemini no proxy, X-Gateway-Key.

═══ FIM DO PROMPT ═══
```

---

## Nota de manutenção (para o time, NÃO faz parte do prompt)

- **Origem do DNA:** o layout acima replica o `.pptx` real das Quartas ("Quartas de final -
  Apresentação.pptx", 11 slides) mostrado pelo owner: capa → stack → conceito-chave central →
  4× tecnologia → 2× conceito-chave de aprofundamento → arquitetura → encerramento.
- **Mapeamento Quartas → Final (por posição de slide):**

  | # | Quartas (real) | Final (este prompt) |
  |---|---|---|
  | 1 | Capa "Identidade dois mundos" | Capa "Voz & Visão — a Copa que te responde" |
  | 2 | Stack (Gateway/CIAM/workforce/Container Apps/SQL) | Stack (MCP/RAG/Gemini/Managed Identity/SignalR) |
  | 3 | Conceito central: desambiguação de identidade | Conceito central: **regra de ouro por construção** |
  | 4 | Tec 1: Gateway YARP | Tec 1: **MCP** |
  | 5 | Tec 2: Entra External ID | Tec 2: **RAG por tool-use** |
  | 6 | Tec 3: Entra ID workforce | Tec 3: **Managed Identity** |
  | 7 | Tec 4: Azure Container Apps | Tec 4: **Azure SignalR / observabilidade** |
  | 8 | Conceito: "só muda a string da authority" | Conceito: **Key Vault, o destino de produção** |
  | 9 | Conceito: "modernizar sem destruir" | Conceito: **"Onde foi o n8n?" (simplificar > substituir)** |
  | 10 | Arquitetura "a foto completa da F2" | Arquitetura "a foto completa da Final" |
  | 11 | Encerramento "você concluiu as Quartas" | Encerramento "você concluiu a Copa do Mundo Azure" |

- **Deck reveal paralelo:** `slides.md` (reveal.js) é a versão navegável e mais granular do mesmo
  conteúdo; as `SPEAKER-NOTES.md` seguem o `slides.md` slide a slide e referenciam o número do
  slide equivalente deste `.pptx` (`≈ Storyboard SN`).
- **Rastreabilidade (Art. IV):** fontes — ADE-008 (re-arquitetura sem n8n), ADE-009 (X-Gateway-Key),
  `docs/runbooks/final-portal-guide.md`, `docs/security/final-security-debt.md`, código real
  (`FifaTicketTools.cs`, `gemini.ts`, `FlowEventType.cs` / `flowNodes.ts`).
