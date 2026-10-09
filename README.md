# 🚀 Triagem, Gestão e Escalamento Automatizado de Chamados (n8n + LLM + BPMN)

Este repositório contém a modelagem de processos e a implementação técnica de uma solução inteligente de triagem, roteamento e suporte automatizado para empresas SaaS. 

A arquitetura combina **modelagem formal em BPMN 2.0**, orquestração de fluxos via **n8n**, inteligência artificial multimodelo com **Guardrails de Segurança (LLM via OpenRouter)** e integração multicanal (**Trello, Slack, Gmail e Google Sheets**).

---

## 📐 Modelagem do Processo (BPMN 2.0)

O fluxo foi desenhado no **Bizagi Process Modeler**, separando claramente as responsabilidades em três raias operacionais (*Swimlanes*):

1. **Cliente:** Envio da solicitação e recebimento das notificações/soluções.
2. **n8n (Automação & IA):** Validação de segurança (Guardrails), classificação semântica com LLM, enriquecimento de dados e roteamento condicional.
3. **Analista de Suporte (Atendimento Humano):** Atuação em chamados urgentes/críticos e gestão do fluxo de escalamento para supervisão.

![Diagrama BPMN de Triagem de Chamados](assets/TriagemAutomacaoChamadosv3.png)

> 💡 **Ficheiro Editável:** [Descarregar projeto no Bizagi (.bpm)](diagrams/TriagemAutomacaoChamadosv3.bpm)

---

## ⚙️ Arquitetura da Solução e Fluxo de Execução

1. **Entrada e Guardrail de Segurança:** O processo é iniciado via Webhook. O conteúdo passa por um nó de verificação com **Gemini 3.1 Pro (via OpenRouter)** com filtros em duas camadas:
   - **Camada 1 (Jailbreak Detection):** Bloqueio de injeção de prompt e manipulação de instrução.
   - **Camada 2 (Topical Alignment):** Filtro de escopo estrito para suporte a software/SaaS.
2. **Enriquecimento de Dados (*Data Enrichment*):** Consulta automática à base no Google Sheets para validar o cadastro do cliente e identificar o plano assinado (*Enterprise, Pro, Basic*).
3. **Triagem por IA e Saída Estruturada:** O agente de IA analisa o chamado, classifica a urgência/categoria e gera a resposta técnica estruturada em JSON via `Structured Output Parser`.
4. **Gestão Kanban (Trello):** Criação automática do card no Trello com etiquetas de prioridade e impacto.
5. **Roteamento Inteligente (Gateways de Decisão):**
   - **Fluxo Padrão (Não Urgente):** O cliente recebe um e-mail formatado em HTML com os passos de resolução e o card segue para aprovação.
   - **Fluxo Crítico (Urgente):** Notificação em tempo real no **Slack** para atuação imediata do Analista de Suporte. Se o problema não for resolvido, um e-mail de escalamento é enviado para a gerência.

---

## 🛠️ Tecnologias e Ferramentas

- **Modelagem de Processos:** Bizagi Process Modeler (BPMN 2.0)
- **Orquestração:** n8n
- **Inteligência Artificial:** Gemini 3.1 Pro via OpenRouter (Guardrails + Agent + Structured Output)
- **Integrações:** Trello (Kanban), Slack (Alertas), Gmail/SMTP (Comunicação) e Google Sheets (CRM/Base)

---

## 📚 Documentação Técnica Aprofundada

Para examinar os detalhes de implementação, prompts e configurações dos nós, aceda aos documentos da pasta `/docs`:

* [🛡️ **01. Guardrails e Verificação de Segurança**](docs/prompt-guardrails.md) — Prompts de Jailbreak, Topical Alignment e limites de modelo.
* [🧠 **02. Agente de IA e Output Parser**](docs/AgentesIA-LLM.md) — System prompt, injeção de contexto e JSON Schema.
* [🔌 **03. Integrações e Gateways**](docs/Integracoes-e-acoes.md) — Configuração do Trello, Slack, Gmail, Google Sheets e regras de negócio dos Gateways.

---

## 📥 Estrutura do Repositório

```text
📁 projeto-triagem-chamados
├── 📄 README.md
├── 📁 assets/                     <-- Imagens usadas no README (BPMN, screenshots)
│   └── 🖼️ TriagemAutomacaoChamadosv3.png
├── 📁 diagrams/                   <-- Ficheiros editáveis de modelagem
│   └── 📄 TriagemAutomacaoChamadosv3.bpm
├── 📁 n8n-flows/                  <-- Código e exportações do n8n
│   └── 📄 TriagemAutomacaoChamadosv3.json
└── 📁 docs/                       <-- Documentação técnica detalhada
    ├── 📄 prompt-guardrails.md
    ├── 📄 AgentesIA-LLM.md
    └── 📄 Integracoes-e-acoes.md
