# Termo de Abertura do Projeto (Project Charter)

**Nome do Projeto:** Data Warehouse para Controladoria e BI
**Data de Início:** 25 de Março de 2026
**Duração Estimada:** 3 meses (12 semanas)
**Patrocinador (Sponsor):** Diretoria Executiva / Controladoria
**Gerente do Projeto / PO:** Rodrigo Silva

## 1. Justificativa e Objetivos do Projeto

Atualmente, a área de Controladoria e demais setores estratégicos da empresa dependem de painéis de Business Intelligence (BI) que se conectam diretamente aos bancos de dados transacionais de produção. Esta arquitetura gera riscos de performance nos sistemas principais e dificulta o cruzamento de dados que percorrem a jornada completa do cliente.

O objetivo deste projeto é construir um Data Warehouse (DW) centralizado em PostgreSQL, que atuará como a única fonte da verdade (Single Source of Truth) para os relatórios analíticos. Este DW integrará os dados desde a prospecção no CRM, passando pela venda no eSolution, a formalização no Sistema de Contratos, até o faturamento e as baixas financeiras no Sienge.

## 2. Escopo do Projeto

O escopo inclui a modelagem, desenvolvimento e implantação do Data Warehouse e de todos os processos de extração, transformação e carga (ETL) necessários. Estão incluídas as seguintes atividades:
* Provisionamento e configuração de um banco de dados PostgreSQL para o DW.
* Mapeamento detalhado das tabelas e campos dos sistemas eSolution (SQL Server), Sienge (PostgreSQL), Sistema de Contratos (PostgreSQL) e Dynamics 365 (Dataverse API).
* Desenvolvimento de pipelines de dados para alimentar as camadas Staging, ODS e Dimensional do DW.
* Criação das tabelas de Fato e Dimensão para suportar as métricas de vendas, contratos, recebimentos e pagamentos.
* Alteração das conexões dos painéis de BI existentes para apontar para o novo DW.
* Homologação dos dados com os relatórios nativos dos sistemas de origem.

Não faz parte do escopo deste projeto a criação de novos painéis de BI do zero, mas sim a migração das conexões dos painéis já existentes e eventuais ajustes nas consultas (DAX/SQL) devido à nova modelagem dimensional.

## 3. Equipe e Papéis

O projeto contará com uma equipe multidisciplinar, operando em horário comercial (GMT-3, 8:30 às 18:00):
* **Rodrigo Silva (PO):** Especialista da Controladoria, responsável pelas regras de negócio e homologação.
* **Marcos Faria (Scrum Master/Especialista):** Condução ágil, arquitetura do DW e desenvolvimento no PostgreSQL.
* **Johnatan Ferraz (Especialista em Controladoria e Dados):** Mapeamento do eSolution e análise de dados.
* **Lucas Mello (Analista de BI):** Migração das conexões de front-end dos painéis e ajustes no BI.
* **Artur Lino (Programador):** Desenvolvimento de rotinas de integração (dedicação de 4h/dia).
* **Daniella Alamy (PMO):** Acompanhamento gerencial e reporte de status.

## 4. Premissas e Restrições

**Premissas:**
* A equipe terá acesso de leitura (read-only) aos bancos de dados de produção do eSolution, Sienge e Sistema de Contratos.
* A API do Dynamics 365 possui documentação e credenciais disponíveis para integração.
* A infraestrutura para o servidor PostgreSQL do DW será providenciada pela área de TI na primeira semana do projeto.

**Restrições:**
* O projeto deve ser concluído no prazo máximo de 3 meses.
* O esforço do programador Artur Lino é limitado a 4 horas diárias.
* Nenhuma alteração pode ser feita na estrutura dos bancos de dados transacionais de origem.

## 5. Marcos do Projeto (Milestones)

* **Kick-off do Projeto:** 25 de Março de 2026 (Reunião de Alinhamento às 14:00).
* **Conclusão da Camada Staging (eSolution e Sienge):** Final da Sprint 2.
* **Conclusão da Modelagem Dimensional Base:** Final da Sprint 3.
* **Integração Completa de Todas as Fontes:** Final da Sprint 4.
* **Go-Live e Encerramento:** Final da Sprint 6.
