# Plano de Arquitetura Tecnica - PAT

**Projeto**: SGMD - Servico de Gerenciamento de Mudancas
**Versao**: 1.0
**Data**: Abril de 2026
**Classificacao**: Uso Interno - SGE/AGU
**Responsavel Tecnico**: Hyago Keller - Gerente de Projetos / Especialista Tecnico ITSM

---

## 1. Resumo Executivo

Este documento descreve a arquitetura tecnica do SGMD, incluindo a infraestrutura de producao, stack tecnologico, integracao com o InvGate Service Management, modelo de dados, seguranca e procedimentos operacionais.

---

## 2. Infraestrutura de Producao

### 2.1 Servidor

| Parametro | Valor |
|---|---|
| **Hostname** | SDF1066 |
| **IP** | 10.208.0.114 |
| **Sistema Operacional** | Windows Server 2025 |
| **RAM** | 16 GB |
| **Armazenamento** | SSD 249 GB |
| **vCPUs** | 4 |
| **Dominio** | sgmd.agu.gov.br |
| **Certificado SSL** | Sim (validade ate 2032) |
| **Disponibilidade** | 24x7 (SLA 100%) |
| **Backup** | Nao se aplica |

### 2.2 Diagrama de Rede

```
                        Internet
                           |
                     [Firewall AGU]
                           |
                    [Proxy Reverso]
                    sgmd.agu.gov.br
                    SSL (ate 2032)
                           |
              ┌────────────┼────────────┐
              |            |            |
         [Frontend]   [Backend]   [MongoDB]
         React App    FastAPI     Banco Local
         Porta 3000   Porta 8001  Porta 27017
              |            |            |
              └────────────┼────────────┘
                           |
                    SDF1066 (10.208.0.114)
                    Windows Server 2025
                           |
              ┌────────────┘
              |
    [InvGate Service Management]
    aguservicos.agu.gov.br
    POST /api/changes (Basic Auth)
```

### 2.3 Portas e Servicos

| Servico | Porta | Protocolo | Acesso |
|---|---|---|---|
| **Frontend (React)** | 3000 | HTTP | Via proxy reverso (HTTPS externo) |
| **Backend (FastAPI)** | 8001 | HTTP | Via proxy reverso, prefixo `/api` |
| **MongoDB** | 27017 | TCP | Local apenas (127.0.0.1) |
| **HTTPS (externo)** | 443 | HTTPS | Publico (sgmd.agu.gov.br) |

---

## 3. Stack Tecnologico

### 3.1 Frontend

| Tecnologia | Versao | Finalidade |
|---|---|---|
| **React.js** | 18.x | Framework de interface |
| **Tailwind CSS** | 3.x | Estilizacao utilitaria |
| **Shadcn/UI** | -- | Componentes de interface |
| **Lucide React** | -- | Biblioteca de icones |
| **Design System** | Gov.br | Identidade visual do Governo Federal |

### 3.2 Backend

| Tecnologia | Versao | Finalidade |
|---|---|---|
| **Python** | 3.11+ | Linguagem principal |
| **FastAPI** | 0.100+ | Framework REST API |
| **Motor** | 3.x | Driver assincrono MongoDB |
| **PyJWT** | 2.x | Autenticacao JWT |
| **bcrypt** | 4.x | Hash de senhas |
| **Uvicorn** | 0.20+ | Servidor ASGI |

### 3.3 Banco de Dados

| Tecnologia | Versao | Finalidade |
|---|---|---|
| **MongoDB** | 7.x | Banco de dados NoSQL |

### 3.4 Infraestrutura

| Tecnologia | Finalidade |
|---|---|
| **Windows Server 2025** | Sistema operacional do servidor |
| **IIS / Proxy Reverso** | Roteamento HTTPS -> servicos internos |
| **Git** | Controle de versao |

---

## 4. Arquitetura da Aplicacao

### 4.1 Visao Geral

```
┌─────────────────────────────────────────────────────┐
│                    CLIENTE (Browser)                  │
│                                                       │
│  React SPA (Gov.br Design)                           │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐             │
│  │Dashboard │ │Calendario│ │  Modal   │              │
│  │ Page     │ │  Grid    │ │ Mudanca  │              │
│  └──────────┘ └──────────┘ └──────────┘              │
│                    |                                  │
│             REACT_APP_BACKEND_URL                     │
└─────────────────────|─────────────────────────────────┘
                      | HTTPS (443)
                      v
┌─────────────────────────────────────────────────────┐
│              PROXY REVERSO (sgmd.agu.gov.br)         │
│                                                       │
│   /api/*  ──────>  Backend (FastAPI :8001)            │
│   /*      ──────>  Frontend (React :3000)             │
└─────────────────────|─────────────────────────────────┘
                      |
┌─────────────────────|─────────────────────────────────┐
│              BACKEND (FastAPI)                         │
│                                                       │
│  ┌────────────────────────────────────────┐           │
│  │         Autenticacao Dual              │           │
│  │  ┌──────────┐    ┌─────────────────┐  │           │
│  │  │JWT Cookie│    │HTTP Basic Auth  │  │           │
│  │  │(Frontend)│    │(InvGate ITSM)   │  │           │
│  │  └──────────┘    └─────────────────┘  │           │
│  └────────────────────────────────────────┘           │
│                      |                                │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐              │
│  │Auth API  │ │Changes   │ │Export    │              │
│  │/api/auth │ │/api/     │ │/api/     │              │
│  │          │ │changes   │ │changes/  │              │
│  │          │ │          │ │export/csv│              │
│  └──────────┘ └──────────┘ └──────────┘              │
│                      |                                │
│               Motor (Async Driver)                    │
└─────────────────────|─────────────────────────────────┘
                      |
┌─────────────────────|─────────────────────────────────┐
│              MONGODB (localhost:27017)                 │
│                                                       │
│  Database: sgmd                                       │
│  ┌──────────┐        ┌──────────────────┐            │
│  │  users   │        │     changes      │            │
│  │          │        │                  │            │
│  │ email    │        │ titulo           │            │
│  │ password │        │ frente_atuacao   │            │
│  │ name     │        │ natureza_mudanca │            │
│  │ role     │        │ categoria_mudanca│            │
│  │          │        │ risco            │            │
│  │          │        │ status           │            │
│  │          │        │ data_inicio/fim  │            │
│  └──────────┘        └──────────────────┘            │
└───────────────────────────────────────────────────────┘
```

### 4.2 Fluxo de Integracao InvGate

```
┌───────────────────────────────────────────────────┐
│            InvGate Service Management              │
│            aguservicos.agu.gov.br                  │
│                                                    │
│  Workflow:                                         │
│  1. [Inicio] Formulario                           │
│       |                                            │
│  2. [Web Service] Integrations                    │
│       | POST https://sgmd.agu.gov.br/api/changes  │
│       | Authorization: Basic (SGMD-Integracao)     │
│       | Content-Type: application/json             │
│       | Body: {titulo, responsavel_negocio,        │
│       |        sistemas_afetados, data_inicio,     │
│       |        data_fim, categoria_mudanca}        │
│       |                                            │
│  3. [Execucao da Mudanca INFRA] Tarefas           │
│       |                                            │
│  4. [E-mail com relatorio de MUD]                 │
└───────────────────────|───────────────────────────┘
                        |
                        v
┌───────────────────────────────────────────────────┐
│              SGMD Backend (FastAPI)                │
│                                                    │
│  verify_basic_auth()                               │
│    -> Valida INVGATE_USER + INVGATE_PASSWORD       │
│    -> created_by = "InvGate ITSM"                  │
│    -> Insere no MongoDB                            │
│    -> Retorna HTTP 201                             │
│                                                    │
│  * Nenhum token e gerado                           │
│  * Autenticacao stateless                          │
└───────────────────────────────────────────────────┘
```

---

## 5. Modelo de Dados

### 5.1 Colecao `users`

| Campo | Tipo | Descricao |
|---|---|---|
| `_id` | ObjectId | Identificador interno MongoDB |
| `email` | String (unique) | E-mail do usuario |
| `password_hash` | String | Senha com hash bcrypt |
| `name` | String | Nome completo |
| `role` | String | `admin` ou `user` |
| `created_at` | String (ISO 8601) | Data de criacao |

### 5.2 Colecao `changes`

| Campo | Tipo | Obrigatorio | Descricao |
|---|---|---|---|
| `id` | String (UUID v4) | Sim (auto) | Identificador unico |
| `titulo` | String | Sim | Titulo da mudanca |
| `descricao` | String | Nao | Descricao detalhada |
| `frente_atuacao` | Enum | Nao (default: `sistemas`) | `infraestrutura`, `sistemas`, `supersapiens` |
| `natureza_mudanca` | Enum | Nao (default: `planejada_normal`) | `planejada_normal`, `baixo_risco`, `emergencial` |
| `categoria_mudanca` | Enum | Nao (default: `corretiva`) | `novo_servico`, `preventiva`, `adaptativa`, `corretiva`, `evolutiva`, `desativacao`, `deploy`, `teste_vulnerabilidade` |
| `status` | Enum | Nao (default: `planejada`) | `planejada`, `aprovada`, `em_execucao`, `concluida`, `cancelada` |
| `risco` | Enum | Nao (default: `medio`) | `alto`, `medio`, `baixo` |
| `resultado_conclusao` | Enum | Condicional | `sucesso`, `sucesso_ressalvas`, `sem_sucesso` |
| `numero_rfc` | String | Nao | Codigo RFC |
| `responsavel_negocio` | String | Nao | Responsavel do negocio |
| `sistemas_afetados` | String | Nao | Sistemas impactados |
| `servicos_impactados` | String | Nao | Servicos afetados |
| `justificativa` | String | Nao | Motivo da mudanca |
| `plano_rollback` | String | Nao | Plano de reversao |
| `ambiente_homologado` | Enum | Nao (default: `nao_se_aplica`) | `sim`, `nao`, `nao_se_aplica` |
| `versao_sistema` | String | Nao | Versao do sistema |
| `data_inicio` | String (datetime) | Sim | Data/hora de inicio |
| `data_fim` | String (datetime) | Nao | Data/hora de termino |
| `created_by` | String | Sim (auto) | Nome do criador ou "InvGate ITSM" |
| `created_at` | String (ISO 8601) | Sim (auto) | Timestamp de criacao |
| `updated_at` | String (ISO 8601) | Sim (auto) | Timestamp de atualizacao |

### 5.3 Indices

| Colecao | Campo | Tipo | Finalidade |
|---|---|---|---|
| `users` | `email` | Unique | Garantir unicidade de e-mail |
| `changes` | `id` | -- | Busca por identificador |
| `changes` | `data_inicio` | -- | Filtragem por periodo |

---

## 6. API REST

### 6.1 Endpoints

| Metodo | Endpoint | Autenticacao | Descricao |
|---|---|---|---|
| `POST` | `/api/auth/login` | Publica | Login (retorna JWT) |
| `POST` | `/api/auth/register` | Publica | Registro de usuario |
| `GET` | `/api/auth/me` | JWT | Dados do usuario logado |
| `POST` | `/api/auth/logout` | JWT | Logout (limpa cookies) |
| `GET` | `/api/changes` | JWT ou Basic Auth | Lista todas as mudancas |
| `POST` | `/api/changes` | JWT ou Basic Auth | Cria nova mudanca |
| `PUT` | `/api/changes/{id}` | JWT ou Basic Auth | Atualiza mudanca |
| `DELETE` | `/api/changes/{id}` | JWT ou Basic Auth | Exclui mudanca |
| `GET` | `/api/changes/export/csv` | JWT ou Basic Auth | Exporta mudancas em CSV |

### 6.2 Autenticacao Dual

```
Requisicao recebida
       |
       v
  Header Authorization?
       |
  ┌────┴────┐
  |         |
Basic?    Bearer/Cookie?
  |         |
  v         v
Valida     Valida
INVGATE_   JWT Token
USER/PASS
  |         |
  v         v
OK: user   OK: user
"InvGate   {nome do
 ITSM"     usuario}
  |         |
  └────┬────┘
       |
       v
  Nenhum valido?
       |
       v
  HTTP 401
```

---

## 7. Seguranca

### 7.1 Autenticacao

| Aspecto | Implementacao |
|---|---|
| **Usuarios do SGMD** | JWT (JSON Web Token) com expiracao de 60 minutos |
| **Integracao InvGate** | HTTP Basic Auth (credenciais em variaveis de ambiente) |
| **Hash de Senhas** | bcrypt com salt automatico |
| **Cookies** | HttpOnly, SameSite=Lax |
| **Comparacao de Credenciais** | `secrets.compare_digest` (resistente a timing attacks) |

### 7.2 Comunicacao

| Aspecto | Implementacao |
|---|---|
| **HTTPS** | Certificado SSL valido ate 2032 |
| **CORS** | Configurado para origens permitidas |
| **MongoDB** | Acesso local apenas (localhost:27017) |

### 7.3 Variaveis de Ambiente

| Variavel | Descricao | Sensivel |
|---|---|---|
| `MONGO_URL` | String de conexao MongoDB | Sim |
| `DB_NAME` | Nome do banco de dados | Nao |
| `JWT_SECRET` | Chave secreta para assinatura JWT | Sim |
| `ADMIN_EMAIL` | E-mail do administrador padrao | Nao |
| `ADMIN_PASSWORD` | Senha do administrador padrao | Sim |
| `INVGATE_BASE_URL` | URL base do InvGate | Nao |
| `INVGATE_USER` | Usuario para HTTP Basic Auth | Sim |
| `INVGATE_PASSWORD` | Senha para HTTP Basic Auth | Sim |
| `CORS_ORIGINS` | Origens CORS permitidas | Nao |
| `FRONTEND_URL` | URL do frontend | Nao |

> **IMPORTANTE**: Todas as credenciais sao armazenadas exclusivamente em variaveis de ambiente. Nenhuma credencial e hardcoded no codigo-fonte.

---

## 8. Estrutura de Diretorios

```
/sgmd
├── backend/
│   ├── .env                    # Variaveis de ambiente (NAO versionado)
│   ├── requirements.txt        # Dependencias Python
│   └── server.py               # API FastAPI (auth, CRUD, integracao)
├── docs/
│   ├── HLD_SGMD_Arquitetura.md           # High-Level Design
│   ├── INTEGRACAO_INVGATE_ORIENTACOES.md  # Guia de Integracao InvGate
│   ├── PGP_SGMD.md                       # Plano de Gerenciamento do Projeto
│   └── PAT_SGMD.md                       # Plano de Arquitetura Tecnica (este)
├── frontend/
│   ├── .env                    # Variaveis frontend (REACT_APP_BACKEND_URL)
│   ├── package.json            # Dependencias Node.js
│   ├── public/
│   │   ├── index.html          # HTML base
│   │   ├── aguservicos-logo.png    # Logo AGU Servicos
│   │   └── supersapiens-logo.png   # Logo SuperSapiens
│   └── src/
│       ├── App.js              # Roteamento principal
│       ├── App.css             # Estilos globais
│       ├── components/
│       │   ├── CalendarGrid.jsx    # Calendario interativo
│       │   ├── ChangeList.jsx      # Lista de mudancas
│       │   ├── ChangeModal.jsx     # Modal de criacao/edicao
│       │   ├── ConflictAlert.jsx   # Alertas de conflito
│       │   ├── GovHeader.jsx       # Header Gov.br
│       │   ├── Legend.jsx          # Legenda do calendario
│       │   ├── MetricsCards.jsx    # Cards de metricas
│       │   └── SuperSapiensIcon.jsx # Icone SuperSapiens
│       ├── contexts/
│       │   └── AuthContext.jsx     # Contexto de autenticacao
│       └── pages/
│           ├── DashboardPage.jsx   # Pagina principal
│           └── LoginPage.jsx       # Pagina de login
└── memory/
    ├── PRD.md                  # Product Requirements Document
    └── test_credentials.md     # Credenciais de teste
```

---

## 9. Procedimentos Operacionais

### 9.1 Deploy

```bash
# 1. Clonar/atualizar repositorio
git pull origin main

# 2. Backend
cd backend
pip install -r requirements.txt
# Configurar .env com variaveis de producao

# 3. Frontend
cd frontend
yarn install
yarn build
# O build gera a pasta build/ com os arquivos estaticos

# 4. Reiniciar servicos
# Reiniciar o backend (uvicorn) e o servidor de arquivos estaticos
```

### 9.2 Monitoramento

| Item | Como Verificar |
|---|---|
| **Backend rodando** | `curl https://sgmd.agu.gov.br/api/auth/me` (deve retornar 401) |
| **Frontend rodando** | Acessar `https://sgmd.agu.gov.br` no browser |
| **MongoDB rodando** | `mongosh --eval "db.runCommand({ping:1})"` |
| **Integracao InvGate** | Verificar logs do backend para requests com `created_by: InvGate ITSM` |

### 9.3 Logs

| Log | Localizacao | Conteudo |
|---|---|---|
| **Backend** | stdout/stderr do processo uvicorn | Requisicoes HTTP, erros, startup |
| **MongoDB** | Log padrao do MongoDB | Operacoes de banco |

---

## 10. Capacidade e Dimensionamento

### 10.1 Estimativas

| Parametro | Estimativa | Justificativa |
|---|---|---|
| **Usuarios simultaneos** | 10-30 | Equipes de TI da SGE |
| **Mudancas por mes** | 50-200 | Volume tipico de RFCs |
| **Tamanho medio por documento** | ~1 KB | Campos de texto e enums |
| **Armazenamento estimado (1 ano)** | ~50 MB | 2400 mudancas x 1KB + indices |
| **Uso de memoria (Backend)** | ~200 MB | FastAPI + Motor async |
| **Uso de memoria (Frontend build)** | ~50 MB | React SPA estatico |
| **Uso de memoria (MongoDB)** | ~500 MB | Dados + cache + indices |

### 10.2 Recursos do Servidor vs Demanda

| Recurso | Disponivel | Demanda Estimada | Margem |
|---|---|---|---|
| **RAM** | 16 GB | ~1 GB (total) | Ampla |
| **vCPUs** | 4 | 1-2 (media) | Suficiente |
| **SSD** | 249 GB | ~5 GB (1 ano) | Ampla |

---

## 11. Evolucoes Futuras

| Prioridade | Item | Descricao |
|---|---|---|
| **P1** | Mapeamento visual de campos | Interface para mapear campos SGMD x InvGate |
| **P2** | Dashboard analitico avancado | Graficos de tendencia, metricas historicas |
| **P2** | Notificacoes | Alertas de mudancas proximas por e-mail |
| **P2** | Permissoes granulares | Controle de acesso por frente de atuacao |
| **P3** | Sync bidirecional | SGMD -> InvGate (enviar mudancas criadas localmente) |
| **P3** | App mobile | Interface responsiva ou PWA |

---

## 12. Aprovacoes

| Papel | Nome | Data | Assinatura |
|---|---|---|---|
| **Responsavel Tecnico** | Hyago Keller | ___/___/2026 | _______________ |
| **Patrocinador** | Uendel Tavares | ___/___/2026 | _______________ |

---

*Documento elaborado como parte do projeto SGMD.*
*SGE - Secretaria de Governanca Estrategica | Advocacia-Geral da Uniao.*
