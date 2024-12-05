# Configuração e instalação
> [!NOTE]
> Atualmente, ao utilizar o template de projeto padrão, podemos habilitar a OpenAPI marcando a opção : Enable OpenAPI support

## 1- instalar o pacote `Swashbuckle.AspNetCore`
```
dotnet add package Swashbuckle.AspNetCore
```

## 2- Adicionar e configurar o middleware do Swagger
```C#
//Program.cs
builder.Services.AddEndpointsApiExplorer();  
builder.Services.AddSwaggerGen(c =>  
{  
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "APICatalogo", Version = "v1" });
} 
```

## 3- Habilitar o middleware
```C#
//Program.cs
if (app.Environment.IsDevelopment())  
{  
    app.UseSwagger();  
    app.UseSwaggerUI();  
    app.ConfigurationExceptionHandler();  
}
```


# Comentários XML

## Gerar documento XML
```C#
//.csproj
<PropertyGroup>  
  <GenerateDocumentationFile>true</GenerateDocumentationFile>  
</PropertyGroup>
```


## Configurar o Swagger para usar o arquivo XML
- Criar um nome de arquivo XML correspondente ao projeto (Reflection)
- Adicionar os comentários XML ao Swagger usando `IncludeXmlComments`
```C#
//Program.cs
builder.Services.AddSwaggerGen(c =>  
{  
    c.SwaggerDoc("v1", new OpenApiInfo  
    {  
        Version = "v1",  
        Title = "APICatalogo",  
        Description = "Catálogos de produtos e Categorias",  
        TermsOfService = new Uri("https://www.google.com"),  
        Contact = new OpenApiContact  
        {  
            Name = "Bruno",  
            Email = "brunodevoliveira@gmail.com",   
			Url = new Uri("https://www.linkedin.com/in/brunodevoliveira/")  
        },        License = new OpenApiLicense  
        {  
            Name = "Usar sobre LICX",  
            Url = new Uri("https://www.linkedin.com/in/brunodevoliveira/")  
        }    });  
	}
	var xmlFilename = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";  
	c.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, xmlFilename));
}
```

# Suprimir avisos de falta de comentários XML
- Inclui tag `<NoWarn>$(NoWarn);1591</NoWarn>` em PropertyGroup do arquivo .csproj

# configurar os controladores para retornar apenas JSON
```C#
[Produces("application/json")] //Actions só irão retornar JSON  
public class CategoriasController : ControllerBase
```

# Convenções em métodos Actions
> [!NOTE]
> Define os tipos de retorno para cada Action no Swagger
>

- Aplicar especificamente em uma Action:
```C#
[HttpPut("/Categorias/{id}")]  
[ApiConventionMethod(typeof(DefaultApiConventions), nameof(DefaultApiConventions.Put))]  
public async Task<ActionResult<CategoriaDTO>> Put(int id, CategoriaDTO categoriaDto)
```

- Aplica para todas as Actions do controlador:
```C#
namespace APICatalogo.Controllers;  
[Route("[controller]")]  
[ApiController]  
[ApiConventionType(typeof(DefaultApiConventions))] //Documenta no Swagger os possíveis status codes que uma Actionpode retornar  
public class ProdutosController : ControllerBase
```


# Analisadores
- Inspeciona os controladores anotados com `[ApiController]` e identifica ações não documentadas
- habilitar `<IncludeOpenAPIAnalyzers>true</IncludeOpenAPIAnalyzers>` na tag `<PropertyGroup>` do .csproj


