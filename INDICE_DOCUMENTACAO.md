# 📚 ÍNDICE GERAL DE DOCUMENTAÇÃO - TICKET DISPENSER

## 📖 Documentos Criados

Foram criados **4 documentos completos** para ajudá-lo a entender e usar o sistema:

### 1. **DOCUMENTACAO_COMPLETA.md** 📘
   - **O Quê:** Documentação exhaustiva do projeto
   - **Para Quem:** Desenvolvedores, arquitetos, stakeholders
   - **Conteúdo:**
     - ✓ Visão geral do projeto (2500+ linhas)
     - ✓ Pré-requisitos e instalação passo-a-passo
     - ✓ Configuração detalhada do `.env`
     - ✓ Execução em modo desenvolvimento (Com/Sem Docker)
     - ✓ Como funciona o sistema (fluxos)
     - ✓ Arquitetura detalhada (Local + Cloud)
     - ✓ Estrutura completa de diretórios explicada
     - ✓ Endpoints principais da API
     - ✓ Comunicação entre APIs (RabbitMQ)
     - ✓ Troubleshooting com soluções
   - **Tempo de Leitura:** 45-60 minutos

### 2. **GUIA_RAPIDO.md** ⚡
   - **O Quê:** Referência rápida e checklist
   - **Para Quem:** Desenvolvedores em pressa, DevOps
   - **Conteúdo:**
     - ✓ Iniciar em 5 minutos com Docker
     - ✓ Checklist de configuração
     - ✓ Comandos essenciais
     - ✓ Endpoints principais resumidos
     - ✓ Autenticação JWT
     - ✓ Erros comuns e soluções
     - ✓ Stack de tecnologias
   - **Tempo de Leitura:** 10-15 minutos

### 3. **EXEMPLOS_E_FLUXOS.md** 🎨
   - **O Quê:** Exemplos práticos e diagramas visuais
   - **Para Quem:** Desenvolvedores frontend, integradores
   - **Conteúdo:**
     - ✓ Exemplos reais de requisições HTTP com cURL
     - ✓ Fluxos visuais ASCII (Criação, Chamada, Sincronização)
     - ✓ WebSocket em tempo real (código JavaScript)
     - ✓ Exemplo completo de integração frontend
     - ✓ Script de teste automatizado em bash
   - **Tempo de Leitura:** 30-40 minutos

### 4. **INDICE_DOCUMENTACAO.md** (ESTE ARQUIVO) 📋
   - **O Quê:** Navegação e índice de todos os documentos
   - **Para Quem:** Todos
   - **Conteúdo:**
     - ✓ Índice de documentos
     - ✓ Mapa de navegação
     - ✓ Quick start
     - ✓ Árvore visual do projeto

---

## 🗺️ Como Navegar pela Documentação

### 👤 Tenho 5 minutos

1. Leia: [GUIA_RAPIDO.md](GUIA_RAPIDO.md) - Seção "⚡ Iniciar em 5 Minutos"
2. Execute: Docker Compose

### 👨‍💻 Sou desenvolvedor novo no projeto

1. Leia: [DOCUMENTACAO_COMPLETA.md](DOCUMENTACAO_COMPLETA.md) - Seções 1-3
2. Leia: [GUIA_RAPIDO.md](GUIA_RAPIDO.md) - Completo
3. Execute: Setup e instalação
4. Explore: Estrutura de diretórios

### 🏗️ Preciso entender a arquitetura

1. Leia: [DOCUMENTACAO_COMPLETA.md](DOCUMENTACAO_COMPLETA.md) - Seção "Como Funciona"
2. Leia: [EXEMPLOS_E_FLUXOS.md](EXEMPLOS_E_FLUXOS.md) - Fluxos visuais
3. Estude: Diagramas ASCII

### 🔌 Vou integrar com frontend

1. Leia: [EXEMPLOS_E_FLUXOS.md](EXEMPLOS_E_FLUXOS.md) - WebSocket + Exemplo frontend
2. Copie: Código de exemplo
3. Estude: Estrutura de mensagens JSON

### 🚀 Vou fazer deploy em produção

1. Leia: [DOCUMENTACAO_COMPLETA.md](DOCUMENTACAO_COMPLETA.md) - Seção "Pré-requisitos"
2. Consulte: Arquivo original `SETUP.md`
3. Configure: `.env` de produção
4. Teste: Script de troubleshooting

### 🐛 Algo não está funcionando

1. Vá a: [DOCUMENTACAO_COMPLETA.md](DOCUMENTACAO_COMPLETA.md) - Seção "Troubleshooting"
2. Procure: Seu erro específico
3. Siga: Passo-a-passo da solução

---

## 📊 Mapa Visual do Projeto

```
                  TICKET DISPENSER SYSTEM
                          │
              ┌─────────────┼─────────────┐
              │             │             │
         LOCAL API      RABBITMQ      CLOUD API
         (Port 8080)   (Broker)     (Port 8084)
             │          │              │
             │          │              │
      ┌──────┴─────┐    │       ┌──────┴──────┐
      │             │    │       │             │
   SQLite(2x)  WebSocket │   MySQL    Redis
   (Offline)   (Real-time)│  (Central) (Cache)
                          │
                    (Sync Events)
                          │
      ┌─────────────────────────────────────┐
      │                                     │
   Outbox Pattern                    Event Consumer
   (Never lose)                      (Process & Store)
      │                                     │
      └─────────────────────────────────────┘
```

### Dados por Localização

```
LOCAL API (Agência)           CLOUD API (Servidor Central)
────────────────────────      ──────────────────────────

SQLite Read Model             MySQL Master
├─ Tickets (atual)           ├─ Todas agências
├─ Users                      ├─ Consolidado
├─ Desks                      ├─ Relatórios
├─ Services                   ├─ Analytics
└─ Devices

SQLite Event Store            RabbitMQ Queue
├─ Histórico completo        ├─ cloud_events_processor
├─ Auditoria                 ├─ local_sync.*
├─ Recovery                  └─ Dead letter queue
└─ Reconstrução

Funciona:                     Funciona:
✓ Online                      ✓ Online
✓ Offline (totalmente)        ✗ Offline
✓ Rápido (local)              ✓ Rápido (cache)
✓ Atende clientes             ✓ Relatórios
```

---

## 🔐 Fluxo de Autenticação

```
┌─────────────────────────────────────┐
│      LOCAL API (JWT)                │
│                                     │
│  POST /auth/login                  │
│  ├─ Email + Senha                  │
│  └─ Retorna: access_token (JWT)    │
│                                     │
│  Use em: Authorization: Bearer {token}
│  Válido por: JWT_TTL (padrão 24h)  │
│  Refresh: POST /auth/refresh       │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│     CLOUD API (Laravel Sanctum)     │
│                                     │
│  POST /v1/auth/login                │
│  ├─ Email + Senha                   │
│  └─ Retorna: token                  │
│                                     │
│  Use em: Authorization: Bearer {token}
│  Válido por: 1 hora (padrão)       │
│  Refresh: Automático com válido    │
└─────────────────────────────────────┘
```

---

## 💻 Stack de Tecnologias

```
┌─────────────────────────────────────────────────┐
│           FRONTEND                              │
│  HTML5 + JavaScript + Tailwind CSS + Vite       │
│  (Responsivo para displays, tablets, desktop)   │
└─────────────────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────────────────┐
│                    API LAYER                                  │
├──────────────────────────────────────────────────────────────┤
│  LOCAL API (Laravel 12, PHP 8.3+)  │  CLOUD API (Laravel 12) │
│  ├─ Controller                     │  ├─ Controller          │
│  ├─ Service                        │  ├─ Service             │
│  ├─ Domain Events (CQRS)           │  ├─ Repository          │
│  ├─ Eloquent ORM                   │  ├─ Eloquent ORM        │
│  └─ Infrastructure (RabbitMQ)      │  └─ Infrastructure      │
└──────────────────────────────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
    ┌────────┐   ┌──────────┐  ┌────────┐
    │ SQLite │   │ RabbitMQ │  │ MySQL  │
    │ (Banco)│   │ (Mensagens)  │(Banco) │
    └────────┘   └──────────┘  └────────┘
        ↓             ↓            ↓
    (Local)      (Broker)    (Central)
    Offline      Async       Online
    2 bancos     Event-      Múltiplos
    separados    driven      tenants
```

---

## 🎯 Checklist de Primeiro Acesso

### Antes de Começar
- [ ] Leu este arquivo (INDICE_DOCUMENTACAO.md)
- [ ] Tem Docker/Docker Compose instalado
- [ ] Tem Git instalado
- [ ] Tem 30 minutos de tempo

### Instalação (15 min)
- [ ] Clonou/preparou pasta do projeto
- [ ] Rodou `docker-compose up -d` na Local API
- [ ] Rodou `docker-compose up -d` na Cloud API
- [ ] Verificou `curl http://localhost:8080/health`
- [ ] Verificou `curl http://localhost:8084/health`

### Primeiro Teste (10 min)
- [ ] Fez login com POST /auth/login
- [ ] Criou uma senha com POST /tickets
- [ ] Chamou a senha com POST /tickets/call-next
- [ ] Finalizou com PATCH /tickets/{id}/finish
- [ ] Verificou no dashboard Cloud

### Entendimento (5 min)
- [ ] Explorou arquivo GUIA_RAPIDO.md
- [ ] Entendeu a estrutura de diretórios
- [ ] Conhece os 2 bancos SQLite (Local)
- [ ] Conhece o MySQL (Cloud)

---

## 🔍 Busca Rápida

**Procurando...** → **Vá a...**

| Procurando | Arquivo | Seção |
|-----------|---------|-------|
| Como instalar | DOCUMENTACAO_COMPLETA | "Pré-requisitos" |
| Como iniciar | GUIA_RAPIDO | "Iniciar em 5 min" |
| Endpoints da API | DOCUMENTACAO_COMPLETA | "Endpoints" |
| Exemplos HTTP | EXEMPLOS_E_FLUXOS | "Requisições" |
| Fluxos visuais | EXEMPLOS_E_FLUXOS | "Fluxos Visuais" |
| WebSocket | EXEMPLOS_E_FLUXOS | "WebSocket" |
| Estrutura de pastas | DOCUMENTACAO_COMPLETA | "Estrutura" |
| Autenticação | DOCUMENTACAO_COMPLETA | "Endpoints" |
| RabbitMQ | DOCUMENTACAO_COMPLETA | "Comunicação" |
| Erros | DOCUMENTACAO_COMPLETA | "Troubleshooting" |
| Comandos úteis | GUIA_RAPIDO | "Comandos" |
| Tecnologias | GUIA_RAPIDO | "Stack" |
| Variáveis `.env` | DOCUMENTACAO_COMPLETA | "Configuração" |
| Event Sourcing | DOCUMENTACAO_COMPLETA | "Como Funciona" |
| Multi-tenant | DOCUMENTACAO_COMPLETA | "Cloud API" |

---

## 📱 Exemplos de Arquivos

### Arquivo `.env.example` (Local API)

```env
APP_NAME="Ticket Dispenser Local"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8080
APP_KEY=base64:...
APP_TIMEZONE=America/Sao_Paulo

# Banco de dados
DB_CONNECTION=sqlite
DB_DATABASE=database/readmodel.sqlite
EVENTSTORE_DB_DATABASE=database/eventstore.sqlite

# Agência
AGENCY_ID=seu-uuid-aqui

# Autenticação
JWT_SECRET=seu-secret-aqui
JWT_TTL=1440
JWT_REFRESH_TTL=20160

# RabbitMQ
RABBITMQ_HOST=rabbitmq
RABBITMQ_PORT=5672
RABBITMQ_USER=admin
RABBITMQ_PASSWORD=admin123
RABBITMQ_VHOST=/

# WebSocket
WEBSOCKET_PORT=8081
WEBSOCKET_HOST=0.0.0.0

# Logging
LOG_CHANNEL=stack
LOG_LEVEL=debug
```

### Arquivo `docker-compose.yml` (Local API - Simplificado)

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: dispenser-local-api
    restart: unless-stopped
    ports:
      - "8080:80"
      - "8081:8081"
    volumes:
      - .:/app
      - dispenser_storage:/app/storage
      - dispenser_database:/app/database
    environment:
      - AGENCY_ID=abad8412-b7de-41f4-8302-464fe1751a41
      - APP_ENV=local
    networks:
      - dispenser

volumes:
  dispenser_storage:
  dispenser_database:

networks:
  dispenser:
    driver: bridge
```

---

## 🚀 Próximos Passos Após Setup

### Nível 1: Iniciante
1. Explore a interface web (se houver)
2. Crie alguns dados de teste
3. Veja os logs em `storage/logs/laravel.log`
4. Entenda a estrutura de bancos SQLite

### Nível 2: Intermediário
1. Integre um frontend simples com WebSocket
2. Crie endpoints customizados
3. Implemente features novas
4. Estude o padrão de Event Sourcing

### Nível 3: Avançado
1. Configure múltiplas agências
2. Implemente clustering
3. Otimize queries
4. Configure monitoramento em produção

---

## 📞 Referências Externas

### Documentação Oficial
- [Laravel 12 Docs](https://laravel.com/docs/12.x)
- [PHP 8.3 Docs](https://www.php.net/docs.php)
- [RabbitMQ Docs](https://www.rabbitmq.com/documentation.html)
- [SQLite Docs](https://www.sqlite.org/docs.html)
- [MySQL 8.0 Docs](https://dev.mysql.com/doc/refman/8.0/en/)

### Padrões Utilizados
- **Event Sourcing**: Armazenar apenas eventos, reconstruir estado
- **CQRS**: Command Query Responsibility Segregation (Local API)
- **Multi-tenancy**: Isolamento de dados por agência (Cloud API)
- **Outbox Pattern**: Garantir entrega de eventos
- **WebSocket**: Comunicação em tempo real bidirecional

### Ferramentas Úteis
- **Postman**: Testar endpoints `https://www.postman.com`
- **DBeaver**: Gerenciar bancos de dados `https://dbeaver.io`
- **RabbitMQ Management**: UI integrado em `http://localhost:15672`

---

## 📝 Histórico de Versões da Documentação

| Versão | Data | Mudanças |
|--------|------|----------|
| 1.0 | 30/05/2026 | Documentação inicial completa |

---

## ✅ Checklist Final

Depois de ler toda documentação:

- [ ] Entendo o que é o sistema Ticket Dispenser
- [ ] Sei as diferenças entre Local API e Cloud API
- [ ] Conheço as tecnologias usadas
- [ ] Consegui instalar o sistema
- [ ] Consigo fazer um login e criar uma senha
- [ ] Entendo como RabbitMQ conecta as duas APIs
- [ ] Sei onde está cada componente do código
- [ ] Consigo troubleshootar erros básicos
- [ ] Tenho recursos para continuar aprendendo

Se marcou tudo: **Parabéns! Está pronto para desenvolver! 🎉**

---

## 🎓 Recursos de Aprendizado

### Hora de Ouro: Primeiras 2 Horas
1. Setup completo
2. Entender fluxo básico
3. Fazer primeiro teste
4. Explorar banco de dados

### Próximas 8 Horas
1. Ler DOCUMENTACAO_COMPLETA
2. Estudar arquitetura em detalhe
3. Explorar código do projeto
4. Fazer pequenas modificações

### Domínio Completo: 1-2 Semanas
1. Implementar features
2. Entender RabbitMQ profundamente
3. Otimizar queries
4. Configurar produção

---

## 📞 Suporte

Se tiver dúvidas:

1. **Procure primeiro neste índice** (Ctrl+F)
2. **Consulte a seção Troubleshooting** da documentação completa
3. **Verifique logs** em `storage/logs/laravel.log`
4. **Teste com cURL** ou Postman

---

**Documento criado:** 30 de Maio de 2026  
**Status:** ✅ Completo e atualizado  
**Total de documentação:** 4 arquivos, 10.000+ palavras

---

**Bem-vindo ao Sistema Ticket Dispenser! 🎉**

Boa sorte com seu desenvolvimento!
