---
sticker: lucide//file-text
---
# 1 Definindo o DBContext
> [!NOTE]
    > O coração da interação com o banco de dados no EF Core é a classe que herda de `DbContext`. é implementado na camada de `Infraestructure`


```csharp
	using CashFlow.Domain.Entities;
	using Microsoft.EntityFrameworkCore;
	namespace CashFlow.Infrastructure.DataAccess;
	
	public class CashFlowDbContext : DbContext 
	{
	    public CashFlowDbContext(DbContextOptions options) : base(options){}
	
	    public DbSet<Expense> Expenses { get; set; }
	}
```

- **O que é o `CashFlowDbContext`?** Ele representa uma <span style="background:rgba(240, 200, 0, 0.2)">sessão com o banco de dados</span>. É através dele que você executa consultas, adiciona, atualiza e remove dados.
  
- **`DbContextOptions options`:** O construtor recebe um objeto `DbContextOptions`. É neste objeto que o <span style="background:rgba(240, 200, 0, 0.2)">EF Core sabe qual provedor de banco de dados usar</span> (SQL Server, SQLite, PostgreSQL, etc.) e qual é a string de conexão. Passar essas opções via construtor (usando `: base(options)`) é a forma padrão de configurar o `DbContext` com DI.
  
- **`public DbSet<Expense> Expenses { get; set; }`:** Esta propriedade `DbSet` mapeia a entidade `Expense` (definida na camada Domain) para uma tabela no banco de dados (por padrão, com o nome "Expenses"). É através desta<span style="background:rgba(240, 200, 0, 0.2)"> propriedade que você acessa e manipula os dados da tabela expense no código</span>.

# 2 Configurando a String de Conexão (appsettings.json)
> [!NOTE]
    > Para que a aplicação saiba onde encontrar o banco de dados, a string de conexão é definida nos arquivos de configuração.


```json
// src/CashFlow.API/appsettings.Development.json
{
  "ConnectionStrings": {
    "Connection": "Server=localhost;Database=cashflowdb;Uid=root;Pwd=3453"
  }
}
```
- **`"ConnectionStrings"`**: Uma seção padrão nos arquivos de configuração do .NET para agrupar strings de conexão.
  
- **`"Connection": "..."`**: A chave `"Connection"` corresponde ao nome que será usado no código para obter a string de conexão. O valor é a string de conexão real com os detalhes do servidor, banco de dados, usuário e senha.

# 3 **Configurando o `DbContext` com a String de Conexão na Injeção de Dependência**

> [!NOTE]
    > Com a string de conexão definida nas configurações, o próximo passo é configurar o Entity Framework Core para usar essa string ao criar instâncias do `CashFlowDbContext`. Isso é feito durante a configuração dos serviços na inicialização da aplicação, tipicamente em um método de extensão dentro da camada de Infrastructure.


```csharp
// Dentro de src/CashFlow.Infrastructure/DependencyInjectionExtension.cs
private static void AddDbContext(IServiceCollection services, IConfiguration configuration)
{
    // 1. Obtém a string de conexão do arquivo de configuração
    var connectionString = configuration.GetConnectionString("Connection"); ;

    // 2. Define a versão do servidor MySQL (necessário para alguns provedores como Pomelo)
    //    Você pode precisar ajustar isso dependendo da sua versão do MySQL.
    var version = new Version(8, 0, 37);
    var serverVesion = new MySqlServerVersion(version);

    // 3. Registra o DbContext no contêiner de DI e configura o provedor e a string de conexão
    services.AddDbContext<CashFlowDbContext>(config => config.UseMySql(connectionString, serverVesion));
}

```

- **`config => config.UseMySql(connectionString, serverVesion)`**: A lambda de configuração (`config => ...`) especifica como o `DbContext` deve ser configurado. `config.UseMySql(...)` (disponível após instalar o pacote do provedor MySQL para EF Core, como `Pomelo.EntityFrameworkCore.MySql`) instrui o EF Core a usar o MySQL e fornece a string de conexão e a versão do servidor necessárias para estabelecer a conexão.

# 4 **Injetando e Utilizando o `DbContext` nos Repositórios**

> [!NOTE]
    > As implementações dos **Repositórios** na camada **Infrastructure** recebem a instância configurada do `CashFlowDbContext` via Injeção de Dependência e a utilizam para interagir com o banco de dados através dos `DbSet`


```csharp
// src/CashFlow.Infrastructure/DataAccess/Repositories/ExpensesRepository.cs
using CashFlow.Domain.Entities;
using CashFlow.Domain.Repositories.Expenses;
using Microsoft.EntityFrameworkCore;

namespace CashFlow.Infrastructure.DataAccess.Repositories;

internal class ExpensesRepository : IExpensesRepository // Implementa a interface do Domain
{
    private readonly CashFlowDbContext _dbContext; // Declara uma referência para o DbContext

    // Injeção de Dependência no construtor: O DI Container fornece uma instância de CashFlowDbContext
    public ExpensesRepository(CashFlowDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    // Método para adicionar uma despesa usando o DbContext
    public async  Task Add(Expense expense)
    {
        await _dbContext.Expenses.AddAsync(expense); // Usa a propriedade DbSet para adicionar a entidade
    }

    // Método para obter todas as despesas usando o DbContext
    public async Task<List<Expense>> GetAlll()
    {
        return await _dbContext.Expenses.AsNoTracking().ToListAsync(); // Consulta o DbSet
    }
}
```

- **`public ExpensesRepository(CashFlowDbContext dbContext)`**: O `ExpensesRepository` solicita uma instância de `CashFlowDbContext` via injeção de dependência no seu construtor. O contêiner de DI, que já configurou o `CashFlowDbContext` no Passo 3, fornece uma instância pronta para uso.
  
- **`_dbContext.Expenses.AddAsync(expense)` e `_dbContext.Expenses.AsNoTracking().ToListAsync()`**: Dentro dos métodos do repositório, a instância injetada do `_dbContext` é utilizada para realizar as operações de persistência. `_dbContext.Expenses` acessa o `DbSet` para a entidade `Expense`, permitindo que o Entity Framework Core traduza essas operações em comandos SQL e os execute no banco de dados configurado.

# 5 **Gerenciando Transações com o Unit of Work**

> [!NOTE]
    > Seu propósito é orquestrar o salvamento das mudanças rastreadas pelo `DbContext` em uma transação única, sendo chamado pela camada de negócio (Application) após todas as operações de persistência de uma determinada funcionalidade terem sido preparadas usando os repositórios.


O `UnitOfWork` não está envolvido na **configuração inicial da conexão** (Passos 1-3) nem em como os **repositórios usam o `DbContext` para preparar operações** (Passo 4).

Ele se encaixa na camada **Application (Use Cases)**. Um Use Case que precisa realizar múltiplas operações de persistência (por exemplo, adicionar uma despesa e atualizar um saldo em outra tabela) usará um ou mais repositórios para preparar essas operações (ex: `_expensesRepository.Add(expense)` e `_balanceRepository.Update(balance)`).

Após preparar _todas_ as operações necessárias para completar a funcionalidade, o Use Case chama o método `Commit()` do `IUnitOfWork` (que ele também recebeu via DI). É neste ponto que a `UnitOfWork` chama `_dbContext.SaveChangesAsync()`, salvando _todas_ as mudanças preparadas pelos diferentes repositórios através da mesma instância do `DbContext` em uma única transação.


```csharp
// src/CashFlow.Infrastructure/DataAccess/UnitOfWork.cs
using CashFlow.Domain.Repositories;

namespace CashFlow.Infrastructure.DataAccess;

public class UnitOfWork : IUnitOfWork // Implementa a interface do Domain
{
    private readonly CashFlowDbContext _dbContext; // Declara uma referência para o DbContext

    // Injeção de Dependência no construtor: Recebe a instância do DbContext
    public UnitOfWork(CashFlowDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    // Implementação do método Commit que chama SaveChangesAsync() do DbContext
    public async Task Commit() => await _dbContext.SaveChangesAsync();

}
```

**Objetivo do Unit of Work no Projeto:**

- **Atomicidade:** Garante que um conjunto de operações de persistência relacionadas ocorra como uma única unidade. Se qualquer parte falhar, toda a transação é revertida. Isso mantém a integridade dos dados.
  
- **Abstração:** A camada Application (Use Cases) interage com a interface `IUnitOfWork`, não diretamente com o `DbContext.SaveChangesAsync()`. Isso desacopla a lógica de negócio do detalhe de infraestrutura de como o salvamento é feito.
  
- **Controle Transacional:** Permite que a camada de negócio controle explicitamente quando uma unidade de trabalho (uma transação) deve ser concluída.