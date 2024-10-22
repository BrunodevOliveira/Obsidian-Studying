
Atualização parcial de um dado. O request `PATCH` inclui um documento de patch, no formato JSON Patch, que descreve as alterações a serem aplicadas no recurso.

Idempotente -> propriedade do protocolo HTTP que significa que se você fizer  a mesma request varias vezes o resultado será o mesmo, como se tivesse feito um única vez.

Não é Idempotente (Depende das operações contidas no documento de patch, por exemplo a operação replace é Idempotente mas a Add não) 

# Operações

1- **replace** -> Substitui um valor existente

```C#
[
    {
        "op": "replace",
        "path": "/estoque",
        "value": 100
    }
]
```

2- **add** -> Adiciona um novo valor (usado principalmente para arrays ou quando a propriedade não existe)
```C#
[
    {
        "op": "add",
        "path": "/estoque",
        "value": 75
    }
]
```

3- **remove** -> Remove um valor (mais comum em arrays ou quando a propriedade é nullable)
```C#
[
    {
        "op": "remove",
        "path": "/estoque"
    }
]
```

4- **copy** -> Copia um valor de um caminho para outro
```C#
[
    {
        "op": "copy",
        "from": "/estoque",
        "path": "/estoqueBackup"
    }
]
```


5- **move** -> Move um valor de um caminho para outro
```C#
[
    {
        "op": "move",
        "from": "/estoque",
        "path": "/estoqueNovo"
    }
]
```

6- **test** -> Testa se um valor existe (útil para validações)
```C#
[
    {
        "op": "test",
        "path": "/estoque",
        "value": 50
    }
]
```

## Exemplos Práticos com Arrays

Se você tiver uma propriedade que é um array, você pode fazer operações específicas:

Classe representando os dados:

```C#
public class ProdutoDTOUpdateRequest
{
    public float Estoque { get; set; }
    public DateTime DataCadastro { get; set; }
    public List<string> Categorias { get; set; }
}
```

1- Adicionar ao final do array:
```C#
[
    {
        "op": "add",
        "path": "/categorias/-",
        "value": "Eletrônicos"
    }
]
```

2- Adicionar em um índice específico:
```C#
[
    {
        "op": "add",
        "path": "/categorias/0",
        "value": "Informática"
    }
]
```

3- Remover um item específico do array:
```C#
[
    {
        "op": "remove",
        "path": "/categorias/1"
    }
]
```


## Exemplo com múltiplas operações
```C#
[
    {
        "op": "test",
        "path": "/estoque",
        "value": 50
    },
    {
        "op": "replace",
        "path": "/estoque",
        "value": 75
    },
    {
        "op": "replace",
        "path": "/datacadastro",
        "value": "2024-10-23T08:41:00"
    }
]
```


## Pacotes

Microsoft.AspNetCore.JsonPatch
- Permite implementar operações de patch em recursos RESTful por meio do método HTTP Patch


Microsoft.AspNetCore.Mvc.NewtonsoftJson
- Habilita o suprote ao JSON Patch na ASP.Net Core Web API
- Fornece um parse e um serializador para o formato JSON Patch


# Implementação

1- Após a instalação dos pacotes, adiciono `AddNewtonsoftJson()` no **Program.cs**

```C#
builder.Services
    .AddControllers(options => options.Filters.Add(typeof(APIExceptionFilter)))
    .AddJsonOptions(options => options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles)
    .AddNewtonsoftJson();
```

2- Crio as classes de DTO para **Resposta** e **Request**:

```C#
namespace APICatalogo.DTOs;

public class ProdutoDTOUpdateResponse
{
    public int ProdutoId { get; set; }

    public string? Nome { get; set; }

    public string? Descricao { get; set; }

    public decimal Preco { get; set; }

    public string? ImagemUrl { get; set; }

    public float Estoque { get; set; }

    public DateTime DataCadastro { get; set; }

    public int CategoriaId { get; set; }
}
```


```C#
namespace APICatalogo.DTOs;

public class ProdutoDTOUpdateRequest : IValidatableObject
{
    [Range(1, 9999, ErrorMessage = "Estoque deve estar entre 1 e 9999")]
    public float Estoque { get; set; }

    public DateTime DataCadastro { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if(DataCadastro.Date <= DateTime.Now.Date)
        {
            yield return new ValidationResult(
                "A data deve ser maior que a data atual", 
                new[] { nameof(this.DataCadastro) }
            );
        }
    }
}
```

3- Dentro da `Mappings` crio uma classe que conterá o mapeamento entre Entidade e DTO feito pela Lib AutoMapper

```C#
namespace APICatalogo.DTOs.Mappings;

public class ProdutoDTOMappingProfile : Profile
{
    public ProdutoDTOMappingProfile()
    {
        CreateMap<Produto, ProdutoDTO>().ReverseMap();
        CreateMap<Categoria, CategoriaDTO>().ReverseMap();
        CreateMap<Produto, ProdutoDTOUpdateRequest>().ReverseMap();
        CreateMap<Produto, ProdutoDTOUpdateResponse>().ReverseMap();
    }
}
```

4- No `Controller` implemento o método responsável pelo **Patch**

```C#
[HttpPatch("{id}/UpdatePartial")]
public ActionResult<ProdutoDTOUpdateResponse> Patch(int id, JsonPatchDocument<ProdutoDTOUpdateRequest> patchProdutoDto)
{
    if (patchProdutoDto is null || id <= 0) return BadRequest();

    var produto = _unitOfWork.ProdutoRepository.Get(c => c.ProdutoId == id);

    if(produto is null) return NotFound();

    var produtoUpdateRequest = _mapper.Map<ProdutoDTOUpdateRequest>(produto);

    //Aplica as atualizaçoes no produto e caso haja erro será armazenado no ModelState
    patchProdutoDto.ApplyTo(produtoUpdateRequest, ModelState);

    /*
        TryValidateModel() executa a validação e adiciona quaisquer erros ao ModelState. TryValidateModel() retorna true caso esteja valido
        Em seguida, verificamos se o ModelState é válido
        Se não for válido, retornamos BadRequest com os erros
     */
    TryValidateModel(produtoUpdateRequest);
    if (!ModelState.IsValid) return BadRequest(ModelState);

    // Mapeio novamente o objeto produtoUpdateRequest para Produto
    _mapper.Map(produtoUpdateRequest, produto);

    //Atualizo no DB
    _unitOfWork.ProdutoRepository.Update(produto);
    _unitOfWork.Commit();

    //Retorno o DTO
    return Ok(_mapper.Map<ProdutoDTOUpdateResponse>(produto));
}
```

