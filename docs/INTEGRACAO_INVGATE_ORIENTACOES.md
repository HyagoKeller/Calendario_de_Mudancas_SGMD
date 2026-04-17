# Orientacoes para Integracao InvGate Service Management x SGMD

**Documento**: Guia de Integracao ITSM
**Sistema**: SGMD - Servico de Gerenciamento de Mudancas
**Data**: Abril de 2026 (Atualizado: 17/04/2026)
**Classificacao**: Uso Interno - DTI
**Objetivo**: Orientar os analistas sobre a integracao entre o SGMD e o InvGate Service Management

---

## 1. Visao Geral da Integracao

O InvGate Service Management envia mudancas automaticamente para o SGMD atraves de um **Web Service** configurado no workflow. Quando um ticket de mudanca e criado/aprovado no InvGate, os dados sao enviados via POST para o SGMD e aparecem no calendario.

### 1.1 Fluxo da Integracao (Implementado)

```
InvGate Service Management
   Workflow:
     1. Inicio (Formulario)
     2. Web Service (Integrations)  ──────┐
     3. Execucao da Mudanca INFRA         |
     4. E-mail com relatorio de MUD       |
                                          |
                                          v
              POST https://sgmd.agu.gov.br/api/changes
              Authorization: Basic (SGMD-Integracao)
              Content-Type: application/json
                                          |
                                          v
                                Backend SGMD (FastAPI)
                                          |
                                          v
                                MongoDB (changes) -> Calendario
```

### 1.2 Direcao da Sincronizacao

| Direcao | Status | Descricao |
|---|---|---|
| **InvGate -> SGMD** | ATIVO | Mudancas do InvGate sao enviadas automaticamente via Web Service |
| **SGMD -> InvGate** | Futuro | Mudancas criadas no SGMD podem ser enviadas ao InvGate |

---

## 2. Metodo de Autenticacao - HTTP Basic Auth

### IMPORTANTE - Mudancas em relacao a versao anterior

| Item | Versao Anterior | Versao Atual |
|---|---|---|
| **Autenticacao InvGate** | OAuth2 Bearer Token (gerava token a cada requisicao) | HTTP Basic Auth (credenciais fixas) |
| **Token por requisicao** | Sim (problema: cada request gerava um token novo) | Nao. Credenciais enviadas direto no header |
| **Configuracao no InvGate** | Manual (token URL, client_id, client_secret) | Credenciais salvas: SGMD-Integracao |

### 2.1 Configuracao no InvGate (Web Service)

Na etapa **Web Service** do workflow:

| Campo | Valor |
|---|---|
| **Endereco URL** | `https://sgmd.agu.gov.br/api/changes` |
| **Metodo** | POST |
| **Modo de autenticacao** | Credenciais salvas |
| **Credencial** | SGMD-Integracao |
| **Formato de envio de dados** | JSON |

### 2.2 Credenciais Globais no InvGate

Acesse: **Integracoes > Credenciais globais > SGMD-Integracao**

| Campo | Valor |
|---|---|
| **Alias** | SGMD-Integracao |
| **Autenticacao** | HTTP (Credenciais basicas) |
| **Usuario** | Keller |
| **Senha** | (configurada no InvGate) |

### 2.3 Como Funciona

O InvGate envia automaticamente o header HTTP:
```
Authorization: Basic base64(Keller:senha)
```

O SGMD verifica essas credenciais contra as variaveis de ambiente `INVGATE_USER` e `INVGATE_PASSWORD`. Nenhum token e gerado. A autenticacao e stateless.

---

## 3. Autenticacao Dual no Backend SGMD

O backend aceita **dois metodos** de autenticacao nos endpoints `/api/changes`:

| Metodo | Quem Usa | Como Funciona |
|---|---|---|
| **HTTP Basic Auth** | InvGate (Web Service) | Header `Authorization: Basic base64(user:pass)` — verificado primeiro |
| **JWT (Cookie/Bearer)** | Usuarios logados no frontend | Token JWT via cookie ou header `Authorization: Bearer <token>` |

- Se Basic Auth for valido → mudanca registrada com `created_by = "InvGate ITSM"`
- Se JWT for valido → mudanca registrada com `created_by = nome do usuario`
- Se nenhum for valido → HTTP 401 "Nao autenticado"

---

## 4. Mapeamento de Campos (InvGate -> SGMD)

### 4.1 Campos Configurados no Web Service (Envio de variaveis)

Estes campos ja estao mapeados no workflow do InvGate:

| Campo SGMD | Variavel InvGate | Tipo |
|---|---|---|
| `titulo` | Inicio - Titulo da Mudanca | Texto |
| `responsavel_negocio` | Inicio - Responsavel do Negocio | Texto |
| `sistemas_afetados` | Inicio - Sistemas ou Areas Impactadas | Texto |
| `data_inicio` | Inicio - Data e Hora de Inicio (Data e hora DMY) | Data/Hora |
| `data_fim` | Inicio - Data e Hora de Finalizacao (Data e hora DMY) | Data/Hora |
| `categoria_mudanca` | Categoria da Mudanca | Lista |

### 4.2 Campos com Valor Padrao

Campos que o SGMD preenche automaticamente quando nao enviados pelo InvGate:

| Campo SGMD | Valor Padrao | Pode ser adicionado ao InvGate? |
|---|---|---|
| `status` | `planejada` | Sim, via Envio de variaveis |
| `frente_atuacao` | `sistemas` | Sim, valores: `infraestrutura`, `sistemas`, `supersapiens` |
| `natureza_mudanca` | `planejada_normal` | Sim, valores: `planejada_normal`, `baixo_risco`, `emergencial` |
| `risco` | `medio` | Sim, valores: `alto`, `medio`, `baixo` |
| `plano_rollback` | (vazio) | Sim |
| `numero_rfc` | (vazio) | Sim |
| `justificativa` | (vazio) | Sim |
| `descricao` | (vazio) | Sim |

### 4.3 Campos Automaticos (gerados pelo SGMD)

| Campo | Descricao |
|---|---|
| `id` | UUID gerado automaticamente |
| `created_at` | Timestamp de criacao (UTC) |
| `updated_at` | Timestamp de atualizacao (UTC) |
| `created_by` | "InvGate ITSM" (quando via Basic Auth) |

### 4.4 Valores Aceitos para Campos de Lista

**categoria_mudanca:**
`novo_servico`, `preventiva`, `adaptativa`, `corretiva`, `evolutiva`, `desativacao`, `deploy`, `teste_vulnerabilidade`

**frente_atuacao:**
`infraestrutura`, `sistemas`, `supersapiens`

**natureza_mudanca:**
`planejada_normal`, `baixo_risco`, `emergencial`

**risco:**
`alto`, `medio`, `baixo`

**status:**
`planejada`, `aprovada`, `em_execucao`, `concluida`, `cancelada`

**ambiente_homologado:**
`sim`, `nao`, `nao_se_aplica`

**resultado_conclusao** (quando status = concluida):
`sucesso`, `sucesso_ressalvas`, `sem_sucesso`

---

## 5. Exemplo de Payload JSON

Exemplo do JSON que o InvGate envia ao SGMD:

```json
{
  "titulo": "Deploy SAPIENS v4.3 - Atualizacao de Modulo",
  "responsavel_negocio": "João Silva",
  "sistemas_afetados": "SAPIENS, SEI",
  "data_inicio": "2026-04-20T10:00",
  "data_fim": "2026-04-20T14:00",
  "categoria_mudanca": "deploy"
}
```

Campos opcionais que podem ser adicionados:
```json
{
  "titulo": "Deploy SAPIENS v4.3 - Atualizacao de Modulo",
  "responsavel_negocio": "João Silva",
  "sistemas_afetados": "SAPIENS, SEI",
  "data_inicio": "2026-04-20T10:00",
  "data_fim": "2026-04-20T14:00",
  "categoria_mudanca": "deploy",
  "frente_atuacao": "supersapiens",
  "natureza_mudanca": "planejada_normal",
  "risco": "alto",
  "numero_rfc": "RFC-2026-0150",
  "plano_rollback": "Reverter para versao 4.2",
  "justificativa": "Correcao de vulnerabilidade critica"
}
```

---

## 6. Variaveis de Ambiente no Servidor SGMD

Variaveis adicionadas no `backend/.env` para a integracao:

| Variavel | Valor | Descricao |
|---|---|---|
| `INVGATE_BASE_URL` | `https://aguservicos.agu.gov.br` | URL base do InvGate |
| `INVGATE_USER` | `Keller` | Usuario para HTTP Basic Auth |
| `INVGATE_PASSWORD` | `JxKR2GVQIFyBW3Um` | Senha para HTTP Basic Auth |

> Essas variaveis devem estar configuradas no servidor de producao (`sgmd.agu.gov.br`).

---

## 7. Diferencas da Build (Ontem vs Hoje)

### 7.1 Arquivos Alterados

| Arquivo | O que Mudou |
|---|---|
| `backend/server.py` | Adicionada autenticacao dual (Basic Auth + JWT). Funcoes `verify_basic_auth()` e `get_user_or_integration()`. Campos `plano_rollback` e `categoria_mudanca` agora opcionais. Imports: `base64`, `secrets` |
| `backend/.env` | Adicionadas variaveis `INVGATE_BASE_URL`, `INVGATE_USER`, `INVGATE_PASSWORD` |
| `frontend/public/index.html` | Sem mudanca funcional (badge Emergent pode ser removido no deploy) |
| `frontend/public/aguservicos-logo.png` | Logo AGU Servicos adicionada |
| `frontend/src/pages/LoginPage.jsx` | Logo AGU centralizada no formulario, texto "Bem-vindo(a)...", placeholder `seuemail@agu.gov.br` |
| `frontend/src/components/GovHeader.jsx` | Logo AGU na barra azul, link clicavel sem negrito |
| `frontend/src/components/CalendarGrid.jsx` | Borda vermelha + tag "EMERGENCIAL" (mantendo cor do status) |

### 7.2 O que Fazer no Deploy

1. **Atualizar `backend/.env`** com as 3 novas variaveis do InvGate
2. **Copiar `aguservicos-logo.png`** para `frontend/public/`
3. **Rebuildar o frontend** (as mudancas de JSX precisam de build)
4. **Reiniciar o backend** (mudancas no server.py + novas variaveis .env)

---

## 8. Teste da Integracao

### 8.1 Teste via curl (simula o InvGate)

```bash
curl -X POST https://sgmd.agu.gov.br/api/changes \
  -u "Keller:JxKR2GVQIFyBW3Um" \
  -H "Content-Type: application/json" \
  -d '{
    "titulo": "Teste Integracao InvGate",
    "responsavel_negocio": "Analista Teste",
    "sistemas_afetados": "SAPIENS",
    "data_inicio": "2026-04-25T10:00",
    "data_fim": "2026-04-25T12:00",
    "categoria_mudanca": "deploy"
  }'
```

**Resposta esperada** (HTTP 201):
```json
{
  "titulo": "Teste Integracao InvGate",
  "status": "planejada",
  "created_by": "InvGate ITSM",
  ...
}
```

### 8.2 Teste de Seguranca

```bash
# Sem autenticacao (deve retornar 401)
curl -X POST https://sgmd.agu.gov.br/api/changes \
  -H "Content-Type: application/json" \
  -d '{"titulo":"Teste","data_inicio":"2026-04-25T10:00"}'

# Senha errada (deve retornar 401)
curl -X POST https://sgmd.agu.gov.br/api/changes \
  -u "Keller:senhaerrada" \
  -H "Content-Type: application/json" \
  -d '{"titulo":"Teste","data_inicio":"2026-04-25T10:00"}'
```

---

## 9. Checklist de Deploy

- [ ] Variaveis `INVGATE_USER`, `INVGATE_PASSWORD`, `INVGATE_BASE_URL` configuradas no servidor
- [ ] `server.py` atualizado com autenticacao dual
- [ ] `aguservicos-logo.png` copiado para `frontend/public/`
- [ ] Frontend rebuildado (`yarn build` ou `npm run build`)
- [ ] Backend reiniciado
- [ ] Credencial SGMD-Integracao configurada no InvGate (HTTP Basic Auth)
- [ ] Web Service do workflow apontando para `https://sgmd.agu.gov.br/api/changes`
- [ ] Teste via curl bem-sucedido
- [ ] Badge "Made with Emergent" removido do `index.html` (se aplicavel)

---

*Documento elaborado como parte da integracao ITSM do projeto SGMD.*
*Departamento de Tecnologia da Informacao.*
