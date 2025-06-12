---
sticker: lucide//file-text
---
1. **Camada API (`src/CashFlow.API`)**: Esta é a porta de entrada da aplicação. É aqui que as requisições HTTP chegam (como um GET para listar despesas, ou um POST para registrar uma nova). O controlador (`src/CashFlow.API/Controllers/ExpensesController.cs`) recebe a requisição, interage com a camada de Application (que contém a lógica de negócio) e retorna a resposta para o cliente. 
	**Importante:** A camada API _não_ sabe nada sobre como os dados são persistidos. Ela apenas coordena a interação entre a requisição externa e a lógica interna da aplicação.
    
2. **Camada Application**: Esta camada contém os "Casos de Uso" (Use Cases) da aplicação. Um caso de uso representa uma funcionalidade específica que o usuário pode executar (ex: "Registrar Nova Despesa", "Listar Todas as Despesas"). Estes casos de uso <span style="background:rgba(240, 200, 0, 0.2)">contêm a lógica de negócio específica</span> para aquela operação. Eles <span style="background:rgba(240, 107, 5, 0.2)">não contêm lógica de persistência de dados diretamente</span>, mas <span style="background:rgba(3, 135, 102, 0.2)">dependem de interfaces para interagir com o mundo externo, incluindo o acesso a dados</span>.
    
3. **Camada Domain**: Esta é o coração da aplicação, o domínio do negócio.
    
    - **Entidades**: Aqui estão definidos os <span style="background:rgba(240, 200, 0, 0.2)">objetos que representam os dados do negócio</span>. No seu caso, a classe `Expense` representa uma despesa. Uma entidade é puramente um objeto que modela a informação relevante do negócio, sem lógica de infraestrutura ou persistência acoplada.
    - **Interfaces de Repositório**: Esta é uma parte crucial da separação de responsabilidades. <span style="background:rgba(240, 200, 0, 0.2)">A camada Domain define contratos para o acesso a dados.</span> O `IExpensesRepository` diz "Eu preciso de uma forma de adicionar despesas, obter despesas, etc.". A camada **<span style="background:rgba(240, 107, 5, 0.2); font-weight: bold">Domain define o que precisa ser feito com os dados, mas não como</span>**. Isso torna a camada <span style="background:rgba(240, 107, 5, 0.2)">Domain completamente independente da tecnologia de banco de dados</span>.
    - **Interface Unit of Work**: Similar aos repositórios, define um <span style="background:rgba(240, 200, 0, 0.2)">contrato para gerenciar transações de banco de dados</span>, garantindo que um conjunto de operações de persistência seja atômico (ou tudo acontece, ou nada acontece).
4. **Camada Infrastructure (`src/CashFlow.Infrastructure`)**: Esta camada é responsável por<span style="background:rgba(240, 200, 0, 0.2)"> implementar os contratos (interfaces) definidos na camada Domain</span> e lidar com detalhes de infraestrutura, como o acesso ao banco de dados usando Entity Framework Core.
    

**Como o Entity Framework se encaixa na Camada Infrastructure e a relação com Domain?**

O Entity Framework Core (EF Core) é um ORM (Object-Relational Mapper). O objetivo dele é permitir que você trabalhe com dados do banco de dados usando objetos .NET (suas Entidades) em vez de escrever SQL diretamente.

Dentro da sua camada `CashFlow.Infrastructure`, você encontrará os seguintes componentes principais relacionados ao EF Core e à persistência:

1. **`DbContext` (`src/CashFlow.Infrastructure/DataAccess/CashFlowDbContext.cs`)**: Este é o ponto central do EF Core.
    
    - `CashFlowDbContext` herda da classe `DbContext` do Entity Framework.
    - <span style="background:rgba(240, 200, 0, 0.2)">Ele representa uma sessão com o banco de dados</span>.
    - **Ele contém propriedades `DbSet<TEntity>` para cada entidade que você quer mapear para uma tabela no banco. No seu caso, você provavelmente terá um `DbSet<Expense>`**.
    - O `DbContext` é responsável por:
        - Consultar dados do banco (transformando linhas da tabela em objetos `Expense`).
        - Monitorar mudanças nos objetos (inserções, atualizações, exclusões).
        - Gerar e executar comandos SQL apropriados quando você chama `SaveChanges`.
        - Gerenciar o ciclo de vida das entidades.
2. **Implementações dos Repositórios (`src/CashFlow.Infrastructure/DataAccess/Repositories/ExpensesRepository.cs`)**: É aqui que a interface `IExpensesRepository` (definida no Domain) é implementada.
    
    - A classe `ExpensesRepository` recebe uma instância do `CashFlowDbContext` via injeção de dependência.
    - Os métodos desta classe (como `Add`, `GetById`, `GetAll`) utilizam o `_context` (a instância do `DbContext`) para interagir com o banco de dados. Por exemplo, um método `Add(Expense expense)` chamaria `_context.Expenses.Add(expense)`. Uma listagem `GetAll()` chamaria `_context.Expenses.AsNoTracking().ToList()`.
    - <span style="background:rgba(240, 200, 0, 0.2)">Esta classe "traduz" as operações de negócio definidas na interface Domain em operações específicas do Entity Framework Core</span>.
3. **Implementação do Unit of Work (`src/CashFlow.Infrastructure/DataAccess/UnitOfWork.cs`)**: Implementa a interface `IUnitOfWork` do Domain.
    
    - Assim como o repositório, <span style="background:rgba(240, 200, 0, 0.2)">ele recebe uma instância do `CashFlowDbContext`.</span>
    - O método `Commit()` desta classe simplesmente chama `_context.SaveChangesAsync()`. É o `DbContext.SaveChangesAsync()` que realmente <span style="background:rgba(240, 200, 0, 0.2)">executa as operações SQL pendentes no banco de dados, garantindo que todas as mudanças monitoradas sejam salvas como uma única transação</span>.

**A Relação entre Domain e Infrastructure:**

<span style="background:rgba(240, 200, 0, 0.2)">A relação é de **implementação**.</span>

- A camada **Domain** define os **contratos** (interfaces `IExpensesRepository`, `IUnitOfWork`) e as **regras** (entidades `Expense`). Ela é abstrata em relação aos detalhes de persistência.
- A camada **Infrastructure** fornece as **implementações concretas** desses contratos (`ExpensesRepository`, `UnitOfWork`) usando uma tecnologia específica (Entity Framework Core) para interagir com o banco de dados.

A camada Application depende das _interfaces_ do Domain. Na hora que a aplicação é inicializada, a configuração de **Injeção** de Dependência (`src/CashFlow.Infrastructure/DependencyInjectionExtension.cs` e possivelmente na API) registra as implementações da camada Infrastructure para as interfaces da camada Domain.

Por exemplo, quando um Use Case na camada Application (como `RegisterExpenseUseCase`) precisa salvar uma despesa, ele recebe no construtor uma instância de `IExpensesRepository`. Ele chama `_expensesRepository.Add(newExpense);`. O Use Case _não sabe_ que a implementação por trás de `_expensesRepository` é a classe `ExpensesRepository` que usa EF Core. Ele apenas sabe que é algo que sabe como adicionar uma despesa.

Essa abordagem (Dependency Inversion Principle) significa que:

- A camada Domain (o core do seu negócio) não depende da Infrastructure.
- A camada Infrastructure depende do Domain (pois implementa suas interfaces).
- A camada Application depende do Domain (usando suas interfaces e entidades).
- A camada API depende da Application (chamando seus Use Cases).

**Em Resumo:**

1. A camada Domain define suas entidades (`Expense`) e interfaces para acesso a dados (`IExpensesRepository`, `IUnitOfWork`).
2. A camada Infrastructure usa Entity Framework Core para criar um `DbContext` (`CashFlowDbContext`) que mapeia as entidades para o banco de dados.
3. A Infrastructure implementa as interfaces de repositório e Unit of Work do Domain, usando o `DbContext` para realizar as operações de persistência (CRUD e transações).
4. A Injeção de Dependência conecta as interfaces do Domain com as implementações do Infrastructure.
5. A camada Application (Use Cases) utiliza as interfaces de repositório e Unit of Work (recebidas via DI) para interagir com os dados, sem se acoplar ao Entity Framework ou ao banco de dados específico.
6. A camada API chama os Use Cases da camada Application para processar as requisições HTTP.

Essa estrutura garante que a sua lógica de negócio (Domain e Application) seja limpa, testável e independente de detalhes de infraestrutura, como o banco de dados que você está usando (SQLite, SQL Server, etc.). Você poderia, teoricamente, substituir a implementação do repositório na camada Infrastructure para usar outro ORM ou até mesmo acesso direto a SQL, sem precisar mudar o código nas camadas Domain ou Application.
