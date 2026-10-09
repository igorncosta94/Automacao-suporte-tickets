# 🔌 Documentação de Integrações, Serviços e Ações (`/docs/Integracoes-e-acoes.md`)

Este documento detalha o funcionamento, contratos de dados e regras operacionais de todos os nós de integração externa (Google Sheets, Trello, Slack e Gmail) e desvios condicionais que compõem a automação.

---

## 📊 1. Enriquecimento de Dados (`consultarBase` / Google Sheets)

- **Tipo de Nó:** Google Sheets / Google Drive
- **Função:** Enriquecimento de dados do chamado (*Data Enrichment*).
- **Mecanismo:** Realiza a busca no cadastro de clientes utilizando o e-mail informado no gatilho `NovoTicket`.
- **Dados Extraídos para a LLM:**
  - `empresa`: Nome da empresa cadastrada.
  - `plano_cliente`: Nível de contrato (*Enterprise*, *Pro*, *Basic*).
  - `cliente_existente`: Validação de cadastro ativo (`true`/`false`).
- **Objetivo no Fluxo:** Permitir que o agente de IA (`AnaliseGravidade`) avalie a prioridade e o SLA com base no contrato do cliente.

---

## 📋 2. Gestão Kanban (`Trello`)

### 2.1. Criação do Ticket (`CriarCard`)
- **Ação:** Cria um novo card na lista inicial do Trello.
- **Mapeamento de Campos:**
  - **Título do Card:** `{{ $json.resumo }}` *(gerado pela IA)*
  - **Descrição:** Nome do cliente, e-mail, empresa, descrição original do problema e impacto identificado.
  - **Etiquetas (Labels):** Mapeadas via `categoria` e `prioridade`.

### 2.2. Movimentação de Status (`MoverCard` / `MoverCard1`)
- **`MoverCard` (Aprovado / Resolvido):** Transfere o card para a lista de chamados concluídos ou aprovados.
- **`MoverCard1` (Rejeitado / Urgentes):** Transfere o card para a lista de auditoria ou tratativa urgente quando o fluxo normal falha.

---

## 💬 3. Comunicação Interna e Alertas (`Slack`)

### Nó: `Notificar Time no Slack`
- **Gatilho:** Ativado apenas quando a regra de urgência é satisfeita (`é Urgente? = Sim`).
- **Canal de Destino:** `#suporte-critico` / `#atendimento-urgente`
- **Conteúdo da Notificação:**
  - Alerta visual com tag de alta prioridade.
  - Dados do cliente, plano e resumo do problema.
  - Link direto para o Card do Trello gerado para atuação imediata do **Analista de Suporte**.

---

## ✉️ 4. Notificações e Respostas ao Cliente (`Gmail / E-mail`)

### 4.1. Resposta ao Cliente (`Enviar Email` / `Send a message`)
- **Destinatário:** `{{ $('NovoTicket').item.json.email }}`
- **Assunto:** `[SaaSPro] Atualização do seu chamado - {{ $json.resumo }}`
- **Corpo:** Envia o HTML renderizado `{{ $json.resposta_cliente_html }}` com as orientações técnicas e solução direta estruturada pela IA.

### 4.2. Escalamento Executivo (`Enviar Email para Superior`)
- **Gatilho:** Ativado caso o analista não consiga resolver o chamado crítico na raia de atendimento humano (`Resolvido = Não`)[cite: 9].
- **Destinatário:** E-mail da gestão/supervisão de suporte.
- **Objetivo:** Notificar a liderança para intervenção de Nível 2 / SLA estourado.

---

## 🔀 5. Regras de Negócio e Gateways de Decisão

| Gateway | Condição | Ação / Destino |
| :--- | :--- | :--- |
| **`Conteúdo Válido?`** | `Pass` (Sem violação/jailbreak) | Avança para análise do agente de IA. |
| | `Fail` (Violação/Out of scope) | Desvia para o término silencioso (`Faça nada`). |
| **`é Urgente?`** | `urgente = true` ou `prioridade = alta` | Dispara alerta no Slack e envia para a raia do Analista. |
| | `urgente = false` | Segue o fluxo normal de confirmação e aprovação. |
| **`Resolvido?`** | `Sim` | Move card no Trello e encerra o chamado. |
| | `Não` | Envia e-mail de escalamento para a gestão. |
