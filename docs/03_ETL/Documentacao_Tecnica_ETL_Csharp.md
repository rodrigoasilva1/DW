# Documentação Técnica: Engenharia de Dados e ETL com C#

**Projeto:** Data Warehouse para Controladoria e BI
**Destinatário:** Artur Lino (Desenvolvedor de Integrações / Engenheiro de Dados)
**Autor:** Marcos Faria (Scrum Master / Especialista de Dados) & Rodrigo Silva (PO)

---

## 1. Introdução ao Mundo de ETL

Olá, Artur! Bem-vindo ao projeto de construção do nosso Data Warehouse (DW). Como você já tem forte base em programação C#, a transição para Engenharia de Dados será natural. Seu principal papel será construir os fluxos de **ETL (Extract, Transform, Load - Extrair, Transformar e Carregar)**.

Em sistemas transacionais (sistemas do dia a dia, como ERPs e CRMs), os bancos de dados são otimizados para inserir e atualizar dados rapidamente (arquitetura transacional). No entanto, eles são péssimos para consultas analíticas pesadas (relatórios e BI). O papel do ETL é:

1.  **Extract (Extrair):** Conectar nas fontes de dados originais e copiar os dados sem impactar a performance do sistema de origem.
2.  **Transform (Transformar):** Limpar, padronizar, cruzar e aplicar regras de negócio (ex: padronizar formatos de data, calcular chaves estrangeiras, tratar nulos).
3.  **Load (Carregar):** Inserir os dados tratados no Data Warehouse (nosso banco PostgreSQL final), que é modelado especificamente para leitura rápida pelos painéis de BI.

### 1.1. Arquitetura em Camadas (Medallion Architecture)

Nosso DW no PostgreSQL será dividido em schemas lógicos, seguindo boas práticas de mercado:

*   **Camada Staging (Bronze):** Cópia exata dos dados da origem (1 para 1). Os dados entram aqui sem transformações complexas, apenas truncando e carregando (Truncate/Insert) ou inserindo dados incrementais. O objetivo é tirar o peso dos bancos de origem o mais rápido possível.
*   **Camada ODS (Silver):** *Operational Data Store*. Aqui os dados da Staging são limpos, tipados corretamente e padronizados.
*   **Camada Dimensional (Gold):** O modelo final (Star Schema). Contém as tabelas **Fato** (métricas, valores, eventos que ocorreram, ex: `fato_vendas`, `fato_recebimentos`) e tabelas **Dimensão** (contexto, descrições, ex: `dim_cliente`, `dim_tempo`, `dim_empreendimento`).

---

## 2. Arquitetura da Solução e Infraestrutura

Nossa infraestrutura principal estará hospedada em uma **Máquina Virtual (VM) na Hostinger**. Nesta máquina, teremos uma instância do PostgreSQL rodando, que abrigará dois bancos distintos:
1.  **Banco de Restauração do Sienge:** Recebe o backup diário do ERP.
2.  **Banco DW (Data Warehouse):** Onde as camadas Staging, ODS e Dimensional viverão.

Sua aplicação em C# (recomendamos um Worker Service ou um Console Application agendado via Cron/Task Scheduler) será o "maestro" de toda essa orquestração.

### 2.1. O Desafio do Sienge (Automação de Backup e Restore)

Não temos conexão direta de leitura ao banco de produção do Sienge. A estratégia é:
1.  A aplicação C# fará o download do arquivo de backup diário do Sienge (via FTP, API, ou diretório mapeado, conforme definido pela infraestrutura).
2.  A aplicação C# executará um comando de sistema operacional para restaurar este backup no PostgreSQL local da VM.
3.  Após o restore, o C# iniciará o processo de ETL, lendo do banco restaurado e gravando na Staging do DW.

**Exemplo de Automação do Restore em C#:**
Você utilizará a classe `System.Diagnostics.Process` para chamar o utilitário `pg_restore` do PostgreSQL.

```csharp
using System.Diagnostics;

public void RestaurarBackupSienge(string caminhoArquivoBackup)
{
    string pgRestorePath = @"/usr/bin/pg_restore"; // Caminho no Linux (Hostinger)
    string connectionStringArgs = "-h localhost -U postgres -d banco_sienge_restaurado --clean";
    
    ProcessStartInfo startInfo = new ProcessStartInfo
    {
        FileName = pgRestorePath,
        Arguments = $"{connectionStringArgs} {caminhoArquivoBackup}",
        RedirectStandardOutput = true,
        RedirectStandardError = true,
        UseShellExecute = false,
        CreateNoWindow = true
    };

    using (Process process = Process.Start(startInfo))
    {
        process.WaitForExit();
        if (process.ExitCode != 0)
        {
            string error = process.StandardError.ReadToEnd();
            throw new Exception($"Erro ao restaurar backup: {error}");
        }
    }
}
```

---

## 3. Mapeamento das Fontes de Dados

Seu pipeline em C# precisará se conectar a diferentes tecnologias. Abaixo estão as bibliotecas recomendadas e o padrão de extração para cada uma.

| Sistema Origem | Tecnologia | Biblioteca C# Recomendada | Estratégia de Extração |
| :--- | :--- | :--- | :--- |
| **Sienge (ERP)** | PostgreSQL (Restaurado) | `Npgsql` + `Dapper` | Leitura em massa (`SELECT *`) das tabelas financeiras para a Staging. |
| **eSolution (Vendas)** | SQL Server | `Microsoft.Data.SqlClient` + `Dapper` | Leitura incremental (baseada em `data_atualizacao`) ou carga full de tabelas de vendas e comissões. |
| **Dynamics 365 (CRM)** | API REST / Dataverse | `HttpClient` / `Microsoft.PowerPlatform.Dataverse.Client` | Consumo de endpoints REST com paginação, serialização via `System.Text.Json`. |
| **Sistema de Contratos** | PostgreSQL (Remoto) | `Npgsql` + `Dapper` | Leitura das tabelas de contratos firmados. |
| **Robô Financeiro** | PostgreSQL (Remoto) | `Npgsql` + `Dapper` | Leitura de logs e lançamentos automatizados. |
| **Planilhas Controladoria** | Arquivos Excel/CSV na Rede | `EPPlus` (Excel) ou `CsvHelper` (CSV) | Leitura de diretório mapeado (`File.ReadAllLines` ou bibliotecas específicas) e conversão para objetos C#. |

---

## 4. Padrões de Desenvolvimento ETL em C#

Para garantir performance e manutenibilidade, evite usar Entity Framework (EF Core) para operações de ETL em massa. O EF Core é excelente para sistemas web e CRUDs, mas gera muito overhead de memória ao rastrear milhares de objetos simultaneamente.

### 4.1. Uso do Dapper para Consultas
O Dapper é um micro-ORM extremamente rápido. Utilize-o para ler dados das origens.

```csharp
using System.Data.SqlClient;
using Dapper;

public async Task<IEnumerable<Venda>> ExtrairVendasEsolutionAsync(string connectionString)
{
    using (var connection = new SqlConnection(connectionString))
    {
        string query = "SELECT IdVenda, DataVenda, Valor, IdCorretor FROM Vendas WHERE DataAtualizacao >= @UltimaData";
        return await connection.QueryAsync<Venda>(query, new { UltimaData = DateTime.Today.AddDays(-1) });
    }
}
```

### 4.2. Inserção em Massa (Bulk Insert) no PostgreSQL DW
O gargalo de um ETL geralmente está na gravação. Nunca faça inserções linha a linha (loops com `INSERT INTO`). No PostgreSQL, a forma mais rápida de gravar milhares de linhas é usando a API `COPY` através do `NpgsqlBinaryImporter`.

**Exemplo de Bulk Insert de Alta Performance:**

```csharp
using Npgsql;

public async Task CarregarStagingVendasAsync(string dwConnectionString, IEnumerable<Venda> vendas)
{
    using (var connection = new NpgsqlConnection(dwConnectionString))
    {
        await connection.OpenAsync();
        
        // Inicia a importação binária (muito mais rápido que INSERT)
        using (var writer = await connection.BeginBinaryImportAsync("COPY staging.vendas (id_venda, data_venda, valor, id_corretor) FROM STDIN (FORMAT BINARY)"))
        {
            foreach (var venda in vendas)
            {
                await writer.StartRowAsync();
                await writer.WriteAsync(venda.IdVenda, NpgsqlTypes.NpgsqlDbType.Integer);
                await writer.WriteAsync(venda.DataVenda, NpgsqlTypes.NpgsqlDbType.Date);
                await writer.WriteAsync(venda.Valor, NpgsqlTypes.NpgsqlDbType.Numeric);
                await writer.WriteAsync(venda.IdCorretor, NpgsqlTypes.NpgsqlDbType.Integer);
            }
            await writer.CompleteAsync(); // Comita a transação do COPY
        }
    }
}
```

### 4.3. Tratamento de Planilhas e Arquivos de Rede
Para arquivos mapeados na rede local (ex: `\\servidor\controladoria\metas.xlsx`), verifique se o usuário que executa a aplicação C# tem permissão de leitura no diretório. Use a biblioteca `EPPlus` (para `.xlsx`) ou `CsvHelper` (para `.csv`).

```csharp
// Exemplo com CsvHelper
using CsvHelper;
using System.Globalization;
using System.IO;

public IEnumerable<Meta> LerPlanilhaMetas(string caminhoArquivo)
{
    using (var reader = new StreamReader(caminhoArquivo))
    using (var csv = new CsvReader(reader, CultureInfo.InvariantCulture))
    {
        return csv.GetRecords<Meta>().ToList();
    }
}
```

---

## 5. Resiliência, Logs e Agendamento

Um bom pipeline de ETL deve ser silencioso quando funciona e muito barulhento quando falha.

1.  **Tratamento de Erros:** Envolva as chamadas de banco de dados e APIs em blocos `try-catch`. Se uma fonte de dados estiver fora do ar (ex: timeout no SQL Server do eSolution), o pipeline deve registrar o erro, alertar a equipe e continuar com as outras fontes, se possível.
2.  **Logs:** Utilize bibliotecas como **Serilog** ou **NLog**. Grave logs estruturados no próprio PostgreSQL (em uma tabela `etl_logs`) ou em arquivos texto, registrando:
    *   Data/Hora de início e fim da extração.
    *   Fonte de dados processada.
    *   Quantidade de linhas extraídas e inseridas.
    *   Mensagens de erro detalhadas (Stack Trace).
3.  **Agendamento:** Recomendamos estruturar a aplicação como um **Worker Service** no .NET (utilizando bibliotecas como **Quartz.NET** ou **Hangfire** para gerenciar os agendamentos internos) ou compilar como um executável e utilizar o `Cron` do Linux na Hostinger para agendar as execuções (ex: rodar a restauração do Sienge às 02:00 da manhã, e as extrações às 03:00).

## 6. Próximos Passos (Sprint 1)

Artur, para as primeiras semanas, seu foco será:
1.  Configurar o repositório C# (Git).
2.  Criar o script C# que executa o `pg_restore` do arquivo de backup do Sienge localmente na VM.
3.  Criar a primeira conexão (usando Npgsql) lendo do banco restaurado e gravando em uma tabela de teste no DW.
4.  Estabelecer a conexão com o SQL Server do eSolution.

Estaremos juntos nas Dailies para tirar qualquer dúvida de modelagem e arquitetura! Bom código!
