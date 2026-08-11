# FieldOps — Plataforma de Inspeção em Campo

> Plataforma digital para planejamento, execução, acompanhamento e revisão de inspeções técnicas em campo, com operação **offline-first** e fluxo integrado entre app mobile, painel administrativo e API central.

---

## 📋 Sobre o projeto

O **FieldOps** substitui processos baseados em formulários impressos, planilhas e mensagens informais por um fluxo digital único e rastreável.

- **Técnicos** usam um **app mobile** para consultar inspeções atribuídas, identificar equipamentos por QR Code, responder checklists, registrar evidências (fotos), localização e não conformidades — inclusive **sem conexão com a internet**.
- **Supervisores e administradores** usam uma **interface web administrativa** para cadastrar dados, criar modelos de inspeção versionados, agendar e atribuir inspeções, acompanhar a execução e aprovar ou reprovar resultados.
- Uma **API REST** centraliza autenticação, autorização, regras de negócio, persistência, sincronização e auditoria.

### Problema que resolve

Inspeções de campo hoje dependem de formulários em papel, planilhas isoladas e fotos sem identificação — gerando retrabalho, perda de dados, baixa rastreabilidade e dificuldade de auditoria. O FieldOps conecta planejamento → execução → revisão em um único fluxo confiável, mesmo em locais sem conectividade.

---

## 🧩 Componentes da solução

| Componente | Tecnologia | Responsabilidade principal |
|---|---|---|
| **App Mobile** | Expo + React Native + TypeScript | Execução das inspeções em campo, offline-first |
| **Interface Administrativa** | Angular + TypeScript | Cadastros, modelos, planejamento, revisão e aprovação |
| **API REST** | Java + Spring Boot | Regras de negócio, autenticação, persistência, sincronização |
| **Infraestrutura de dados** | PostgreSQL + SQLite + Object Storage | Dados centrais, dados locais e armazenamento de evidências |

```
┌─────────────────────────────┐
│ App Mobile (Expo + RN)      │
│ SQLite + Secure Store       │
└──────────────┬──────────────┘
               │ HTTPS / JSON / Multipart
               ▼
┌─────────────────────────────┐
│ API REST (Spring Boot)      │
│ Segurança + Regras          │
└───────┬──────────────┬──────┘
        ▼              ▼
┌──────────────┐  ┌──────────────────┐
│ PostgreSQL   │  │ Storage de       │
│              │  │ evidências       │
└──────────────┘  └──────────────────┘
        ▲
        │ HTTPS / JSON
┌───────┴─────────────────────┐
│ Interface Administrativa    │
│ (Angular)                   │
└──────────────────────────────┘
```

---

## 🚀 Escopo do MVP

O MVP demonstra o fluxo completo ponta a ponta:

1. Administrador cadastra dados básicos (usuários, clientes, locais, equipamentos).
2. Supervisor cria e publica um modelo de inspeção.
3. Supervisor agenda a inspeção e atribui a um técnico.
4. Técnico recebe a inspeção no app e a executa **offline**, com fotos, QR Code e localização.
5. App sincroniza os dados pendentes de forma **idempotente** (sem duplicidade).
6. Supervisor revisa e **aprova ou reprova** o resultado.

### Fora do escopo do MVP
Portal público do cliente, multiempresa, faturamento, roteirização automática, chamadas de vídeo, assinatura eletrônica com validade jurídica, integração com ERPs/IoT, IA para diagnóstico automático.

---

## 👥 Personas

| Persona | Perfil | Objetivo principal |
|---|---|---|
| **Carlos** | Técnico de campo | Executar inspeções rapidamente, mesmo sem internet |
| **Marina** | Supervisora | Planejar, acompanhar e revisar inspeções |
| **Ana** | Administradora | Manter cadastros e usuários sem acesso direto ao banco |
| **Roberto** | Responsável do cliente | Consultar resultados e não conformidades (futuro) |

## 🔐 Perfis e permissões

`ADMIN` · `SUPERVISOR` · `TECHNICIAN` · `CLIENT_VIEWER` (futuro)

A autorização é sempre validada **no backend** — a interface nunca é o único mecanismo de segurança. Técnicos só acessam suas próprias inspeções; supervisores atuam em seu escopo operacional.

---

## 🔄 Fluxo e estados da inspeção

```
RASCUNHO → ATRIBUÍDA → EM_ANDAMENTO → ENVIADA → EM_REVISÃO → APROVADA
                                                          ↘ REPROVADA → EM_ANDAMENTO (correção)
                                          (Supervisor/Admin) → CANCELADA
```

> ⚠️ O **estado de negócio** da inspeção é sempre separado do **estado local de sincronização** (`SINCRONIZADO`, `PENDENTE`, `SINCRONIZANDO`, `ERRO`, `CONFLITO`).

---

## 🏗️ Modelo de dados (resumo)

Principais entidades: `User`, `Client`, `InspectionSite`, `Equipment`, `InspectionTemplate` / `InspectionTemplateVersion`, `TemplateSection` / `TemplateItem`, `Inspection`, `InspectionItemSnapshot`, `InspectionResponse`, `Evidence`, `NonConformity`, `InspectionReview`, `AuditEvent`, `SyncOperation`, `SyncMetadata`.

Decisões centrais:
- Identificadores **UUID** em todas as entidades sincronizáveis.
- Modelos de inspeção são **versionados de forma imutável**; toda alteração gera nova versão.
- Cada inspeção preserva um **snapshot** dos itens no momento da execução (garante fidelidade histórica).
- Controle de concorrência **otimista** (campo `version`).
- Exclusão lógica (inativação) em vez de exclusão física para entidades históricas.

---

## 📡 API REST

- Prefixo: `/api/v1`
- Formato: JSON · Upload: `multipart/form-data`
- Autenticação: Bearer Token (JWT)
- Documentação: OpenAPI / Swagger
- Erros padronizados (`timestamp`, `status`, `code`, `message`, `fieldErrors`)
- Sincronização mobile via `POST /api/v1/mobile/sync` (operações idempotentes com `operationId`)

Principais grupos de endpoints: `auth`, `users`, `clients`, `sites`, `equipment`, `inspection-templates`, `inspections`, `responses`, `evidence`, `non-conformities`, `reviews`, `mobile/sync`, `dashboard`.

---

## 🛠️ Stack tecnológica

**App Mobile**
- Expo + React Native + TypeScript + Expo Router
- TanStack Query (dados remotos) · SQLite (offline) · Zustand/Context (estado de UI)
- React Hook Form + Zod (formulários e validação)
- Secure Store (sessão) · Câmera, QR Code e Geolocalização nativos

**Interface Administrativa**
- Angular + TypeScript, rotas protegidas por guards, formulários reativos

**API**
- Java + Spring Boot, JPA + PostgreSQL, JWT, migrações versionadas, OpenAPI

---

## 📂 Estrutura sugerida do repositório

```
fieldops/
├── mobile/          # App Expo + React Native
│   └── src/app | features | components | domain | infrastructure
├── admin-web/        # Interface administrativa Angular
│   └── src/app/core | shared | features
├── api/              # API Spring Boot
│   └── com.fieldops/ auth | user | client | site | equipment
│                      template | inspection | evidence
│                      synchronization | review | audit | shared
└── docs/              # OpenAPI, diagramas, backlog, decisões
```

---

## ▶️ Como executar localmente



**API (Spring Boot)**
```bash
cd api
./mvnw spring-boot:run
```

**Interface Administrativa (Angular)**
```bash
cd admin-web
npm install
ng serve
```

**App Mobile (Expo)**
```bash
cd mobile
npm install
npx expo start
# Escaneie o QR Code com o Expo Go ou rode em emulador Android
```

Cada ambiente (local, integração, teste, produção acadêmica) deve possuir URL própria da API, configuração de banco, armazenamento e credenciais separadas.

---

## 🗺️ Roadmap (16 semanas / 8 sprints)

| Sprint | Semanas | Incremento demonstrável |
|---|---|---|
| 1 | 1–2 | Três aplicações executáveis com contrato inicial |
| 2 | 3–4 | Autenticação integrada e cadastros disponíveis |
| 3 | 5–6 | Supervisor cria e atribui uma inspeção |
| 4 | 7–8 | Técnico executa uma inspeção online |
| 5 | 9–10 | Inspeção online com recursos nativos (foto, QR, GPS) |
| 6 | 11–12 | Inspeção concluída offline e sincronizada |
| 7 | 13–14 | Ciclo completo com reprovação e correção |
| 8 | 15–16 | Produto integrado apresentado ponta a ponta |

---

## ✅ Critérios de aceitação e qualidade

- Cada funcionalidade possui cenários de aceitação (`Dado / Quando / Então`) cobrindo caminho feliz e exceções.
- **Definition of Done** em camadas: história → mobile/admin/API → sprint → MVP.
- Regras de sincronização **idempotentes**: reenvio de uma operação nunca duplica dados.
- Nenhum dado local pendente é descartado silenciosamente em caso de falha ou cancelamento remoto.

---

## 👨‍💻 Equipe

Giovana Rafaela · Juliana Nascimento · Maria Eduarda · Milenna Portella

---

## 📄 Licença

Projeto de caráter acadêmico, desenvolvido para fins de aprendizado integrado em desenvolvimento mobile, backend, frontend web e engenharia de software.
