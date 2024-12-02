# Overview CORS
É uma <span style="color:rgb(255, 255, 0)">política de segurança</span> implementada pelos navegadores web para <span style="color:rgb(255, 255, 0)">proteger</span> os usuários contra <span style="color:rgb(255, 255, 0)">requisições</span> maliciosas de <span style="color:rgb(255, 255, 0)">origens diferentes</span>

## O que é Origem
É uma combinação de esquema, domínio e porta:
- Esquema (Schema) -> Geralmente, `Http` ou `Https`
- Domínio (Domain) -> O nome do host, como `www.exemplo.com`
- Porta (Port) -> A porta onde o recurso é servido geralmente 80 pata `http` e 443 para `https`

<mark style="background-color: #fff88f; color: black">Duas páginas da web são consideradas da mesma origem se possuírem a mesma combinação de esquema, domínio e porta</mark>

Origens diferentes
URL base -> https://dominio.com
As seguintes origens são diferentes
- http://dominio.com -> Protocolo http diferente
- https://dominio.com:8000 -> porta diferente 
- https://dominio.net -> domínio diferente
 
## Política CORS
A política CORS tem como objetivo permitir ou restringir requisições de recursos entre origens diferentes em aplicações Web

O servidor deve incluir cabeçalhos CORS nas respostas HTTP para indicar se uma origem específica tem permissão para acessar os recursos.
Exp.: Access-Control-Allow-Origin: `<Origem>`

Durante a execução de uma requisição,<span style="color:rgb(255, 255, 0)"> cabeçalhos CORS</span> são adicionados tanto no <span style="color:rgb(255, 255, 0)">request</span>quanto no <span style="color:rgb(255, 255, 0)">response</span> para indicar as permissões e restrições relacionadas ao <span style="color:rgb(255, 255, 0)">CORS</span>.

## Habilitando CORS
> [!NOTE]
> Existem 3 maneiras para habilitar o CORS 

### 1- No Middleware usando uma política nomeada ou uma política padrão

configuro o Middleware na classe `Program.cs` com as origens permitidas
```C#
var OrigensComAcessoPermitido = "_origensComAcessoPermitido";  
builder.Services.AddCors(options =>  
{  
    options.AddPolicy(name: OrigensComAcessoPermitido,  
        policy =>  
        {  
            policy.WithOrigins("https://apirequest.io");  
        });});
```

Habilito o Middleware  na classe `Program.cs` sempre <span style="color:rgb(254, 0, 65)">após</span> o `app.UseRouting` e <span style="color:rgb(107, 255, 174)">antes</span> do `app.UseAuthorization`:
```C#
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseCors(OrigensComAcessoPermitido);

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```


### 2- Usando roteamento de endpoint
```C#
public void Configure(IApplicationBuilder app)
{
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers()
            .RequireCors("MinhaPolicy");
    });
}
```

### 3- Com o atributo `[EnablreCors]`

Aplica uma<mark style="background-color: #fff88f; color: black"> política nomeada somente aos endpoints selecionados</mark> que exigem a política CORS

#### Habilitando

- Na Action -> Atributo `[EnableCors("PoliticaCORSExemplo")]`
- No Controledor -> Atributo `[EnableCors("PoliticaCORSExemplo")]`
	- O atributo `DisableCors` desabilita o CORS para um recurso em um método Action específico
```C#
// No controlador ou método: Aplica para todas as Actions
[EnableCors("MinhaPolicy")]
public class MeuController : ControllerBase 
{
    [EnableCors("PoliticaClientes")] //Aplica para a Action especifica
    public IActionResult GetProdutos() { }
	
	// Método com política diferente 
	[EnableCors("PoliticaAdmin")] 
	public IActionResult CriarProduto() { }
}
```



# Rate Limiting
Consiste em restringir o número de requisições que podem ser feitas em um período de tempo a uma aplicação

## Configuração

### 1- Registrar o serviço de limitação de Taxa (Program.cs)

#### Limitador de janela fica
- `AddFixedWindowLimiter` -> configura um limitador de janela fixa
- Permite um <mark style="background-color: #fff88f; color: black">número fixo de requests dentro de uma janela de tempo</mark> específica e <span style="color:rgb(255, 6, 255)">todos os requests subsequentes são postergados</span>
```C#
builder.Services.AddRateLimiter(rateLimiterOptions =>  
{  
    //Permite 3 requests a cada 10 segundos   
	rateLimiterOptions.AddFixedWindowLimiter("fixed", options =>  
	{  
		options.PermitLimit = 3;  
		options.Window = TimeSpan.FromSeconds(10);  
		// ordem dos requests a sererm processados: Do mais antigop ao mais recente:
		options.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;
		// Número máximo de requisições que podem ser enfileiradas após o limite de requisições ser excedido durante a janela de tempo:
		options.QueueLimit = 1;  
	});  
	      
	rateLimiterOptions.RejectionStatusCode = StatusCodes.Status429TooManyRequests;  
});
```

#### Limitador de janela deslizante
- Método `AddSlidingWindowLimiter` configura um limitador de janela deslizante
- Semelhante ao de janela fixa, mas introduz segmentos em uma janela
	- Cada janela de tempo é dividida em vários segmentos
	- A janela desliza um, segmento em cada intervalo de segmento
	- O intervalo do segmento é obtido assim: (window_time) / (segments_per_window)
	- Quando um segmento expira, as requisições recebidas nesse segmento são adicionadas ao segmento atual
```C#
builder.Services.AddRateLimiter(rateLimiterOptions =>  
{  
    rateLimiterOptions.AddSlidingWindowLimiter("sliding", options =>  
    {  
        options.PermitLimit = 5;  
        options.Window = TimeSpan.FromSeconds(10);  
        options.SegmentsPerWindow = 2;  
        options.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;  
        options.QueueLimit = 2;  
    });        
    
    rateLimiterOptions.RejectionStatusCode = StatusCodes.Status429TooManyRequests;  
});
```


#### Limitador Token bucket (cesta)
- Cada token na cesta representa um request que pode ser usado
- O número total de tokens nunca pode exceder o limite de token 
- Se a cesta fica vazia o próximo request será rejeitado ou postergado
- é semelhante à janela deslizante, mas em vez de adicionar novamente as requisições do segmento expirado, um número fixo de token é adicionado após cada período de reposição.
```C#
builder.Services.AddRateLimiter(rateLimiterOptions =>  
{  
    rateLimiterOptions.AddTokenBucketLimiter("token", options =>  
    {  
        options.TokenLimit = 3; //A cesta poderá conter até 3 request
        options.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;  
        options.QueueLimit = 2;  
        options.ReplenishmentPeriod = TimeSpan.FromSeconds(5);//Período de reabastecimento da cesta  
        options.TokensPerPeriod = 2;//Quantidade de tokens a serem adicionados na cesta  
        options.AutoReplenishment = true;//Reabastecimento automatico da cesta  
    });    
    
    rateLimiterOptions.RejectionStatusCode = StatusCodes.Status429TooManyRequests;  
});
```


#### Limitador de Simultaneidade ou concorrência
- Limita apenas o número de requisições simultâneas
```C#
builder.Services.AddRateLimiter(rateLimiterOptions =>  
{ 
    rateLimiterOptions.AddConcurrencyLimiter("concurrency", options =>  
    {  
        options.PermitLimit = 5;//Apenas os 5 primeiro requests terão acesso a um recurso de forma simultanea  
        options.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;  
        options.QueueLimit = 1;  
    });    
    rateLimiterOptions.RejectionStatusCode = StatusCodes.Status429TooManyRequests;  
});
```

### 2- Adicionar o Middleware ao pipeline de requisições da aplicação (após app.Routing)

```C#
//Precisa estar depois do app.UseRouting();
app.UseRateLimiter();
```

### 3- Usar os atributos `EnableRateLimiting` e `DisableRateLimiting` nos controladores
```C#
namespace APICatalogo.Controllers;  
[Route("[controller]")]  
[ApiController]  
[EnableRateLimiting("fixedwindow")]  
public class CategoriasController : ControllerBase  
{
	[HttpGet]  
	[ServiceFilter(typeof(ApiLoggingFilter))]  
	[DisableRateLimiting]  
	public async Task<ActionResult<IEnumerable<CategoriaDTO>>> Get()  
	{  
	    var categorias = await _unitOfWork.CategoriaRepository.GetAllAsync();  
	  
	    if (categorias is null) return NotFound("Não existem categorias...");  
	  
	    var categoriasDto = CategoriaDTOMappingExtensions.ToCategoriasDtoList(categorias);  
	  
	    return Ok(categoriasDto);  
	}
}
```

### Podemos definir um limitador de taxa global
- Vai atuar em todos os controladores
- Nesse caso não precisamos utilizar os atributos `EnableRateLimiting` e `DisableRateLimiting`
