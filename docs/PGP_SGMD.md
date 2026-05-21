# Plano de Gerenciamento do Projeto - PGP

**Projeto**: SGMD - Servico de Gerenciamento de Mudancas
**Versao**: 1.0
**Data**: Abril de 2026
**Classificacao**: Uso Interno - SGE/AGU

---

## 1. Informacoes Gerais do Projeto

| Campo | Descricao |
|---|---|
| **Nome do Projeto** | SGMD - Servico de Gerenciamento de Mudancas |
| **Sigla** | SGMD |
| **Patrocinador** | Uendel Tavares - CGGOV/SGE |
| **Gerente do Projeto** | Hyago Keller - Gerente de Projetos / Especialista Tecnico ITSM |
| **Area Solicitante** | SGE - Secretaria de Governanca Estrategica |
| **Orgao** | Advocacia-Geral da Uniao (AGU) |
| **Orcamento** | Nao se aplica (recursos internos) |

---

## 2. Descricao do Projeto

### 2.1 Contexto e Justificativa

A Secretaria de Governanca Estrategica (SGE) da AGU necessita de uma ferramenta centralizada para gerenciar o ciclo de vida das mudancas de TI (ITIL v4), permitindo o planejamento, acompanhamento e controle das intervencoes nos sistemas e infraestrutura do orgao.

Atualmente, o controle de mudancas e realizado de forma descentralizada, dificultando a visibilidade dos agendamentos, a deteccao de conflitos e o acompanhamento dos resultados. O SGMD resolve esses problemas ao oferecer:

- Calendario visual unificado com todas as mudancas programadas
- Integracao automatica com o InvGate Service Management (ITSM da AGU)
- Alertas de conflitos de agendamento entre mudancas
- Rastreabilidade completa dos registros de mudanca (RFC)
- Exportacao de dados para auditorias e relatorios

### 2.2 Objetivo Geral

Desenvolver e implantar uma aplicacao web para gerenciamento visual de mudancas de TI, integrada ao InvGate Service Management, seguindo os padroes do design system Gov.br e as praticas ITIL v4.

### 2.3 Objetivos Especificos

1. Prover um calendario interativo com visoes semanal, mensal, semestral e anual das mudancas programadas
2. Implementar CRUD completo de registros de mudanca com campos ITIL v4
3. Integrar com o InvGate Service Management via HTTP Basic Auth para recepcao automatica de mudancas
4. Implementar metricas e indicadores visuais (status, risco, frente de atuacao)
5. Alertar automaticamente sobre conflitos de agendamento
6. Exportar dados em formato CSV para relatorios e auditorias
7. Garantir disponibilidade 24x7 no ambiente de producao

---

## 3. Equipe do Projeto

| Nome | Papel | Responsabilidades |
|---|---|---|
| **Uendel Tavares** | Patrocinador (CGGOV/SGE) | Aprovacao estrategica, alocacao de recursos, validacao de entregas |
| **Hyago Keller** | Gerente de Projetos / Especialista Tecnico ITSM | Planejamento, coordenacao, arquitetura tecnica, integracao ITSM, gestao de riscos |
| **Andre Nascimento** | Analista de Automacao Senior | Desenvolvimento backend/frontend, automacao de processos, integracao InvGate |
| **Yan Basilio** | Analista de Automacao Pleno | Desenvolvimento frontend, testes, documentacao tecnica |

---

## 4. Partes Interessadas (Stakeholders)

| Stakeholder | Interesse | Nivel de Influencia |
|---|---|---|
| **SGE - Secretaria de Governanca Estrategica** | Patrocinador. Governanca e visibilidade das mudancas | Alto |
| **CGGOV - Coordenacao-Geral de Governanca** | Supervisao direta do projeto | Alto |
| **Coordenacoes vinculadas a SGE** | Usuarios finais. Registro e acompanhamento de mudancas | Medio |
| **Equipes de Infraestrutura** | Executores de mudancas de infraestrutura | Medio |
| **Equipes de Sistemas** | Executores de mudancas de sistemas e aplicacoes | Medio |
| **Equipe SuperSapiens** | Executores de mudancas no ecossistema SAPIENS | Medio |
| **Gestores de Negocio** | Responsaveis pela aprovacao e justificativa das mudancas | Medio |

---

## 5. Escopo do Projeto

### 5.1 Escopo Incluido

| # | Entrega | Descricao |
|---|---|---|
| 1 | **Modulo de Autenticacao** | Login com JWT, controle de sessao, seed de administrador |
| 2 | **CRUD de Mudancas** | Criacao, leitura, atualizacao e exclusao de registros de mudanca com campos ITIL v4 |
| 3 | **Calendario Interativo** | Visoes semanal, mensal, semestral e anual com indicadores visuais por status, risco e frente de atuacao |
| 4 | **Dashboard de Metricas** | Cards de contagem por status (Planejada, Aprovada, Em Execucao, Concluida, Cancelada) e por frente de atuacao (Infraestrutura, Sistemas, SuperSapiens) |
| 5 | **Alertas de Conflito** | Deteccao automatica de sobreposicao de agendamentos |
| 6 | **Integracao InvGate** | Recepcao automatica de mudancas via HTTP Basic Auth (Web Service do workflow InvGate) |
| 7 | **Exportacao CSV** | Exportacao de dados para relatorios e auditorias |
| 8 | **Documentacao** | PGP, PAT, HLD, Guia de Integracao |
| 9 | **Deploy em Producao** | Implantacao no servidor SDF1066 (sgmd.agu.gov.br) |

### 5.2 Escopo Excluido

- Desenvolvimento de aplicativo mobile nativo
- Integracao com sistemas externos alem do InvGate Service Management
- Modulo de aprovacao/workflow interno (utiliza o workflow do InvGate)
- Backup automatizado (nao se aplica conforme definicao do projeto)
- Treinamento presencial de usuarios (sera fornecida documentacao)

---

## 6. Cronograma

### 6.1 Fases do Projeto

| Fase | Atividade | Status |
|---|---|---|
| **Fase 1 - Planejamento** | Levantamento de requisitos, definicao de arquitetura, PGP e PAT | Concluida |
| **Fase 2 - Desenvolvimento Core** | Backend (FastAPI + MongoDB), Autenticacao JWT, CRUD de mudancas | Concluida |
| **Fase 3 - Frontend** | Dashboard Gov.br, Calendario interativo, Metricas, Alertas de conflito | Concluida |
| **Fase 4 - Campos ITIL v4** | Frente de Atuacao, Natureza, Categoria, Risco, Rollback, Homologacao | Concluida |
| **Fase 5 - Integracao InvGate** | HTTP Basic Auth, Web Service, mapeamento de campos | Concluida |
| **Fase 6 - Deploy e Documentacao** | Implantacao no servidor, PGP, PAT, HLD, Guia de Integracao | Em andamento |
| **Fase 7 - Validacao e Ajustes** | Testes em producao, ajustes de mapeamento InvGate, validacao com stakeholders | Pendente |

---

## 7. Gestao de Riscos

| # | Risco | Probabilidade | Impacto | Mitigacao |
|---|---|---|---|---|
| R1 | Indisponibilidade do servidor SDF1066 | Baixa | Alto | Monitoramento 24x7, SLA 100% |
| R2 | Falha na integracao InvGate (campos mapeados incorretamente) | Media | Medio | Documentacao de mapeamento, testes com payloads reais |
| R3 | Mudanca nas APIs do InvGate sem aviso | Baixa | Alto | Versionamento da API, logs de erro, alertas |
| R4 | Crescimento inesperado do volume de dados | Baixa | Baixo | MongoDB escala horizontalmente, indices otimizados |
| R5 | Perda de credenciais de integracao | Baixa | Alto | Credenciais em variaveis de ambiente, acesso restrito ao servidor |
| R6 | Resistencia dos usuarios a nova ferramenta | Media | Medio | Interface intuitiva (Gov.br), integracao automatica reduz trabalho manual |

---

## 8. Gestao da Comunicacao

| Tipo | Frequencia | Participantes | Meio |
|---|---|---|---|
| **Reuniao de Status** | Semanal | Equipe do projeto | Microsoft Teams |
| **Reporte ao Patrocinador** | Quinzenal | Hyago Keller, Uendel Tavares | E-mail / Reuniao |
| **Comunicacao Tecnica** | Sob demanda | Andre, Yan, Keller | Teams / Documentacao no Git |
| **Release Notes** | A cada deploy | Todos os stakeholders | E-mail |

---

## 9. Gestao da Qualidade

### 9.1 Criterios de Aceitacao

| Criterio | Descricao |
|---|---|
| **Funcionalidade** | Todas as funcionalidades do escopo devem estar operacionais |
| **Integracao** | InvGate deve conseguir enviar mudancas via Web Service com HTTP Basic Auth |
| **Usabilidade** | Interface seguindo padroes Gov.br, responsiva, com indicadores visuais claros |
| **Seguranca** | Autenticacao JWT para usuarios, Basic Auth para integracao, sem exposicao de credenciais |
| **Disponibilidade** | Sistema acessivel 24x7 via sgmd.agu.gov.br |

### 9.2 Testes

| Tipo de Teste | Responsavel | Ferramenta |
|---|---|---|
| Testes Unitarios (Backend) | Andre Nascimento | pytest |
| Testes de Integracao (InvGate) | Hyago Keller | curl / Postman |
| Testes de Interface (Frontend) | Yan Basilio | Playwright / Manual |
| Teste de Aceitacao | Stakeholders SGE | Manual (ambiente de producao) |

---

## 10. Gestao de Mudancas do Projeto

Alteracoes no escopo, cronograma ou requisitos devem seguir o fluxo:

1. **Solicitacao**: Registrada pelo solicitante ao Gerente do Projeto
2. **Analise de Impacto**: Avaliacao tecnica pela equipe (Keller + Andre + Yan)
3. **Aprovacao**: Validacao pelo Patrocinador (Uendel Tavares) para mudancas de alto impacto
4. **Implementacao**: Desenvolvimento, teste e deploy
5. **Comunicacao**: Notificacao aos stakeholders impactados

---

## 11. Encerramento do Projeto

O projeto sera considerado encerrado quando:

- [ ] Todas as entregas do escopo estiverem implementadas e validadas
- [ ] A integracao InvGate estiver operacional em producao
- [ ] A documentacao estiver completa e atualizada no repositorio Git
- [ ] O sistema estiver acessivel via sgmd.agu.gov.br com disponibilidade 24x7
- [ ] O Patrocinador formalizar a aceitacao das entregas

---

## 12. Aprovacoes

| Papel | Nome | Data | Assinatura |
|---|---|---|---|
| **Gerente do Projeto** | Hyago Keller | ___/___/2026 | _______________ |
| **Patrocinador** | Uendel Tavares | ___/___/2026 | _______________ |

---

*Documento elaborado como parte do projeto SGMD.*
*SGE - Secretaria de Governanca Estrategica | Advocacia-Geral da Uniao.*
