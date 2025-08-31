---
aliases:
sticker: lucide//file-text
---
# Camadas Principais (Core)

   1. `CashFlow.Domain`: Este é o coração da sua aplicação.
       * Responsabilidade: Contém as regras de negócio mais puras e centrais. Aqui ficam as entidades (como Expense.cs), enums e,
         crucialmente, as interfaces dos repositórios (IExpensesRepository.cs, IUnitOfWork.cs).
       * Regra Principal: Este projeto não deve depender de nenhum outro projeto da solução. Ele é o centro do universo da sua aplicação.

   2. `CashFlow.Aplication`: A camada de aplicação.
       * Responsabilidade: Orquestrar o fluxo de negócio. Ela contém os Use Cases (casos de uso), como RegisterExpenseUseCase.cs. É aqui
         que a lógica da aplicação é executada: ela recebe dados da camada de apresentação, utiliza o domínio para executar as regras de
         negócio e invoca a infraestrutura (através de interfaces) para persistir os dados.
       * Dependências: Depende do CashFlow.Domain, mas não conhece detalhes de infraestrutura (como o banco de dados) ou da API.
 
# Camadas Externas (External)

   3. `CashFlow.API`: A camada de apresentação (Presentation Layer).
       * Responsabilidade: Expor a funcionalidade da aplicação para o mundo exterior, neste caso, através de uma API REST. Ela contém os
         Controllers (como ExpensesController.cs) que recebem as requisições HTTP, as validam (superficialmente) e chamam os casos de uso
          na camada de aplicação.
       * Dependências: Depende do CashFlow.Aplication e do CashFlow.Communication.

   4. `CashFlow.Infrastructure`: A camada de infraestrutura.
       * Responsabilidade: Implementar os detalhes técnicos e lidar com preocupações externas. Isso inclui acesso a banco de dados
         (CashFlowDbContext.cs, ExpensesRepository.cs), comunicação com outros sistemas, logging, etc. Ela contém as implementações 
         concretas das interfaces definidas no CashFlow.Domain (como ExpensesRepository implementando IExpensesRepository).
       * Dependências: Depende do CashFlow.Domain e CashFlow.Aplication para implementar suas interfaces e extensões.

# Projetos de Suporte (Cross-Cutting Concerns)

   5. `CashFlow.Communication`: Projeto de Contratos de Dados (DTOs).
       * Responsabilidade: Definir os objetos de transferência de dados (Data Transfer Objects - DTOs) que são usados para a comunicação
         entre a API e os clientes. As pastas Requests e Responses deixam isso claro. Ele garante que a camada de apresentação e o
         domínio não estejam firmemente acoplados.
       * Uso: Usado principalmente pelo CashFlow.API e CashFlow.Aplication.

   6. `CashFlow.Exception`: Gerenciamento centralizado de exceções.
       * Responsabilidade: Centralizar as exceções customizadas da aplicação (CashFlowException.cs) e o tratamento de mensagens de erro,
         inclusive com suporte a múltiplos idiomas (como indicam os arquivos .resx). Isso padroniza o tratamento de erros em todo o
         sistema.

# Projetos de Teste

   7. `Validators.Test`: Testes para os validadores.
       * Responsabilidade: Contém testes unitários focados especificamente nos validadores da aplicação, como o RegisterExpenseValidator.

   8. `CommonTestUtilities`: Utilitários para os testes.
       * Responsabilidade: Fornecer código de ajuda para a criação de testes, como o RequestRegisterExpenseJsonBuilder.cs. O padrão
         Builder é excelente aqui para criar objetos complexos necessários para os cenários de teste, tornando os testes mais legíveis e
         fáceis de manter.****



# Estrutura Detalhada do Projeto (src)  
O diagrama a seguir detalha a responsabilidade de cada diretório dentro da pasta src, ilustrando o fluxo de dados e a separação de responsabilidades da arquitetura.  
  
```  
src/  
│  
├───Base.API/  (Ponto de Entrada da Aplicação)  
│   │  
│   ├───Controllers/  
│   │   └─── ExpensesController.cs: Define os endpoints da API (ex: /api/expenses). Recebe requisições HTTP, extrai dados (do corpo, da rota) e invoca o Caso de Uso correspondente na camada de Aplicação. Ele é o "porteiro" da sua API.  
│   │  
│   ├───Filters/  
│   │   └─── ExceptionFilter.cs: Um filtro global que captura TODAS as exceções que ocorrem na aplicação. Ele inspeciona o tipo da exceção e formata uma resposta de erro padronizada (JSON), retornando o status code apropriado (400 para erros de validação, 500 para erros inesperados, etc.). Centraliza o tratamento de erros.  
│   │  
│   ├───Middleware/  
│   │   └─── CultureMiddleware.cs: Intercepta cada requisição para ler o header `Accept-Language`. Com base nesse valor, define a cultura (idioma e formato regional) da thread atual. Isso permite que a aplicação, especialmente o projeto `Base.Exception`, retorne mensagens de erro no idioma solicitado pelo cliente (pt-BR, en-US, fr, etc.).  
│   │  
│   ├───Properties/  
│   │   └─── launchSettings.json: Arquivo de configuração do ambiente de desenvolvimento. Define perfis para executar a aplicação localmente, especificando URLs, portas e variáveis de ambiente (como `ASPNETCORE_ENVIRONMENT`).  
│   │  
│   ├─── appsettings.json / appsettings.Development.json: Arquivos de configuração da aplicação. O `.json` principal contém configurações padrão (ex: nível de log), enquanto o `.Development.json` contém configurações específicas para o ambiente de desenvolvimento, como a `ConnectionStrings` do banco de dados.  
│   │  
│   └─── Program.cs: O coração da inicialização da API. É aqui que tudo é configurado:  
│       - Registra os serviços para injeção de dependência (Controllers, Swagger, Filtros).  
│       - Chama os métodos de extensão `AddAplication()` e `AddInfrastructure()` para registrar todas as dependências dessas camadas.  
│       - Configura o pipeline de requisições HTTP (a ordem em que os middlewares, como o `CultureMiddleware`, são executados).  
│  
├───Base.Application/  (Orquestração e Lógica da Aplicação)  
│   │  
│   ├───AutoMapper/  
│   │   └─── AutoMapping.cs: Configura o AutoMapper. Define "receitas" para converter objetos de um tipo para outro automaticamente (ex: de um `RequestRegisterExpenseJson` para uma entidade `Expense`). Evita código repetitivo de atribuição de propriedades.  
│   │  
│   ├───UseCases/  (Implementação dos Casos de Uso de Negócio)  
│   │   └───Expenses/  
│   │       ├───GetAll/: Contém a lógica para buscar todas as despesas. A interface `IGetAllExpenseUseCase` define o contrato, e a classe `GetAllExpenseUseCase` o implementa, usando o repositório para buscar os dados e o AutoMapper para formatar a resposta.  
│   │       └───Register/: Contém a lógica para registrar uma nova despesa.  
│   │           ├─── IRegisterExpenseUseCase.cs: A interface (o "o quê").  
│   │           ├─── RegisterExpenseUseCase.cs: A implementação (o "como"). Orquestra a validação, o mapeamento do DTO para a entidade, a adição no repositório e o commit da transação.  
│   │           └─── RegisterExpenseValidator.cs: Usando `FluentValidation`, define as regras de validação para uma nova despesa (ex: título é obrigatório, valor > 0). Lança uma `ErrorOnValidationException` se as regras não forem atendidas.  
│   │  
│   └─── DependencyInjectionExtension.cs: Define o método de extensão `AddAplication()`. Sua única responsabilidade é agrupar o registro de todos os serviços desta camada (Casos de Uso, AutoMapper) para a injeção de dependência, mantendo o `Program.cs` da API mais limpo.  
│  
├───Base.Communication/  (Contratos de Comunicação - DTOs)  
│   │  
│   ├───Enums/: Contém enums (`PaymentType`) que são parte do contrato público da API.  
│   │  
│   ├───Requests/: Classes (DTOs) que modelam os dados que a API ESPERA RECEBER dos clientes. Ex: `RequestRegisterExpenseJson`.  
│   │  
│   └───Responses/: Classes (DTOs) que modelam os dados que a API VAI ENVIAR para os clientes. Ex: `ResponseRegisteredExpenseJson`. O uso de DTOs desacopla a representação externa dos dados do seu modelo de domínio interno.  
│  
├───Base.Domain/  (O Coração do Negócio - Regras e Entidades)  
│   │  
│   ├───Entities/  
│   │   └─── Expense.cs: Representa uma despesa no seu domínio de negócio. É uma classe "pura", contendo apenas as propriedades que definem uma despesa. Não sabe sobre banco de dados ou API.  
│   │  
│   ├───Enums/: Contém enums (`PaymentType`) que são parte integrante do modelo de domínio.  
│   │  
│   └───Repositories/  
│       ├─── IUnitOfWork.cs: Define o contrato para o padrão "Unit of Work", que garante que múltiplas operações de banco de dados sejam salvas em uma única transação atômica através do método `Commit()`.  
│       └───Expenses/  
│           └─── IExpensesRepository.cs: Define o contrato para o repositório de despesas. Diz "o que" pode ser feito com as despesas (ex: `Add`, `GetAlll`), mas não "como". A camada de aplicação depende desta abstração, não da implementação concreta.  
│  
├───Base.Exception/  (Gerenciamento de Erros e Mensagens)  
│   │  
│   ├───ExceptionsBase/: Contém as classes base para as exceções customizadas da aplicação. `BaseException` é a base de todas, e `ErrorOnValidationException` é uma especialização para carregar uma lista de erros de validação.  
│   │  
│   └─── ResourceErrorMessages.*.resx / .Designer.cs: Arquivos de recursos para internacionalização (i18n). O `.resx` armazena as mensagens de erro em diferentes idiomas (pt-BR, fr, etc.), e o `.Designer.cs` é uma classe gerada que permite acessar essas mensagens de forma fortemente tipada (ex: `ResourceErrorMessages.TITLE_REQUIRED`).  
│  
└───Base.Infrastructure/  (Detalhes de Implementação - Mundo Externo)  
    │    ├───DataAccess/    │   ├───Repositories/    │   │   └─── ExpensesRepository.cs: A IMPLEMENTAÇÃO CONCRETA da interface `IExpensesRepository`. É aqui que a lógica de acesso a dados realmente acontece, usando o Entity Framework Core (`_dbContext`) para interagir com o banco de dados MySQL.│   │   ├─── UnitOfWork.cs: A IMPLEMENTAÇÃO CONCRETA da interface `IUnitOfWork`. Sua implementação do método `Commit()` simplesmente chama `_dbContext.SaveChangesAsync()`.  
│   │   └─── BaseDbContext.cs: A representação do seu banco de dados em código (contexto do Entity Framework). A propriedade `DbSet<Expense>` mapeia a entidade `Expense` para a tabela "Expenses" no banco.  
│   │  
│   └─── DependencyInjectionExtension.cs: Define o método de extensão `AddInfrastructure()`. Responsável por registrar as implementações concretas desta camada (DbContext, Repositórios, UnitOfWork) no container de injeção de dependência, associando as interfaces (ex: `IExpensesRepository`) às suas implementações (ex: `ExpensesRepository`).  
  
```  
  
# Fluxo de Desenvolvimento: Do Domínio para a API  
---  
## 📋 Índice Completo  
  
1. [Preparação do Ambiente e Projeto](#1-preparação-do-ambiente-e-projeto)  
2. [Domain - A Base de Tudo](#2-domain---a-base-de-tudo)  
3. [Communication - Contratos de Comunicação](#3-communication---contratos-de-comunicação)  
4. [Exception - Sistema de Erros](#4-exception---sistema-de-erros)  
5. [Application - Casos de Uso](#5-application---casos-de-uso)  
6. [Infrastructure - Acesso a Dados](#6-infrastructure---acesso-a-dados)  
7. [API - Endpoints HTTP](#7-api---endpoints-http)  
8. [Configuração Final e Testes](#8-configuração-final-e-testes)  
  
---  
  
## 1. 🏗️ Preparação do Ambiente e Projeto  
  
### Para Projeto Novo  
  
#### Estrutura de Pastas  
```bash  
# Criar solução  
dotnet new sln -n CashFlow  
  
# Criar estrutura de pastas  
mkdir src  
mkdir tests  
  
# Criar projetos  
cd src  
dotnet new classlib -n CashFlow.Domain  
dotnet new classlib -n CashFlow.Communication  
dotnet new classlib -n CashFlow.Exception  
dotnet new classlib -n CashFlow.Application  # Note: No projeto original está "Aplication" (typo)  
dotnet new classlib -n CashFlow.Infrastructure  
dotnet new webapi -n CashFlow.API  
  
cd ../tests  
dotnet new xunit -n Validators.Test  
dotnet new classlib -n CommonTestUtilities  
  
# Adicionar projetos à solução  
cd ..  
dotnet sln add src/CashFlow.Domain/CashFlow.Domain.csproj  
dotnet sln add src/CashFlow.Communication/CashFlow.Communication.csproj  
dotnet sln add src/CashFlow.Exception/CashFlow.Exception.csproj  
dotnet sln add src/CashFlow.Application/CashFlow.Application.csproj  
dotnet sln add src/CashFlow.Infrastructure/CashFlow.Infrastructure.csproj  
dotnet sln add src/CashFlow.API/CashFlow.API.csproj  
dotnet sln add tests/Validators.Test/Validators.Test.csproj  
dotnet sln add tests/CommonTestUtilities/CommonTestUtilities.csproj  
```  
  
#### Configurar Referências Entre Projetos  
```bash  
# Domain não referencia ninguém (camada mais interna)  
  
# Communication não referencia ninguém  
cd src/CashFlow.Communication  
  
# Exception não referencia ninguém  
cd ../CashFlow.Exception  
  
# Application referencia Communication, Domain e Exception  
cd ../CashFlow.Application  
dotnet add reference ../CashFlow.Communication/CashFlow.Communication.csproj  
dotnet add reference ../CashFlow.Domain/CashFlow.Domain.csproj  
dotnet add reference ../CashFlow.Exception/CashFlow.Exception.csproj  
  
# Infrastructure referencia Domain  
cd ../CashFlow.Infrastructure  
dotnet add reference ../CashFlow.Domain/CashFlow.Domain.csproj  
  
# API referencia Application, Communication, Exception e Infrastructure  
cd ../CashFlow.API  
dotnet add reference ../CashFlow.Application/CashFlow.Application.csproj  
dotnet add reference ../CashFlow.Communication/CashFlow.Communication.csproj  
dotnet add reference ../CashFlow.Exception/CashFlow.Exception.csproj  
dotnet add reference ../CashFlow.Infrastructure/CashFlow.Infrastructure.csproj  
```  
  
### Para Nova Feature em Projeto Existente  
  
Antes de começar, certifique-se de:  
- [ ] Ter o projeto clonado e funcionando  
- [ ] Banco de dados MySQL configurado e rodando  
- [ ] Entender a estrutura atual do projeto  
- [ ] Ter as migrations aplicadas: `dotnet ef database update`  
  
---  
  
## 2. 🎯 Domain - A Base de Tudo  
  
O Domain é o coração da aplicação. Aqui definimos as entidades e contratos sem nenhuma dependência externa.  
  
### 📝 Checklist Detalhado  
  
#### A. Criar/Atualizar Entidade  
  
**Localização:** `src/CashFlow.Domain/Entities/`  
  
```csharp  
// Exemplo: src/CashFlow.Domain/Entities/Expense.cs  
using CashFlow.Domain.Enums;  
  
namespace CashFlow.Domain.Entities;  
  
public class Expense  
{  
    public long Id { get; set; }    public string Title { get; set; } = string.Empty;    public string? Description { get; set; }  // Note o uso de nullable    public DateTime Date { get; set; }    public decimal? Amount { get; set; }    public PaymentType PaymentType { get; set; }}  
```  
  
**Dicas para o Júnior:**  
- Use `string.Empty` para inicializar strings obrigatórias  
- Use `?` para propriedades opcionais (nullable)  
- `long` para IDs é uma boa prática (suporta mais registros que `int`)  
  
#### B. Criar Enums (se necessário)  
  
**Localização:** `src/CashFlow.Domain/Enums/`  
  
```csharp  
// Exemplo: src/CashFlow.Domain/Enums/PaymentType.cs  
namespace CashFlow.Domain.Enums;  
  
public enum PaymentType  
{  
    Cash = 0,    CreditCard = 1,    DebitCard = 2,    EletronicTransfer = 3,  // Note: Typo intencional mantido por compatibilidade}  
```  
  
#### C. Definir Interface do Repositório  
  
**Localização:** `src/CashFlow.Domain/Repositories/[NomeDaEntidade]/`  
  
```csharp  
// Exemplo: src/CashFlow.Domain/Repositories/Expenses/IExpensesRepository.cs  
using CashFlow.Domain.Entities;  
  
namespace CashFlow.Domain.Repositories.Expenses;  
  
public interface IExpensesRepository  
{  
    Task Add(Expense expense);    Task<List<Expense>> GetAlll();  // Note: Typo no original (três 'l')    Task<Expense?> GetById(long id);    Task Update(Expense expense);    Task Delete(long id);    Task<bool> ExistActiveExpenseForUser(long userId);}  
```  
  
**⚠️ Importante:** Sempre use `Task` para operações assíncronas!  
  
#### D. Interface Unit of Work  
  
**Localização:** `src/CashFlow.Domain/Repositories/IUnitOfWork.cs`  
  
```csharp  
namespace CashFlow.Domain.Repositories;  
  
public interface IUnitOfWork  
{  
    Task Commit();}  
```  
  
---  
  
## 3. 📡 Communication - Contratos de Comunicação  
  
Define os DTOs (Data Transfer Objects) para comunicação com o mundo externo.  
  
### 📝 Checklist Detalhado  
  
#### A. Criar Request DTOs  
  
**Localização:** `src/CashFlow.Communication/Requests/`  
  
```csharp  
// Exemplo: src/CashFlow.Communication/Requests/RequestRegisterExpenseJson.cs  
using CashFlow.Communication.Enums;  
  
namespace CashFlow.Communication.Requests;  
  
public class RequestRegisterExpenseJson  
{  
    public string Title { get; set; } = String.Empty;    public string Description { get; set; } = String.Empty;    public DateTime Date { get; set; }    public decimal Amount { get; set; }    public PaymentType PaymentType { get; set; }}  
```  
  
**💡 Dica:** Note que temos enums duplicados em Communication e Domain. Isso é intencional para manter as camadas independentes!  
  
#### B. Criar Response DTOs  
  
**Localização:** `src/CashFlow.Communication/Responses/`  
  
```csharp  
// Para resposta simples  
public class ResponseRegisteredExpenseJson  
{  
    public string Title { get; set; } = String.Empty;}  
  
// Para lista de itens  
public class ResponseExpenseJson  
{  
    public List<ResponseShortExpenseJson> Expenses { get; set; } = new List<ResponseShortExpenseJson>();}  
  
// Para item resumido  
public class ResponseShortExpenseJson  
{  
    public long Id { get; set; }    public string Title { get; set; }    public decimal Amount { get; set; }}  
```  
  
#### C. Criar Response de Erro  
  
```csharp  
// src/CashFlow.Communication/Responses/ResponseErrorJson.cs  
namespace CashFlow.Communication.Responses;  
  
public class ResponseErrorJson  
{  
    public List<string> ErrorMessages { get; set; }  
    // Construtor para erro único    public ResponseErrorJson(string errorMessage)    {        ErrorMessages = new List<string> { errorMessage };    }  
    // Construtor para múltiplos erros    public ResponseErrorJson(List<string> errorMessage)    {        ErrorMessages = errorMessage;    }}  
```  
  
#### D. Duplicar Enums (se necessário)  
  
```csharp  
// src/CashFlow.Communication/Enums/PaymentType.cs  
namespace CashFlow.Communication.Enums;  
  
public enum PaymentType  
{  
    Cash = 0,    CreditCard = 1,    DebitCard = 2,    EletronicTransfer = 3,}  
```  
  
---  
  
## 4. ⚠️ Exception - Sistema de Erros  
  
Gerencia exceções customizadas e mensagens internacionalizadas.  
  
### 📝 Checklist Detalhado  
  
#### A. Criar Exceção Base (se não existir)  
  
```csharp  
// src/CashFlow.Exception/ExceptionsBase/CashFlowException.cs  
namespace CashFlow.Exception.ExceptionsBase;  
  
public abstract class CashFlowException : SystemException  
{  
    // Classe base abstrata para todas as exceções do projeto}  
```  
  
#### B. Criar Exceções Específicas  
  
```csharp  
// src/CashFlow.Exception/ExceptionsBase/ErrorOnValidationException.cs  
namespace CashFlow.Exception.ExceptionsBase;  
  
public class ErrorOnValidationException : CashFlowException  
{  
    public List<string> Errors { get; set; }        public ErrorOnValidationException(List<string> errorMessages)  
    {        Errors = errorMessages;    }}  
```  
  
#### C. Configurar Recursos de Mensagens  
  
**Passo 1:** Adicionar mensagens no arquivo de recursos principal  
  
```xml  
<!-- src/CashFlow.Exception/ResourceErrorMessages.resx -->  
<data name="TITLE_REQUIRED" xml:space="preserve">  
    <value>The title is required</value></data>  
<data name="AMOUNT_MUST_BE_GREATER_THAN_ZERO" xml:space="preserve">  
    <value>The Amount must be greater than zero.</value></data>  
```  
  
**Passo 2:** Adicionar traduções  
  
```xml  
<!-- src/CashFlow.Exception/ResourceErrorMessages.pt-BR.resx -->  
<data name="TITLE_REQUIRED" xml:space="preserve">  
    <value>O titulo é obrigatório</value></data>  
```  
  
**Passo 3:** Configurar o .csproj  
  
```xml  
<!-- src/CashFlow.Exception/CashFlow.Exception.csproj -->  
<ItemGroup>  
    <Compile Update="ResourceErrorMessages.Designer.cs">        <DesignTime>True</DesignTime>        <AutoGen>True</AutoGen>        <DependentUpon>ResourceErrorMessages.resx</DependentUpon>    </Compile></ItemGroup>  
  
<ItemGroup>  
    <EmbeddedResource Update="ResourceErrorMessages.resx">        <Generator>PublicResXFileCodeGenerator</Generator>        <LastGenOutput>ResourceErrorMessages.Designer.cs</LastGenOutput>    </EmbeddedResource></ItemGroup>  
```  
  
---  
  
## 5. 🧠 Application - Casos de Uso  
  
Implementa a lógica de negócio e orquestra as operações.  
  
### 📝 Checklist Detalhado  
  
#### A. Instalar Pacotes Necessários  
  
```bash  
cd src/CashFlow.Applicationdotnet add package AutoMapper --version 13.0.1dotnet add package FluentValidation --version 11.9.0dotnet add package Microsoft.Extensions.DependencyInjection --version 8.0.0```  
  
#### B. Configurar AutoMapper  
  
```csharp  
// src/CashFlow.Application/AutoMapper/AutoMapping.cs  
using AutoMapper;  
using CashFlow.Communication.Requests;  
using CashFlow.Communication.Responses;  
using CashFlow.Domain.Entities;  
  
namespace CashFlow.Application.AutoMapper;  
  
public class AutoMapping : Profile  
{  
    public AutoMapping()    {        RequestToEntity();        EntityToResponse();    }        private void RequestToEntity()  
    {        // Request -> Entity        CreateMap<RequestRegisterExpenseJson, Expense>();    }  
    private void EntityToResponse()    {        // Entity -> Response        CreateMap<Expense, ResponseRegisteredExpenseJson>();        CreateMap<Expense, ResponseShortExpenseJson>();    }}  
```  
  
#### C. Criar Validator  
  
```csharp  
// src/CashFlow.Application/UseCases/Expenses/Register/RegisterExpenseValidator.cs  
using CashFlow.Communication.Requests;  
using CashFlow.Exception;  
using FluentValidation;  
  
namespace CashFlow.Application.UseCases.Expenses.Register;  
  
public class RegisterExpenseValidator : AbstractValidator<RequestRegisterExpenseJson>  
{  
    public RegisterExpenseValidator()    {        RuleFor(expense => expense.Title)            .NotEmpty()            .WithMessage(ResourceErrorMessages.TITLE_REQUIRED);                    RuleFor(expense => expense.Amount)  
            .GreaterThan(0)            .WithMessage(ResourceErrorMessages.AMOUNT_MUST_BE_GREATER_THAN_ZERO);                    RuleFor(expense => expense.Date)  
            .LessThanOrEqualTo(DateTime.UtcNow)            .WithMessage(ResourceErrorMessages.EXPENSE_CANNOT_FOR_THE_FUTURE);                    RuleFor(expense => expense.PaymentType)  
            .IsInEnum()            .WithMessage(ResourceErrorMessages.PAYMENT_TYPE_INVALID);    }}  
```  
  
#### D. Criar Interface do Use Case  
  
```csharp  
// src/CashFlow.Application/UseCases/Expenses/Register/IRegisterExpenseUseCase.cs  
using CashFlow.Communication.Requests;  
using CashFlow.Communication.Responses;  
  
namespace CashFlow.Application.UseCases.Expenses.Register;  
  
public interface IRegisterExpenseUseCase  
{  
    Task<ResponseRegisteredExpenseJson> Execute(RequestRegisterExpenseJson request);}  
```  
  
#### E. Implementar Use Case  
  
```csharp  
// src/CashFlow.Application/UseCases/Expenses/Register/RegisterExpenseUseCase.cs  
using AutoMapper;  
using CashFlow.Communication.Requests;  
using CashFlow.Communication.Responses;  
using CashFlow.Domain.Entities;  
using CashFlow.Domain.Repositories;  
using CashFlow.Domain.Repositories.Expenses;  
using CashFlow.Exception.ExceptionsBase;  
  
namespace CashFlow.Application.UseCases.Expenses.Register;  
  
public class RegisterExpenseUseCase : IRegisterExpenseUseCase  
{  
    private readonly IExpensesRepository _repository;    private readonly IUnitOfWork _unitOfWork;    private readonly IMapper _mapper;        public RegisterExpenseUseCase(  
        IExpensesRepository repository,        IUnitOfWork unitOfWork,  
        IMapper mapper)    {        _repository = repository;        _unitOfWork = unitOfWork;        _mapper = mapper;    }  
    public async Task<ResponseRegisteredExpenseJson> Execute(RequestRegisterExpenseJson request)    {        // 1. Validar dados de entrada        Validate(request);  
        // 2. Mapear DTO para Entidade        var entity = _mapper.Map<Expense>(request);  
        // 3. Adicionar ao repositório        await _repository.Add(entity);  
        // 4. Confirmar transação        await _unitOfWork.Commit();  
        // 5. Retornar resposta mapeada        return _mapper.Map<ResponseRegisteredExpenseJson>(entity);    }  
    private void Validate(RequestRegisterExpenseJson request)    {        var validator = new RegisterExpenseValidator();        var result = validator.Validate(request);  
        if (result.IsValid == false)        {            var errorMessages = result.Errors.Select(f => f.ErrorMessage).ToList();            throw new ErrorOnValidationException(errorMessages);        }    }}  
```  
  
#### F. Configurar Injeção de Dependência  
  
```csharp  
// src/CashFlow.Application/DependencyInjectionExtension.cs  
using CashFlow.Application.AutoMapper;  
using CashFlow.Application.UseCases.Expenses.GetAll;  
using CashFlow.Application.UseCases.Expenses.Register;  
using Microsoft.Extensions.DependencyInjection;  
  
namespace CashFlow.Application;  
  
public static class DependencyInjectionExtension  
{  
    public static void AddAplication(this IServiceCollection services)    {        AddAutoMapper(services);        AddUseCases(services);    }  
    private static void AddAutoMapper(IServiceCollection services)    {        services.AddAutoMapper(typeof(AutoMapping));    }        private static void AddUseCases(IServiceCollection services)  
    {        services.AddScoped<IRegisterExpenseUseCase, RegisterExpenseUseCase>();        services.AddScoped<IGetAllExpenseUseCase, GetAllExpenseUseCase>();    }}  
```  
  
---  
  
## 6. 🔧 Infrastructure - Acesso a Dados  
  
Implementa o acesso ao banco de dados usando Entity Framework Core.  
  
### 📝 Checklist Detalhado  
  
#### A. Instalar Pacotes  
  
```bash  
cd src/CashFlow.Infrastructuredotnet add package Microsoft.EntityFrameworkCore --version 7.0.20dotnet add package Pomelo.EntityFrameworkCore.MySql --version 7.0.0```  
  
#### B. Criar DbContext  
  
```csharp  
// src/CashFlow.Infrastructure/DataAccess/CashFlowDbContext.cs  
using CashFlow.Domain.Entities;  
using Microsoft.EntityFrameworkCore;  
  
namespace CashFlow.Infrastructure.DataAccess;  
  
public class CashFlowDbContext : DbContext  
{  
    public CashFlowDbContext(DbContextOptions options) : base(options)    {    }  
    public DbSet<Expense> Expenses { get; set; }  
    // Configurações adicionais se necessário    protected override void OnModelCreating(ModelBuilder modelBuilder)    {        base.OnModelCreating(modelBuilder);                // Exemplo de configuração  
        modelBuilder.Entity<Expense>()            .Property(e => e.Amount)            .HasPrecision(18, 2);    }}  
```  
  
#### C. Implementar Repository  
  
```csharp  
// src/CashFlow.Infrastructure/DataAccess/Repositories/ExpensesRepository.cs  
using CashFlow.Domain.Entities;  
using CashFlow.Domain.Repositories.Expenses;  
using Microsoft.EntityFrameworkCore;  
  
namespace CashFlow.Infrastructure.DataAccess.Repositories;  
  
internal class ExpensesRepository : IExpensesRepository  
{  
    private readonly CashFlowDbContext _dbContext;        public ExpensesRepository(CashFlowDbContext dbContext)  
    {        _dbContext = dbContext;    }        public async Task Add(Expense expense)  
    {        await _dbContext.Expenses.AddAsync(expense);    }  
    public async Task<List<Expense>> GetAlll()    {        // AsNoTracking melhora performance para leitura        return await _dbContext.Expenses.AsNoTracking().ToListAsync();    }        public async Task<Expense?> GetById(long id)  
    {        return await _dbContext.Expenses            .AsNoTracking()            .FirstOrDefaultAsync(expense => expense.Id == id);    }        public async Task Update(Expense expense)  
    {        _dbContext.Expenses.Update(expense);    }        public async Task Delete(long id)  
    {        var expense = await _dbContext.Expenses.FindAsync(id);        if (expense != null)        {            _dbContext.Expenses.Remove(expense);        }    }}  
```  
  
#### D. Implementar Unit of Work  
  
```csharp  
// src/CashFlow.Infrastructure/DataAccess/UnitOfWork.cs  
using CashFlow.Domain.Repositories;  
  
namespace CashFlow.Infrastructure.DataAccess;  
  
public class UnitOfWork : IUnitOfWork  
{  
    private readonly CashFlowDbContext _dbContext;        public UnitOfWork(CashFlowDbContext dbContext)  
    {        _dbContext = dbContext;    }  
    public async Task Commit() => await _dbContext.SaveChangesAsync();}  
```  
  
#### E. Configurar Injeção de Dependência  
  
```csharp  
// src/CashFlow.Infrastructure/DependencyInjectionExtension.cs  
using CashFlow.Domain.Repositories;  
using CashFlow.Domain.Repositories.Expenses;  
using CashFlow.Infrastructure.DataAccess;  
using CashFlow.Infrastructure.DataAccess.Repositories;  
using Microsoft.EntityFrameworkCore;  
using Microsoft.Extensions.Configuration;  
using Microsoft.Extensions.DependencyInjection;  
  
namespace CashFlow.Infrastructure;  
  
public static class DependencyInjectionExtension  
{  
    public static void AddInfrastructure(this IServiceCollection services, IConfiguration configuration)    {        AddDbContext(services, configuration);        AddRepositories(services);    }  
    private static void AddRepositories(IServiceCollection services)    {        services.AddScoped<IUnitOfWork, UnitOfWork>();        services.AddScoped<IExpensesRepository, ExpensesRepository>();    }  
    private static void AddDbContext(IServiceCollection services, IConfiguration configuration)    {        var connectionString = configuration.GetConnectionString("Connection");        var version = new Version(8, 0, 37);        var serverVersion = new MySqlServerVersion(version);  
        services.AddDbContext<CashFlowDbContext>(config =>            config.UseMySql(connectionString, serverVersion));  
    }}  
```  
  
#### F. Criar e Aplicar Migrations  
  
```bash  
# Instalar ferramenta do EF Core (apenas uma vez)  
dotnet tool install --global dotnet-ef  
  
# Criar migration  
cd src/CashFlow.API  
dotnet ef migrations add InitialCreate -p ../CashFlow.Infrastructure -s .  
  
# Aplicar migration  
dotnet ef database update -p ../CashFlow.Infrastructure -s .  
```  
  
---  
  
## 7. 🌐 API - Endpoints HTTP  
  
Expõe os endpoints e configura a aplicação web.  
  
### 📝 Checklist Detalhado  
  
#### A. Configurar appsettings.json  
  
```json  
// src/CashFlow.API/appsettings.Development.json  
{  
  "ConnectionStrings": {    "Connection": "Server=localhost;Database=cashflowdb;Uid=root;Pwd=3453"  },  "Logging": {    "LogLevel": {      "Default": "Information",      "Microsoft.AspNetCore": "Warning"    }  }}  
```  
  
#### B. Criar Exception Filter  
  
```csharp  
// src/CashFlow.API/Filters/ExceptionFilter.cs  
using CashFlow.Communication.Responses;  
using CashFlow.Exception;  
using CashFlow.Exception.ExceptionsBase;  
using Microsoft.AspNetCore.Mvc;  
using Microsoft.AspNetCore.Mvc.Filters;  
  
namespace CashFlow.API.Filters;  
  
public class ExceptionFilter : IExceptionFilter  
{  
    public void OnException(ExceptionContext context)    {        if (context.Exception is CashFlowException)        {            HandleProjectException(context);        }        else        {            ThrowUnknowError(context);        }    }  
    private void HandleProjectException(ExceptionContext context)    {        if (context.Exception is ErrorOnValidationException)        {            var ex = (ErrorOnValidationException)context.Exception;            var errorResponse = new ResponseErrorJson(ex.Errors);                        context.HttpContext.Response.StatusCode = StatusCodes.Status400BadRequest;  
            context.Result = new BadRequestObjectResult(errorResponse);        }        else        {            var errorResponse = new ResponseErrorJson(context.Exception.Message);            context.HttpContext.Response.StatusCode = StatusCodes.Status400BadRequest;            context.Result = new BadRequestObjectResult(errorResponse);        }    }  
    private void ThrowUnknowError(ExceptionContext context)    {        var errorResponse = new ResponseErrorJson(ResourceErrorMessages.UNKNOWN_ERROR);        context.HttpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;        context.Result = new ObjectResult(errorResponse);    }}  
```  
  
#### C. Criar Middleware de Cultura  
  
```csharp  
// src/CashFlow.API/Middleware/CultureMiddleware.cs  
using System.Globalization;  
  
namespace CashFlow.API.Middleware;  
  
public class CultureMiddleware  
{  
    private readonly RequestDelegate _next;        public CultureMiddleware(RequestDelegate next)  
    {        _next = next;    }        public async Task Invoke(HttpContext context)  
    {        var supportedLanguages = CultureInfo.GetCultures(CultureTypes.AllCultures).ToList();        var requestedCulture = context.Request.Headers.AcceptLanguage.FirstOrDefault();  
        var cultureInfo = new CultureInfo("en");  
        if (string.IsNullOrWhiteSpace(requestedCulture) == false            && supportedLanguages.Exists(l => l.Name.Equals(requestedCulture)))  
        {            cultureInfo = new CultureInfo(requestedCulture);        }  
        CultureInfo.CurrentCulture = cultureInfo;        CultureInfo.CurrentUICulture = cultureInfo;  
        await _next(context);    }}  
```  
  
#### D. Criar Controller  
  
```csharp  
// src/CashFlow.API/Controllers/ExpensesController.cs  
using CashFlow.Application.UseCases.Expenses.GetAll;  
using CashFlow.Application.UseCases.Expenses.Register;  
using CashFlow.Communication.Requests;  
using CashFlow.Communication.Responses;  
using Microsoft.AspNetCore.Mvc;  
  
namespace CashFlow.API.Controllers;  
  
[Route("api/[controller]")]  
[ApiController]  
public class ExpensesController : ControllerBase  
{  
    [HttpPost]    [ProducesResponseType(typeof(ResponseRegisteredExpenseJson), StatusCodes.Status201Created)]    [ProducesResponseType(typeof(ResponseErrorJson), StatusCodes.Status400BadRequest)]    public async Task<IActionResult> Register(        [FromServices] IRegisterExpenseUseCase useCase,        [FromBody] RequestRegisterExpenseJson request)    {        var response = await useCase.Execute(request);        return Created(String.Empty, response);    }  
    [HttpGet]    [ProducesResponseType(typeof(ResponseExpenseJson), StatusCodes.Status200OK)]    [ProducesResponseType(StatusCodes.Status204NoContent)]    public async Task<IActionResult> GetAllExpenses(        [FromServices] IGetAllExpenseUseCase useCase)    {        var response = await useCase.Execute();                if (response.Expenses.Count != 0)  
        {            return Ok(response);        }  
        return NoContent();    }}  
```  
  
#### E. Configurar Program.cs  
  
```csharp  
// src/CashFlow.API/Program.cs  
using CashFlow.API.Filters;  
using CashFlow.API.Middleware;  
using CashFlow.Application;  
using CashFlow.Infrastructure;  
  
var builder = WebApplication.CreateBuilder(args);  
  
// Adicionar serviços ao container  
builder.Services.AddControllers();  
builder.Services.AddEndpointsApiExplorer();  
builder.Services.AddSwaggerGen();  
  
// Adicionar filtro de exceções  
builder.Services.AddMvc(options => options.Filters.Add(typeof(ExceptionFilter)));  
  
// Adicionar camadas da aplicação  
builder.Services.AddInfrastructure(builder.Configuration);  
builder.Services.AddAplication();  
  
var app = builder.Build();  
  
// Configurar pipeline  
if (app.Environment.IsDevelopment())  
{  
    app.UseSwagger();    app.UseSwaggerUI();}  
  
app.UseMiddleware<CultureMiddleware>();  
app.UseHttpsRedirection();  
app.UseAuthorization();  
app.MapControllers();  
  
app.Run();  
```  
  
---  
  
## 8. 🧪 Configuração Final e Testes  
  
### 📝 Checklist de Validação  
  
#### A. Testes com Swagger  
  
1. Execute o projeto:  
```bash  
cd src/CashFlow.APIdotnet run```  
  
2. Acesse: `https://localhost:7134/swagger`  
  
3. Teste cada endpoint:  
   - POST com dados válidos  
   - POST com dados inválidos  
   - GET para verificar retorno  
  
#### B. Testes Unitários Básicos  
  
```csharp  
// tests/Validators.Test/Expenses/Register/RegisterExpenseValidatorTest.cs  
using CashFlow.Application.UseCases.Expenses.Register;  
using CashFlow.Communication.Requests;  
using FluentAssertions;  
using Xunit;  
  
public class RegisterExpenseValidatorTest  
{  
    [Fact]    public void Success()    {        // Arrange        var validator = new RegisterExpenseValidator();        var request = new RequestRegisterExpenseJson        {            Title = "Cinema",            Amount = 55,            Date = DateTime.Now.AddDays(-1),            PaymentType = CashFlow.Communication.Enums.PaymentType.Cash        };  
        // Act        var result = validator.Validate(request);  
        // Assert        result.IsValid.Should().BeTrue();    }  
    [Fact]    public void Error_Title_Empty()    {        // Arrange        var validator = new RegisterExpenseValidator();        var request = new RequestRegisterExpenseJson        {            Title = string.Empty,            Amount = 55,            Date = DateTime.Now,            PaymentType = CashFlow.Communication.Enums.PaymentType.Cash        };  
        // Act        var result = validator.Validate(request);  
        // Assert        result.IsValid.Should().BeFalse();        result.Errors.Should().ContainSingle()            .And.Contain(e => e.ErrorMessage.Equals("The title is required"));    }}  
```  
  
**Lembre-se:** A ordem é importante! Sempre comece pelo **Domain** e siga o fluxo sequencialmente para manter a consistência arquitetural.