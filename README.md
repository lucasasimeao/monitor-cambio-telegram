# Monitor de Câmbio com Alertas no Telegram

> Automação em Node.js que consulta o dólar (USD/BRL), grava o histórico no Airtable e avisa no Telegram quando a cotação cai abaixo de um piso definido.

![Node.js](https://img.shields.io/badge/Node.js-18+-339933?logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-5-000000?logo=express&logoColor=white)
![Axios](https://img.shields.io/badge/Axios-HTTP-5A29E4?logo=axios&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-Bot%20API-26A5E4?logo=telegram&logoColor=white)
![Airtable](https://img.shields.io/badge/Airtable-API-18BFFF?logo=airtable&logoColor=white)

## Sobre o projeto

Acompanhar o câmbio manualmente é repetitivo e fácil de esquecer. Este projeto automatiza o ciclo completo **coleta → persistência → notificação**, orquestrando três APIs externas:

1. **AwesomeAPI** fornece a cotação USD/BRL em tempo real;
2. **Airtable** guarda cada leitura (data, moeda e valor), formando uma base histórica;
3. **Telegram Bot API** envia o alerta quando o dólar fica abaixo do piso e um boletim diário em horário agendado.

Projeto acadêmico da graduação em IA e Automação Digital (UniFECAF), com foco em **integração e orquestração de APIs**.

## Funcionalidades

- **Consulta de câmbio** USD/BRL via AwesomeAPI.
- **Alerta condicional no Telegram** quando a cotação fica abaixo de `DOLLAR_ALERT_THRESHOLD`.
- **Boletim diário agendado**: o servidor verifica a cada minuto e envia o resumo no horário de `DAILY_REPORT_TIME` (uma vez por dia).
- **Histórico no Airtable**: cada verificação vira um registro na tabela.
- **Limpeza da tabela** com paginação (`offset`) e exclusão em lotes de 10, respeitando o limite da API do Airtable.
- **Painel web** para disparar verificação, boletim e limpeza, com escape de HTML nos dados exibidos.
- **Validação de configuração**: cada integração falha com mensagem clara se faltar alguma variável no `.env`.

## Tecnologias

- Node.js 18+, Express 5, Axios, dotenv
- AwesomeAPI (REST, sem autenticação), Airtable API (Bearer token), Telegram Bot API (`parse_mode: HTML`)
- HTML/CSS/JavaScript no painel

## Como executar

Pré-requisitos:
- Node.js 18+
- Um bot do Telegram criado com o [BotFather](https://core.telegram.org/bots) e o ID do chat de destino
- Uma base no [Airtable](https://airtable.com) com uma tabela contendo os campos `Data` (Date), `Moeda` (texto) e `Valor` (número), e um Personal Access Token com os escopos `data.records:read` e `data.records:write`

```bash
git clone https://github.com/lucasasimeao/projeto_08.git
cd projeto_08/painel-financeiro
npm install
cp .env.example .env   # no Windows: copy .env.example .env
```

| Variável | Descrição |
| --- | --- |
| `PORT` | Porta do servidor (padrão `3000`) |
| `DOLLAR_ALERT_THRESHOLD` | Piso do dólar para disparar o alerta (ex.: `5.30`) |
| `DAILY_REPORT_ENABLED` | Liga/desliga o boletim diário (`true`/`false`) |
| `DAILY_REPORT_TIME` | Horário do boletim (`HH:MM`) |
| `TELEGRAM_BOT_TOKEN` | Token do bot |
| `TELEGRAM_CHAT_ID` | Chat ou grupo que recebe as mensagens |
| `AIRTABLE_API_KEY` | Personal Access Token do Airtable |
| `AIRTABLE_BASE_ID` | ID da base (começa com `app...`) |
| `AIRTABLE_TABLE_NAME` | Nome da tabela (padrão `Cotacoes`) |

```bash
npm start      # ou: npm run dev (reinicia ao salvar)
```

Painel em `http://localhost:3000`.

### Endpoints

| Método | Rota | Descrição |
| --- | --- | --- |
| GET | `/health` | Verifica se o servidor está no ar |
| GET | `/config` | Mostra a configuração do boletim diário |
| POST | `/verificar` | Busca a cotação, salva no Airtable e alerta no Telegram se estiver abaixo do piso |
| POST | `/resumo-diario` | Envia o boletim no Telegram na hora |
| POST | `/limpar-tabela` | Remove todos os registros da tabela no Airtable |

## Estrutura do projeto

```text
painel-financeiro/
├── public/index.html    # Painel web
├── src/server.js        # Rotas, regras de alerta e agendamento do boletim
├── src/awesomeapi.js    # Cotação USD/BRL
├── src/airtable.js      # Gravação, listagem paginada e exclusão em lotes
├── src/telegram.js      # Envio de mensagens
└── .env.example
```

## Segurança e LGPD

- Credenciais apenas no `.env` (ignorado pelo Git); o `.env.example` documenta as variáveis sem valores.
- O painel escapa o HTML dos dados externos antes de exibi-los (proteção contra XSS).
- Nenhum dado pessoal é tratado: apenas cotações públicas e o horário da consulta.

## Competências demonstradas

- **Automação de processos e orquestração de APIs** (REST, autenticação por token, paginação, limites de lote).
- **Bots e notificações**: integração com a Telegram Bot API.
- **Coleta e armazenamento de dados** em base no-code (Airtable), gerando série histórica pronta para análise.
- **Agendamento** de tarefas recorrentes no próprio servidor.
- **Boas práticas**: variáveis de ambiente, separação por módulos e tratamento de erros por integração.

## Próximos passos

- Exportar o histórico do Airtable e analisar a variação do câmbio (Python/Pandas ou Power BI).
- Proteger a rota `/limpar-tabela` (hoje qualquer acesso ao painel pode acioná-la).
- Usar um agendador como `node-cron` e permitir monitorar outras moedas.
