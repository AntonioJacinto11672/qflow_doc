# 🎯 COMEÇAR AQUI - RESUMO DE UMA PÁGINA

> **Leia isto primeiro para entender tudo em 2 minutos!**

---

## O QUÊ É O SISTEMA?

**Ticket Dispenser** é um **sistema de fila digital para agências** que funciona assim:

```
CLIENTE → [Pega Senha #42] → [Vê em Display] → [Operador Chama] → [Atendimento] → [Dados na Nuvem]
```

---

## AS DUAS PARTES DO SISTEMA

### 🏪 LOCAL API (Cada Agência)
- **Localização:** Dentro de cada agência
- **Banco:** SQLite (rápido, offline)
- **Funciona:** Mesmo sem internet ✓
- **Responsável:** Criar senhas, chamar clientes
- **Porta:** 8080

### ☁️ CLOUD API (Servidor Central)
- **Localização:** Servidor remoto
- **Banco:** MySQL (centralizado)
- **Funciona:** Precisa de internet
- **Responsável:** Dashboards, relatórios
- **Porta:** 8084

### 🔗 RabbitMQ (Mensageiro)
- **Função:** Conecta Local e Cloud
- **Garante:** Nenhum evento se perde
- **Tempo:** Síncrono em tempo real
- **Porta:** 5672 (ou 15672 Management UI)

---

## COMEÇAR EM 3 PASSOS

### 1️⃣ Instalar Dependências (Opcional com Docker)

```bash
# Com Docker (MAIS FÁCIL)
cd dispenser-local-api-main
docker-compose up -d

cd ../dispenser-remote-api-main
docker-compose up -d

# Pronto! ✓
```

### 2️⃣ Testar a API

```bash
# Verificar se está online
curl http://localhost:8080/health
curl http://localhost:8084/health

# Ambas devem retornar 200 OK
```

### 3️⃣ Fazer Login

```bash
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "operador@agencia.com.br",
    "password": "senha123"
  }'
```

---

## OPERAÇÕES PRINCIPAIS

| O Que | Endpoint | O Que Faz |
|-------|----------|----------|
| **Criar Senha** | `POST /tickets` | Cliente pega nova senha |
| **Chamar Senha** | `POST /tickets/call-next` | Operador chama no balcão |
| **Finalizar** | `PATCH /tickets/{id}/finish` | Termina atendimento |
| **Ver Dashboard** | `GET /dashboard/overview` | Gerente vê relatórios |
| **Exportar Relatório** | `GET /reports/daily` | Dados do dia |

---

## ESTRUTURA DE PASTAS (O Que Está Onde)

```
📁 dispenser-local-api-main/
  ├─ 📁 app/Http/Controllers/  ← Recebem requisições
  ├─ 📁 app/Services/          ← Lógica de negócio
  ├─ 📁 app/Models/            ← Tabelas do banco
  ├─ 📁 app/Domain/Events/     ← Eventos do domínio
  ├─ 📁 database/              ← Migrations e seeds
  ├─ 📁 routes/                ← Definição de URLs
  ├─ 📁 storage/logs/          ← Arquivos de log
  ├─ 📄 .env                   ← Configurações (IMPORTANTE!)
  └─ 📄 docker-compose.yml     ← Configuração Docker

📁 dispenser-remote-api-main/  ← Mesma estrutura
```

---

## CONFIGURAÇÃO CRÍTICA (`.env`)

### Local API - O Que NUNCA Esquecer

```env
AGENCY_ID=seu-uuid-da-agencia    # Identifica a agência
APP_ENV=local                     # Modo desenvolvimento
JWT_SECRET=gerado-pelo-artisan   # Chave de autenticação
DB_DATABASE=database/readmodel.sqlite
EVENTSTORE_DB_DATABASE=database/eventstore.sqlite
```

### Cloud API - O Que Mudar

```env
APP_ENV=local
DB_HOST=mysql
DB_DATABASE=ticket_dispenser
RABBITMQ_HOST=rabbitmq
```

---

## FLUXO EM 5 SEGUNDOS

```
🎯 Cliente chega
   ↓
💻 Pressiona botão no kiosk
   ↓
🏪 Local API: Cria evento "TicketCreated"
   ↓
💾 Persiste em SQLite Event Store
   ↓
📡 Publica em RabbitMQ
   ↓
☁️ Cloud API: Consome e atualiza MySQL
   ↓
📊 Dashboard em tempo real atualiza
   ↓
✓ Tudo sincronizado!
```

---

## TECNOLOGIAS (Stack)

| Camada | Tecnologia |
|--------|-----------|
| Linguagem | PHP 8.3+ |
| Framework | Laravel 12 |
| Banco Local | SQLite |
| Banco Cloud | MySQL 8.0 |
| Message Broker | RabbitMQ |
| Frontend | Vite + Tailwind CSS |
| Autenticação | JWT (Local) + Sanctum (Cloud) |

---

## COMANDOS ÚTEIS

```bash
# Ver logs em tempo real
tail -f storage/logs/laravel.log

# Criar usuário de teste
php artisan tinker
>>> \App\Models\User::create(['name' => 'Teste', 'email' => 'teste@test.com', 'password' => bcrypt('123456')])

# Processar fila de eventos
php artisan queue:work

# Limpar cache
php artisan cache:clear
php artisan config:clear

# Reiniciar tudo
docker-compose down
docker-compose up -d
```

---

## ERROS COMUNS

| Erro | Solução |
|------|---------|
| `Port 8080 in use` | `lsof -i :8080` e mata o processo |
| `JWT_SECRET not set` | `php artisan jwt:secret` |
| `Connection refused` | Verifica se docker está rodando: `docker ps` |
| `401 Unauthorized` | Faz novo login, token expirou |
| `Database locked` | SQLite corrompido: `rm database/*.sqlite && php artisan migrate` |

---

## PRÓXIMOS PASSOS

### Ler Documentação Completa 📖
1. **INDICE_DOCUMENTACAO.md** - Índice de tudo
2. **DOCUMENTACAO_COMPLETA.md** - 5000+ linhas, tudo em detalhe
3. **GUIA_RAPIDO.md** - Checklist e referência rápida
4. **EXEMPLOS_E_FLUXOS.md** - Código real e diagramas

### Integração Frontend 💻
- Conectar WebSocket para atualizações em tempo real
- Criar display de senhas (tela TV)
- Integrar painel operador (desktop)
- Fazer app mobile (opcional)

### Deployment 🚀
- Configurar certificado SSL
- Montar em servidor Linux
- Configurar backups MySQL
- Monitoramento (Prometheus/Grafana)

---

## 🎯 ARQUIVOS CRIADOS

Foram gerados **4 documentos** nesta pasta:

```
c:\xampp\htdocs\externa_api\
├─ 📄 INDICE_DOCUMENTACAO.md          ← Índice completo (nav)
├─ 📄 DOCUMENTACAO_COMPLETA.md        ← 10.000+ palavras
├─ 📄 GUIA_RAPIDO.md                  ← Referência rápida
├─ 📄 EXEMPLOS_E_FLUXOS.md            ← Código + Diagramas
└─ 📄 COMECE_AQUI.md                  ← ESTE ARQUIVO
```

---

## ⏱️ TEMPO ESTIMADO

| Atividade | Tempo |
|-----------|-------|
| Instalar tudo | 15 min |
| Fazer primeiro teste | 5 min |
| Ler GUIA_RAPIDO | 15 min |
| Ler DOCUMENTACAO_COMPLETA | 60 min |
| Entender completamente | 4-8 horas |
| Desenvolver feature nova | +4 horas |

---

## 🎓 MAPA MENTAL

```
TICKET DISPENSER
    │
    ├─ Local API (Agência)
    │   ├─ Cria senhas
    │   ├─ Chama clientes
    │   ├─ Funciona offline
    │   └─ SQLite
    │
    ├─ RabbitMQ (Mensageiro)
    │   ├─ Publica eventos
    │   ├─ Garante entrega
    │   └─ Síncrono
    │
    └─ Cloud API (Central)
        ├─ Consolida dados
        ├─ Gera relatórios
        ├─ Dashboards
        └─ MySQL
```

---

## ✅ SUCESSO QUANDO...

- [ ] Conseguiu instalar com Docker
- [ ] Fez login na Local API
- [ ] Criou uma senha
- [ ] Chamou a senha
- [ ] Viu no dashboard Cloud
- [ ] Entendeu o fluxo RabbitMQ
- [ ] Explorou estrutura de código
- [ ] Conseguiu ler logs

---

## 📞 AJUDA RÁPIDA

**Não consegue?** → **Leia:**

- "Não sei instalar" → DOCUMENTACAO_COMPLETA.md (Seção: Instalação)
- "API não funciona" → GUIA_RAPIDO.md (Seção: Troubleshooting)
- "Quero exemplos" → EXEMPLOS_E_FLUXOS.md (Requisições HTTP)
- "Preciso entender fluxos" → EXEMPLOS_E_FLUXOS.md (Diagramas)
- "Dúvida sobre pastas" → DOCUMENTACAO_COMPLETA.md (Estrutura)
- "Preciso de resumo" → GUIA_RAPIDO.md (Checklist)

---

## 🚀 PRÓXIMO: QUAL ARQUIVO LER?

```
Tenho 5 minutos? 
  ↓
  → GUIA_RAPIDO.md - Seção "Iniciar em 5 min"

Sou novo no projeto?
  ↓
  → INDICE_DOCUMENTACAO.md - Mapa de navegação
  → DOCUMENTACAO_COMPLETA.md - Leitura completa

Preciso integrar frontend?
  ↓
  → EXEMPLOS_E_FLUXOS.md - WebSocket + Código

Algo quebrou?
  ↓
  → DOCUMENTACAO_COMPLETA.md - Seção "Troubleshooting"
```

---

## 🎉 CONCLUSÃO

O **Ticket Dispenser** é um sistema bem estruturado de:
- ✓ 2 APIs (Local + Cloud)
- ✓ 2 Bancos de dados (SQLite + MySQL)
- ✓ Comunicação assíncrona (RabbitMQ)
- ✓ Arquitetura event-driven
- ✓ Funcionalidade offline-first

**Está pronto para começar?** Escolha um dos 4 documentos acima e comece!

---

**Versão:** 1.0  
**Data:** 30 de Maio de 2026  
**Status:** ✅ Documentação completa  

**Bem-vindo ao projeto! 🚀**
