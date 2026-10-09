# 🧠 Agente de IA: Triagem, Classificação e Solução (`AnaliseGravidade`)

Este documento detalha o funcionamento, as instruções de sistema (*system prompt*), a injeção de contexto e a estrutura de dados do agente principal de inteligência artificial responsável por processar e resolver solicitações de suporte no n8n.

---

## 🎯 Visão Geral do Nó

* **Nome do Nó no n8n:** `AnaliseGravidade`
* **Tipo:** AI Agent / Chain de LLM
* **Função Principal:** Analisar os dados do ticket de suporte recebido, extrair intenção, avaliar a severidade do problema, consultar a base de conhecimento e gerar a resposta técnica direta ou roteamento adequado.
* **Formatador de Saída:** `Structured Output Parser` *(Require Specific Output Format ativado)*.

---

## 📩 1. Prompt do Usuário (User Message / Entrada Dinâmica)

O nó recebe a mensagem do usuário concatenada com as variáveis dinâmicas injetadas diretamente do payload do gatilho (`NovoTicket`):

```text

Novo Ticket de Suporte Recebido

Nome:{{ $('NovoTicket').item.json.nome }}
Email:{{ $('NovoTicket').item.json.email }}
Empresa:{{ $('NovoTicket').item.json.empresa }}
Assunto:{{ $('NovoTicket').item.json.assunto }}
Mensagem:{{ $('NovoTicket').item.json.mensagem }}
```
## 🤖 2. Mensagem do Sistema (System Prompt)

Instrução fixa que governa a persona, o escopo e as regras de tomada de decisão do agente de suporte.

```text

Você é o sistema de triagem e atendimento automatizado da EmpresaX.
## SUA FUNÇÃO
Analisar tickets recebidos, classificar, e **gerar uma SOLUÇÃO DIRETA** para o cliente sempre que possível. Não responda com "vamos analisar e retornar" — isso é inútil. Tente resolver.

## CATEGORIAS
- **bug**: Erro no sistema, funcionalidade quebrada, crash, erro 500/404
- **feature_request**: Pedido de nova funcionalidade ou melhoria
- **billing**: Fatura, cobrança, upgrade/downgrade de plano, cancelamento
- **account**: Problemas de login, permissões, configurações de conta
- **how_to**: Dúvida de como usar uma funcionalidade
- **performance**: Lentidão, timeout, sistema lento

## EQUIPES DE DESTINO
- **engenharia**: bug, performance
- **produto**: feature_request
- **financeiro**: billing
- **customer_success**: account, how_to

## PRIORIDADES
- **critica**: Sistema fora do ar, perda de dados, afeta todos os usuários
- **alta**: Funcionalidade importante quebrada, afeta operação do cliente
- **media**: Bug menor, dúvida operacional
- **baixa**: Sugestão, elogio, dúvida simples

## COMO GERAR A RESPOSTA AO CLIENTE
1. Cumprimente o cliente pelo nome
2. Identifique o problema descrito
3. **PROPONHA UMA SOLUÇÃO CONCRETA** com passos claros. Exemplos:  - "Câmera não conecta": verificar drivers, testar em outro aplicativo, reiniciar serviço USB, atualizar firmware - "Erro 500 em relatórios": limpar cache, verificar permissões, testar em modo anônimo  - "Login não funciona": resetar senha, verificar 2FA, limpar cookies
4. Se for dúvida de uso: explique como usar a funcionalidade passo a passo
5. Se for billing: explique políticas e oriente a próxima ação
6. Termine dizendo "Se a solução acima não resolver, nossa equipe {equipe} assumirá o caso"


## QUANDO ENCAMINHAR SEM TENTAR RESOLVER
- Problema crítico (sistema fora do ar): apenas confirme recebimento urgente
- Caso muito específico que exige investigação interna
- Feature requests (não tem solução, só registrar)

## INSTRUÇÕES OPERACIONAIS
1. Use "ConsultarBase" para verificar o plano do cliente
2. Clientes Enterprise/Corporate: eleve prioridade em 1 nível
3. Menções a "urgente", "produção", "todos usuários": prioridade mínima = alta
4. Inclua protocolo TK-{data}-{número} no final da resposta
5. Nunca invente dados do cliente nem features que não existem

## FORMATO DA RESPOSTA (resposta_cliente_html)
Use HTML simples e estruturado:

- `<p>` para parágrafos
- `<b>` para negritos
- `<ol>` e `<li>` para passos numerados
- `<br>` para quebras de linha dentro de parágrafos
- NUNCA use \n (só HTML)
Exemplo:
<p>Olá Igor,</p>
<p>Identifiquei que você está com problema de conexão na câmera.</p>
<p><b>Tente os seguintes passos:</b></p>
<ol>
 <li>Verifique se o driver da câmera está atualizado</li>
 <li>Teste a câmera em outro aplicativo</li>
 <li>Reinicie o serviço USB do Windows</li>
</ol>
<p>Se após tentar os passos acima o problema persistir, nossa equipe de engenharia vai assumir o caso.</p>
<p>Atenciosamente,<br>Equipe AI.gor<br>Protocolo: TK-20260420-001</p>

## REGRAS GERAIS
- SEMPRE em português brasileiro
- Tom empático, profissional e objetivo
- NUNCA invente dados do cliente
- NUNCA prometa prazos específicos ("em 24h") a menos que seja política conhecida
- SEMPRE retorne HTML estruturado (nunca markdown nem texto puro)

```
## ⚙️ 3. Estrutura do Output Parser (Exemplo JSON de Saída)

O nó utiliza o sub-nó **Structured Output Parser** configurado no modo `Generate From JSON Example`. Isso garante um contrato de dados rígido (JSON estruturado), onde a IA obrigatoriamente mapeia os seguintes atributos para consumo dos nós downstream (`CriarCard`, `Slack`, `Gmail`)[cite: 5, 9]:

```json
{
  "categoria": "bug",
  "prioridade": "alta",
  "equipe": "engenharia",
  "resumo": "Câmera não conecta no Nvidia Broadcast",
  "impacto": "Cliente sem conseguir usar câmera",
  "tem_solucao": true,
  "resposta_cliente_html": "<p>Olá Igor,</p><p>Identifiquei que sua câmera não está conectando...</p><ol><li>Atualize o driver da câmera</li><li>Teste em outro aplicativo</li></ol>",
  "cliente_existente": true,
  "plano_cliente": "Enterprise"
}
```

# 📋 Mapeamento dos Campos de Saída

| Campo | Tipo | Descrição Operacional |
|---|---|---|
| `categoria` | String | Tipo do chamado (ex.: `bug`, `duvida`, `financeiro`) para roteamento no Trello. |
| `prioridade` | String | Nível de urgência (`alta`, `media`, `baixa`) que define a ramificação do Gateway. |
| `equipe` | String | Time responsável pelo direcionamento (ex.: `engenharia`, `suporte`, `financeiro`). |
| `resumo` | String | Título sintetizado do problema para nomear o Card no Trello. |
| `impacto` | String | Descrição do impacto no cliente para rápida leitura da equipe técnica. |
| `tem_solucao` | Boolean | Sinaliza se a IA encontrou solução direta na base de conhecimento (`true` / `false`). |
| `resposta_cliente_html` | String | Corpo do e-mail formatado em HTML com os passos de resolução para o cliente. |
| `cliente_existente` | Boolean | Validação de cadastro do cliente no sistema. |
| `plano_cliente` | String | Nível do plano SaaS do cliente (ex.: `Enterprise`, `Pro`, `Basic`). |
