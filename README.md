# 🌤️ Chatbot de Temperatura via Telegram + N8N

Chatbot no Telegram que informa a temperatura atual de qualquer cidade do Brasil. O usuário envia o nome da cidade e estado, o N8N consulta a API do OpenWeather e responde com uma mensagem amigável.

**Exemplo de uso:**
```
Usuário:  São Paulo, SP
Bot:      🌤️ A temperatura em São Paulo é de 20°C
```

---

## 🗂️ Estrutura do Projeto

```
.
├── docker-compose.yml   # Infraestrutura: N8N, Postgres, Redis, Ngrok
└── workflow.json        # Workflow exportado do N8N (importar manualmente)
```

---

## 🐳 Pré-requisitos

- [Docker](https://docs.docker.com/get-docker/) e Docker Compose instalados
- Conta no [Telegram](https://telegram.org/) com um bot criado via [@BotFather](https://t.me/BotFather)
- Chave de API gratuita no [OpenWeather](https://openweathermap.org/api)
- Conta no [Ngrok](https://ngrok.com/) com um domínio estático configurado

---

## ⚙️ Configuração das Variáveis de Ambiente

Abra o arquivo `docker-compose.yml` e substitua os placeholders `{...}` pelos seus valores reais. Faça isso nos dois serviços (`n8n-editor` e `n8n-worker`):

```yaml
# Antes
- OPENWEATHER_API_KEY={OPENWEATHER_API_KEY}
- TELEGRAM_BOT_TOKEN={TELEGRAM_BOT_TOKEN}

# Depois (exemplo)
- OPENWEATHER_API_KEY=a1b2c3
- TELEGRAM_BOT_TOKEN=123456789:AABBccDDeeFFggHH-exemplo
```

Faça o mesmo para o serviço `ngrok`:

```yaml
# Antes
- NGROK_AUTHTOKEN={NGROK_AUTHTOKEN}
- WEBHOOK_URL={WEBHOOK_URL}
- --domain={WEBHOOK_URL_DOMINIO}

# Depois (exemplo)
- NGROK_AUTHTOKEN=2abc123seu_token_aqui
- WEBHOOK_URL=https://seudominio.ngrok-free.dev/
- --domain=seudominio.ngrok-free.dev/
```

> ⚠️ Cuidado para não subir o `docker-compose.yml` com os valores reais para um repositório público. Se for versionar o projeto, prefira manter os placeholders `{...}` no repositório e preencher os valores apenas localmente.

### Onde obter cada credencial

| Variável | Como obter |
|---|---|
| `TELEGRAM_BOT_TOKEN` | Fale com [@BotFather](https://t.me/BotFather) no Telegram → `/newbot` |
| `OPENWEATHER_API_KEY` | Crie uma conta em [openweathermap.org](https://openweathermap.org) → *API keys* |
| `NGROK_AUTHTOKEN` | Acesse [dashboard.ngrok.com](https://dashboard.ngrok.com) → *Your Authtoken* |

---

## 🚀 Como Executar

### 1. Subir a infraestrutura via Portainer (extensão do Docker Desktop)

1. Abra o **Docker Desktop**
2. No menu lateral, clique em **Extensions** e abra o **Portainer**
3. Vá em **Stacks → Add stack**
4. Dê um nome para a stack (ex.: `automacoes-ftr`)
5. Cole o conteúdo do `docker-compose.yml` (já com os valores preenchidos) no editor
6. Clique em **Update the stack**

Aguarde todos os containers aparecerem com status **Running** na aba de containers do Portainer.

### 2. Acessar o N8N

Abra o navegador pela URL do Ngrok que você configurou no `docker-compose.yml`, por exemplo:

**https://seu-dominio.ngrok-free.dev**

Na primeira vez, crie um usuário administrador quando solicitado.

### 3. Importar o Workflow

1. No menu lateral, clique em **Workflows**
2. Clique em **Import from file**
3. Selecione o arquivo `.json`

### 4. Configurar as Credenciais no N8N

As variáveis `OPENWEATHER_API_KEY` e `TELEGRAM_BOT_TOKEN` já são injetadas via `docker-compose.yml` e ficam disponíveis como variáveis de ambiente dentro do N8N.

Para usá-las nos nodes:

**No node do Telegram (Trigger e Send Message):**
1. Abra o node e verifique o campo **Credential**
2. Caso esteja vazio, clique em → **Create new credential**
3. No campo *Access Token*, insira: `{{ $env.TELEGRAM_BOT_TOKEN }}`
4. Salve a credencial

**No node HTTP Request (OpenWeather):**
1. Abra o node e verifique se o campo **Query Parameters** contém o Name = appied e o valor `{{ $env.OPENWEATHER_API_KEY }}`
2. Caso não contenha, adicione num novo parâmetro e insira: `{{ $env.OPENWEATHER_API_KEY }}`
4. Salve a credencial

---

## 🧪 Testando o Chatbot

1. Abra o Telegram e encontre o seu bot pelo nome definido no BotFather
2. Envie uma mensagem no formato `Cidade, UF`:

| Mensagem enviada | Resposta esperada |
|---|---|
| `Curitiba, PR` | 🌤️ A temperatura em Curitiba é de 18°C |
| `São Paulo, SP` | 🌤️ A temperatura em São Paulo é de 20°C |
| `Grifnória` | ❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP). |

### Formato aceito

- ✅ `São Paulo, SP`
- ✅ `Rio de Janeiro,RJ` (sem espaço também funciona)
- ✅ `Campinas` (somente a cidade também funciona)
- ❌ Nomes fictícios ou inexistentes retornam mensagem de erro amigável

---
