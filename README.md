# Data Warehouse para Controladoria e BI

**Projeto:** Construção de um Data Warehouse centralizado em PostgreSQL para consolidar dados de múltiplas fontes e suportar painéis de Business Intelligence.

**Duração:** 3 meses (12 semanas) em 6 Sprints quinzenais
**Início:** 26 de Março de 2026
**Go-Live:** Final de Junho de 2026

---

## 📋 Equipe do Projeto

| Papel | Nome | Responsabilidade |
| :--- | :--- | :--- |
| **Product Owner** | Rodrigo Silva | Especialista da Controladoria em dados, visão do produto e regras de negócio |
| **Scrum Master / Especialista** | Marcos Faria | Condução ágil, arquitetura do DW e desenvolvimento PostgreSQL |
| **Especialista Controladoria e Dados** | Johnatan Ferraz | Mapeamento do eSolution (SQL Server) e análise de dados |
| **Analista de BI** | Lucas Mello | Migração das conexões de BI e ajustes no front-end analítico |
| **Programador / Engenheiro de Dados** | Artur Lino | Desenvolvimento de rotinas de ETL e integração (4h/dia) |
| **PMO de Projetos** | Daniella Alamy | Acompanhamento gerencial e alinhamento estratégico |

---

## 🏗️ Arquitetura do Projeto

O Data Warehouse será estruturado em **3 camadas** (Medallion Architecture):

1. **Staging (Bronze):** Cópia exata dos dados das origens, sem transformações complexas.
2. **ODS (Silver):** Dados limpos, tipados e padronizados.
3. **Dimensional (Gold):** Star Schema com tabelas Fato e Dimensão para análises BI.

### Fontes de Dados Integradas

- **Sienge (PostgreSQL):** ERP financeiro - contas a receber, contas a pagar, fluxo de caixa, baixas.
- **eSolution (SQL Server):** Sistema de vendas - detalhes de vendas, sala de vendas, corretores, comissões.
- **Dynamics 365 (API REST / Dataverse):** CRM - prospecção, leads, relacionamento com clientes.
- **Sistema de Contratos (PostgreSQL):** Lançamento e gestão de contratos.
- **Robô Financeiro (PostgreSQL):** Logs e lançamentos automatizados.
- **Planilhas Controladoria (Excel/CSV):** Dados manuais alimentados pela controladoria.

---

## 📁 Estrutura do Repositório

```
DW/
├── README.md                          # Este arquivo
├── docs/
│   ├── 01_Planejamento/
│   │   ├── Termo_Abertura_DW.md       # Project Charter oficial
│   │   ├── Plano_de_Projeto_DW.md     # Plano detalhado com cronograma
│   │   └── Projeto_DW_Controladoria.md # Documento completo do projeto
│   ├── 02_Arquitetura/
│   │   └── Dicionario_e_Plano_ETL.md  # Dicionário de dados e plano de ETL
│   ├── 03_ETL/
│   │   └── Documentacao_Tecnica_ETL_Csharp.md # Guia técnico para o desenvolvedor
│   └── 04_Modelagem/
│       └── (Modelos SQL e scripts de criação de tabelas)
├── src/
│   └── (Código-fonte C# dos pipelines de ETL)
└── scripts/
    └── (Scripts SQL, Bash, PowerShell para automação)
```

---

## 🚀 Como Começar

### Para o Desenvolvedor (Artur Lino)

1. Leia o documento **`docs/03_ETL/Documentacao_Tecnica_ETL_Csharp.md`** para entender os padrões e arquitetura de ETL.
2. Comece pela automação do backup do Sienge (restore automático na VM da Hostinger).
3. Implemente a primeira conexão com o PostgreSQL restaurado.
4. Progresse para as outras fontes de dados conforme as Sprints.

### Para o Product Owner e Stakeholders

1. Consulte o **`docs/01_Planejamento/Termo_Abertura_DW.md`** para entender o escopo e objetivos.
2. Acompanhe o progresso através do **`docs/01_Planejamento/Plano_de_Projeto_DW.md`** que detalha as 6 Sprints.

### Para o Arquiteto de Dados (Marcos Faria)

1. Revise o **`docs/02_Arquitetura/Dicionario_e_Plano_ETL.md`** para o design das tabelas Fato e Dimensão.
2. Valide as transformações de dados propostas.

---

## 📅 Cronograma das Sprints

| Sprint | Período | Foco Principal |
| :--- | :--- | :--- |
| **Sprint 1** | Semanas 1-2 (26/03 - 09/04) | Setup infraestrutura, mapeamento Sienge e eSolution |
| **Sprint 2** | Semanas 3-4 (10/04 - 23/04) | Pipelines de extração (Sienge e eSolution) para Staging |
| **Sprint 3** | Semanas 5-6 (24/04 - 07/05) | Star Schema, Fatos e Dimensões, integração de chaves |
| **Sprint 4** | Semanas 7-8 (08/05 - 21/05) | Integração Contratos e Dynamics 365 |
| **Sprint 5** | Semanas 9-10 (22/05 - 04/06) | Data Quality e migração das conexões BI |
| **Sprint 6** | Semanas 11-12 (05/06 - 18/06) | Homologação final, documentação e Go-Live |

---

## 🔄 Ritos Ágeis

- **Daily Stand-up:** 15 min, diária (8:30 - 18:00, GMT-3)
- **Sprint Planning:** 2h, a cada 15 dias
- **Sprint Review & Retrospective:** 1h 30m, ao final de cada Sprint
- **Weekly com PMO:** 30 min, semanal com Daniella Alamy

---

## 📞 Contato e Suporte

Para dúvidas técnicas sobre ETL e C#, contate **Artur Lino**.
Para questões de negócio e priorização, contate **Rodrigo Silva** (Product Owner).
Para acompanhamento gerencial, contate **Daniella Alamy** (PMO).

---

**Última atualização:** 27 de Março de 2026
**Versão:** 1.0
