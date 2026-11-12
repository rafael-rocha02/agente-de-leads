# Agente de leads

Agente de IA construído no n8n que lê o CRM, cruza a origem de cada lead com a nota dada pelo SDR e com o motivo de descarte, e envia ao time de marketing um resumo diário no Slack.

Construí este agente durante meu estágio em pré-vendas na VExpenses, para resolver um problema que eu vivia como SDR: a percepção sobre a qualidade dos leads de cada landing page chegava ao marketing de forma indireta, passando por várias pessoas. Esta é uma recriação publicada com dados fictícios.

![Fluxo no n8n](imagens/fluxo-n8n.jpg)

## Como funciona

1. **Busca os negócios no Pipedrive**, pela API, com origem, estrelas, status e observações do SDR.
2. **O Analista calcula os números** em código, sem IA: leads, média de estrelas e % de perdidos por origem no dia anterior (na segunda, de sexta a domingo), comparados com os 7 dias anteriores do calendário.
3. **O Classificador** (Gemini) agrupa o texto livre dos SDRs em motivos de perda, como "só pesquisando" ou "empresa pequena demais".
4. **O Redator** (Gemini) escreve a mensagem para o marketing, com aviso quando o volume é baixo demais para concluir.
5. **O Revisor** (Claude Haiku 4.5) compara cada número da mensagem com os dados do Analista. Por ser de outro fornecedor, tende a não repetir os erros do Gemini. Se encontrar erro, a mensagem volta para correção.
6. **A mensagem vai para o Slack**, no canal do marketing, de segunda a sexta às 8h (horário de Brasília).

O fluxo orquestra agentes de dois fornecedores de IA: o Gemini classifica e escreve, e o Claude revisa. As contas ficam com o código, não com a IA. O Classificador e o Revisor respondem em JSON, para o fluxo ler a saída sem interpretar texto.

## O Revisor em ação

Na execução de exemplo, o Redator (Gemini) citou as contagens dos temas do Classificador, e não os motivos de perda registrados no CRM. O Revisor (Claude) comparou cada número com os dados oficiais, reprovou a mensagem e apontou os três números que não batiam. A versão corrigida foi a enviada.

| Trecho da mensagem | Primeira versão | Nos dados oficiais | Versão enviada |
|---|---|---|---|
| LP Cartões, "só pesquisando" | 10 | 12 | 12 |
| LP Cartões, "empresa pequena demais" | 7 | 12 | 12 |
| Site institucional, "já usa concorrente" | 6 | 5 | 5 |

Usar um modelo de outro fornecedor no Revisor é proposital: quem escreve e quem confere não compartilham os mesmos vieses, então um erro de um tem menos chance de passar pelo outro.

![Saída do Revisor](imagens/revisor.jpg)

![Mensagem no Slack](imagens/mensagem-slack.png)

## Arquivos

- [`fluxo/agente-de-leads.json`](fluxo/agente-de-leads.json): o fluxo completo, pronto para importar no n8n.
- [`prompts.md`](prompts.md): os prompts de cada agente.

## Como usar

1. No n8n, importe `fluxo/agente-de-leads.json`.
2. Configure as credenciais do Pipedrive, do Google Gemini e da Anthropic (Claude) nos nós correspondentes.
3. No nó "Enviar no Slack", troque `COLE_AQUI_A_URL_DO_WEBHOOK_DO_SLACK` pela URL de um Incoming Webhook do seu Slack.
4. No início do nó "Analista", `DATA_EXECUCAO` está preenchida com `2026-11-03` para a demonstração com os dados fictícios, que terminam em 02/11/2026. Deixe vazia (`''`) para o agente usar a data de hoje.
5. Os campos personalizados do Pipedrive (data de entrada, estrelas, origem e reunião agendada) são lidos pelas chaves definidas no início do nó "Analista". Ajuste para as chaves da sua conta.

## Ferramentas

n8n, API do Pipedrive, API do Gemini, API do Claude e Slack.

---

Feito por [Rafael Augusto Barbosa da Rocha](https://www.linkedin.com/in/rafael-rocha02/).
