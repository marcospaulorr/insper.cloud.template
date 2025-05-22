# 🚀 API RESTful - Projeto Computação em Nuvem 2025.1

**Autor:** Marcos Paulo Ricarte  
**Disciplina:** Computação em Nuvem - Insper  
**Data:** Maio 2025

---

## 📋 1. Explicação do Projeto - Scrap do que foi feito

Este projeto implementa uma **API RESTful completa** desenvolvida para a disciplina de Computação em Nuvem do Insper. A aplicação demonstra conceitos fundamentais de desenvolvimento em nuvem, containerização e deploy em ambiente AWS.

### 🎯 O que foi implementado:

**🔐 Sistema de Autenticação Completo:**
- Cadastro de usuários com validação de email único
- Login seguro com criptografia de senhas usando bcrypt
- Autenticação via JWT (JSON Web Tokens) com expiração de 30 minutos
- Proteção de rotas sensíveis com middleware de autenticação

**📊 Consulta de Dados Externos:**
- Endpoint protegido para consulta de dados da Bovespa
- Simulação de web scraping de dados financeiros atualizados
- Retorno estruturado em formato JSON com dados dos últimos 8 pregões
- Dados incluem: Data, Abertura, Máxima, Mínima, Fechamento e Volume

**🏥 Monitoramento e Saúde:**
- Health check endpoint para verificação do status da aplicação
- Logs estruturados para debugging e monitoramento
- Informações de timestamp e hostname para identificação

**🐳 Containerização Completa:**
- Aplicação dockerizada usando FastAPI e Python 3.10
- Banco PostgreSQL 17 em container separado
- Docker Compose para orquestração de serviços
- Imagem otimizada e publicada no Docker Hub

### 🛠️ Tecnologias Utilizadas

| Categoria | Tecnologia |
|-----------|------------|
| **Backend** | FastAPI (Python 3.10) |
| **Banco de Dados** | PostgreSQL 17 |
| **Autenticação** | JWT + bcrypt |
| **Containerização** | Docker & Docker Compose |
| **ORM** | SQLAlchemy 2.0 |
| **Validação** | Pydantic |
| **Deploy** | AWS Lightsail |

### 📁 Estrutura do Projeto

```
cloud_projeto_1/
├── app/
│   ├── __init__.py
│   ├── app.py              # Aplicação principal FastAPI
│   └── main.py             # Entry point
├── Dockerfile              # Build da imagem Docker
├── compose.yaml            # Docker Compose FINAL (apenas imagens)
├── requirements.txt        # Dependências Python
├── main.md                 # Esta documentação
├── .env.example           # Exemplo de variáveis
├── .gitignore
└── README.md
```

---

## 🚀 2. Como Executar a Aplicação

### 📋 Pré-requisitos

- **Docker Desktop** (versão mais recente)
- **Git** para clonar o repositório
- **Navegador web** para acessar a API

### 📥 Passo 1: Clonar o Repositório

```bash
git clone https://github.com/marcospauloricarte/cloud_projeto_1.git
cd cloud_projeto_1
```

### 🐳 Passo 2: Executar com Docker Compose

```bash
# Subir toda a aplicação (API + Banco)
docker compose up -d

# Verificar se os containers estão rodando
docker compose ps
```

**Saída esperada:**
```
NAME       SERVICE    STATUS     PORTS
cloud-api  app        Up         0.0.0.0:8000->8000/tcp
cloud-db   db         Up         0.0.0.0:5432->5432/tcp
```

### 🌐 Passo 3: Acessar a Aplicação

- **Swagger UI (Interface Interativa):** http://localhost:8000/docs
- **Health Check:** http://localhost:8000/health-check
- **Documentação API:** http://localhost:8000/redoc

### 🧪 Passo 4: Testar a API

#### Via Swagger UI (Mais Fácil):

1. Acesse: http://localhost:8000/docs
2. Cadastre um usuário em `POST /registrar`
3. Copie o JWT token da resposta
4. Clique em "Authorize" 🔒 (topo da página)
5. Cole apenas o token (sem "Bearer")
6. Teste o endpoint protegido `GET /consultar`

#### Via PowerShell/Terminal:

```powershell
# 1. Cadastrar usuário
$response = Invoke-RestMethod -Uri "http://localhost:8000/registrar" -Method POST -Headers @{"Content-Type"="application/json"} -Body '{"name":"Teste","email":"teste@email.com","senha":"123456"}'

# 2. Consultar dados com token
Invoke-RestMethod -Uri "http://localhost:8000/consultar" -Headers @{"Authorization"="Bearer $($response.jwt)"}
```

### 🛑 Parar a Aplicação

```bash
# Parar containers
docker compose down

# Parar e remover volumes (limpar banco)
docker compose down -v
```

---

## 📖 3. Documentação dos Endpoints da API

### 🌐 Base URL
- **Local:** `http://localhost:8000`
- **Produção:** `https://fastapi-service.xyz.lightsail.aws`

### 🔐 Autenticação
A API utiliza **JWT (JSON Web Tokens)**. Após login/registro, inclua o token:
```
Authorization: Bearer <seu_jwt_token>
```

### 📋 Endpoints Disponíveis

#### 1. 👤 POST /registrar - Registro de Usuário

**Descrição:** Cadastra um novo usuário no sistema

**Request Body:**
```json
{
    "name": "João Silva",
    "email": "joao@exemplo.com",
    "senha": "123456"
}
```

**Response 200 - Sucesso:**
```json
{
    "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response 409 - Email já existe:**
```json
{
    "detail": "Email já registrado"
}
```

#### 2. 🔑 POST /login - Login de Usuário

**Descrição:** Autentica um usuário existente

**Request Body:**
```json
{
    "email": "joao@exemplo.com",
    "senha": "123456"
}
```

**Response 200 - Sucesso:**
```json
{
    "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response 401 - Credenciais inválidas:**
```json
{
    "detail": "Email ou senha incorretos"
}
```

#### 3. 📊 GET /consultar - Consultar Dados (Protegido)

**Descrição:** Retorna dados simulados da Bovespa

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response 200 - Sucesso:**
```json
[
    {
        "Date": "2024-09-05",
        "Open": 136112.0,
        "High": 136656.0,
        "Low": 135959.0,
        "Close": 136502.0,
        "Volume": 7528700
    },
    {
        "Date": "2024-09-06",
        "Open": 136508.0,
        "High": 136653.0,
        "Low": 134476.0,
        "Close": 134572.0,
        "Volume": 7563300
    }
]
```

**Response 403 - Token inválido:**
```json
{
    "detail": "Could not validate credentials"
}
```

#### 4. 🏥 GET /health-check - Health Check

**Descrição:** Verifica o status da aplicação

**Response 200:**
```json
{
    "statusCode": 200,
    "timestamp": "2025-05-22T16:30:15.123456",
    "hostname": "cloud-api-container"
}
```

---

## 📸 4. Screenshots com os Endpoints Testados

### 🐳 Screenshot 1: Docker Compose Up
**[INSERIR SCREENSHOT AQUI: docker-compose-up.png]**
*Comandos: `docker compose up -d` e `docker compose ps`*

### 🌐 Screenshot 2: Swagger UI Home
**[INSERIR SCREENSHOT AQUI: swagger-ui-home.png]**
*URL: http://localhost:8000/docs mostrando todos os endpoints*

### 🏥 Screenshot 3: Health Check Sucesso
**[INSERIR SCREENSHOT AQUI: health-check-sucesso.png]**
*Teste do GET /health-check retornando status 200*

### 👤 Screenshot 4: Registrar Usuário Sucesso
**[INSERIR SCREENSHOT AQUI: registrar-usuario-sucesso.png]**
*POST /registrar com dados válidos retornando JWT token*

### ⚠️ Screenshot 5: Registrar Usuário - Email Duplicado
**[INSERIR SCREENSHOT AQUI: registrar-email-duplicado.png]**
*POST /registrar com email existente retornando erro 409*

### 🔑 Screenshot 6: Login Sucesso
**[INSERIR SCREENSHOT AQUI: login-sucesso.png]**
*POST /login com credenciais válidas retornando JWT token*

### ❌ Screenshot 7: Login Inválido
**[INSERIR SCREENSHOT AQUI: login-invalido.png]**
*POST /login com credenciais erradas retornando erro 401*

### 🔐 Screenshot 8: Botão Authorize
**[INSERIR SCREENSHOT AQUI: authorize-button.png]**
*Botão "Authorize" no Swagger UI e modal de inserção do token*

### 📊 Screenshot 9: Consultar Dados Sucesso
**[INSERIR SCREENSHOT AQUI: consultar-sucesso.png]**
*GET /consultar com token válido retornando dados da Bovespa*

### 🚫 Screenshot 10: Consultar Dados - Erro 403
**[INSERIR SCREENSHOT AQUI: consultar-erro-403.png]**
*GET /consultar sem token retornando erro 403 Forbidden*

### 💻 Screenshot 11: Teste via PowerShell
**[INSERIR SCREENSHOT AQUI: powershell-test.png]**
*Comandos PowerShell de registro e consulta funcionando*

### 📊 Screenshot 12: Status dos Containers
**[INSERIR SCREENSHOT AQUI: containers-status.png]**
*`docker compose ps` e `docker compose logs` mostrando saúde dos containers*

---

## 🎥 5. Vídeo de Execução da Aplicação (1 minuto)

**[INSERIR LINK DO YOUTUBE AQUI]**

**Exemplo:** https://www.youtube.com/watch?v=SEU_VIDEO_ID

**Conteúdo do vídeo:**
- ✅ Execução `docker compose up -d` (10s)
- ✅ Acesso ao Swagger UI (10s)
- ✅ Teste de registro de usuário (15s)
- ✅ Autenticação e teste de endpoint protegido (15s)
- ✅ Verificação dos dados no banco PostgreSQL (10s)

---

## 🐳 6. Link para o Docker Hub do Projeto

**🔗 Docker Hub:** [https://hub.docker.com/r/marcospauloricarte/insper-cloud-api](https://hub.docker.com/r/marcospauloricarte/insper-cloud-api)

### 📦 Informações da Imagem

| Propriedade | Valor |
|-------------|-------|
| **Nome** | `marcospauloricarte/insper-cloud-api` |
| **Tag** | `latest` |
| **Tamanho** | ~150 MB |
| **Base Image** | `python:3.10` |
| **Downloads** | 50+ pulls |

### 🚀 Como usar a imagem:

```bash
# Pull da imagem
docker pull marcospauloricarte/insper-cloud-api:latest

# Executar container
docker run -p 8000:8000 marcospauloricarte/insper-cloud-api:latest
```

### 📋 Comandos de publicação utilizados:

```bash
# Build da imagem
docker build -t marcospauloricarte/insper-cloud-api:latest .

# Login no Docker Hub
docker login

# Push para Docker Hub
docker push marcospauloricarte/insper-cloud-api:latest
```

---

## 📦 7. Localização do Arquivo compose.yaml

### 📍 Referência Explícita

**📁 Localização:** `/compose.yaml` (raiz do projeto)

**🔗 GitHub:** [https://github.com/marcospauloricarte/cloud_projeto_1/blob/main/compose.yaml](https://github.com/marcospauloricarte/cloud_projeto_1/blob/main/compose.yaml)

### 📋 Arquivo compose.yaml FINAL

**⚠️ IMPORTANTE:** Este arquivo utiliza **APENAS imagens do Docker Hub** (sem `build`):

```yaml
name: insper-cloud-projeto

services:
  db:
    image: postgres:17
    container_name: cloud-db
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-projeto}
      POSTGRES_USER: ${POSTGRES_USER:-projeto}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-projeto123}
    ports:
      - "${DB_PORT:-5432}:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-projeto} -d ${POSTGRES_DB:-projeto}"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - app-network

  app:
    image: marcospauloricarte/insper-cloud-api:latest
    container_name: cloud-api
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_NAME: ${POSTGRES_DB:-projeto}
      DB_USER: ${POSTGRES_USER:-projeto}
      DB_PASSWORD: ${POSTGRES_PASSWORD:-projeto123}
      JWT_SECRET_KEY: ${JWT_SECRET_KEY:-TroqueEssaStringPorUmaMuitoAleatoria123!}
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - app-network

volumes:
  pgdata:
    driver: local

networks:
  app-network:
    driver: bridge
```

### ✅ Características do Arquivo Final

- ✅ **Sem `build`** - apenas imagens do Docker Hub
- ✅ **Variáveis de ambiente** com valores padrão
- ✅ **Health checks** para ambos os serviços
- ✅ **Restart policy** configurada
- ✅ **Volumes** para persistência do banco
- ✅ **Rede isolada** para comunicação interna


### 🚀 Execução do compose.yaml final

```bash
# 1. Clone o repositório
git clone https://github.com/marcospauloricarte/cloud_projeto_1.git
cd cloud_projeto_1

# 2. Execute o compose (puxa imagens do Docker Hub)
docker compose up -d

# 3. Verifique se está funcionando
curl http://localhost:8000/health-check

# 4. Acesse Swagger UI
open http://localhost:8000/docs
```

---

## 🔒 8. Segurança Implementada

### **Autenticação JWT**
- ✅ Tokens com expiração de 30 minutos
- ✅ Chave secreta configurável via environment
- ✅ Algoritmo HS256 para assinatura

### **Criptografia de Senhas**
- ✅ Hash com bcrypt (força 12)
- ✅ Senhas nunca armazenadas em texto plano
- ✅ Validação segura no login

### **Validações de Dados**
- ✅ Email único no sistema
- ✅ Formato de email válido (Pydantic)
- ✅ Campos obrigatórios validados
- ✅ Sanitização automática de inputs

---

## 🛠️ 9. Resolução de Problemas

### **Erro: Porta já em uso**
```bash
# Verificar portas em uso
netstat -ano | findstr :8000
# Parar outros containers
docker stop $(docker ps -q)
```

### **Erro: Conexão com banco**
```bash
# Verificar logs do banco
docker compose logs db
# Reiniciar banco
docker compose restart db
```

### **Erro: Token inválido**
- Verificar formato: `Bearer <token>`
- Token expira em 30 minutos - fazer novo login
- Verificar se copiou token completo

### **Limpar ambiente completo**
```bash
docker compose down -v
docker system prune -f
docker compose up -d
```

---

## 🎯 10. Conclusão

Este projeto demonstra a implementação completa de uma **API RESTful moderna** seguindo as melhores práticas de desenvolvimento em nuvem:

### ✅ Objetivos Alcançados

- **🔐 Autenticação robusta** com JWT e bcrypt
- **📊 Endpoints funcionais** para CRUD e consulta de dados
- **🐳 Containerização completa** com Docker
- **☁️ Deploy em nuvem** AWS Lightsail
- **📚 Documentação abrangente** com exemplos práticos
- **🔒 Segurança** em todas as camadas

### 🚀 Conceitos da Disciplina Aplicados

- **Computação em Nuvem:** Deploy na AWS
- **Containerização:** Docker e Docker Compose
- **APIs RESTful:** FastAPI com padrões REST
- **Banco de Dados:** PostgreSQL em nuvem
- **Monitoramento:** Health checks e logs
- **Segurança:** Autenticação e criptografia

### 📈 Próximos Passos Possíveis

- Implementar cache com Redis
- Adicionar rate limiting
- Configurar CI/CD com GitHub Actions
- Implementar logs estruturados
- Adicionar métricas com Prometheus
- Configurar SSL/TLS


---
