# Tutorial: Implementando Exceções Personalizadas em uma API .NET com Filtro de Exceção

  https://studio.firebase.google.com/excecaopersolanizada-22105026

Este tutorial demonstra como criar e gerenciar exceções personalizadas em uma API .NET utilizando um filtro de exceção global. Este padrão ajuda a centralizar o tratamento de erros e a fornecer respostas consistentes da API.

  
O projeto "ExcecaoPersonalizada" serve como exemplo dessa implementação.


## Estrutura e Objetivo das Classes:

### 1. Definindo a Exceção Base Personalizada

*   **Objetivo:** Criar uma classe base que sirva como **marcador para todas as suas exceções de negócio personalizadas**. O filtro de exceção usará essa base para identificar quais exceções ele deve tratar.

*   **Classe:** `BaseExcecaoPersonalizada`

*   **Descrição:** Herda diretamente de `SystemException`. Não requer lógica interna complexa, sua principal função é a herança para identificação.

```csharp

public abstract class BaseExcecaoPersonalizada : SystemException{}

```

### 2. Criando Exceções Específicas Personalizadas

*   **Objetivo:** Definir classes de exceção para cenários de erro específicos do seu domínio de negócio.

*   **Classe:** `RegistroPessoaExcecaoPersonalizada`

*   **Descrição:** Representa um erro ocorrido durante o processo de registro de uma pessoa. Inclui uma propriedade `Errors` (lista de strings) para detalhar as mensagens de erro associadas.

  

```csharp

public class RegistroPessoaExcecaoPersonalizada : BaseExcecaoPersonalizada {

    public List<string> Errors { get; set; }

  

    public RegistroPessoaExcecaoPersonalizada(List<string> errorMessages)

    {

        Errors = errorMessages;

    }

}

```

*   **Nota:** Você pode criar outras classes de exceção personalizadas (ex: `UsuarioNaoEncontradoExcecao`, `PedidoInvalidoExcecao`) herdando de `BaseExcecaoPersonalizada` para outros tipos de erros.

### 3. Padronizando a Resposta de Erro

*   **Objetivo:** Garantir que a resposta enviada ao cliente quando ocorre um erro tratado tenha um formato consistente.

*   **Classe:** `RespostaErroJson`

*   **Descrição:** Uma classe simples para serializar a informação do erro. Contém uma lista de strings (`ErrorMessages`) para as mensagens de erro. Possui construtores para conveniência, aceitando uma única mensagem ou uma lista.


```csharp

public class RespostaErroJson {

    public List<string> ErrorMessages { get; set; }

  

    public RespostaErroJson(string erroMessage)

    {

        ErrorMessages = [erroMessage];

    }

  

    public RespostaErroJson(List<string> erroMessages)

    {

        ErrorMessages = erroMessages;

    }

}

```

  

### 4. Implementando o Filtro de Exceções
*   **Objetivo:** Criar um ponto centralizado no pipeline de requisição para interceptar exceções e gerar respostas HTTP apropriadas.

*   **Classe:** `FiltroExcecoes`

*   **Arquivo:** `Excecoes/ExcecaoExemplo.cs`

*   **Descrição:** Implementa a interface `IExceptionFilter`. O método `OnException` é o coração deste filtro. Ele verifica o tipo da exceção que ocorreu:

    *   Se for uma `BaseExcecaoPersonalizada` (ou qualquer classe que herde dela), chama `ProcessarExcecao`.

    *   Caso contrário (exceções não tratadas/inesperadas), chama `ErroSemTratamento`.

    ``` csharp
using FluentValidation;

using Microsoft.AspNetCore.Mvc;

using Microsoft.AspNetCore.Mvc.Filters;

public class FiltroExcecoes : IExceptionFilter {
  
    public void OnException(ExceptionContext context) {

        // Verifica se a exceção é uma das nossas exceções personalizadas

        if(context.Exception is BaseExcecaoPersonalizada) {

            ProcessarExcecao(context);

        } else {

            // Lida com exceções que não são personalizadas (erros inesperados)

            ErroSemTratamento(context);

        }

        // Indica que a exceção foi tratada pelo filtro

        context.ExceptionHandled = true;

    }

  

    private void ProcessarExcecao(ExceptionContext context) {
        // Verifica o tipo específico da exceção personalizada para tratamento diferenciado, se necessário

        if(context.Exception is RegistroPessoaExcecaoPersonalizada) {

            var ex = context.Exception as RegistroPessoaExcecaoPersonalizada;

            // Cria a resposta padronizada usando as mensagens de erro da exceção

            var respostaErro = new RespostaErroJson(ex.Errors);

            // Define o status code HTTP para 400 Bad Request (comum para erros de validação/negócio)

            context.HttpContext.Response.StatusCode = StatusCodes.Status400BadRequest;

            // Define o resultado da action como um BadRequestObjectResult contendo a resposta de erro

            context.Result = new BadRequestObjectResult(respostaErro);

        } else {

            // Tratamento padrão para outras exceções BaseExcecaoPersonalizada se não houver lógica específica por tipo

            var respostaErro = new RespostaErroJson(context.Exception.Message);

  

            context.HttpContext.Response.StatusCode = StatusCodes.Status400BadRequest;

            context.Result = new BadRequestObjectResult(respostaErro);

        }

    }


    private void ErroSemTratamento(ExceptionContext context) {

        // Cria uma resposta genérica para erros não personalizados

        var respostaErro = new RespostaErroJson("Erro Sem Tratamento -> " + context.Exception.Message);

  

        // Define o status code HTTP para 500 Internal Server Error

        context.HttpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;

        // Retorna um ObjectResult com a resposta de erro

        context.Result = new ObjectResult(respostaErro);

    }

}
```
*   **Métodos:**

    *   `OnException(ExceptionContext context)`: O ponto de entrada do filtro.

    *   `ProcessarExcecao(ExceptionContext context)`: Lida com exceções personalizadas. Adapta o status code e a resposta com base no tipo específico da exceção.

    *   `ErroSemTratamento(ExceptionContext context)`: Lida com exceções inesperadas, retornando um erro 500.

### 5. Utilizando a Exceção Personalizada (Exemplo com Validação)
*   **Objetivo:** Demonstrar como lançar a exceção personalizada a partir da sua lógica de negócio.

*   **Classes de Exemplo:** `RegrasValidacaoPessoa` (utilizando FluentValidation) e `RegistrarPessoaRegraNegocio`.

*   **Arquivos:** `Excecoes/ExcecaoExemplo.cs` (RegrasValidacaoPessoa e RegistrarPessoaRegraNegocio), `Controllers/HomeController.cs` (RequestRegistroPessoaJson, RespostaRegistroPessoaJson e a Action no Controller).

*   **Descrição:**

    *   `RegrasValidacaoPessoa`: Define as regras de validação para o objeto `RequestRegistroPessoaJson`.

    *   `RegistrarPessoaRegraNegocio`: Contém a lógica de negócio para registrar uma pessoa. O método `Execute` chama a validação. Se `resultado.IsValid` for `false`, ele coleta as mensagens de erro e *lança* uma nova instância de `RegistroPessoaExcecaoPersonalizada` contendo esses erros.

    *   O Controller (`PessoaController`) na action `RegistrarPessoa` chama `RegistrarPessoaRegraNegocio().Execute(request)`. Quando a exceção é lançada pelo método `Execute`, ela é propagada para o pipeline e interceptada pelo `FiltroExcecoes`.


    ``` csharp
public class RegrasValidacaoPessoa : AbstractValidator<RequestRegistroPessoaJson> // RequestRegistroPessoaJson está no arquivo do Controller

{

    public RegrasValidacaoPessoa()

    {

        RuleFor(pessoa => pessoa.Nome).NotEmpty().WithMessage("Nome é obrigatório");

        RuleFor(pessoa => pessoa.Idade).GreaterThan(0).WithMessage("Idade é obrigatório");

        RuleFor(pessoa => pessoa.Altura).GreaterThan(0).WithMessage("Altura é obrigatório");

        RuleFor(pessoa => pessoa.Genero).NotEmpty().WithMessage("Gênero é obrigatório");

        RuleFor(pessoa => pessoa.Genero).IsInEnum().WithMessage("Escolha um Gênero válido");

    }

}

// Classe de Regra de Negócio que orquestra a validação e lança a exceção

public class RegistrarPessoaRegraNegocio() {

    public RespostaRegistroPessoaJson Execute(RequestRegistroPessoaJson request) { // RequestRegistroPessoaJson e RespostaRegistroPessoaJson estão no arquivo do Controller

        ValidarRegistroPessoa(request);

        // Lógica real para registrar a pessoa no banco de dados ou outro serviço...

        return new RespostaRegistroPessoaJson {

            Nome = request.Nome // Exemplo de retorno em caso de sucesso

        };

    }

    private void ValidarRegistroPessoa(RequestRegistroPessoaJson request) {

        var validador = new RegrasValidacaoPessoa();

        var resultado = validador.Validate(request);

  

        if(resultado.IsValid == false) {

            var mensagenErro = resultado.Errors.Select(e => e.ErrorMessage).ToList();

            // LANÇA a exceção personalizada com a lista de erros de validação

            throw new RegistroPessoaExcecaoPersonalizada(mensagenErro);

        }

    }

}

// No arquivo Controllers/HomeController.cs (renomeado para PessoaController para clareza)

using Microsoft.AspNetCore.Mvc;

using Excecoes; // Importa o namespace onde estão as exceções e o filtro

  

[Route("api/[controller]")]

[ApiController]

public class PessoaController : ControllerBase

{

    [HttpPost]

    public IActionResult RegistrarPessoa([FromBody] RequestRegistroPessoaJson request) {

        var regraNegocio = new RegistrarPessoaRegraNegocio();

        var resposta = regraNegocio.Execute(request); // Chama a regra de negócio que pode lançar a exceção

        return Ok(resposta); // Retorna sucesso se a exceção não for lançada

    }

}

// Definições dos modelos de requisição e resposta (exemplo)

public class RequestRegistroPessoaJson() {

    public string Nome { get; set; }

    public  int Idade { get; set; }

    public float Altura { get; set; }

    public GeneroType Genero { get; set; } // Assumindo um enum GeneroType

    public DateTime Criacao { get; set; } = DateTime.Now;

}

  

public class RespostaRegistroPessoaJson() {

    public string Nome { get; set; }

    public DateTime Criacao { get; set; } = DateTime.Now;

}

// Exemplo de Enum (em algum lugar no projeto)

public enum GeneroType

{

    Masculino,

    Feminino,

    Outro

}
    ```
### 6. Registrando o Filtro de Exceção no Pipeline da Aplicação
*   **Objetivo:** Adicionar o `FiltroExcecoes` aos serviços do ASP.NET Core para que ele seja executado automaticamente quando exceções não tratadas ocorrerem nas actions dos controllers.

*   **Arquivo:** `Program.cs`

*   **Como registrar (baseado no seu `Program.cs`):** No seu caso, o filtro está sendo adicionado às opções do MVC.


 ```csharp
using Excecoes; // Importa o namespace do filtro

var builder = WebApplication.CreateBuilder(args);

// Adiciona os serviços de controllers

builder.Services.AddControllers();

// Adiciona o filtro de exceção às opções do MVC

builder.Services.AddMvc(opt => opt.Filters.Add(typeof(FiltroExcecoes)));


var app = builder.Build();

// Mapeia os endpoints dos controllers

app.MapControllers();

// Inicia a aplicação, escutando na URL especificada

app.Run(url); // url definida anteriormente com base na variável de ambiente PORT
    ```

### Fluxo Resumido:

Requisição -> Controller Action -> Lógica de Negócio/Validação -> Lança `RegistroPessoaExcecaoPersonalizada` -> Exceção Propagada -> `FiltroExcecoes` Intercepta -> `FiltroExcecoes` Chama `ProcessarExcecao` -> `ProcessarExcecao` Identifica o tipo específico, Cria `RespostaErroJson`, Define Status 400, Define `context.Result` -> Resposta 400 JSON enviada ao Cliente.

Este fluxo garante que seus erros de negócio sejam tratados de forma consistente, retornando respostas padronizadas com códigos de status HTTP apropriados para o cliente da API.