# ChatBot IA de Suporte com n8n, Google Sheets e Groq

Projeto desenvolvido durante um processo seletivo para simular um **chatbot de suporte técnico com IA**, usando uma base de conhecimento estruturada no Google Sheets e um workflow automatizado no n8n.

A proposta do projeto é permitir que um cliente envie uma dúvida pelo chat, o sistema consulte uma base de conhecimento, filtre os conteúdos mais relevantes e envie essas informações para uma IA gerar uma resposta objetiva, profissional e baseada apenas nos dados disponíveis.

## Objetivo

Criar um assistente de suporte capaz de:

- Receber perguntas de usuários por chat;
- Consultar uma base de conhecimento no Google Sheets;
- Encontrar os registros mais relevantes para a dúvida enviada;
- Montar um prompt estruturado para a IA;
- Gerar uma resposta clara e segura;
- Retornar a resposta ao usuário dentro do próprio chat do n8n.

## Tecnologias utilizadas

- **n8n** — automação do fluxo e integração entre serviços;
- **Docker** — execução local/self-hosted do n8n;
- **Google Sheets** — base de conhecimento do chatbot;
- **Groq API** — chamada ao modelo de IA;
- **Llama 3.3 70B Versatile** — modelo utilizado para geração das respostas;
- **JavaScript** — tratamento, filtro e ranqueamento dos dados dentro dos nodes de código do n8n.

## Como o fluxo funciona

O workflow segue a seguinte lógica:

```text
Usuário envia pergunta no chat
        ↓
n8n recebe a mensagem
        ↓
Google Sheets retorna a base de conhecimento
        ↓
Node de código filtra as respostas mais relevantes
        ↓
Outro node monta o prompt para a IA
        ↓
HTTP Request envia a requisição para a Groq API
        ↓
Resposta da IA é formatada
        ↓
Chat retorna a resposta final ao usuário
```

## Estrutura do workflow

O workflow exportado possui os seguintes nodes principais:

| Node | Função |
|---|---|
| `When chat message received` | Recebe a pergunta enviada pelo usuário |
| `Base_Conhecimento` | Consulta a planilha do Google Sheets |
| `Filtro Base_Conhecimento` | Normaliza a pergunta e ranqueia os registros mais relevantes |
| `Prompt IA` | Monta o corpo da requisição para a IA |
| `HTTP Request` | Envia a requisição para a API da Groq |
| `Formatar Resposta` | Extrai e organiza o texto retornado pela IA |
| `Chat` | Envia a resposta final ao usuário |

## Base de conhecimento

A base de conhecimento foi organizada no Google Sheets com uma estrutura simples, permitindo fácil manutenção e expansão.

Exemplo de colunas utilizadas:

| Coluna | Descrição |
|---|---|
| `categoria` | Área ou tema da dúvida |
| `pergunta` | Pergunta comum do usuário |
| `resposta` | Resposta cadastrada na base |
| `palavras_chave` | Termos usados para melhorar a busca |

## Filtro de relevância

Antes de enviar a pergunta para a IA, o workflow executa um filtro em JavaScript.

Esse filtro:

1. Normaliza a pergunta do usuário;
2. Remove pontuações;
3. Divide a pergunta em palavras;
4. Compara essas palavras com os campos da base de conhecimento;
5. Calcula uma pontuação para cada linha;
6. Ordena os resultados por relevância;
7. Seleciona os 5 registros mais relevantes.

Dessa forma, a IA recebe apenas os dados mais úteis para responder à pergunta.

## Uso da IA

A IA recebe dois blocos principais:

1. **Prompt de sistema**, com regras de comportamento;
2. **Pergunta do cliente + base encontrada**, com os dados recuperados da planilha.

O prompt foi configurado para orientar a IA a:

- Responder apenas com base na base de conhecimento;
- Não inventar funcionalidades ou procedimentos;
- Recomendar abertura de chamado quando não houver informação suficiente;
- Responder em passos numerados quando possível;
- Solicitar evidências caso a dúvida indique possível bug;
- Manter tom profissional, simples e humano.

## Estrutura sugerida do repositório

```bash
n8n-ai-support-chatbot/
├── workflows/
│   └── ChatBot-IA-sanitized.json
├── docs/
│   └── base_conhecimento_exemplo.csv
├── docker-compose.example.yml
├── .env.example
├── .gitignore
└── README.md
```

## Como executar localmente com Docker

### 1. Clonar o repositório

```bash
git clone https://github.com/SEU-USUARIO/n8n-ai-support-chatbot.git
cd n8n-ai-support-chatbot
```

### 2. Criar o arquivo `.env`

Copie o arquivo de exemplo:

```bash
cp .env.example .env
```

No Windows PowerShell:

```powershell
copy .env.example .env
```

Depois, edite o `.env` com suas informações locais.

### 3. Subir o n8n com Docker

```bash
docker compose -f docker-compose.example.yml up -d
```

Depois acesse:

```text
http://localhost:5678
```

## Como importar o workflow no n8n

1. Abra o n8n no navegador;
2. Clique em **Import workflow**;
3. Selecione o arquivo:

```text
workflows/ChatBot-IA-sanitized.json
```

4. Configure novamente as credenciais necessárias:
   - Google Sheets;
   - Groq API;
5. Confirme a URL da planilha usada como base de conhecimento;
6. Salve o workflow;
7. Ative o workflow;
8. Teste enviando uma pergunta pelo chat.

## Configuração das credenciais

Por segurança, as credenciais não devem ser enviadas para o GitHub.

Depois de importar o workflow, configure manualmente no n8n:

### Google Sheets

- Crie ou selecione uma credencial do Google;
- Garanta acesso à planilha usada como base;
- Confirme o nome da aba, por exemplo:

```text
Base_Conhecimento
```

### Groq API

- Crie uma credencial para a Groq API no n8n;
- Configure sua chave de API;
- Confirme o endpoint usado no HTTP Request:

```text
https://api.groq.com/openai/v1/chat/completions
```

## Exemplo de pergunta

```text
Como faço para resetar a senha?
```

Exemplo de comportamento esperado:

```text
1. Verifique se o e-mail informado está correto.
2. Acesse a opção de recuperação de senha.
3. Siga as instruções enviadas por e-mail.
4. Caso o problema continue, abra um chamado com o time de suporte.
```

## Cuidados de segurança

Antes de publicar o projeto no GitHub, não envie:

- Chaves de API;
- Arquivo `.env` real;
- Credenciais do n8n;
- Banco local do n8n;
- Volume `.n8n`;
- Dados sensíveis de clientes;
- Links privados de planilhas, caso a base contenha dados internos.

Use sempre arquivos de exemplo, como:

- `.env.example`;
- `docker-compose.example.yml`;
- `ChatBot-IA-sanitized.json`.

## Melhorias futuras

Algumas melhorias possíveis para o projeto:

- Criar uma busca semântica com embeddings;
- Adicionar memória de conversa;
- Registrar perguntas não respondidas em uma nova aba do Google Sheets;
- Criar métricas de atendimento, como dúvidas mais frequentes;
- Adicionar integração com WhatsApp, Slack ou sistema de chamados;
- Criar fallback automático para abertura de ticket;
- Separar ambientes de desenvolvimento e produção.

## Status

Projeto finalizado como desafio de processo seletivo e mantido como demonstração prática de automação, integração com IA e construção de chatbot com base de conhecimento.
