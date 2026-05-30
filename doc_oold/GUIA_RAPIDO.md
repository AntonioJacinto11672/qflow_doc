# 🚀 GUIA RÁPIDO - TICKET DISPENSER

## ⚡ Iniciar em 5 Minutos (Com Docker)

### Local API
```bash
cd dispenser-local-api-main
docker-compose up -d
# Acesso: http://localhost:8080
```

### Cloud API
```bash
cd dispenser-remote-api-main
docker-compose up -d
# Acesso: http://localhost:8084
# RabbitMQ UI: http://localhost:15672 (admin/admin123)
```

---

## 📋 Checklist de Configuração

### 1. Clonar/Preparar Projeto
- [ ] Pasta `dispenser-local-api-main` existe
- [ ] Pasta `dispenser-remote-api-main` existe
- [ ] Ambas têm arquivo `.env`

### 2. Variáveis Críticas do `.env`

#### Local API (OBRIGATÓRIO)
```env
AGENCY_ID=seu-uuid-aqui              # UUID da agência
APP_ENV=local
JWT_SECRET=gerado-pelo-artisan
DB_DATABASE=database/readmodel.sqlite
EVENTSTORE_DB_DATABASE=database/eventstore.sqlite
```

#### Cloud API
```env
APP_ENV=local
DB_HOST=mysql
DB_DATABASE=ticket_dispenser
DB_USERNAME=dispenser_user
RABBITMQ_HOST=rabbitmq
```

### 3. Instalar Dependências
```bash
# Local API
cd dispenser-local-api-main
composer install
npm install
npm run dev

# Cloud API
cd dispenser-remote-api-main
composer install
npm install
npm run dev
```

### 4. Gerar Chaves
```bash
# Local API
php artisan key:generate
php artisan jwt:secret

# Cloud API
php artisan key:generate
```

### 5. Criar Bancos
```bash
# Ambas as APIs
php artisan migrate
# Opcional: php artisan db:seed
```

### 6. Iniciar
```bash
# Com Docker
docker-compose up -d

# Ou sem Docker
php artisan serve --port=8080  # Local
php artisan serve --port=8084  # Cloud (em outro terminal)
```

---

## 🔗 Endpoints Essenciais

### Local API (JWT)
| Método | URL | O Que Faz |
|--------|-----|-----------|
| POST | `/auth/login` | Faz login, retorna token |
| GET | `/tickets` | Lista senhas |
| POST | `/tickets` | Cria nova senha |
| POST | `/tickets/call-next` | Chama próxima senha |
| PATCH | `/tickets/{id}/finish` | Finaliza atendimento |
| GET | `/health` | Verifica se está online |

### Cloud API (Sanctum)
| Método | URL | O Que Faz |
|--------|-----|-----------|
| POST | `/v1/auth/login` | Faz login |
| GET | `/v1/agencies` | Lista agências |
| GET | `/v1/dashboard/overview` | Dashboard |
| GET | `/v1/reports/daily` | Relatórios do dia |

---

## 🔐 Autenticação

### Local API (JWT)

```bash
# 1. Fazer login
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "password"}'

# Resposta:
# {"access_token": "eyJ0eXA...", "token_type": "Bearer", "expires_in": 3600}

# 2. Usar em requisições
curl -X GET http://localhost:8080/tickets \
  -H "Authorization: Bearer eyJ0eXA..."
```

### Cloud API (Sanctum)

```bash
# Similar ao Local API
curl -X POST http://localhost:8084/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@example.com", "password": "password"}'
```

---

## 📡 RabbitMQ - O "Mensageiro"

### O que faz?
- Local API → Cloud API: "Nova senha criada!"
- Cloud API → Local API: "Novo serviço disponível"

### Acessar Management UI
```
http://localhost:15672
Usuário: admin
Senha: admin123
```

### Verificar Fila de Eventos
1. Acesse http://localhost:15672
2. Vá em "Queues"
3. Veja `cloud_events_processor` (eventos da Local para Cloud)
4. Veja `local_sync.*` (configurações da Cloud para Local)

---

## 📁 Onde Está Cada Coisa?

```
Códigos dos controllers    → app/Http/Controllers/
Lógica de negócio         → app/Services/
Dados (modelos)           → app/Models/
Eventos de domínio        → app/Domain/Events/ (Local API)
Sincronização RabbitMQ    → app/Infrastructure/RabbitMQ/
Rotas da API              → routes/api.php
Configurações             → .env
Banco de dados (estrutura)→ database/migrations/
```

---

## 🔧 Comandos Úteis

### Logs em Tempo Real
```bash
# Local API
cd dispenser-local-api-main
tail -f storage/logs/laravel.log

# Cloud API
cd dispenser-remote-api-main
tail -f storage/logs/laravel.log
```

### Reiniciar Tudo
```bash
# Com Docker
docker-compose down
docker-compose up -d

# Sem Docker
php artisan cache:clear
php artisan config:clear
php artisan serve
```

### Criar Usuário de Teste
```bash
php artisan tinker

# Local API
>>> \App\Models\User::create([
  'name' => 'Operador Teste',
  'email' => 'operador@test.com',
  'password' => bcrypt('123456'),
  'agency_id' => 'seu-agency-id'
])

# Cloud API
>>> \App\Models\User::create([
  'name' => 'Admin Teste',
  'email' => 'admin@test.com',
  'password' => bcrypt('123456')
])
```

### Processar Fila de Eventos
```bash
# Local API - publicar eventos para RabbitMQ
php artisan command:process-outbox

# Cloud API - consumir eventos do RabbitMQ
php artisan queue:work rabbitmq --queue=cloud_events_processor
```

### Limpar Bancos de Dados
```bash
# ⚠️ CUIDADO - Isso deleta TUDO!
php artisan migrate:refresh
# Ou
php artisan migrate:reset
php artisan migrate
```

---

## 🚨 Erros Comuns e Soluções

| Erro | Solução |
|------|---------|
| `SQLSTATE[HY000]: General error` | Banco SQLite corrompido: `rm database/*.sqlite && php artisan migrate` |
| `JWT_SECRET not set` | `php artisan jwt:secret` |
| `Connection refused` (RabbitMQ) | Verificar se RabbitMQ está rodando: `docker ps \| grep rabbitmq` |
| `401 Unauthorized` | Token expirou: fazer novo login |
| `port 8080 in use` | Porta ocupada: `php artisan serve --port=8081` |
| `MySQL connection error` | Verificar credentials no `.env` |

---

## 📊 Estrutura de Dados

### Ticket (Senha)
```json
{
  "id": "uuid",
  "number": 42,
  "service_id": "uuid",
  "status": "waiting|called|finished|absent",
  "priority": "normal|priority",
  "created_at": "2025-12-18T16:27:00Z",
  "called_at": null,
  "finished_at": null
}
```

### User (Operador)
```json
{
  "id": "uuid",
  "name": "João Silva",
  "email": "joao@agencia.com.br",
  "role": "operator|admin|manager",
  "agency_id": "uuid",  // Local API
  "desk_id": null
}
```

### Desk (Balcão)
```json
{
  "id": "uuid",
  "number": 1,
  "name": "Balcão 1",
  "agency_id": "uuid",
  "current_ticket_id": null,
  "status": "available|busy|offline"
}
```

### Service (Serviço)
```json
{
  "id": "uuid",
  "name": "Abertura de Conta",
  "description": "Serviço de abertura de nova conta",
  "estimated_time": 300,  // segundos
  "agency_id": "uuid"
}
```

---

## 🔄 Fluxo de Dados Simplificado

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENTE                              │
│                 Pega senha (kiosk)                      │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
          ┌──────────────────────┐
          │   Local API - Cria   │
          │  TicketCreated Event │
          └──────────┬───────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
   ┌─────────────┐      ┌──────────────────┐
   │ SQLite      │      │   RabbitMQ       │
   │ Event Store │      │ (outbox pattern) │
   └─────────────┘      └────────┬─────────┘
        │                        │
        ▼                        ▼
   ┌─────────────┐      ┌──────────────────┐
   │Projector    │      │  Cloud API       │
   │Atualiza     │      │  Consumer        │
   │Read Model   │      │  Processa        │
   └─────────────┘      └────────┬─────────┘
        │                        │
        ▼                        ▼
   ┌─────────────┐      ┌──────────────────┐
   │ Display     │      │ MySQL            │
   │ Mostra      │      │ Atualiza         │
   │ Senha       │      │ Dashboard        │
   └─────────────┘      └──────────────────┘
```

---

## 🛠️ Stack de Tecnologias

| Componente | Tecnologia | Versão |
|------------|-----------|--------|
| Linguagem | PHP | 8.3+ |
| Framework | Laravel | 12.0 |
| Web Frontend | Vite + Tailwind | 4.0 |
| API Local | JWT | 2.2 |
| API Cloud | Sanctum | 4.2 |
| Banco Local | SQLite | - |
| Banco Cloud | MySQL | 8.0 |
| Message Broker | RabbitMQ | 3.13 |
| Cache | Redis | (opcional) |
| Docker | Docker Compose | - |

---

## 📞 Referências Rápidas

- 📄 Documentação Completa: `DOCUMENTACAO_COMPLETA.md`
- 📄 Arquitetura Detalhada: `ARCHITECTURE.md`
- 📄 Setup Produção: `SETUP.md`
- 📄 WebSocket Guide: `docs/WEBSOCKET_FRONTEND_GUIDE.md`
- 📄 Multi-Agency: `MULTI_AGENCY_SETUP.md`

---

**Última atualização:** 30 de Maio de 2026
