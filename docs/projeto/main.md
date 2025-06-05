# 🚀 API RESTful - Projeto Computação em Nuvem 2025.1

**Autores:** Marcos Paulo Ricarte e Roberta Barros Teixeira  
**Disciplina:** Computação em Nuvem - Insper  
**Data:** Maio 2025

---

## 📋 1. Explicação do Projeto - Scrap do que foi feito

Este projeto implementa uma **API RESTful** para cadastro e autenticação de usuários, além de permitir a consulta de dados externos. Foi desenvolvida para a disciplina de Computação em Nuvem do Insper.

### 🎯 O que foi implementado:

**🔐 Sistema de Autenticação:**
- Cadastro de usuários com validação de email único
- Login seguro com criptografia de senhas usando bcrypt
- Autenticação via JWT (JSON Web Tokens)
- Proteção de rotas sensíveis

**📊 Consulta de Dados Externos:**
- Endpoint protegido para consulta da cotação USD/BRL
- Web scraping de dados da API AwesomeAPI
- Retorno estruturado em formato JSON com cotação atual

**🏥 Monitoramento:**
- Health check endpoint para verificação do status da aplicação
- Informações de timestamp e hostname

**🐳 Containerização:**
- Aplicação dockerizada usando FastAPI e Python 3.10
- Banco PostgreSQL 17 em container separado
- Docker Compose para orquestração de serviços
- Imagem publicada no Docker Hub

### 🛠️ Tecnologias Utilizadas

- **Backend:** FastAPI (Python 3.10)
- **Banco de Dados:** PostgreSQL 17
- **Autenticação:** JWT + bcrypt
- **Containerização:** Docker & Docker Compose
- **ORM:** SQLAlchemy
- **Validação:** Pydantic
- **Requisições HTTP:** httpx (assíncrono)

---

## 🚀 2. Como Executar a Aplicação

### 📋 Pré-requisitos

- **Docker Desktop** (versão mais recente)
- **Git** para clonar o repositório

### 📥 Passo 1: Clonar o Repositório

```bash
git clone https://github.com/marcospauloricarte/cloud_projeto_1.git
cd cloud_projeto_1
```

### 🔄 Passo 2: Configurar variáveis de ambiente (opcional)

```bash
cp .env.example .env
# Edite o arquivo .env se necessário
```

### 🐳 Passo 3: Executar com Docker Compose

```bash
# Subir toda a aplicação (API + Banco)
docker compose up -d

# Verificar se os containers estão rodando
docker compose ps
```

### 🌐 Passo 4: Acessar a Aplicação

- **Swagger UI (Interface Interativa):** http://localhost:8000/docs
- **Health Check:** http://localhost:8000/health-check

### 🛑 Parar a Aplicação

```bash
# Parar containers
docker compose down
```

---

## 📖 3. Documentação dos Endpoints da API

### 🌐 Base URL
- **Local:** `http://localhost:8000`

### 🔐 Autenticação
A API utiliza **JWT (JSON Web Tokens)**. Após login/registro, inclua o token no header Authorization:
`Authorization: Bearer SEU_TOKEN_JWT`

### 📋 Endpoints Disponíveis

#### 1. 👤 POST /registrar - Registro de Usuário

**Descrição:** Cadastra um novo usuário no sistema

**Request Body:**
```json
{
    "nome": "Usuário Teste",
    "email": "teste@example.com",
    "senha": "senha123"
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
    "detail": "Email já registrado."
}
```

#### 2. 🔑 POST /login - Login de Usuário

**Descrição:** Autentica um usuário existente

**Request Body:**
```json
{
    "email": "teste@example.com",
    "senha": "senha123"
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
    "detail": "Credenciais inválidas."
}
```

#### 3. 📊 GET /consultar - Consultar Cotação USD/BRL (Protegido)

**Descrição:** Retorna a cotação atual do dólar em relação ao real

**Header necessário:**
```
Authorization: Bearer SEU_TOKEN_JWT
```

**Response 200 - Sucesso:**
```json
{
    "date": "2025-05-24 14:30:45",
    "usd_brl": 5.1234
}
```

**Response 403 - Token inválido:**
```json
{
    "detail": "Token inválido ou expirado"
}
```

#### 4. 🏥 GET /health_check - Health Check

**Descrição:** Verifica o status da aplicação

**Response 200:**
```json
{
    "statusCode": 200,
    "timestamp": "2025-05-24T14:30:45.123456",
    "hostname": "cloud-api-container"
}
```

---

## 📸 4. Screenshots com os Endpoints Testados

### 🏥 Screenshot 1: Health Check Sucesso
![image](https://github.com/user-attachments/assets/851a64f1-e43c-44dc-8c7e-7bbc35f542dd)
*Teste do GET /health_check retornando status 200*

### 👤 Screenshot 2: Registrar Usuário Sucesso
![image](https://github.com/user-attachments/assets/28a0c4cf-8834-4045-8494-e689a6cae1ce)
*POST /registrar com dados válidos retornando JWT token*

### 🔑 Screenshot 3: Login Sucesso
![image](https://github.com/user-attachments/assets/987e3545-b5a3-4344-b9bc-9521af6a94dc)
*POST /login com credenciais válidas retornando JWT token*

### 📊 Screenshot 4: Consultar Dados Sucesso
![image](https://github.com/user-attachments/assets/211b05a1-f907-4369-82ee-ccf745e237b5)
*GET /consultar funcionando após autenticação*

### 🐳 Screenshot 5: Containers Rodando
![image](https://github.com/user-attachments/assets/527223ff-5c29-4a34-8314-31c76623847b)
*Docker containers em execução (docker ps)*

---

## 🎥 5. Vídeo de Execução da Aplicação 

https://youtu.be/rPPLQLgoXgw

---

## 🐳 6. Link para o Docker Hub do Projeto

**🔗 Docker Hub:** [https://hub.docker.com/r/marcospauloricarte/insper-cloud-api](https://hub.docker.com/r/marcospauloricarte/insper-cloud-api)

### 📦 Informações da Imagem

| Propriedade | Valor |
|-------------|-------|
| **Nome** | `marcospauloricarte/insper-cloud-api` |
| **Tag** | `latest` |
| **Base Image** | `python:3.10` |

### 🚀 Como usar a imagem:

```bash
# Pull da imagem
docker pull marcospauloricarte/insper-cloud-api:latest

# Executar container (exemplo)
docker run -p 8000:8000 marcospauloricarte/insper-cloud-api:latest
```

---

## 📦 7. Localização do Arquivo compose.yaml

### 📍 Referência Explícita

**📁 Localização:** `/compose.yaml` (raiz do projeto)

**🔗 GitHub:** [https://github.com/marcospaulorr/cloud_projeto_1](https://github.com/marcospaulorr/cloud_projeto_1)

### 📋 Arquivo compose.yaml FINAL

```yaml
name: insper-cloud-projeto

services:
  db:
    image: postgres:17
    container_name: cloud-db
    environment:
      POSTGRES_DB: ${DB_NAME:-projeto}
      POSTGRES_USER: ${DB_USER:-projeto}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-projeto123}
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
    image: marcospauloricarte/insper-cloud-api:latest  # Usando imagem do Docker Hub
    container_name: cloud-api
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_NAME: ${POSTGRES_DB:-projeto}
      DB_USER: ${POSTGRES_USER:-projeto}
      DB_PASSWORD: ${POSTGRES_PASSWORD:-projeto123}
      JWT_SECRET_KEY: ${JWT_SECRET_KEY:-TroqueEssaStringPorUmaMuitoAleatoria123!}
      ALGORITHM: ${ALGORITHM:-HS256}
      ACCESS_TOKEN_EXPIRE_MINUTES: ${ACCESS_TOKEN_EXPIRE_MINUTES:-30}
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

- ✅ **`image: marcospauloricarte/insper-cloud-api:latest`** - usa imagem do Docker Hub
- ✅ **Variáveis de ambiente** com valores padrão
- ✅ **Health checks** para o banco de dados
- ✅ **Restart policy** configurada
- ✅ **Volumes** para persistência do banco
- ✅ **Rede isolada** para comunicação interna

---

## 🔒 8. Segurança Implementada

### **Autenticação JWT**
- ✅ Tokens com expiração de 30 minutos
- ✅ Chave secreta configurável via environment
- ✅ Algoritmo HS256 para assinatura

### **Criptografia de Senhas**
- ✅ Hash com bcrypt
- ✅ Senhas nunca armazenadas em texto plano
- ✅ Validação segura no login

---

## 🔧 9. Resolução de Problemas Comuns

- **Erro de porta já em uso**: Verifique se a porta 8000 já está sendo usada por outro serviço
- **Erro de conexão com banco**: Verifique se o container do PostgreSQL está rodando
- **Token inválido**: Certifique-se de incluir "Bearer " antes do token JWT
- **Limpar ambiente**: Use `docker compose down -v` para remover todos os containers e volumes

---

## 🏗️ 10. AWS Lightsail - Etapa 2

### ☁️ Screenshot da Infraestrutura Funcionando na AWS

#### 🖥️ Container Service no Lightsail
![image](https://github.com/user-attachments/assets/ff2d86e2-89f5-42b8-a58f-dafb88a0f6bc)
*Container Service ativo no AWS Lightsail mostrando status e configurações*

#### 🗄️ Banco de Dados no Lightsail
![image](https://github.com/user-attachments/assets/d204c68d-ebbe-479a-a4a7-6a043c06b6df)
*Banco de dados PostgreSQL gerenciado no AWS Lightsail*

### 🌐 Screenshots dos Endpoints AWS Testados

#### 🏥 Health Check no Lightsail
![image](https://github.com/user-attachments/assets/c63a22ba-15d6-41d4-a93d-041da67d6629)
*Endpoint health_check funcionando no ambiente Lightsail*

#### 👤 Registro no Lightsail
![image](https://github.com/user-attachments/assets/f5682262-c3fc-4564-ada7-ddf88f2c4c3c)
*Endpoint de registro funcionando no Lightsail*

#### 🔑 Login no Lightsail
![image](https://github.com/user-attachments/assets/66cf434b-7ddb-4f8b-8e01-ed69b5d396d5)
*Endpoint de login funcionando no Lightsail*

#### 📊 Consulta USD/BRL no Lightsail
![image](https://github.com/user-attachments/assets/7cf560b5-1153-4459-9ed7-23232e00a0b8)
*Endpoint de consulta de cotação funcionando no Lightsail*

### 💰 Custos da AWS

#### 💵 Custos Atuais
![image](https://github.com/user-attachments/assets/90034c61-9a5a-467f-80d0-ff2fe546c7c9)
*Custos da AWS no dia da submissão*

#### 📈 Projeção de Custos para Diferentes Escalas

| Configuração | Recursos | Custo Mensal Estimado (USD) |
|--------------|----------|------------------------------|
| **1 instância (atual)** | 1x Container Micro (1 vCPU, 512MB RAM)<br>1x PostgreSQL Micro (1 vCPU, 1GB RAM) | $7.00 (Container)<br>$15.00 (DB)<br>**Total: $22.00** |
| **5 instâncias** | 5x Container Micro (1 vCPU, 512MB RAM)<br>1x PostgreSQL Micro (1 vCPU, 1GB RAM) | $35.00 (Containers)<br>$15.00 (DB)<br>**Total: $50.00** |
| **10 instâncias** | 10x Container Micro (1 vCPU, 512MB RAM)<br>1x PostgreSQL Small (1 vCPU, 2GB RAM) | $70.00 (Containers)<br>$30.00 (DB)<br>**Total: $100.00** |

### 🎥 Vídeo da Aplicação Funcionando no Lightsail

https://youtu.be/qpfAiKzi50o

**Conteúdo do vídeo:**
- Acesso ao AWS Lightsail Container Service
- Demonstração da API funcionando na nuvem
- Teste de cadastro e login de usuário
- Armazenamento de dados no banco PostgreSQL gerenciado
- Consulta da cotação USD/BRL através da API

### 🏗️ Arquitetura da Solução na AWS

```
                      ┌─────────────────┐
                      │                 │
 Internet ───────────►│  Load Balancer  │
                      │                 │
                      └────────┬────────┘
                               │
                               ▼
                      ┌─────────────────┐
                      │  Container      │
                      │  Service        │
                      │  (FastAPI)      │
                      └────────┬────────┘
                               │
                               ▼
                      ┌─────────────────┐
                      │  PostgreSQL     │
                      │  Database       │
                      │                 │
                      └─────────────────┘
```

### 🛡️ Segurança na Nuvem

- ✅ **Configuração segura de banco de dados**: Credenciais gerenciadas via variáveis de ambiente
- ✅ **HTTPS**: Tráfego criptografado entre cliente e API
- ✅ **Isolamento de rede**: Configuração adequada de VPC e regras de firewall
- ✅ **Health checks**: Monitoramento contínuo para detectar problemas

### 📊 Monitoramento e Gerenciamento

- ✅ **Dashboard Lightsail**: Métricas de uso de CPU e memória
- ✅ **Logs de aplicação**: Registros de acessos e erros
- ✅ **Alertas**: Configurados para notificação em caso de alta utilização ou falhas

---

## 🏆 11. Conclusão do Projeto

A implementação deste projeto demonstra a aplicação prática dos conceitos de computação em nuvem:

- **Containerização eficiente** com Docker
- **Arquitetura de microsserviços** separando aplicação e banco de dados
- **Implantação em nuvem** utilizando AWS Lightsail
- **Gerenciamento de custos** com projeções para diferentes escalas
- **Monitoramento e observabilidade** através de health checks e logs

A solução desenvolvida é escalável, segura e atende aos requisitos propostos na disciplina, aplicando boas práticas de desenvolvimento em todas as etapas, desde o ambiente local até o deploy em produção na AWS.

---
