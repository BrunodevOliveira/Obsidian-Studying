Aplicar a paginação usando parâmetros na URL para definir o número e tamanho da página:

`https://localhost:7014/produtos?pageNumber=2&pageSize=3`
pageNumber -> número da página
pageSize -> quantidade de registros por página


## [[Excalidraw/Paginação.md|Implementação]]
Em um método Action de ProdutosController podemos incluir parâmetro para definir o número e tamanho da página:

- `[FromQuery]` -> vincula os parâmetros a valores fornecidos na string de consulta (query string)
- `ProdutosParameters` -> Classe quie encapsula os parâmetro usados
- `GetProdutos` -> Método do repositório de produtos usado para obter os produtos com os parâmetros definidos
![[Paginacao_implementacao.png]]




# Filtros
Recuperar um **subconjunto específico de dados** de um conjunto maior com base em critérios ou condições específicas

## Maneiras de filtrar dados

Filtragem por consulta (Query Filtering) -> Usa parâmetros de consulta na URL do request para especificar critérios de filtro

Filtragem por rota(Route Filtering) -> <mark style="background-color: #fff88f; color: black">Definir rotas específicas em seu controlador que aceitam parâmetros no próprio URL da solicitação</mark>

Filtragem avançada -> Implementar lógica de filtragem mais complexa usando classes de parâmetros personalizadas

Usar pacotes de terceiros -> Pacote Sieve


## Implementação Route Filtering
![[Pasted image 20241024211402.png]]

1- Criar a classe que conterá os filtros vindos dos parâmetros da rota
- A classe herda de QueryStringParameters os filtros de tamanho de página e número de página

```C#
public class CategoriasFiltroNome : QueryStringParameters
{
    public string? Nome { get; set; }
}
```


2- Adiciona na Interface a assinatura do método que será responsável pela filtragem

```C#
namespace APICatalogo.Repositories;

public interface ICategoriaRepository : IRepository<Categoria>
{
    PagedList<Categoria> GetCategorias(CategoriasParameters categoriasParams);

    PagedList<Categoria> GetCategoriasFiltroNome(CategoriasFiltroNome categoriasParams);
}
```

3- Criar a função responsável por retornar os dados filtrados
```C#
namespace APICatalogo.Repositories;

public class CategoriaRepository : Repository<Categoria>, ICategoriaRepository
{
    public CategoriaRepository(AppDbContext context) : base(context)
    {
    }
	....

    public PagedList<Categoria> GetCategoriasFiltroNome(CategoriasFiltroNome categoriasParams)
    {
        var categorias = GetAll().AsQueryable();

        if(!string.IsNullOrEmpty(categoriasParams.Nome))
        {
            //Se não entrar nesse IF será retornado os valores de GetAll()
            categorias = categorias.Where(c => c.Nome.ToLower().Contains(categoriasParams.Nome.ToLower()));
        }

        var categortiasFiltradas = PagedList<Categoria>
              .ToPagedList(categorias, categoriasParams.PageNumber, categoriasParams.PageSize);

        return categortiasFiltradas;
    }
}
```

4- Criar no controller a rota responsável pelo filtragem
```C#
[HttpGet("filter/nome/pagination")]
public ActionResult<IEnumerable<CategoriaDTO>> GetCategoriasFiltradas([FromQuery] CategoriasFiltroNome categoriasFiltro)
   {
       var categorias = _unitOfWork.CategoriaRepository.GetCategoriasFiltroNome(categoriasFiltro);

       return ObterCategorias(categorias);

   }

private ActionResult<IEnumerable<CategoriaDTO>> ObterCategorias(PagedList<Categoria> categorias)
   {
       var metadata = new
       {
           categorias.TotalCount,
           categorias.PageSize,
           categorias.CurrentPage,
           categorias.TotalPages,
           categorias.HasNext,
           categorias.HasPrevious
       };

       Response.Headers.Append("X-Pagination", JsonConvert.SerializeObject(metadata));

       var categoriasDto = CategoriaDTOMappingExtensions.ToCategoriasDtoList(categorias);

       return Ok(categoriasDto);
	}
```

