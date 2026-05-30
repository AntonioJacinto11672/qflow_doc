# 🎨 EXEMPLOS PRÁTICOS E FLUXOS VISUAIS

## 🌐 Exemplos de Requisições HTTP

### Cenário: Sistema Funcional Completo

#### 1️⃣ Login na Local API

```bash
# Request
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "operador@agencia.com.br",
    "password": "senha123"
  }'

# Response (200 OK)
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODAiLCJpYXQiOjE3Njg2NTAwNDAsImV4cCI6MTc2ODczNjQ0MCwibmJmIjoxNzY4NjUwMDQwLCJqdGkiOiJ2bXM4TTdJRzhqUGo0N3ZKIiwic3ViIjoiMSIsInBydiI6IjIzYmQ1YzRhNDlhMDZhMDEwYzk2ZjZiNGNhODczZTAwMDAwMDAwMDAwIiwibmFtZSI6Ik9wZXJhZG9yIn0.abc123...",
  "token_type": "Bearer",
  "expires_in": 86400
}

# Guardar o token para as próximas requisições
TOKEN="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
```

#### 2️⃣ Criar uma Nova Senha

```bash
# Request
curl -X POST http://localhost:8080/tickets \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "service_id": "550e8400-e29b-41d4-a716-446655440000",
    "priority": "normal"
  }'

# Response (201 Created)
{
  "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "number": 42,
  "service_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "waiting",
  "priority": "normal",
  "created_at": "2025-12-18T16:27:00.000Z",
  "called_at": null,
  "finished_at": null
}

# Nova senha: NÚMERO 42 - Esperando ser chamada
```

#### 3️⃣ Listar Todas as Senhas (para painel operador)

```bash
# Request
curl -X GET "http://localhost:8080/tickets?status=waiting&limit=10" \
  -H "Authorization: Bearer $TOKEN"

# Response (200 OK)
{
  "data": [
    {
      "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "number": 42,
      "service_id": "550e8400-e29b-41d4-a716-446655440000",
      "service_name": "Abertura de Conta",
      "status": "waiting",
      "priority": "normal",
      "created_at": "2025-12-18T16:27:00.000Z",
      "wait_time_seconds": 120
    },
    {
      "id": "a3b5d7f1-89c4-4e2b-8f1a-5c6d7e8f9g0h",
      "number": 43,
      "service_id": "660e8400-e29b-41d4-a716-446655440111",
      "service_name": "Saque",
      "status": "waiting",
      "priority": "normal",
      "created_at": "2025-12-18T16:28:00.000Z",
      "wait_time_seconds": 60
    }
  ],
  "total": 2,
  "per_page": 10,
  "current_page": 1
}
```

#### 4️⃣ Chamar Próxima Senha (operador clica botão)

```bash
# Request
curl -X POST http://localhost:8080/tickets/call-next \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "desk_id": "770e8400-e29b-41d4-a716-446655440222"
  }'

# Response (200 OK)
{
  "ticket_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "ticket_number": 42,
  "service_name": "Abertura de Conta",
  "desk_number": "Balcão 1",
  "status": "called",
  "called_at": "2025-12-18T16:29:00.000Z"
}

# WebSocket publica para displays: "SENHA 42 - BALCÃO 1"
```

#### 5️⃣ Finalizar Atendimento (operador terminou)

```bash
# Request
curl -X PATCH http://localhost:8080/tickets/f47ac10b-58cc-4372-a567-0e02b2c3d479/finish \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "duration": 480  # 8 minutos em segundos
  }'

# Response (200 OK)
{
  "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "ticket_number": 42,
  "status": "finished",
  "started_at": "2025-12-18T16:29:00.000Z",
  "finished_at": "2025-12-18T16:36:00.000Z",
  "duration_seconds": 420,
  "service_name": "Abertura de Conta",
  "desk_number": "Balcão 1"
}

# Evento é publicado em RabbitMQ para Cloud API atualizar relatórios
```

#### 6️⃣ Acessar Dashboard Cloud (gerente vê consolidado)

```bash
# Primeiro, login na Cloud API
curl -X POST http://localhost:8084/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "gerente@example.com",
    "password": "senha123"
  }'

# Response
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "type": "Bearer"
}

CLOUD_TOKEN="eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."

# Agora acessar dashboard
curl -X GET "http://localhost:8084/v1/dashboard/overview?agency_id=550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer $CLOUD_TOKEN"

# Response (200 OK)
{
  "agency_id": "550e8400-e29b-41d4-a716-446655440000",
  "agency_name": "Agência Centro",
  "date": "2025-12-18",
  "summary": {
    "total_tickets": 450,
    "priority_tickets": 35,
    "abandoned_tickets": 8,
    "average_wait_time_seconds": 180,
    "average_service_time_seconds": 420,
    "tickets_per_hour": 56.25,
    "peak_hour": "10:00-11:00"
  },
  "desks": [
    {
      "id": "770e8400-e29b-41d4-a716-446655440222",
      "number": 1,
      "name": "Balcão 1",
      "status": "available",
      "tickets_served": 45,
      "average_service_time": 420,
      "current_ticket": null
    },
    {
      "id": "880e8400-e29b-41d4-a716-446655440333",
      "number": 2,
      "name": "Balcão 2",
      "status": "busy",
      "tickets_served": 50,
      "average_service_time": 450,
      "current_ticket": 43
    },
    {
      "id": "990e8400-e29b-41d4-a716-446655440444",
      "number": 3,
      "name": "Balcão 3",
      "status": "offline",
      "tickets_served": 0,
      "average_service_time": 0,
      "current_ticket": null
    }
  ],
  "services": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Abertura de Conta",
      "requests_today": 150,
      "average_service_time": 450,
      "average_wait_time": 200
    },
    {
      "id": "660e8400-e29b-41d4-a716-446655440111",
      "name": "Saque",
      "requests_today": 200,
      "average_service_time": 360,
      "average_wait_time": 150
    }
  ]
}
```

#### 7️⃣ Gerar Relatório do Dia

```bash
# Request
curl -X GET "http://localhost:8084/v1/reports/daily?agency_id=550e8400-e29b-41d4-a716-446655440000&date=2025-12-18" \
  -H "Authorization: Bearer $CLOUD_TOKEN"

# Response (200 OK)
{
  "date": "2025-12-18",
  "agency_id": "550e8400-e29b-41d4-a716-446655440000",
  "agency_name": "Agência Centro",
  "summary": {
    "total_tickets": 450,
    "priority_tickets": 35,
    "normal_tickets": 415,
    "abandoned_tickets": 8,
    "completed_tickets": 442,
    "total_wait_time_minutes": 1350,  # Total de espera de todos
    "total_service_time_minutes": 3150,  # Total de atendimento
    "average_wait_time_seconds": 180,
    "average_service_time_seconds": 420,
    "peak_hour": {
      "hour": 10,
      "tickets": 67
    }
  },
  "by_service": [
    {
      "service_id": "550e8400-e29b-41d4-a716-446655440000",
      "service_name": "Abertura de Conta",
      "total_requests": 150,
      "completed": 145,
      "abandoned": 5,
      "average_service_time": 450,
      "average_wait_time": 200,
      "total_time_minutes": 1125
    },
    {
      "service_id": "660e8400-e29b-41d4-a716-446655440111",
      "service_name": "Saque",
      "total_requests": 200,
      "completed": 200,
      "abandoned": 0,
      "average_service_time": 360,
      "average_wait_time": 150,
      "total_time_minutes": 1700
    },
    {
      "service_id": "770e8400-e29b-41d4-a716-446655440222",
      "service_name": "Consulta",
      "total_requests": 100,
      "completed": 97,
      "abandoned": 3,
      "average_service_time": 240,
      "average_wait_time": 120,
      "total_time_minutes": 600
    }
  ],
  "by_desk": [
    {
      "desk_id": "770e8400-e29b-41d4-a716-446655440222",
      "desk_number": 1,
      "desk_name": "Balcão 1",
      "tickets_served": 45,
      "average_service_time": 420,
      "total_time_minutes": 315,
      "utilization_percent": 85
    },
    {
      "desk_id": "880e8400-e29b-41d4-a716-446655440333",
      "desk_number": 2,
      "desk_name": "Balcão 2",
      "tickets_served": 50,
      "average_service_time": 450,
      "total_time_minutes": 375,
      "utilization_percent": 90
    }
  ],
  "hourly_breakdown": [
    {
      "hour": 9,
      "tickets": 45,
      "average_wait_time": 150,
      "average_service_time": 360
    },
    {
      "hour": 10,
      "tickets": 67,
      "average_wait_time": 240,
      "average_service_time": 480
    },
    {
      "hour": 11,
      "tickets": 60,
      "average_wait_time": 200,
      "average_service_time": 420
    }
  ]
}
```

---

## 🎨 Fluxos Visuais Completos

### Fluxo 1: Criação de Senha (Local API Offline)

```
┌─────────────────────────────────────────────────────────────────────┐
│                      CLIENTE NA AGÊNCIA                             │
│                                                                     │
│  1. Cliente chega na agência                                       │
│     ↓                                                               │
│  2. Pressiona botão: "SERVIÇO: Abertura Conta"                    │
│     ↓                                                               │
│  3. Kiosk (Windows App ou Web) envia POST /tickets                │
└────────────────────────┬────────────────────────────────────────────┘
                         │
        ┌────────────────▼────────────────┐
        │     LOCAL API (8080)             │
        │                                  │
        │  TicketController::store()      │
        │  ├─ Valida dados                │
        │  ├─ Service cria Ticket         │
        │  ├─ Gera TicketCreated Event   │
        │  └─ Retorna novo ticket        │
        └────────┬─────────────────────────┘
                 │
    ┌────────────┴────────────┐
    │                         │
    ▼                         ▼
┌──────────────┐     ┌──────────────────────┐
│  Event Store │     │  Projector           │
│  (SQLite)    │     │  ├─ Lê evento       │
│              │     │  ├─ Gera SQL        │
│ Persiste     │     │  └─ Atualiza        │
│ evento       │     └─────────┬────────────┘
│              │               │
└──────────────┘               ▼
                        ┌──────────────────┐
                        │  Read Model      │
                        │  (SQLite)        │
                        │                  │
                        │  Ticket #42      │
                        │  Status: waiting │
                        │  Criado: 16:27   │
                        └────────┬─────────┘
                                 │
                    ┌────────────┼────────────┐
                    │            │            │
                    ▼            ▼            ▼
            ┌────────────┐ ┌─────────┐ ┌──────────┐
            │ WebSocket  │ │ Outbox  │ │ Response │
            │ Broadcast  │ │ Table   │ │ JSON     │
            │            │ │ (RabbitMQ)          │
            │ Display:   │ │         │ │ {"id":   │
            │ "42"       │ │ Pub     │ │  ...}    │
            └────────────┘ │ Later   │ └──────────┘
                           └─────────┘
                                │
                                ▼
                        ┌──────────────────────┐
                        │    RabbitMQ Queue    │
                        │  Quando conectar     │
                        │  cloud_events_       │
                        │  processor           │
                        │                      │
                        │ {evento json}        │
                        └──────────────────────┘
```

### Fluxo 2: Sincronização com Cloud API

```
┌──────────────────────────────────────────────────────────┐
│             LOCAL API                                    │
│   Evento TicketCreated está em fila (outbox)            │
└────────────────────┬─────────────────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │ Background Job:         │
        │ process-outbox          │
        │ (rodando a cada 1 min)  │
        └────────┬────────────────┘
                 │
        ┌────────▼────────────────┐
        │ Lê tabela outbox        │
        │ Eventos não publicados  │
        └────────┬────────────────┘
                 │
        ┌────────▼────────────────┐
        │ Publica em RabbitMQ:    │
        │                         │
        │ Exchange:               │
        │ ticket_dispenser_events │
        │                         │
        │ Routing Key:            │
        │ local.ticket.created    │
        │                         │
        │ Message Body:           │
        │ {evento completo json}  │
        └────────┬────────────────┘
                 │
        ┌────────▼────────────────────────┐
        │ Marca na outbox como publicado   │
        │ Continua processando próximos   │
        └────────┬────────────────────────┘
                 │
     ┌───────────┴────────────────┐
     │                            │
     │  SE FALHAR:                │
     │  ↓                         │
     │ Deixa na outbox            │
     │ Tenta novamente depois     │
     │ NÃO perde evento           │
     │                            │
     │  SE SUCESSO:               │
     │  ↓                         │
     │ Continua normalmente       │
     └───────────┬────────────────┘
                 │
                 ▼
     ┌─────────────────────────────┐
     │      RabbitMQ               │
     │                             │
     │ Exchange:                   │
     │ ticket_dispenser_events     │
     │                             │
     │ Queue:                      │
     │ cloud_events_processor      │
     │                             │
     │ Mensagem esperando ser      │
     │ consumida...                │
     └──────────┬──────────────────┘
                │
                ▼
     ┌────────────────────────────────┐
     │      CLOUD API (8084)          │
     │                                │
     │  Consumer:                     │
     │  CloudEventsProcessor          │
     │  (rodando continuamente)       │
     │                                │
     │  1. Escuta fila               │
     │  2. Recebe mensagem           │
     │  3. TicketEventHandler processa│
     │  4. Cria Ticket em MySQL      │
     │  5. Atualiza Cache Redis      │
     │  6. Marca como consumido      │
     │  7. Volta a escutar           │
     │                                │
     └──────────┬────────────────────┘
                │
                ▼
     ┌────────────────────────────────┐
     │  Dashboard WebSocket           │
     │                                │
     │  Gerente conectado em:         │
     │  http://localhost:8084/        │
     │  dashboard/agencia/123         │
     │                                │
     │  WebSocket recebe:             │
     │  "Nova senha criada"           │
     │                                │
     │  JavaScript atualiza DOM       │
     │  em tempo real (sem refresh)   │
     └────────────────────────────────┘
```

### Fluxo 3: Chamada de Senha

```
OPERADOR CLICA EM "CHAMAR PRÓXIMA"
            │
            ▼
  ┌──────────────────────────────┐
  │  POST /tickets/call-next     │
  │  {"desk_id": "xyz"}          │
  └──────────┬───────────────────┘
             │
             ▼
  ┌──────────────────────────────────┐
  │  LOCAL API TicketController       │
  │                                  │
  │  1. Encontra próxima senha       │
  │     Status = "waiting"           │
  │     Prioridade = higher first    │
  │                                  │
  │  2. Cria evento TicketCalled     │
  │     {                            │
  │       ticket_id: "42",           │
  │       desk_id: "xyz",            │
  │       called_at: "16:29:00",     │
  │       status: "called"           │
  │     }                            │
  └──────────┬───────────────────────┘
             │
        ┌────┴────┐
        │          │
        ▼          ▼
  ┌─────────┐  ┌────────────────────┐
  │ Event   │  │ Projector          │
  │ Store   │  │ ├─ Atualiza ticket │
  │ (SQLite)│  │ │ Status = called  │
  │         │  │ └─ Em tempo real   │
  │ Salvo   │  └────────┬───────────┘
  │         │           │
  └─────────┘           ▼
                   ┌──────────────┐
                   │ Read Model   │
                   │ (SELECT      │
                   │ rápido)      │
                   └──────┬───────┘
                          │
             ┌────────────┼──────────────┐
             │            │              │
             ▼            ▼              ▼
      ┌────────────┐ ┌──────────┐ ┌───────────┐
      │ WebSocket  │ │ Outbox   │ │ Resposta  │
      │ BROADCAST  │ │ (RabbitMQ)           │
      │            │ │          │ │ {"ticket" │
      │ Para 2     │ │ Pub para │ │  _number" │
      │ clientes:  │ │ cloud    │ │  : 42}    │
      │            │ │          │ │           │
      │ 1. Display │ └──────────┘ └───────────┘
      │    cliente │
      │ 2. Painel  │
      │    operador│
      │            │
      │ Ambos:     │
      │ "SENHA 42" │
      │ "BALCÃO 1" │
      └────────────┘
```

---

## 🔌 WebSocket em Tempo Real

### Cliente JavaScript Conectado a WebSocket

```javascript
// No frontend (HTML/Vue/React)

const ws = new WebSocket('ws://localhost:8081');

ws.addEventListener('open', function() {
  console.log('Conectado ao WebSocket');
  
  // Subscribe ao canal de eventos
  ws.send(JSON.stringify({
    action: 'subscribe',
    channel: 'tickets',
    agency_id: 'abad8412-b7de-41f4-8302-464fe1751a41'
  }));
});

ws.addEventListener('message', function(event) {
  const data = JSON.parse(event.data);
  
  // Diferentes tipos de eventos
  if (data.type === 'ticket.created') {
    // Nova senha criada
    console.log(`Nova senha: ${data.ticket.number}`);
    updateQueueDisplay(data.ticket);
  }
  else if (data.type === 'ticket.called') {
    // Senha foi chamada
    console.log(`Chamando: ${data.ticket.number} no ${data.desk.name}`);
    playSound('alert.mp3');
    updateDisplay(`SENHA ${data.ticket.number} - ${data.desk.name}`);
  }
  else if (data.type === 'ticket.finished') {
    // Atendimento finalizado
    console.log(`Finalizado: ${data.ticket.number}`);
    removeFromDisplay(data.ticket.id);
  }
});

ws.addEventListener('close', function() {
  console.log('Desconectado do WebSocket');
  // Reconectar automaticamente
  setTimeout(() => connectWebSocket(), 5000);
});

ws.addEventListener('error', function(error) {
  console.error('Erro WebSocket:', error);
});
```

### Mensagens WebSocket Típicas

```json
{
  "type": "ticket.created",
  "timestamp": "2025-12-18T16:27:00Z",
  "ticket": {
    "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "number": 42,
    "service": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Abertura de Conta"
    },
    "priority": "normal",
    "status": "waiting"
  }
}
```

```json
{
  "type": "ticket.called",
  "timestamp": "2025-12-18T16:29:00Z",
  "ticket": {
    "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "number": 42
  },
  "desk": {
    "id": "770e8400-e29b-41d4-a716-446655440222",
    "number": 1,
    "name": "Balcão 1"
  }
}
```

```json
{
  "type": "ticket.finished",
  "timestamp": "2025-12-18T16:36:00Z",
  "ticket": {
    "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "number": 42,
    "duration_seconds": 420
  }
}
```

---

## 📊 Exemplo de Integração Frontend

### Display de Agência (Tela de Senhas)

```html
<!DOCTYPE html>
<html>
<head>
  <title>Display de Senhas - Agência Centro</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      background: #333;
      margin: 0;
    }
    
    .display-container {
      background: #000;
      color: #fff;
      padding: 40px;
      border-radius: 10px;
      text-align: center;
      min-width: 800px;
    }
    
    .agencia-name {
      font-size: 24px;
      margin-bottom: 30px;
      text-transform: uppercase;
    }
    
    .ticket-box {
      background: #222;
      margin: 20px 0;
      padding: 30px;
      border-radius: 10px;
      border: 3px solid #00cc00;
    }
    
    .ticket-number {
      font-size: 120px;
      font-weight: bold;
      color: #00ff00;
      text-shadow: 0 0 10px #00ff00;
    }
    
    .ticket-desk {
      font-size: 48px;
      color: #ffff00;
      margin-top: 20px;
    }
    
    .waiting-queue {
      background: #111;
      padding: 20px;
      margin-top: 30px;
      border-radius: 10px;
    }
    
    .queue-item {
      background: #333;
      padding: 15px;
      margin: 10px 0;
      border-left: 4px solid #00cc00;
      text-align: left;
    }
  </style>
</head>
<body>
  <div class="display-container">
    <div class="agencia-name">Agência Centro</div>
    
    <div id="current-ticket" style="display: none;">
      <div class="ticket-box">
        <div class="ticket-number" id="ticket-number">--</div>
        <div class="ticket-desk" id="ticket-desk">--</div>
      </div>
    </div>
    
    <div id="no-ticket" class="ticket-box">
      <div style="font-size: 48px; color: #888;">
        Nenhuma senha sendo chamada
      </div>
    </div>
    
    <div class="waiting-queue">
      <h3>Próximas Senhas:</h3>
      <div id="queue-list">
        <!-- Preenchido por JavaScript -->
      </div>
    </div>
  </div>
  
  <script>
    let ws;
    let currentTicket = null;
    let queueTickets = [];
    
    function connectWebSocket() {
      ws = new WebSocket('ws://localhost:8081');
      
      ws.addEventListener('open', () => {
        console.log('Conectado ao WebSocket');
        ws.send(JSON.stringify({
          action: 'subscribe',
          channel: 'display',
          agency_id: 'abad8412-b7de-41f4-8302-464fe1751a41'
        }));
      });
      
      ws.addEventListener('message', (event) => {
        const data = JSON.parse(event.data);
        handleMessage(data);
      });
      
      ws.addEventListener('close', () => {
        console.log('Desconectado, reconectando em 5s...');
        setTimeout(connectWebSocket, 5000);
      });
    }
    
    function handleMessage(data) {
      if (data.type === 'ticket.created') {
        queueTickets.push(data.ticket);
        updateQueue();
      }
      else if (data.type === 'ticket.called') {
        currentTicket = data.ticket;
        document.getElementById('ticket-number').textContent = data.ticket.number;
        document.getElementById('ticket-desk').textContent = 'BALCÃO ' + data.desk.number;
        document.getElementById('no-ticket').style.display = 'none';
        document.getElementById('current-ticket').style.display = 'block';
        playSound();
      }
      else if (data.type === 'ticket.finished') {
        currentTicket = null;
        document.getElementById('current-ticket').style.display = 'none';
        document.getElementById('no-ticket').style.display = 'block';
      }
    }
    
    function updateQueue() {
      const list = document.getElementById('queue-list');
      list.innerHTML = '';
      queueTickets.slice(0, 5).forEach(ticket => {
        const item = document.createElement('div');
        item.className = 'queue-item';
        item.textContent = `Senha ${ticket.number} - ${ticket.service.name}`;
        list.appendChild(item);
      });
    }
    
    function playSound() {
      // Toca um bip simples
      const audioContext = new (window.AudioContext || window.webkitAudioContext)();
      const oscillator = audioContext.createOscillator();
      const gain = audioContext.createGain();
      
      oscillator.connect(gain);
      gain.connect(audioContext.destination);
      
      oscillator.frequency.value = 1000;
      oscillator.type = 'sine';
      
      gain.gain.setValueAtTime(0.3, audioContext.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.5);
      
      oscillator.start(audioContext.currentTime);
      oscillator.stop(audioContext.currentTime + 0.5);
    }
    
    // Iniciar
    connectWebSocket();
  </script>
</body>
</html>
```

---

## 📱 Teste Rápido com cURL

### Script Completo de Teste

```bash
#!/bin/bash

# Cores para output
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

echo -e "${BLUE}=== TESTE COMPLETO DO SISTEMA ===${NC}"

# 1. Login
echo -e "\n${BLUE}1. Fazendo login...${NC}"
TOKEN=$(curl -s -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "operador@agencia.com.br", "password": "senha123"}' \
  | jq -r '.access_token')

if [ -z "$TOKEN" ] || [ "$TOKEN" = "null" ]; then
  echo "❌ Falha no login"
  exit 1
fi

echo -e "${GREEN}✓ Token: ${TOKEN:0:20}...${NC}"

# 2. Criar senha
echo -e "\n${BLUE}2. Criando nova senha...${NC}"
TICKET=$(curl -s -X POST http://localhost:8080/tickets \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"service_id": "550e8400-e29b-41d4-a716-446655440000", "priority": "normal"}')

TICKET_ID=$(echo $TICKET | jq -r '.id')
TICKET_NUMBER=$(echo $TICKET | jq -r '.number')

echo -e "${GREEN}✓ Senha criada: #${TICKET_NUMBER} (ID: ${TICKET_ID:0:8}...)${NC}"

# 3. Listar senhas
echo -e "\n${BLUE}3. Listando senhas na fila...${NC}"
curl -s -X GET http://localhost:8080/tickets \
  -H "Authorization: Bearer $TOKEN" \
  | jq '.data | length' | xargs echo -e "${GREEN}✓ Senhas na fila:${NC}"

# 4. Chamar senha
echo -e "\n${BLUE}4. Chamando a senha no balcão 1...${NC}"
CALL=$(curl -s -X POST http://localhost:8080/tickets/call-next \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"desk_id": "770e8400-e29b-41d4-a716-446655440222"}')

echo -e "${GREEN}✓ Senha chamada:${NC}"
echo $CALL | jq '.ticket_number, .desk_number'

# 5. Finalizar atendimento
echo -e "\n${BLUE}5. Finalizando atendimento (480 segundos)...${NC}"
FINISH=$(curl -s -X PATCH http://localhost:8080/tickets/${TICKET_ID}/finish \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"duration": 480}')

echo -e "${GREEN}✓ Atendimento finalizado:${NC}"
echo $FINISH | jq '.status, .duration_seconds'

# 6. Verificar se sincronizou com Cloud
echo -e "\n${BLUE}6. Verificando sincronização com Cloud API...${NC}"
sleep 2  # Aguardar processamento
CLOUD_TICKET=$(curl -s -X GET "http://localhost:8084/v1/tickets/${TICKET_ID}" \
  -H "Authorization: Bearer $CLOUD_TOKEN" 2>/dev/null)

if echo $CLOUD_TICKET | jq -e '.id' > /dev/null 2>&1; then
  echo -e "${GREEN}✓ Sincronizado com sucesso!${NC}"
else
  echo "⚠ Ainda não sincronizado (pode levar alguns segundos)"
fi

echo -e "\n${BLUE}=== TESTE CONCLUÍDO ===${NC}"
```

---

**Última atualização:** 30 de Maio de 2026
