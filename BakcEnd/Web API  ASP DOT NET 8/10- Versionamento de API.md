# Abordagens e configuração

## 1- Querystring
Inclui o número da versão como um parâmetro de consulta na URL
`https://api.exemplo.com/resource?version=1`
`https://api.exemplo.com/resource?version=2`

## 2- URI
Inclui a versão diretamente na URL da API
`https://api.exemplo.com/v1/resource`
`https://api.exemplo.com/v2/resource`

## 3- Headers
Especifica a versão desejada no cabeçalho(header) do request HTTP
```
GET/resource HTTP/1.1
Host: api.exemplo.com
Accept: application/json
X-API-Version: 1
```

## 4- Media Type
Usar diferentes tipos de mídia para representar versões diferentes de API
`Accept: application/vnd.exemplo.v1+json`

# Configuração

## Pacotes 
- `Asp.Versioning.Mvc.ApiExplorer`
- `Asp.Versioning.Http`

## Configuração em `Program.cs`
```C#
// Adicionar serviços de versionamento 
builder.Services.AddApiVersioning(options => 
{ 
	options.DefaultApiVersion = new ApiVersion(1, 0); 
	options.AssumeDefaultVersionWhenUnspecified = true; 
	
	// Escolher um método de leitura de versão 
	// options.ApiVersionReader = new QueryStringApiVersionReader("v"); 
	options.ApiVersionReader = ApiVersionReader.Combine( 
		new QueryStringApiVersionReader("v"), 
		new UrlSegmentApiVersionReader(), 
		new HeaderApiVersionReader("X-API-Version") 
	); 
});
```

## Utilizando nos controladores

### Utilizando URI
```C#
// Exemplo de Controlador com Versionamento por URI 
[ApiController] 
[Route("api/v{version:apiVersion}/[controller]")] 
public class ProductsController : ControllerBase { 
[HttpGet] public IActionResult GetProducts() { 
	return Ok(new[] { new { Id = 1, Name = "Product A" }, new { Id = 2, Name = "Product B" } }); 
} }
```

### Utilizando Query String
```C#
[ApiController]
[Route("api/[controller]")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class UsersController : ControllerBase
{
    // Método para a versão 1.0
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetUsersV1()
    {
        return Ok(new[] { 
            new { Id = 1, Name = "John Doe" },
            new { Id = 2, Name = "Jane Smith" }
        });
    }

    // Método para a versão 2.0
    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetUsersV2()
    {
        return Ok(new[] { 
            new { 
                Id = 1, 
                Name = "John Doe", 
                Email = "john.doe@example.com",
                CreatedAt = DateTime.UtcNow 
            },
            new { 
                Id = 2, 
                Name = "Jane Smith", 
                Email = "jane.smith@example.com",
                CreatedAt = DateTime.UtcNow 
            }
        });
    }
}
```
