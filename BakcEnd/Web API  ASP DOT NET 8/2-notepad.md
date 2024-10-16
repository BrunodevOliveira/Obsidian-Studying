Web  API média

# Roteamento
A rota é determinada com base nos atributos definidos nos controladores e métodos Action

[Route("api/[controller]")] ->Definindo a rota no controlador
[ApiController]
public class CategoriasController : ControllerBase{

  [HttpGet("{id}", Name = "ObterCategoria")] -> complementando a rota através da action
  public ActionResult<Categoria> Get(int id){}
}

## Ignorar o nome da rota atribuído em Route
Basta na action adicionar uma '/' no início do nome do endpoint
[HttpGet("/produtos")] --> Assim o endpoint será /p rodutos e não mais api/categorias/produtos


# Métodos Actions Assíncronos
Utilizamos as palavras *async* e *await* junto com o tipo *Task* para transformar as Actions em assíncronas

public async Task<Actionresult><IEnumerable><Categoria>>> GetCategoriasAsync()
{
	return await _context.Categorias.ToListAsync();
}

Essa Action retorna um objeto do tipo Task<Actionresult><IEnumerable><T>>>, faz uso dos modificadores async e await e seu nome termina com Async


# Model Binding - Fonte de dados

1- Valores de formulários: (POST e PUT) enviados no corpo do request

2- Rotas: [Route("api/[Controller]")] ou HttpGet("{id}")

3- Query Strings: api/produtos/4?nome=suco&ativo=true

## Atributos para definir se o Model Bindig vai ocorrer ou não

1- BindRequired - Este atributo adiciona um erro ao ModelState se a vinculação de dados aos parâmetros não puder ocorrer


2- BindNerver - Informa ao Model Binder para não vincular a informação ao parâmetro

public class Categoria 
{
	public int CAtegoriaID
	public string Nome
	[BindNever]
	public string imagemUrl
}


## Atributos que indicam a fonte de dados dos parâmetros

1- FromForm - utilize somente os dados recebidos do formulário enviado

2- FromRoute - vincula apenas os dados oriundos da rota de dados

3- FromQuery - Recebe apenas os dados da cadeia de consulta (querystring)

4- FromHeader - vincula os valores que vêm no cabeçalho da requisição HTTP

5- FromBody - Vincula os dados a partir do Body do request

6- FromServices - vincula o valor especificado à implementação que foi configurada no seu container de injeção de dependência

Permite injetar as dependências diretamente no método Action do controlados que reques a dependência.

Como utilizar: Definir o serviço -> Registrar os serviços -> Aplicar o atributo [FromServices] ao método Action (esse passo apenas para versões anteriores ao .Net 7)

# Data Notations - Validação personalizada 

1- Criar atributos customizados

Características:
- O seu foco é validar uma propriedade
- Pode ser utilizada em diversos modelos e propriedades

Passos:
- Criar uma classe que herda de ValidationAttribute e sobrescreve o método IsValid

O método IsValid aceita dois parâmetros:
1- um objeto value, que é a entrada a ser validada
2- um objeto ValidationContext, que fornece informações adicionais, como a instância de modelo criada pro model binding

- O método IsValid retorna a classe ValidationResult, através de seu campo estático Success ou inicializando uma nova instância (no caso de um erro)

public class PrimeiraLetraMaiusculaAttribute : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext validation)
    {
        if(value == null || string.IsNullOrEmpty(value.ToString())) return ValidationResult.Success;

        var primeiraLetra = value.ToString()[0].ToString();
        if (primeiraLetra != primeiraLetra.ToUpper())
        {
            return new ValidationResult("A primeira letra do nome do produto deve ser maiúscula");
        }

        return ValidationResult.Success; 
    }
}

2- Implementar IValidatebleObject no seu modelo

Características:
- Pode acessar todas as propriedades do modelo e realizar uma validação mais complexa
- Não pode ser reutilizada em outros modelos 

Passos:
- Implementar a interface IValidatableObject na classe do modelo
- criar um método que utilize responsável por validar os campos

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if(!string.IsNullOrEmpty(this.Nome))
        {
            var primeiraLetra = this.Nome.ToString();
            if(primeiraLetra != primeiraLetra.ToUpper())
            {
                yield return new ValidationResult("A primeira letra do nome do produto deve ser maiúscula",
                    new[] { nameof(this.Nome) }
                );
            }
        }

        if(this.Estoque <= 0)
        {
            yield return new ValidationResult("O estoque deve ser maior que 0",
                   new[] { nameof(this.Nome) }
               );
        }
    }
}

# Ler arquivos de configuração (appsetings.json)

Para ler em classes:
- Na classe injeto uma instância  de IConfiguration:
	 private readonly IConfiguration _configuration;
	 public CategoriasController(AppDbContext context, IConfiguration configuration)
 {
     _context = context;
     _configuration = configuration;
 }
- Para acessar os valores utilizo a notação de ["chave"]:
        var valor1 = _configuration["chave1"];
        var valor2 = _configuration["chave2"];
        var secao1 = _configuration["secao1:chave2"]; //Acessa valores aninhados


Para ler na classe program posso utilizar a variável *builder*
var builder = WebApplication.CreateBuilder(args);
var valor1 = builder.Configuration["chave1"]

builder representa uma instância de WebApplicationBuilder e possui uma propriedade Configuration que é uma instância de IConfiguration

## launchSettings.json
Nesse arquivo indicamos qual appsetings será executado para cada ambiente através da chave *ASPNETCORE_ENVIRONMENT* 

# Middleware
- O middleware é um software montado em um pipeline de aplicativo para manipular solicitações e respostas
- Pode executar o código antes e depois do próximo componente no pipeline.
- Middleware são configurados no arquivo Program.cs
- Possuem a nomenclatura "Use"

app.Use(async (context, next) =>
{
    //adicionar codigo antes do request
    await next(context);
    //adicionar o código depois do request
});

// Middleware terminal:
app.Run(async (context) =>
{
    await context.Response.WriteAsync("Middleware final");
});

## Passos para implementação
- Criar uma classe
public class ErrorDetails
{
    public int StatusCode { get; set; }
    public string? Message { get; set; }
    public string? Trace { get; set; }

    public override string ToString()
    {
        return JsonSerializer.Serialize(this);
    }
}
- Criar um método de Extensão que implemente IApplicationBuilder
public static class ApiExceptionMiddlewareExtensions
{
    public static void ConfigurationExceptionHandler(this IApplicationBuilder app){}
}
- Usar o Middleware UseExceptionHandler e configurar o tratamento de erros
public static void ConfigurationExceptionHandler(this IApplicationBuilder app){
	app.UseExceptionHandler(appError =>{})
}
- Habilitar o uso do método de Extensão ConfigureExceptionHandler na classe Program (app.ConfigureExceptionHandler())

# Filtros

São atributos anexados às classes ou métodos dos controladores que injetam lógica extra ao processamento de requisição e permitem a implementação de funcionalidades relacionadas a autorização, exception, log e cache de forma simples e elegante.

Eles permitem executar um código personalizado antes ou depois de executar um método Action.

Permite também realizar tarefas repetitivas comuns a métodos Actions e são chamados em certos estágios do pipeline. 

## Escopo e ordem de execução

Um filtro pode ser adicionado ao pipeline em um dos três escopos:
1- Pelo método Action;
2- Pela classe do Controlador;
3- Globalmente (é aplicado a todos os controladores e actions)

A ordem padrão de execução é a seguinte:
1- O filtro *global* é aplicado primeiro;
2- Depois o filtro de nível de *classe* é aplicado;
3- Finalmente, o filtro de nível de *método* é aplicado


## Filters vs Middleware
Os Filters são usados principalmente no ASP.NET Core MVC para adicionar lógica que é executada antes e depois das ações dos controladores.

O Middleware é usado em todo o ASP.NET Core, não apenas em MVC, e é um pipeline de componentes que processam uma solicitação HTTP. Ele permite que você adicione componentes reutilizáveis que processam solicitações HTTP conforme elas passam pelo pipeline de middleware.

O Middleware é configurado e executado antes do pipeline de MVC. Ele pode lidar com tarefas como roteamento, autenticação, manipulação de erros e muito mais. Um exemplo comum é o Middleware de roteamento, que roteia solicitações HTTP para os controladores corretos com base na URL.

Assim, a recomendação é usar Filters quando precisar de lógica específica para ações do controlador em um contexto MVC, e usar Middleware para processar solicitações HTTP em um nível mais global na aplicação ASP.NET Core.

## Passos para implementação
- Criar uma classe que implemente IActionFilter
- Injetar ILogger na classe para ter acesso as funcionalidades de logger
- Implementar os métodos OnActionExecuting e OnActionExecuted
- No Program.cs registro o filtro criado -> builder.Services.AddScoped<ApiLoggingFilter>();
- Aplico o filtro em um controlador:
[HttpGet]
[ServiceFilter(typeof(ApiLoggingFilter))] //ApiLoggingFilter é o nome do filtro(classe) criado
 