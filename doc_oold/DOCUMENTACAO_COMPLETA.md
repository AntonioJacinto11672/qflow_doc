# 📚 DOCUMENTAÇÃO COMPLETA - SISTEMA TICKET DISPENSER

**Data:** Maio 2026  
**Versão do Documento:** 1.0  
**Status:** Documentação Operacional Completa

---

## 📖 ÍNDICE

1. [Visão Geral do Projeto](#visão-geral-do-projeto)
2. [Pré-requisitos e Instalação](#pré-requisitos-e-instalação)
3. [Configuração Inicial](#configuração-inicial)
4. [Execução em Modo Desenvolvimento](#execução-em-modo-desenvolvimento)
5. [Como Funciona o Sistema](#como-funciona-o-sistema)
6. [Arquitetura Detalhada](#arquitetura-detalhada)
7. [Estrutura de Diretórios](#estrutura-de-diretórios)
8. [Endpoints Principais](#endpoints-principais)
9. [Comunicação entre APIs](#comunicação-entre-apis)
10. [Troubleshooting](#troubleshooting)

---

## 🎯 Visão Geral do Projeto

### O Que É?

**Ticket Dispenser** é um **sistema distribuído de gerenciamento de senhas para agências** que funciona em tempo real, permitindo:

- ✅ **Dispensar senhas digitais** em filas de agências
- ✅ **Atender clientes** chamando as senhas em balcões
- ✅ **Gerar relatórios** consolidados de múltiplas agências
- ✅ **Funcionar offline** em cada agência (independente do servidor central)
- ✅ **Sincronizar automaticamente** quando há conexão com a internet

### Cenário de Uso Real

```
Cliente chega na agência
         ↓
Pega uma senha no dispensador
         ↓
Espera ser chamado no display
         ↓
Operador chama a senha no sistema
         ↓
Evento é enviado para servidor central
         ↓
Relatório em tempo real é atualizado na nuvem
```

### Dois Sistemas Complementares

| Aspecto | Local API | Cloud API |
|---------|-----------|-----------|
| **Localização** | Em cada agência (on-premise) | Servidor central (nuvem) |
| **Banco de dados** | SQLite (local) | MySQL (centralizado) |
| **Autenticação** | JWT | Laravel Sanctum |
| **Conectividade** | Funciona offline | Requer internet |
| **Responsabilidade** | Operações em tempo real | Relatórios e dashboards |
| **Comunicação** | Publica eventos para RabbitMQ | Consome eventos do RabbitMQ |

---

## 📦 Pré-requisitos e Instalação

### O Que Você Precisa

#### Sistema Operacional
- **Windows** 10/11, **macOS** 10.15+, ou **Linux** (Ubuntu 20.04+)

#### Software Obrigatório

1. **Docker Desktop** (recomendado para produção/desenvolvimento)
   - Download: https://www.docker.com/products/docker-desktop
   - Inclui Docker Compose automaticamente

2. **PHP 8.3+** (se rodar sem Docker)
   - Download: https://www.php.net/downloads.php
   - Windows: Use o instalador oficial

3. **Composer** (gerenciador de pacotes PHP)
   - Download: https://getcomposer.org/download/
   - Instala-se globalmente

4. **Node.js + npm** (para build do frontend)
   - Download: https://nodejs.org/ (versão LTS)
   - npm vem incluído

5. **Git** (controle de versão)
   - Download: https://git-scm.com/

### Instalação Passo a Passo

#### Passo 1: Preparar o Ambiente

```bash
# No Windows (PowerShell)
cd C:\xampp\htdocs\externa_api

# No Linux/macOS
cd ~/www/externa_api
```

#### Passo 2: Clonar o Repositório (ou usar pasta existente)

Se estiver usando pasta já existente, pule para Passo 3.

```bash
# Se precisar clonar o repositório
git clone <seu-repositório> .
```

#### Passo 3: Instalar Dependências PHP

**Opção A: Com Composer (recomendado)**

```bash
# Para LOCAL API
cd dispenser-local-api-main
composer install
cd ..

# Para CLOUD API
cd dispenser-remote-api-main
composer install
cd ..
```

**Opção B: Com Docker**
- Pule este passo, o Dockerfile cuida disso automaticamente

#### Passo 4: Instalar Dependências Node.js/Frontend

```bash
# Para LOCAL API
cd dispenser-local-api-main
npm install
npm run build  # ou npm run dev para desenvolvimento
cd ..

# Para CLOUD API
cd dispenser-remote-api-main
npm install
npm run build
cd ..
```

#### Passo 5: Gerar Chaves e Configurações

```bash
# Para LOCAL API
cd dispenser-local-api-main
cp .env.example .env
php artisan key:generate          # Gera APP_KEY
php artisan jwt:secret            # Gera JWT_SECRET (se usando JWT)
cd ..

# Para CLOUD API
cd dispenser-remote-api-main
cp .env.example .env
php artisan key:generate
cd ..
```

---

## ⚙️ Configuração Inicial

### LOCAL API - Arquivo `.env`

#### Configurações Críticas

```env
# Aplicação
APP_NAME="Ticket Dispenser Local"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8080
APP_TIMEZONE=America/Sao_Paulo

# Banco de Dados - SQLite (dois bancos separados)
DB_CONNECTION=sqlite
DB_DATABASE=database/readmodel.sqlite    # Base de leitura
EVENTSTORE_DB_DATABASE=database/eventstore.sqlite  # Event Store

# Identificação da Agência (OBRIGATÓRIO)
AGENCY_ID=abad8412-b7de-41f4-8302-464fe1751a41  # UUID da sua agência

# Autenticação JWT
JWT_SECRET=seu-secret-aqui-gerado-pelo-artisan
JWT_TTL=1440  # Token válido por 1440 minutos (24 horas)
JWT_REFRESH_TTL=20160  # Refresh token válido por 20160 minutos (14 dias)

# RabbitMQ - Sincronização com Cloud
RABBITMQ_HOST=rabbitmq.exemplo.com
RABBITMQ_PORT=5672
RABBITMQ_USER=dispenser_user
RABBITMQ_PASSWORD=sua_senha
RABBITMQ_VHOST=/

# WebSocket (para atualizações em tempo real)
WEBSOCKET_PORT=8081
WEBSOCKET_HOST=0.0.0.0

# Atualizações
ZIP_WEBAPPS_PATH=/var/www/webapps
WINDOWS_APP_UPDATES_PATH=/var/www/windows
APK_UPDATES_PATH=/var/www/apk

# Cache e Fila
CACHE_DRIVER=file
QUEUE_CONNECTION=sync  # ou 'rabbitmq' para processar via RabbitMQ

# Logging
LOG_CHANNEL=stack
LOG_LEVEL=debug
```

#### O Que Cada Variável Faz

| Variável | Propósito | Obrigatória? |
|----------|-----------|------------|
| `APP_ENV` | Modo (local, production) | Sim |
| `AGENCY_ID` | UUID que identifica a agência | Sim |
| `DB_DATABASE` | Caminho do banco SQLite principal | Sim |
| `EVENTSTORE_DB_DATABASE` | Caminho do Event Store SQLite | Sim |
| `JWT_SECRET` | Chave secreta para gerar tokens | Sim |
| `RABBITMQ_*` | Credenciais de RabbitMQ | Não (opcional offline) |
| `WEBSOCKET_*` | Configuração do WebSocket | Não (opcional) |

### CLOUD API - Arquivo `.env`

#### Configurações Críticas

```env
# Aplicação
APP_NAME="Ticket Dispenser Cloud"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8084
APP_TIMEZONE=America/Sao_Paulo

# Banco de Dados - MySQL
DB_CONNECTION=mysql
DB_HOST=mysql.exemplo.com
DB_PORT=3306
DB_DATABASE=ticket_dispenser
DB_USERNAME=dispenser_user
DB_PASSWORD=sua_senha

# Autenticação - Laravel Sanctum
SANCTUM_STATEFUL_DOMAINS=localhost,127.0.0.1,*.example.com
SESSION_DOMAIN=localhost

# RabbitMQ - Comunicação com Local APIs
RABBITMQ_HOST=rabbitmq.exemplo.com
RABBITMQ_PORT=5672
RABBITMQ_USER=dispenser_user
RABBITMQ_PASSWORD=sua_senha
RABBITMQ_VHOST=/

# Cache e Fila
CACHE_DRIVER=redis
CACHE_REDIS_HOST=redis.exemplo.com
CACHE_REDIS_PORT=6379
QUEUE_CONNECTION=rabbitmq

# Logging
LOG_CHANNEL=stack
LOG_LEVEL=debug

# Mail (opcional para notificações)
MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=465
MAIL_USERNAME=seu_user
MAIL_PASSWORD=sua_password
```

---

## 🚀 Execução em Modo Desenvolvimento

### Opção 1: Com Docker Compose (RECOMENDADO)

Mais simples e reproduzível em qualquer máquina.

#### Para LOCAL API

```bash
# Acesse a pasta da Local API
cd dispenser-local-api-main

# Inicie os contêineres
docker-compose up -d

# Verifique os logs
docker-compose logs -f

# Para parar
docker-compose down
```

**Acesso:**
- API: http://localhost:8080
- WebSocket: localhost:8081

#### Para CLOUD API

```bash
# Acesse a pasta da Cloud API
cd dispenser-remote-api-main

# Inicie os contêineres (inclui MySQL e RabbitMQ)
docker-compose up -d

# Verifique os logs
docker-compose logs -f

# Para parar
docker-compose down
```

**Acesso:**
- API: http://localhost:8084
- MySQL: localhost:3377
- RabbitMQ Management: http://localhost:15672 (user: admin, pass: admin123)

### Opção 2: Sem Docker (Direto na Máquina)

Requer PHP, MySQL e RabbitMQ instalados localmente.

#### Para LOCAL API

```bash
cd dispenser-local-api-main

# 1. Gerar as bases SQLite
php artisan migrate

# 2. (Opcional) Popular com dados de teste
php artisan db:seed

# 3. Compilar assets frontend (Vite)
npm run dev

# 4. Iniciar servidor de desenvolvimento
php artisan serve --port=8080

# 5. (Opcional) Iniciar WebSocket em outro terminal
php artisan tinker
# Dentro do tinker:
# \App\Services\WebSocketService::start()
```

**Logs em tempo real:**
```bash
# Em outro terminal
tail -f storage/logs/laravel.log
```

#### Para CLOUD API

Requer MySQL e RabbitMQ rodando (use Docker se precisar):

```bash
# Apenas MySQL via Docker (deixe rodando)
docker run -d \
  --name mysql_ticket_dispenser \
  -e MYSQL_DATABASE=ticket_dispenser \
  -e MYSQL_USER=dispenser_user \
  -e MYSQL_PASSWORD=dispenser_pass \
  -e MYSQL_ROOT_PASSWORD=root_pass \
  -p 3306:3306 \
  mysql:8.0

# Apenas RabbitMQ via Docker (deixe rodando)
docker run -d \
  --name rabbitmq_ticket_dispenser \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin123 \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3.13-management

# Agora rode a Cloud API
cd dispenser-remote-api-main

# 1. Criar banco de dados (migrations)
php artisan migrate

# 2. (Opcional) Popular dados
php artisan db:seed

# 3. Compilar assets
npm run dev

# 4. Iniciar servidor
php artisan serve --port=8084

# 5. (Em outro terminal) Iniciar consumer de RabbitMQ
php artisan queue:work rabbitmq --queue=cloud_events_processor
```

### Monitoramento em Desenvolvimento

#### Ver Logs em Tempo Real

```bash
# LOCAL API
cd dispenser-local-api-main
tail -f storage/logs/laravel.log

# CLOUD API
cd dispenser-remote-api-main
tail -f storage/logs/laravel.log
```

#### Verificar Status das APIs

```bash
# LOCAL API
curl http://localhost:8080/health

# CLOUD API
curl http://localhost:8084/health
```

#### Acessar RabbitMQ Management UI

```
URL: http://localhost:15672
User: admin
Pass: admin123
```

Aqui você pode:
- Ver número de mensagens nas filas
- Monitorar exchanges e bindings
- Testar conexões

---

## 🔄 Como Funciona o Sistema

### Fluxo 1: Cliente Pega Senha na Agência (LOCAL API)

```
┌─────────────────────────────────────────────────────────────────┐
│ LOCAL API - Operação em Tempo Real                              │
└─────────────────────────────────────────────────────────────────┘

1. Cliente chega na agência
   ↓
2. Cliente pressiona botão no kiosk (Windows App / Web)
   ↓
3. Requisição HTTP: POST /tickets
   {
     "service_id": "abc123",
     "priority": "normal"
   }
   ↓
4. TicketController recebe e valida
   ↓
5. TicketService cria a senha
   ↓
6. Domain Event TicketCreated é gerado
   ↓
7. Event é persistido no Event Store (SQLite)
   ↓
8. Projector atualiza Read Model (SQLite)
   ↓
9. WebSocket publica em tempo real para displays
   ↓
10. Display mostra: "SENHA 0042 - BALCÃO 2"
   ↓
11. Event é publicado no RabbitMQ (Outbox Pattern)
   ↓
12. Cloud API consome e atualiza relatórios
```

### Fluxo 2: Operador Chama Senha (LOCAL API)

```
┌─────────────────────────────────────────────────────────────────┐
│ LOCAL API - Chamada de Senha                                    │
└─────────────────────────────────────────────────────────────────┘

1. Operador clica em "Chamar Próxima"
   ↓
2. Requisição: POST /tickets/call-next
   {
     "desk_id": "desk-123"
   }
   ↓
3. TicketController procura próxima senha da fila
   ↓
4. Domain Event TicketCalled é criado
   ↓
5. Event é persistido + Read Model atualizado
   ↓
6. Dois WebSockets ativados:
   - Display mostra senha chamada
   - Painel do operador atualiza
   ↓
7. Event enviado para RabbitMQ
   ↓
8. Cloud API recebe e:
   - Atualiza tempo de atendimento
   - Incrementa métrica do balcão
   - Atualiza dashboard em tempo real
```

### Fluxo 3: Sincronização Cloud → Local (Configuration Push)

```
┌─────────────────────────────────────────────────────────────────┐
│ CLOUD API - Envia configuração para Local API                   │
└─────────────────────────────────────────────────────────────────┘

1. Admin Cloud cria novo serviço
   ↓
2. Cloud API publica evento em RabbitMQ:
   Exchange: cloud.sync
   Routing Key: cloud.sync.{AGENCY_ID}.service.created
   Payload: {serviço json}
   ↓
3. Local API consumer escuta:
   - Consumer filtra por seu AGENCY_ID
   - Recebe apenas eventos relevantes
   ↓
4. Local API processa e:
   - Cria o serviço no banco local
   - Atualiza cache
   - Gera confirmação de receipt
   ↓
5. Local continua funcionando normalmente
```

### Padrão Event Sourcing Simplificado

O sistema usa um padrão de **Event Sourcing** na Local API:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Command     │────▶│  Domain      │────▶│  Event       │
│              │     │  Service     │     │  Store       │
└──────────────┘     └──────────────┘     └──────────────┘
                            ▲                     │
                            │                     ▼
                            │              ┌──────────────┐
                            │              │   Projector  │
                            │              └──────────────┘
                            │                     │
                            │                     ▼
                            │              ┌──────────────┐
                            └──────────────│ Read Model   │
                                           │ (Consultas)  │
                                           └──────────────┘
```

**O Que Significa:**

1. **Command**: Usuário faz uma ação (criar senha)
2. **Domain Service**: Lógica de negócio válida o comando
3. **Event Store**: Evento é registrado permanentemente
4. **Projector**: Transforma evento em dados legíveis
5. **Read Model**: Base otimizada para consultas rápidas

**Benefícios:**
- ✅ Histórico completo (auditoria)
- ✅ Recuperação de falhas
- ✅ Reconstrução de estado
- ✅ Escalabilidade (CQRS - Command Query Responsibility Segregation)

### Padrão RabbitMQ: Outbox

```
┌─────────────────────────────────────────────────────────────────┐
│ LOCAL API - Garantir Entrega de Eventos                         │
└─────────────────────────────────────────────────────────────────┘

Quando um evento ocorre:

1. Event é salvo no Event Store
2. Read Model é atualizado
3. Evento é salvo na tabela 'outbox' (mesmo banco)
   ↓
   (Se RabbitMQ falhar aqui, não há problema - está salvo)
   ↓
4. Background job processa outbox
   ↓
5. Publica eventos para RabbitMQ
   ↓
6. Marca como publicado e remove da outbox
   ↓
   (Se falhar na publicação, fica na outbox e tenta novamente)
```

**Por Que?** Garante que NENHUM evento se perde, mesmo se RabbitMQ cair.

---

## 🏗️ Arquitetura Detalhada

### Local API - Arquitetura Interna

#### Camadas de Negócio

```
┌────────────────────────────────────────────────────┐
│              HTTP Routes                           │
│            (routes/api.php)                        │
└────────────────┬─────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────┐
│           Controllers                              │
│  (app/Http/Controllers/Api/)                      │
│  - TicketController                               │
│  - UserController                                 │
│  - DeskController                                 │
└────────────────┬─────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────┐
│           Services                                 │
│  (app/Services/)                                  │
│  - TicketService (lógica de negócio)            │
│  - UserService                                    │
│  - DeskService                                    │
└────────────────┬─────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────┐
│        Domain Layer                                │
│  (app/Domain/Events/)                            │
│  - Domain Events                                  │
│  - Event Factory                                  │
│  - Value Objects                                  │
└────────────────┬─────────────────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
┌───────▼──────┐   ┌──────▼──────┐
│ Event Store  │   │  Projectors  │
│   (SQLite)   │   │              │
│   Histórico  │   │ Atualiza Read│
│              │   │ Model        │
└──────────────┘   └──────┬───────┘
                          │
                   ┌──────▼──────┐
                   │ Read Model  │
                   │ (SQLite)    │
                   │ Consultas   │
                   └─────────────┘
```

#### Fluxo de Dados para Query (Leitura)

```
GET /tickets
    ↓
TicketController::index()
    ↓
TicketService::getAll()
    ↓
Ticket::all()  (Query na Read Model SQLite)
    ↓
Retorna JSON
```

#### Fluxo de Dados para Command (Escrita)

```
POST /tickets
{service_id: "123"}
    ↓
TicketController::store()
    ↓
TicketService::create($data)
    ↓
TicketCreated::dispatch($data)  (Domain Event)
    ↓
Event Store: SAVE evento
    ↓
TicketProjector::handle($event)  (Atualiza Read Model)
    ↓
Outbox: SAVE para publicar depois
    ↓
Queue job publica em RabbitMQ
    ↓
Retorna nova senha JSON
```

### Cloud API - Arquitetura Multi-tenant

#### Camadas de Negócio

```
┌────────────────────────────────────────────────────┐
│              HTTP Routes                           │
│            (routes/api.php)                        │
└────────────────┬─────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────┐
│           Controllers                              │
│  (app/Http/Controllers/Api/)                      │
│  Middleware: authority, agency_context            │
└────────────────┬─────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────┐
│           Services                                 │
│  (app/Services/)                                  │
│  - AgencyService                                  │
│  - DashboardService                               │
│  - ReportService                                  │
└────────────────┬─────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────┘
│        Repository Layer                            │
│  (app/Repositories/)                              │
│  - Filtra automaticamente por agency_id           │
│  - Protege dados de outras agências              │
└────────────────┬─────────────────────────────────┐
                 │
┌────────────────▼─────────────────────────────────┐
│            Models                                  │
│  (app/Models/)                                    │
│  - Agency                                         │
│  - Ticket                                         │
│  - User                                           │
│  - Desk                                           │
│  - TicketEvent (sincronia com Local)             │
└────────────────┬─────────────────────────────────┐
                 │
         ┌───────▼────────┐
         │                │
    ┌────▼────┐    ┌─────▼─────┐
    │  MySQL  │    │ RabbitMQ  │
    │  8.0    │    │ Consumer  │
    │         │    │  (Queue)  │
    └─────────┘    └───────────┘
```

#### Fluxo Multi-tenant

Cada query filtra automaticamente por `agency_id`:

```
User A faz: GET /v1/reports
    ↓
Controller verifica token e extrai agency_id
    ↓
Service chama: Report::where('agency_id', $agency_id)
    ↓
Retorna APENAS dados da agência A
    ↓
User B não vê os dados de User A
```

### Comunicação via RabbitMQ

#### Estrutura de Exchanges e Queues

```
┌───────────────────────────────────────────────────┐
│                RabbitMQ                           │
├───────────────────────────────────────────────────┤
│                                                   │
│ Exchange: ticket_dispenser_events (topic)         │
│ ├─ Routing Key: local.ticket.created             │
│ ├─ Routing Key: local.ticket.called              │
│ ├─ Routing Key: local.user.*                     │
│ └─ Routing Key: local.desk.*                     │
│         ↓ Bind to Queue                          │
│ Queue: cloud_events_processor                    │
│         ↓                                        │
│ Cloud API Consumer                               │
│                                                   │
├───────────────────────────────────────────────────┤
│                                                   │
│ Exchange: cloud.sync (topic)                     │
│ ├─ cloud.sync.AGENCY_1.service.created          │
│ ├─ cloud.sync.AGENCY_2.user.updated             │
│ └─ cloud.sync.AGENCY_N.*.*                       │
│         ↓ Bind to Queue                          │
│ Queue: local_sync.AGENCY_1                       │
│ Queue: local_sync.AGENCY_2                       │
│ ...                                              │
│         ↓                                        │
│ Cada Local API Consumer (filtra seu AGENCY_ID)   │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 📂 Estrutura de Diretórios

### Local API (`dispenser-local-api-main/`)

```
dispenser-local-api-main/
│
├── 📄 ARCHITECTURE.md              # Documentação de arquitetura
├── 📄 SETUP.md                     # Guia de instalação
├── 📄 README.md                    # Visão geral
├── 📄 composer.json                # Dependências PHP
├── 📄 package.json                 # Dependências Node/npm
├── 📄 .env.example                 # Template de configuração
├── 📄 docker-compose.yml           # Configuração Docker
├── 📄 Dockerfile                   # Imagem Docker
│
├── 📁 app/                         # Código-fonte principal
│   ├── 📁 Http/
│   │   ├── 📁 Controllers/         # Controllers da API
│   │   │   ├── TicketController.php    # Gerencia senhas
│   │   │   ├── UserController.php      # Gerencia usuários
│   │   │   ├── DeskController.php      # Gerencia balcões
│   │   │   ├── ServiceController.php   # Gerencia serviços
│   │   │   ├── DeviceController.php    # Gerencia dispositivos
│   │   │   └── AuthController.php      # Autenticação JWT
│   │   ├── 📁 Requests/           # Validação de requests
│   │   ├── 📁 Resources/          # Recursos de resposta
│   │   └── 📁 Middleware/         # Middlewares (JWT, CORS)
│   │
│   ├── 📁 Domain/                 # Lógica de domínio (Event Sourcing)
│   │   └── 📁 Events/             # Domain Events
│   │       ├── TicketCreated.php
│   │       ├── TicketCalled.php
│   │       ├── TicketFinished.php
│   │       ├── UserCreatedEvent.php
│   │       └── ... (outros eventos)
│   │
│   ├── 📁 Services/               # Serviços de negócio
│   │   ├── TicketService.php      # Lógica de senhas
│   │   ├── UserService.php        # Lógica de usuários
│   │   ├── DeskService.php        # Lógica de balcões
│   │   ├── EventService.php       # Gerenciamento de eventos
│   │   └── RabbitMQService.php    # Publicação em RabbitMQ
│   │
│   ├── 📁 Models/                 # Modelos (Eloquent ORM)
│   │   ├── Ticket.php             # Modelo de senha
│   │   ├── User.php               # Modelo de usuário
│   │   ├── Desk.php               # Modelo de balcão
│   │   ├── Service.php            # Modelo de serviço
│   │   ├── Device.php             # Modelo de dispositivo
│   │   ├── TicketEvent.php        # Evento persistido
│   │   └── Agency.php             # Agência
│   │
│   ├── 📁 Infrastructure/         # Infraestrutura e implementações
│   │   ├── 📁 Projectors/         # Projectors (atualizam Read Model)
│   │   │   ├── TicketProjector.php
│   │   │   ├── UserProjector.php
│   │   │   └── DeskProjector.php
│   │   ├── 📁 RabbitMQ/
│   │   │   ├── Publisher.php      # Publica eventos
│   │   │   └── Consumer.php       # Consome eventos do Cloud
│   │   └── 📁 Persistence/
│   │       ├── EventRepository.php
│   │       └── EventStore.php
│   │
│   ├── 📁 Console/                # Comandos Artisan
│   │   ├── Commands/
│   │   │   ├── ProcessOutbox.php  # Processa fila de eventos
│   │   │   ├── SyncWithCloud.php  # Sincronização manual
│   │   │   └── MigrateData.php    # Migrações de dados
│   │
│   ├── 📁 Providers/              # Service Providers
│   │   ├── AppServiceProvider.php
│   │   ├── EventServiceProvider.php
│   │   └── RouteServiceProvider.php
│   │
│   └── helpers.php                # Funções auxiliares
│
├── 📁 bootstrap/                  # Bootstrap da aplicação
│   ├── app.php                    # Criação da aplicação
│   └── providers.php              # Carregamento de providers
│
├── 📁 config/                     # Configurações
│   ├── app.php                    # Config da app
│   ├── auth.php                   # Config de autenticação
│   ├── database.php               # Config de banco
│   ├── jwt.php                    # Config de JWT
│   ├── queue.php                  # Config de filas
│   ├── rabbitmq.php               # Config de RabbitMQ
│   ├── cache.php
│   ├── filesystems.php
│   ├── logging.php
│   ├── mail.php
│   ├── session.php
│   └── websocket.php              # Config de WebSocket
│
├── 📁 database/                   # Banco de dados
│   ├── 📁 migrations/             # Migrações (schema)
│   │   ├── create_tickets_table.php
│   │   ├── create_users_table.php
│   │   ├── create_desks_table.php
│   │   ├── create_services_table.php
│   │   ├── create_ticket_events_table.php
│   │   └── ... (mais migrações)
│   ├── 📁 seeders/                # Dados iniciais
│   │   ├── DatabaseSeeder.php
│   │   ├── UserSeeder.php
│   │   └── ... (mais seeders)
│   └── 📁 factories/              # Factories para testes
│       ├── TicketFactory.php
│       └── UserFactory.php
│
├── 📁 routes/                     # Rotas da aplicação
│   ├── api.php                    # Rotas da API
│   ├── web.php                    # Rotas web (se houver)
│   └── console.php                # Rotas de console
│
├── 📁 resources/                  # Assets e views
│   ├── 📁 js/                     # JavaScript (Vite)
│   │   ├── app.js
│   │   └── ... (componentes)
│   ├── 📁 css/                    # Estilos
│   │   └── app.css
│   └── 📁 views/                  # Blade templates (se houver)
│
├── 📁 storage/                    # Arquivos em tempo de execução
│   ├── 📁 app/                    # Uploads de usuários
│   ├── 📁 logs/                   # Arquivos de log
│   ├── 📁 framework/              # Cache, sessions
│   │   ├── 📁 cache/
│   │   └── 📁 sessions/
│   └── 📁 dispatcher/             # Dados operacionais
│
├── 📁 public/                     # Raiz web
│   ├── index.php                  # Entry point
│   ├── robots.txt
│   └── logos/
│
├── 📁 tests/                      # Testes
│   ├── TestCase.php               # Base para testes
│   ├── 📁 Feature/                # Testes de features
│   │   ├── TicketControllerTest.php
│   │   └── ...
│   ├── 📁 Unit/                   # Testes unitários
│   │   ├── TicketServiceTest.php
│   │   └── ...
│   └── phpunit.xml                # Configuração do PHPUnit
│
├── 📁 vendor/                     # Dependências (auto-gerado)
│
├── 📁 node_modules/               # Dependências Node (auto-gerado)
│
├── artisan                        # CLI do Laravel
├── vite.config.js                 # Config do Vite
└── phpunit.xml                    # Config de testes

```

### Cloud API (`dispenser-remote-api-main/`)

Estrutura similar, com algumas diferenças:

```
dispenser-remote-api-main/
│
├── 📄 ARCHITECTURE.md              # Documentação
├── 📄 IMPLEMENTATION_GUIDE.md       # Guia de implementação
├── 📄 README.md                    # Visão geral
├── 📄 composer.json
├── 📄 package.json
├── 📄 docker-compose.yml           # MySQL + RabbitMQ + App
│
├── 📁 app/
│   ├── 📁 Http/Controllers/Api/
│   │   ├── AgencyController.php         # Agências
│   │   ├── DashboardController.php      # Dashboards
│   │   ├── ReportController.php         # Relatórios
│   │   ├── UserController.php
│   │   ├── DeskController.php
│   │   ├── ServiceController.php
│   │   ├── TicketController.php
│   │   ├── DeviceController.php
│   │   ├── AlertController.php
│   │   ├── ContentController.php
│   │   ├── TemplateController.php
│   │   └── AuthController.php
│   │
│   ├── 📁 Services/
│   │   ├── AgencyService.php
│   │   ├── DashboardService.php
│   │   ├── ReportService.php
│   │   ├── TicketService.php
│   │   ├── RabbitMQEventPublisher.php   # Publica para Local
│   │   └── ... (outros serviços)
│   │
│   ├── 📁 Models/
│   │   ├── Agency.php
│   │   ├── Ticket.php
│   │   ├── TicketEvent.php              # Sincronia com Local
│   │   ├── User.php
│   │   ├── Desk.php
│   │   ├── Service.php
│   │   ├── Device.php
│   │   ├── DeviceVersionHistory.php
│   │   ├── Alert.php
│   │   ├── Content.php
│   │   ├── OperatorSession.php
│   │   └── ... (mais modelos)
│   │
│   ├── 📁 Infrastructure/
│   │   ├── 📁 RabbitMQ/
│   │   │   ├── Consumer.php             # Consome eventos do Local
│   │   │   ├── Publisher.php            # Publica para Local
│   │   │   ├── 📁 Handlers/
│   │   │   │   ├── TicketEventHandler.php
│   │   │   │   ├── AssignmentEventHandler.php
│   │   │   │   └── ResourceSyncHandler.php
│   │   │   └── EventProcessor.php
│   │   └── 📁 RateLimiting/
│   │       └── ApiRateLimiter.php
│   │
│   ├── 📁 Jobs/
│   │   ├── ProcessLocalApiEvent.php
│   │   ├── PublishEventToLocal.php
│   │   └── SyncMetrics.php
│   │
│   └── ... (mesmo padrão da Local API)
│
├── 📁 database/
│   ├── 📁 migrations/
│   │   ├── create_agencies_table.php
│   │   ├── create_users_table.php
│   │   ├── create_tickets_table.php
│   │   ├── create_ticket_events_table.php
│   │   ├── create_devices_table.php
│   │   ├── create_alerts_table.php
│   │   └── ... (mais tabelas)
│   └── 📁 seeders/
│       ├── AgencySeeder.php
│       ├── UserSeeder.php
│       └── ...
│
├── 📁 docker/
│   ├── Dockerfile
│   └── ... (configs Docker)
│
└── ... (mesma estrutura do resto)
```

### O Que Cada Pasta Faz

| Pasta | Propósito |
|-------|-----------|
| `app/Http/Controllers/` | Recebem requisições HTTP e delegam para Services |
| `app/Services/` | Lógica de negócio principal |
| `app/Models/` | Representam tabelas do banco de dados |
| `app/Domain/Events/` | Eventos de domínio (apenas Local API) |
| `app/Infrastructure/` | Implementações técnicas (RabbitMQ, Projectors, etc) |
| `database/migrations/` | Versionamento do schema (estrutura) do banco |
| `database/seeders/` | Dados iniciais para testes/desenvolvimento |
| `config/` | Variáveis de configuração (usam valores do `.env`) |
| `routes/` | Definição de endpoints da API |
| `storage/` | Arquivos gerados em tempo de execução (logs, cache) |
| `tests/` | Testes automatizados (unitários e de feature) |
| `vendor/` | Pacotes PHP (não editar, auto-gerado pelo Composer) |
| `node_modules/` | Pacotes Node (não editar, auto-gerado pelo npm) |

---

## 🔌 Endpoints Principais

### LOCAL API

#### Autenticação (Público)

```http
POST /auth/login
Content-Type: application/json

{
  "email": "operador@agencia.com.br",
  "password": "senha123"
}

Response 200:
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

```http
POST /auth/refresh
Authorization: Bearer {token}

Response 200:
{
  "access_token": "novo_token..."
}
```

#### Senhas (Protegido - Requer JWT)

```http
GET /tickets
Authorization: Bearer {token}

Response 200:
[
  {
    "id": "uuid-123",
    "number": 42,
    "service_id": "uuid-service",
    "status": "waiting", // waiting, called, finished, absent
    "created_at": "2025-12-18T16:27:00Z",
    "called_at": null
  },
  ...
]
```

```http
POST /tickets
Authorization: Bearer {token}
Content-Type: application/json

{
  "service_id": "uuid-service",
  "priority": "normal" // normal, priority
}

Response 201:
{
  "id": "uuid-123",
  "number": 43,
  "service_id": "uuid-service",
  "status": "waiting",
  "created_at": "2025-12-18T16:30:00Z"
}
```

```http
POST /tickets/call-next
Authorization: Bearer {token}
Content-Type: application/json

{
  "desk_id": "uuid-desk"
}

Response 200:
{
  "ticket_id": "uuid-123",
  "number": 42,
  "called_at": "2025-12-18T16:31:00Z"
}
```

```http
PATCH /tickets/{id}/finish
Authorization: Bearer {token}
Content-Type: application/json

{
  "duration": 180 // segundos
}

Response 200:
{
  "id": "uuid-123",
  "status": "finished",
  "finished_at": "2025-12-18T16:35:00Z"
}
```

#### Informações (Público)

```http
GET /health

Response 200:
{
  "status": "ok",
  "version": "1.0.0"
}
```

```http
GET /templates

Response 200:
[
  {
    "id": "uuid-1",
    "name": "template-default",
    "html": "<html>...</html>"
  },
  ...
]
```

### CLOUD API

#### Autenticação (Público)

```http
POST /v1/auth/login
Content-Type: application/json

{
  "email": "admin@example.com",
  "password": "senha123"
}

Response 200:
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "type": "Bearer"
}
```

#### Agências (Protegido)

```http
GET /v1/agencies
Authorization: Bearer {token}

Response 200:
[
  {
    "id": "uuid-1",
    "name": "Agência Centro",
    "city": "São Paulo",
    "desks_count": 5,
    "devices_count": 3,
    "users_count": 10,
    "created_at": "2025-01-01T00:00:00Z"
  },
  ...
]
```

#### Dashboards (Protegido)

```http
GET /v1/dashboard/overview?agency_id={agency_id}
Authorization: Bearer {token}

Response 200:
{
  "tickets_today": 450,
  "average_wait_time": 180, // segundos
  "average_service_time": 300,
  "desks": [
    {
      "id": "uuid-desk-1",
      "name": "Balcão 1",
      "current_ticket": 42,
      "tickets_served": 25,
      "average_time": 320
    }
  ],
  "services": [
    {
      "id": "uuid-service-1",
      "name": "Abertura Conta",
      "requests": 150,
      "average_time": 450
    }
  ]
}
```

#### Relatórios (Protegido)

```http
GET /v1/reports/daily?agency_id={agency_id}&date=2025-12-18
Authorization: Bearer {token}

Response 200:
{
  "date": "2025-12-18",
  "agency_id": "uuid-1",
  "summary": {
    "total_tickets": 450,
    "priority_tickets": 50,
    "abandoned_tickets": 15,
    "total_service_time": 135000, // minutos
    "peak_hour": "10:00-11:00"
  },
  "by_service": [
    {
      "service_id": "uuid-service-1",
      "service_name": "Abertura Conta",
      "count": 150,
      "average_time": 450
    }
  ]
}
```

---

## 🔄 Comunicação entre APIs

### Fluxo Completo: Criar Senha na Local e Sincronizar com Cloud

#### 1️⃣ **Passo 1: Cliente Cria Senha na Local API**

```
Cliente: POST /tickets
Local API: Cria Domain Event "TicketCreated"
    ↓
Event Store: Persiste evento
    ↓
Projector: Atualiza Read Model
    ↓
Outbox: Salva para publicar
```

#### 2️⃣ **Passo 2: Local API Publica em RabbitMQ**

```
Background Job: Lê tabela outbox
    ↓
Publifica mensagem:
    Exchange: ticket_dispenser_events
    Routing Key: local.ticket.created
    Payload: {evento json completo}
    ↓
Marca como publicado na outbox
```

#### 3️⃣ **Passo 3: Cloud API Consome Mensagem**

```
Cloud Consumer: Escuta fila "cloud_events_processor"
    ↓
Recebe evento "local.ticket.created"
    ↓
TicketEventHandler: Processa evento
    ├─ Cria/atualiza Ticket no MySQL
    ├─ Cria TicketEvent para auditoria
    └─ Atualiza métricas (cache Redis)
    ↓
Event processado com sucesso
```

#### 4️⃣ **Passo 4: Dashboard Atualiza em Tempo Real**

```
Cloud API: Atualiza cache Redis
    ↓
Dashboard WebSocket: Publica para clientes conectados
    ↓
Browser WebSocket Client: Recebe atualização
    ↓
UI: Atualiza em tempo real (sem refresh)
```

### Fluxo Reverso: Cloud Envia Configuração para Local

#### 1️⃣ **Admin Cria Serviço na Cloud**

```
Admin Cloud: POST /v1/services
Cloud API: Cria serviço no MySQL
```

#### 2️⃣ **Cloud Publica para Agências**

```
Cloud API: Publica evento
    Exchange: cloud.sync
    Routing Key: cloud.sync.{AGENCY_ID}.service.created
    Payload: {dados do novo serviço}
```

#### 3️⃣ **Local API Consome (Se AGENCY_ID Bate)**

```
Local Consumer: Escuta cloud.sync.{SEU_AGENCY_ID}.*.*
    ↓
Recebe evento porque AGENCY_ID combina
    ↓
ServiceHandler: Processa
    ├─ Cria serviço no Event Store
    ├─ Projector atualiza Read Model
    └─ Gera TicketEventCreated (receipt)
    ↓
Publica receipt para Cloud indicando sucesso
```

#### 4️⃣ **Local API Continua Funcionando**

```
Mesmo que Cloud API caia, Local continua:
✓ Dispensando senhas
✓ Atendendo clientes
✓ Armazenando dados localmente
✗ Não sincroniza até Cloud voltar
```

---

## 🚨 Troubleshooting

### Problema: API Não Inicia

**Sintoma:** `php artisan serve` retorna erro

**Solução:**

```bash
# 1. Verificar variáveis de ambiente
cat .env | grep APP_

# 2. Gerar chaves necessárias
php artisan key:generate
php artisan jwt:secret  # Local API apenas

# 3. Criar bases de dados (SQLite/MySQL)
php artisan migrate

# 4. Limpar cache
php artisan cache:clear
php artisan config:clear

# 5. Verificar permissões de pastas
chmod -R 775 storage bootstrap/cache
```

### Problema: WebSocket Não Conecta

**Sintoma:** Display mostra mensagem de erro de conexão

**Solução:**

```bash
# 1. Verificar se WebSocket está rodando
curl http://localhost:8081/health

# 2. Verificar porta no firewall
# Windows: netstat -an | findstr 8081
# Linux: sudo lsof -i :8081

# 3. Configurar no .env
WEBSOCKET_HOST=0.0.0.0
WEBSOCKET_PORT=8081
WEBSOCKET_SECRET=chave-secreta

# 4. Reiniciar
docker-compose restart  # ou
php artisan tinker
\App\Services\WebSocketService::restart()
```

### Problema: RabbitMQ Não Conecta

**Sintoma:** Erros ao processar fila, eventos não sincronizam

**Solução:**

```bash
# 1. Verificar se RabbitMQ está rodando
curl http://rabbitmq:15672/api/overview

# 2. Verificar credenciais no .env
RABBITMQ_HOST=rabbitmq
RABBITMQ_PORT=5672
RABBITMQ_USER=admin
RABBITMQ_PASSWORD=admin123
RABBITMQ_VHOST=/

# 3. Testar conexão
php artisan tinker
\App\Infrastructure\RabbitMQ\Publisher::test()

# 4. Verificar exchanges e queues na UI
# http://localhost:15672 (admin/admin123)
```

### Problema: Senhas Não Aparecem no Dashboard

**Sintoma:** Local API cria senhas, mas Cloud não mostra

**Solução:**

```bash
# 1. Verificar se eventos estão na outbox (Local API)
SELECT * FROM outbox WHERE published_at IS NULL;

# 2. Verificar se RabbitMQ está processando
# Ver queue "cloud_events_processor" na UI

# 3. Verificar logs do Cloud Consumer
tail -f storage/logs/laravel.log | grep -i rabbitmq

# 4. Reprocessar manualmente
# Local API:
php artisan command:process-outbox

# Cloud API:
php artisan queue:work rabbitmq --queue=cloud_events_processor
```

### Problema: Erros de Autenticação JWT (Local API)

**Sintoma:** `401 Unauthorized` nas requisições

**Solução:**

```bash
# 1. Verificar se JWT_SECRET está gerado
grep JWT_SECRET .env

# 2. Se vazio, gerar
php artisan jwt:secret

# 3. Verificar formato do token
# Authorization: Bearer {token}

# 4. Verificar expiração do token
# JWT_TTL padrão é 1440 minutos (24 horas)

# 5. Verificar se middleware está habilitado
# routes/api.php deve ter 'auth:api' no grupo
```

### Problema: SQLite Corrompido (Local API)

**Sintoma:** Erros de banco de dados, travamentos

**Solução:**

```bash
# 1. Verificar integridade
php artisan tinker
SQLite3::queryIntegrity();

# 2. Se corrompido, restaurar backup
cp database/readmodel.sqlite.backup database/readmodel.sqlite

# 3. Se sem backup, recriar
rm database/readmodel.sqlite database/eventstore.sqlite
php artisan migrate

# 4. Reprocessar eventos
php artisan command:rebuild-projections
```

### Problema: Disco Cheio (Event Store Crescendo)

**Sintoma:** Event Store cresceu demais (GB)

**Solução:**

```bash
# 1. Arquivar eventos antigos
php artisan command:archive-events --before=2025-01-01

# 2. Verificar tamanho
du -h database/eventstore.sqlite

# 3. Otimizar banco
php artisan command:optimize-eventstore

# 4. Se necessário, limpar tudo (CUIDADO!)
# Apenas se for recomeçar:
rm database/eventstore.sqlite
php artisan migrate
```

---

## 📞 Suporte e Próximos Passos

### Documentos Complementares

Recomenda-se ler também:
- `ARCHITECTURE.md` - Detalhes técnicos profundos
- `SETUP.md` - Guia de instalação em producção
- `MULTI_AGENCY_SETUP.md` - Como configurar múltiplas agências
- `WEBSOCKET_FRONTEND_GUIDE.md` - Como integrar WebSocket no frontend
- `ZIP_UPDATES_FRONTEND_INTEGRATION.md` - Como integrar atualizações web

### Monitoramento em Produção

Para ambiente de produção, recomenda-se:

1. **Logs Centralizados:** ELK Stack, Datadog, ou similar
2. **Alertas:** Configure alertas para fila RabbitMQ cheia
3. **Backups:** Automatize backups do MySQL
4. **Monitoramento:** Prometheus + Grafana para métricas
5. **Health Checks:** Cron jobs para verificar saúde da API

### Melhorias Futuras

- [ ] Implementar autenticação OAuth2
- [ ] Adicionar suporte a múltiplos idiomas
- [ ] Integração com impressoras de senhas
- [ ] App mobile para clientes (fila em tempo real)
- [ ] Machine Learning para predição de filas
- [ ] Integração com sistemas de CRM

---

## 📝 Notas Finais

- ✅ **Local API** é o coração operacional - sempre funciona mesmo offline
- ✅ **Cloud API** é o cérebro - consolida dados e fornece insights
- ✅ **RabbitMQ** é o nervoso central - conecta tudo de forma confiável
- ✅ **SQLite** é rápido e eficiente para Local API
- ✅ **MySQL** é robusto para Cloud API centralizado

**Documento gerado:** 30 de Maio de 2026  
**Versão do Sistema:** 1.0.0

---

**Fim da Documentação**
