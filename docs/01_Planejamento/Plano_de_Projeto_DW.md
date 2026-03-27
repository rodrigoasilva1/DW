# Plano de Projeto: Data Warehouse para Controladoria e BI

**Data:** 25 de Março de 2026
**Autor:** Rodrigo Silva (Product Owner - Especialista de Controladoria em Dados)

## 1. Informações Gerais do Projeto

Este documento consolida o planejamento para a construção do Data Warehouse (DW) centralizado em PostgreSQL, visando integrar os dados transacionais para a criação de painéis de Business Intelligence (BI) para a área de Controladoria e demais setores estratégicos. O projeto seguirá a metodologia Ágil (Scrum) com duração de 3 meses, distribuídos em 6 Sprints quinzenais.

O fuso horário adotado para o projeto é o GMT-3, com horário comercial de segunda a sexta-feira, das 8:30 às 18:00. O projeto terá início oficial com uma reunião de alinhamento hoje às 14:00.

## 2. Equipe do Projeto

A equipe principal do projeto é composta por especialistas dedicados à construção e validação do Data Warehouse e dos painéis de BI:

| Papel | Nome | Responsabilidades |
| :--- | :--- | :--- |
| Product Owner (PO) | Rodrigo Silva | Especialista da Controladoria em dados, responsável pela visão do produto, validação das regras de negócio e priorização do backlog. |
| Scrum Master e Especialista | Lucas e Vinicius | Especialista em BI e banco de dados PostgreSQL. Responsável por conduzir os ritos ágeis, remover impedimentos e atuar na modelagem e construção do banco. |
| Especialista em Controladoria e Dados | Johnatan Ferraz | Especialista em eSolution (SQL Server) e mapeamento de dados. |
| Analista de BI | Lucas Mello | Responsável por alterar as conexões do BI, telas e front-end. |
| PMO de Projetos | Daniella Alamy | Responsável pelo acompanhamento gerencial do projeto, garantindo o alinhamento com a estratégia da empresa através de reuniões semanais. |
| Programador | Artur Lino | Atuará no desenvolvimento das rotinas de ETL e integração de dados, com dedicação parcial de até 4 horas por dia no projeto. |

## 3. Ritos Ágeis e Governança

Para garantir a fluidez e o alinhamento constante da equipe, os seguintes ritos ágeis serão adotados, respeitando o horário comercial (8:30 às 18:00, GMT-3):

| Rito | Duração | Frequência | Participantes |
| :--- | :--- | :--- | :--- |
| Daily Stand-up | 15 minutos | Diária | Rodrigo, Johnatan, Lucas, Artur |
| Sprint Planning | 2 horas | A cada 15 dias | Toda a equipe técnica |
| Sprint Review | 1 hora e 30 minutos | Ao final de cada Sprint | Toda a equipe técnica e PMO |
| Weekly com PMO | 30 minutos | Semanal | Toda a equipe e Daniella Alamy |

## 4. Arquitetura e Fontes de Dados

A arquitetura do Data Warehouse será baseada em PostgreSQL e seguirá uma abordagem em camadas (Staging, ODS e Dimensional). As fontes de dados integradas serão:

1. **Dynamics 365 (Dataverse):** CRM responsável pela prospecção e gestão de leads.
2. **eSolution (SQL Server):** Sistema de Vendas, origem dos detalhes granulares das negociações, salas de vendas e comissões.
3. **Sistema de Contratos (PostgreSQL):** Sistema para lançamento e gestão de contratos.
4. **Sienge (PostgreSQL):** ERP Financeiro principal, responsável pelo contas a receber, contas a pagar e baixas financeiras.

## 5. Cronograma de Sprints (3 Meses)

O cronograma está dividido em 6 Sprints de duas semanas cada:

| Sprint | Período | Objetivo Principal |
| :--- | :--- | :--- |
| Sprint 1 | Semanas 1-2 | Setup da infraestrutura PostgreSQL e mapeamento inicial detalhado do eSolution (SQL Server) e Sienge. |
| Sprint 2 | Semanas 3-4 | Desenvolvimento dos pipelines de extração (ETL) e carga na camada Staging para eSolution e Sienge. |
| Sprint 3 | Semanas 5-6 | Modelagem dimensional (Fatos e Dimensões) e resolução de chaves de integração entre vendas e financeiro. |
| Sprint 4 | Semanas 7-8 | Inclusão dos dados do Sistema de Contratos e integração com a API do Dynamics 365. |
| Sprint 5 | Semanas 9-10 | Validação da qualidade dos dados e adaptação das conexões dos painéis de BI para o novo DW. |
| Sprint 6 | Semanas 11-12 | Homologação final, documentação completa e Go-Live com desligamento das conexões antigas. |
