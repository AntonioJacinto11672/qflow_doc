# 📚 DOCUMENTAÇÃO REVISADA - WORKSPACE ATUAL

**Data:** Maio 2026
**Versão do Documento:** 1.0
**Status:** Documentação sincronizada com o código existente

---

## 1. Projetos reais disponíveis neste workspace

Este workspace contém exatamente dois projetos:

1. `dispenser-remote-api-main`
   - **Laravel 12 Cloud API**.
   - Baseado em `routes/api.php`.
   - Usa `auth:sanctum` e `tenant.context` em muitos endpoints.
   - Fornece o backend central de gerenciamento de agências, usuários, desks, serviços, templates, relatórios e BI.

2. `dispenser-backoffice-main/dispenser-backoffice-main`
   - **Next.js Backoffice**.
   - Inclui uma API de desenvolvimento mock em `app/api/v1`.
   - Serve como interface administrativa e console de teste.
   - Não é um backend de produção real; responde com dados em memória e tokens simulados.

> Observação importante: não há atualmente um projeto `dispenser-local-api-main` neste workspace.

---

## 2. Como usar os dois projetos juntos

### 2.1 `dispenser-remote-api-main` (Cloud API)

Passos mínimos:

```bash
cd c:\xampp\htdocs\externa_api\dispenser-remote-api-main
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan serve --port=8084
```

A API ficará disponível em `http://localhost:8084`.

### 2.2 `dispenser-backoffice-main/dispenser-backoffice-main` (Backoffice)

Passos mínimos:

```bash
cd c:\xampp\htdocs\externa_api\dispenser-backoffice-main\dispenser-backoffice-main
npm install
npm run dev
```

A aplicação será iniciada em `http://localhost:3000`.

### 2.3 O relacionamento entre os projetos

- O Backoffice é um painel administrativo que roda localmente em Next.js.
- Ele contém rotas mock em `app/api/v1` para simular dados internos.
- Essas rotas não estão implementadas como um cliente direto da Cloud API neste workspace.
- Para integrar o Backoffice à Cloud API, é preciso ajustar o código do frontend e/ou as rotas mock para consumir `http://localhost:8084`.

---

## 3. Observações de arquitetura do workspace

- `dispenser-remote-api-main` é o único serviço Laravel real disponível aqui.
- `dispenser-backoffice-main/dispenser-backoffice-main` é um frontend Next.js com backend de mock.
- A Cloud API prepara e expõe um conjunto de endpoints REST completos, incluindo health, autenticação, gerenciamento de agências, desks, serviços, dashboards, relatórios e BI.
- O Backoffice expõe apenas um conjunto de endpoints de desenvolvimento para uso interno do painel.

---

## 4. Rotas do Cloud API (`dispenser-remote-api-main`)

### 4.1 Base de URL

`http://localhost:8084`

### 4.2 Rotas públicas e health

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/health` | Verifica se o serviço está disponível e se DB/RabbitMQ estão saudáveis |
| POST | `/v1/auth/login` | Autentica usuário e retorna token |
| GET | `/v1/agencies/{id}` | Retorna dados públicos de uma agência por ID |

### 4.3 Autenticação e sessão

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/auth/me` | Retorna o usuário autenticado atual |
| POST | `/v1/auth/logout` | Encerra a sessão atual |
| POST | `/v1/auth/logout-all` | Encerra todas as sessões do usuário |
| POST | `/v1/auth/refresh` | Atualiza token de sessão |

> Nota: Estas rotas estão protegidas por `auth:sanctum`.

### 4.4 Agências

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/agencies` | Lista agências do tenant |
| POST | `/v1/agencies` | Cria uma nova agência |
| GET | `/v1/agencies/{agency}` | Mostra detalhes de uma agência |
| PUT/PATCH | `/v1/agencies/{agency}` | Atualiza agência |
| DELETE | `/v1/agencies/{agency}` | Remove agência |
| GET | `/v1/agencies/{agency}/services` | Lista serviços vinculados à agência |
| POST | `/v1/agencies/{agency}/services` | Vincula serviços à agência |
| DELETE | `/v1/agencies/{agency}/services` | Desvincula serviços da agência |

### 4.5 Usuários

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/users` | Lista usuários do tenant |
| POST | `/v1/users` | Cria usuário |
| GET | `/v1/users/{user}` | Mostra usuário |
| PUT/PATCH | `/v1/users/{user}` | Atualiza usuário |
| DELETE | `/v1/users/{user}` | Remove usuário |
| GET | `/v1/users/without-desk` | Lista usuários sem desk atribuído |

### 4.6 Desks

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/desks` | Lista desks |
| POST | `/v1/desks` | Cria desk |
| GET | `/v1/desks/{desk}` | Mostra desk |
| PUT/PATCH | `/v1/desks/{desk}` | Atualiza desk |
| DELETE | `/v1/desks/{desk}` | Remove desk |
| GET | `/v1/desks/{desk}/user` | Retorna o usuário associado ao desk |
| POST | `/v1/desks/{desk}/users/{user}` | Atribui um usuário ao desk |
| DELETE | `/v1/desks/{desk}/users/{user}` | Remove usuário do desk |
| GET | `/v1/desks/{desk}/users` | Lista usuários do desk (deprecated) |
| GET | `/v1/desks/{desk}/services` | Lista serviços do desk |
| POST | `/v1/desks/{desk}/services` | Vincula múltiplos serviços ao desk |
| DELETE | `/v1/desks/{desk}/services` | Desvincula múltiplos serviços do desk |
| POST | `/v1/desks/{desk}/services/{service}` | Atribui serviço específico ao desk |
| DELETE | `/v1/desks/{desk}/services/{service}` | Remove serviço específico do desk |

### 4.7 Serviços

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/services` | Lista serviços |
| POST | `/v1/services` | Cria serviço |
| GET | `/v1/services/{service}` | Mostra serviço |
| PUT/PATCH | `/v1/services/{service}` | Atualiza serviço |
| DELETE | `/v1/services/{service}` | Remove serviço |
| GET | `/v1/services/{service}/agencies` | Lista agências vinculadas |
| POST | `/v1/services/{service}/agencies` | Vincula agências ao serviço |
| DELETE | `/v1/services/{service}/agencies` | Desvincula agências do serviço |

### 4.8 Devices

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/devices` | Lista dispositivos registrados |
| POST | `/v1/devices` | Registra dispositivo |
| GET | `/v1/devices/{device}` | Mostra dispositivo |
| PUT/PATCH | `/v1/devices/{device}` | Atualiza dispositivo |
| DELETE | `/v1/devices/{device}` | Remove dispositivo |

### 4.9 Contents

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/contents` | Lista conteúdos |
| POST | `/v1/contents` | Cria conteúdo |
| GET | `/v1/contents/{content}` | Mostra conteúdo |
| PUT/PATCH | `/v1/contents/{content}` | Atualiza conteúdo |
| DELETE | `/v1/contents/{content}` | Remove conteúdo |

### 4.10 Templates

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/templates` | Lista templates |
| POST | `/v1/templates` | Cria template |
| GET | `/v1/templates/{template}` | Mostra template |
| PUT/PATCH | `/v1/templates/{template}` | Atualiza template |
| DELETE | `/v1/templates/{template}` | Remove template |
| POST | `/v1/templates/{template}/set-default` | Define template padrão |
| POST | `/v1/templates/upload-image` | Faz upload de imagem para template |
| POST | `/v1/templates/upload-video` | Faz upload de vídeo para template |

### 4.11 Dashboard

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/dashboard/overview` | Dados de overview do dashboard |
| GET | `/v1/dashboard/metrics` | Métricas da agência |
| GET | `/v1/dashboard/realtime` | Dados em tempo real |
| GET | `/v1/dashboard/desks-status` | Status de desks |
| GET | `/v1/dashboard/traffic-by-service` | Tráfego por serviço |
| GET | `/v1/dashboard/wait-time-by-service` | Tempo de espera por serviço |
| GET | `/v1/dashboard/wait-time-by-desk` | Tempo de espera por desk |
| GET | `/v1/dashboard/daily-traffic-by-service` | Tráfego diário por serviço |
| GET | `/v1/dashboard/daily-traffic` | Tráfego diário por agência |
| GET | `/v1/dashboard/traffic-by-agency` | Tráfego por agência |
| GET | `/v1/dashboard/daily-traffic-by-agency` | Detalhe diário por agência |
| GET | `/v1/dashboard/sla-wait-time` | SLA de espera por prioridade |

### 4.12 Relatórios

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/reports/general-attendance` | Relatório de atendimento geral |
| GET | `/v1/reports/detailed-attendance` | Relatório detalhado de atendimento |
| GET | `/v1/reports/average-wait-time` | Tempo médio de espera |
| GET | `/v1/reports/desk-performance` | Desempenho de desks |
| GET | `/v1/reports/ticket-status` | Status de tickets |
| GET | `/v1/reports/detailed-ticket-status` | Status detalhado de tickets |

### 4.13 Operator Sessions

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/operator-sessions` | Lista sessões de operadores |
| GET | `/v1/operator-sessions/stats` | Estatísticas de sessões |
| GET | `/v1/operator-sessions/{id}` | Mostra sessão específica |

### 4.14 BI Endpoints

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/v1/bi/tickets` | Exporta dados brutos de tickets |
| GET | `/v1/bi/sessions` | Exporta dados brutos de sessões |
| GET | `/v1/bi/services` | Exporta dados brutos de serviços |
| GET | `/v1/bi/desks` | Exporta dados brutos de desks |
| GET | `/v1/bi/agencies` | Exporta dados brutos de agências |

---

## 5. Rotas do Backoffice Mock (`dispenser-backoffice-main/dispenser-backoffice-main`)

### 5.1 Base de URL

`http://localhost:3000/api/v1`

### 5.2 Autenticação Backoffice

| Método | URL | Finalidade |
|--------|-----|------------|
| POST | `/auth/login` | Simula login e retorna token mock |
| GET | `/auth/me` | Retorna dados do usuário atual simulado |
| POST | `/auth/logout` | Simula logout |

### 5.3 Agências e usuários de agência

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/agencies` | Lista agências mock |
| POST | `/agencies` | Cria agência mock |
| GET | `/agencies/{id}` | Detalhes da agência mock |
| DELETE | `/agencies/{id}` | Remove agência mock |
| GET | `/agency-users` | Lista usuários de agência mock |
| POST | `/agency-users` | Cria usuário de agência mock |
| GET | `/agency-users/{id}` | Detalha usuário de agência mock |
| PATCH | `/agency-users/{id}` | Atualiza usuário de agência mock |
| DELETE | `/agency-users/{id}` | Remove usuário de agência mock |

### 5.4 Cloud users e conteúdos

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/cloud-users` | Lista cloud users mock |
| POST | `/cloud-users` | Cria cloud user mock |
| GET | `/cloud-users/{id}` | Retorna cloud user mock |
| PATCH | `/cloud-users/{id}` | Atualiza cloud user mock |
| DELETE | `/cloud-users/{id}` | Remove cloud user mock |
| GET | `/contents` | Lista conteúdos mock |
| POST | `/contents` | Cria conteúdo mock |
| GET | `/contents/{id}` | Retorna conteúdo mock |
| DELETE | `/contents/{id}` | Remove conteúdo mock |

### 5.5 Desks e serviços

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/desks` | Lista desks mock |
| POST | `/desks` | Cria desk mock |
| GET | `/desks/{id}` | Retorna desk mock |
| PATCH | `/desks/{id}` | Atualiza desk mock |
| DELETE | `/desks/{id}` | Remove desk mock |
| PATCH | `/desks/{id}/status` | Atualiza status do desk mock |
| POST | `/desks/{id}/users` | Atribui usuários em massa ao desk mock |
| DELETE | `/desks/{id}/users` | Remove usuários em massa do desk mock |
| POST | `/desks/{id}/users/{userId}` | Atribui usuário ao desk mock |
| DELETE | `/desks/{id}/users/{userId}` | Remove usuário do desk mock |
| POST | `/desks/{id}/services` | Atribui serviços em massa ao desk mock |
| DELETE | `/desks/{id}/services` | Remove serviços em massa do desk mock |
| POST | `/desks/{id}/services/{serviceId}` | Atribui serviço ao desk mock |
| DELETE | `/desks/{id}/services/{serviceId}` | Remove serviço do desk mock |

### 5.6 Serviços e agências de serviço

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/services` | Lista serviços mock |
| POST | `/services` | Cria serviço mock |
| GET | `/services/{id}` | Retorna serviço mock |
| PATCH | `/services/{id}` | Atualiza serviço mock |
| DELETE | `/services/{id}` | Remove serviço mock |
| GET | `/services/{id}/agencies` | Lista agências vinculadas ao serviço mock |
| POST | `/services/{id}/agencies` | Vincula agência ao serviço mock |
| DELETE | `/services/{id}/agencies` | Remove agência do serviço mock |

### 5.7 Templates

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/templates` | Lista templates mock |
| POST | `/templates` | Cria template mock |
| GET | `/templates/{id}` | Retorna template mock |
| PATCH | `/templates/{id}` | Atualiza template mock |
| DELETE | `/templates/{id}` | Remove template mock |
| POST | `/templates/{id}/set-default` | Define template padrão mock |
| POST | `/templates/{id}/upload-image` | Faz upload de imagem mock |
| POST | `/templates/{id}/upload-video` | Faz upload de vídeo mock |

### 5.8 Relatórios do Backoffice

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/reports/general-attendance` | Relatório de atendimento geral mock |
| GET | `/reports/detailed-attendance` | Relatório detalhado mock |
| GET | `/reports/average-wait-time` | Tempo médio de espera mock |
| GET | `/reports/desk-performance` | Desempenho de desks mock |

### 5.9 Usuários do Backoffice

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/users` | Lista usuários mock |
| POST | `/users` | Cria usuário mock |
| GET | `/users/{id}` | Retorna usuário mock |
| PATCH | `/users/{id}` | Atualiza usuário mock |
| DELETE | `/users/{id}` | Remove usuário mock |

### 5.10 Dashboard do Backoffice

| Método | URL | Finalidade |
|--------|-----|------------|
| GET | `/dashboard/overview` | Dados principais de painel mock |
| GET | `/dashboard/metrics` | Métricas mock |
| GET | `/dashboard/realtime` | Dados em tempo real mock |
| GET | `/dashboard/desks-status` | Status de desks mock |

---

## 6. Notas importantes para uso imediato

- Se você precisa de um backend real conectado ao Cloud API, use `dispenser-remote-api-main`.
- Se você quer testar a IU e as rotas internas de admin, use `dispenser-backoffice-main/dispenser-backoffice-main`.
- Atualmente, o Backoffice usa dados mock em memória:
  - `app/api/v1/_mock-db.ts`
  - `app/api/v1/*/route.ts`
- Para documentar ou alterar a API real, foque em `routes/api.php` dentro do Cloud API.

---

## 7. Caminho rápido para roteirizar mudanças

### Se você quiser ligar o Backoffice ao Cloud API real

1. Identifique chamadas de API no frontend do Backoffice.
2. Altere a origem de `http://localhost:3000/api/v1` para `http://localhost:8084/v1` ou use um proxy.
3. Ajuste o token/headers de autenticação conforme o esquema do Cloud API.

### Se você quiser adicionar o Local API faltante

- Adicione a pasta `dispenser-local-api-main` ao workspace com o código do Local API.
- Garanta a sincronização de `RABBITMQ` entre Local e Cloud.
- Atualize a documentação para refletir os dois projetos e o broker RabbitMQ.
