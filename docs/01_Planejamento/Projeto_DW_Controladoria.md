# Projeto de Data Warehouse para Controladoria e BI

**Data:** 25 de Março de 2026
**Autor:** Rodrigo Silva (Product Owner - Especialista de Controladoria em Dados)

## 1. Resumo Executivo e Equipe

Este documento detalha o planejamento e a arquitetura para a construção de um Data Warehouse (DW) centralizado em PostgreSQL. O objetivo principal é consolidar dados de múltiplas fontes transacionais para suportar a criação de painéis de Business Intelligence (BI) para a área de Controladoria e demais setores estratégicos da empresa. O projeto adotará a metodologia Ágil (Scrum) com duração estimada de 3 meses, divididos em Sprints quinzenais.

A equipe principal do projeto é composta por Rodrigo Silva atuando como Product Owner (PO), sendo o especialista da Controladoria em dados. Marcos Faria assume o papel de Scrum Master e Especialista em BI, auxiliando também nas construções no banco de dados PostgreSQL. Johnatan Ferraz atua como Especialista em Controladoria e Dados, especialista em consultas no banco SQL Server do eSolution. Lucas Mello atua como Analista de BI e será responsável por alterar as conexões do BI e telas. O acompanhamento gerencial será feito por Daniella Alamy, PMO de Projetos da empresa.

## 2. Arquitetura de Dados e Fontes

O DW integrará dados de quatro sistemas principais, respeitando a jornada do cliente e do dado financeiro. O Dynamics 365 (Dataverse) atua como CRM responsável pela prospecção, gestão de leads e relacionamento inicial com o cliente. O eSolution (SQL Server) é o sistema de vendas, sendo a origem principal das vendas e contendo detalhes granulares como sala de vendas, corretores, comissões e informações detalhadas da negociação antes da integração financeira. O Sistema de Contratos (PostgreSQL) é responsável pelo lançamento e gestão de contratos gerados a partir das vendas. Por fim, o Sienge (PostgreSQL) é o ERP principal financeiro e de controladoria. Ele recebe os dados de venda do eSolution via API e é a fonte da verdade para o contas a receber, contas a pagar, fluxo de caixa, baixas de parcelas, renegociações e toda a movimentação financeira e contábil.

### 2.1. Estrutura do Data Warehouse (PostgreSQL)

A arquitetura seguirá uma abordagem em camadas para garantir governança e performance. A Camada Staging (Bronze) servirá como área de pouso dos dados extraídos das fontes originais em seu formato bruto. A Camada ODS / Silver conterá dados limpos, padronizados e integrados, realizando a resolução de chaves entre sistemas, como o mapeamento do ID da Venda no eSolution com o ID do Título no Sienge. A Camada Dimensional (Gold / Data Marts) utilizará modelagem em Star Schema, com Fatos e Dimensões otimizadas para as consultas dos painéis de BI.

### 2.2. Modelagem Dimensional Preliminar

A modelagem dimensional preliminar define as principais tabelas de contexto (Dimensões) e de métricas (Fatos) que comporão o DW.

| Tipo de Tabela | Nome da Tabela | Descrição e Origem |
| :--- | :--- | :--- |
| Dimensão | `dim_tempo` | Calendário padrão corporativo para análises temporais. |
| Dimensão | `dim_cliente` | Dados consolidados do cliente, com origem no Dynamics e Sienge. |
| Dimensão | `dim_empreendimento` | Dados do projeto, obra ou loteamento. |
| Dimensão | `dim_unidade` | Informações sobre lotes, apartamentos e salas, com origem no Sienge e eSolution. |
| Dimensão | `dim_corretor` | Dados da equipe de vendas, extraídos do eSolution. |
| Dimensão | `dim_sala_vendas` | Locais de venda e filiais, extraídos do eSolution. |
| Fato | `fato_vendas` | Dados detalhados de cada venda realizada, com origem no eSolution. |
| Fato | `fato_contratos` | Status e evolução dos contratos, com origem no Sistema de Contratos. |
| Fato | `fato_financeiro_recebimentos` | Títulos a receber, parcelas, baixas, juros e multas, com origem principal no Sienge. |
| Fato | `fato_financeiro_pagamentos` | Títulos a pagar e despesas da obra, com origem principal no Sienge. |

## 3. Cronograma e Sprints Ágeis (3 Meses)

O projeto será executado em 6 Sprints de 2 semanas cada, totalizando 12 semanas (3 meses). Reuniões de acompanhamento ocorrerão semanalmente com a presença da PMO.

### 3.1. Sprints 1 a 3: Infraestrutura, Extração e Modelagem Inicial

A **Sprint 1 (Semanas 1-2)** tem como objetivo preparar o ambiente do DW e iniciar o mapeamento das fontes de dados mais críticas. As atividades incluem o provisionamento do servidor PostgreSQL para o DW e a criação da estrutura de bancos e schemas. Nesta etapa, Johnatan Ferraz realizará o mapeamento detalhado do banco de dados do eSolution, focando nas tabelas de vendas, sala de vendas, corretores e comissões. Paralelamente, Rodrigo Silva e Marcos Faria mapearão as tabelas financeiras essenciais no Sienge, focando em contas a receber e baixas. Também será definida a ferramenta de ETL a ser utilizada.

A **Sprint 2 (Semanas 3-4)** foca em trazer os dados brutos dos dois sistemas principais para a camada Staging. Serão desenvolvidos os pipelines de extração do eSolution e do Sienge para o DW PostgreSQL. A equipe realizará a validação da volumetria e integridade dos dados extraídos e manterá reuniões de alinhamento semanais com a PMO.

A **Sprint 3 (Semanas 5-6)** visa criar as primeiras tabelas Fato e Dimensão, resolvendo as chaves de integração. Serão criadas as dimensões principais, como tempo, cliente, empreendimento, unidade e sala de vendas. Em seguida, serão construídas a `fato_vendas` com dados do eSolution e a `fato_financeiro_recebimentos` com dados do Sienge. O ponto crucial desta sprint é o mapeamento e cruzamento da chave de venda, enviada via API do eSolution para o Sienge, garantindo a rastreabilidade financeira da venda.

### 3.2. Sprints 4 a 6: Integração Secundária, Validação e Go-Live

A **Sprint 4 (Semanas 7-8)** tem como objetivo integrar as fontes secundárias para enriquecer a jornada do cliente. As tarefas incluem o mapeamento e extração do Sistema de Contratos para a Staging e a conexão com a API do Dynamics 365 para extração de dados de leads e prospecção. A modelagem dimensional será atualizada para incluir a `fato_contratos` e enriquecer a `dim_cliente`. Marcos Faria também realizará uma revisão de performance das queries no PostgreSQL.

A **Sprint 5 (Semanas 9-10)** foca em garantir a qualidade dos dados e iniciar a virada dos painéis de BI. Rodrigo Silva será responsável pela homologação dos dados da `fato_financeiro_recebimentos` contra os relatórios nativos do Sienge. Lucas Mello iniciará a alteração das conexões de dados nos painéis de BI existentes para apontar para o novo DW em PostgreSQL, realizando os ajustes necessários de DAX e SQL conforme a nova modelagem dimensional.

A **Sprint 6 (Semanas 11-12)** marca a entrega final do projeto, documentação e treinamento. As atividades incluem a aprovação final dos painéis de BI pela Controladoria e Diretoria, e a documentação detalhada do Dicionário de Dados do DW, dos fluxos de ETL e dos agendamentos. O projeto será encerrado formalmente em uma reunião com a PMO Daniella Alamy, culminando no Go-Live com o desligamento das conexões diretas de BI nos bancos de produção e a virada oficial para o DW.

## 4. Governança e Ritos Ágeis

Para garantir o alinhamento e o progresso contínuo, a equipe adotará ritos ágeis rigorosos. Reuniões diárias de 15 minutos (Daily Stand-up), de forma assíncrona ou síncrona, serão realizadas entre Rodrigo, Marcos, Johnatan e Lucas para alinhamento técnico. A cada 15 dias, ocorrerá a Sprint Planning para definir as tarefas do próximo ciclo. Semanalmente, haverá uma reunião de 30 minutos com a PMO Daniella Alamy para reporte de status, riscos e impedimentos. Ao final de cada Sprint, a equipe conduzirá uma Sprint Review e Retrospective para demonstrar os incrementos construídos e discutir melhorias para o próximo ciclo.
