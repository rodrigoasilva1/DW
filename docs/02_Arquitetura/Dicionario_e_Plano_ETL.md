# Dicionário de Dados e Plano de ETL

**Data:** 25 de Março de 2026
**Projeto:** Data Warehouse para Controladoria e BI

## 1. Estratégia de ETL (Extração, Transformação e Carga)

O processo de integração de dados seguirá uma arquitetura em três camadas lógicas dentro do banco de dados PostgreSQL. A equipe de desenvolvimento, liderada por Marcos Faria com apoio do programador Artur Lino, será responsável por construir esses pipelines garantindo a consistência e a rastreabilidade dos dados.

A primeira camada, denominada Staging (ou Bronze), atuará como uma área de pouso para os dados brutos. As tabelas nesta camada refletirão exatamente a estrutura das tabelas de origem, recebendo cargas incrementais diárias, ou integrais (full load) para tabelas de domínio menores. O objetivo principal da Staging é minimizar o impacto de leitura nos bancos de dados de produção do Sienge, eSolution e Contratos.

A segunda camada, ODS (Operational Data Store) ou Silver, será responsável pela limpeza, padronização e integração dos dados. É nesta etapa que ocorrerá a resolução de chaves entre os diferentes sistemas. Por exemplo, o ID do cliente gerado no Dynamics 365 será mapeado para o código de cliente no eSolution e no Sienge. Da mesma forma, o código da venda gerado no eSolution será cruzado com o contrato gerado no Sistema de Contratos e com os títulos financeiros criados no Sienge via integração de API.

A terceira e última camada, Dimensional (ou Gold), conterá as tabelas modeladas em Star Schema, otimizadas para consultas analíticas dos painéis de BI. Os dados serão transformados em Fatos e Dimensões, agregando valor para as análises da Controladoria.

## 2. Dicionário de Dados Preliminar (Camada Dimensional)

Abaixo estão detalhadas as principais tabelas que comporão o Data Warehouse na sua camada final de consumo.

### 2.1. Tabelas de Dimensão (Contexto)

| Tabela | Descrição | Campos Principais | Origem Principal |
| :--- | :--- | :--- | :--- |
| `dim_tempo` | Calendário corporativo para análises temporais. | `id_data`, `data_completa`, `ano`, `mes`, `dia`, `trimestre`, `semestre`, `dia_semana`. | Gerada via script no PostgreSQL. |
| `dim_cliente` | Dados cadastrais consolidados dos clientes. | `id_cliente_dw`, `cpf_cnpj`, `nome_razao_social`, `email`, `telefone`, `id_origem_crm`, `id_origem_sienge`. | Dynamics 365 e Sienge. |
| `dim_empreendimento` | Cadastro de obras, projetos ou loteamentos. | `id_empreendimento`, `codigo_obra`, `nome_empreendimento`, `cidade`, `estado`, `status_obra`. | Sienge. |
| `dim_unidade` | Detalhes das unidades comercializadas (lotes, salas). | `id_unidade`, `id_empreendimento`, `bloco`, `numero_unidade`, `metragem`, `valor_tabela`. | Sienge e eSolution. |
| `dim_corretor` | Informações da equipe de vendas e parceiros. | `id_corretor`, `nome_corretor`, `imobiliaria`, `cpf`, `status_ativo`. | eSolution. |
| `dim_sala_vendas` | Locais físicos onde as vendas foram originadas. | `id_sala`, `nome_sala`, `cidade`, `regiao`, `gerente_responsavel`. | eSolution. |

### 2.2. Tabelas de Fato (Métricas)

| Tabela | Descrição | Métricas e Chaves Estrangeiras | Origem Principal |
| :--- | :--- | :--- | :--- |
| `fato_vendas` | Registro granular de cada venda efetivada. | `id_venda`, `fk_data_venda`, `fk_cliente`, `fk_unidade`, `fk_corretor`, `fk_sala`, `vlr_venda`, `vlr_comissao`, `vlr_desconto`. | eSolution. |
| `fato_contratos` | Acompanhamento do status de formalização. | `id_contrato`, `fk_venda`, `fk_data_assinatura`, `status_contrato`, `vlr_contrato`. | Sistema de Contratos. |
| `fato_financeiro_recebimentos` | Títulos a receber, parcelas e baixas. | `id_titulo`, `fk_contrato`, `fk_data_vencimento`, `fk_data_baixa`, `vlr_parcela`, `vlr_pago`, `vlr_juros`, `vlr_multa`, `status_titulo`. | Sienge. |
| `fato_financeiro_pagamentos` | Despesas e contas a pagar relacionadas às obras. | `id_pagamento`, `fk_empreendimento`, `fk_data_vencimento`, `fk_data_pagamento`, `vlr_documento`, `vlr_pago`, `centro_custo`. | Sienge. |
