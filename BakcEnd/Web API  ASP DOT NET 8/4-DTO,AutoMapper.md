# Data transfer Object

é uma estratégia que visa não expor todo o modelo de domínio  da nossa API. Devemos evitar retornar entidades de domínio a partir dos endpoints de uma API. 

Entidades de domínio não devem depender de lógica externa e devem ser isoladas das camadas externas da aplicação

Um DTO é um contêiner de dados usado para transportar dados entre diferentes partes ou camadas e um aplicação que define como os dados serão enviados pela rede


## DTO de entrada

São usados para recebere dados nos requests HTTP


## DTO no retorno
São usados para modelar os dados que os controladores retornam aos clientes


# AutoMapper Biblioteca

Realiza o mapeamento entre os objetos que representam nossas entidades e os objetos que representam os nossos DTOs filtrando as propriedades que desejamos expor

**Pacote necessário:**
- `Automapper`


## Implementação de Mapping Manual

1- Criar uma Classe de DTO com os dados a serem trafegados da entidade

```C#
namespace APICatalogo.DTOs;

public class CategoriaDTO
{
    [Key]
    public int CategoriaId { get; set; }

    [Required]
    [StringLength(80)]
    public string? Nome { get; set; }

    [Required]
    [StringLength(300)]
    public string? ImagemUrl { get; set; }
}
```

2- Criar uma Classe estática com os métodos de conversão 

```C#
namespace APICatalogo.DTOs.Mappings;

public static class CategoriaDTOMappingExtensions
{
    public static CategoriaDTO? ToCategoriaDto(this Categoria categoria)
    {
        if (categoria is null)return null;

        var categoriaDTO = new CategoriaDTO()
        {
            CategoriaId = categoria.CategoriaId,
            Nome = categoria.Nome,
            ImagemUrl = categoria.ImagemUrl,
        };

        return categoriaDTO;
    }

    public static Categoria? ToCategoria(this CategoriaDTO categoriaDTO)
    {
        if (categoriaDTO is null) return null;

        var categoria = new Categoria()
        {
            CategoriaId = categoriaDTO.CategoriaId,
            Nome = categoriaDTO.Nome,
            ImagemUrl = categoriaDTO.ImagemUrl,
        };

        return categoria;
    }

    public static IEnumerable<CategoriaDTO>? ToCategoriasDtoList(this IEnumerable<Categoria> categorias)
    {
        if (categorias is null || !categorias.Any()) return new List<CategoriaDTO>();

        return categorias.Select(c => new CategoriaDTO() {
            CategoriaId = c.CategoriaId,
            Nome = c.Nome,
            ImagemUrl = c.ImagemUrl,
        } as CategoriaDTO).ToList();

    }
}
```

3- Executo os métodos da classe estática nos controllers que necessitam de conversão para entrada e saída de dados

``` C#
namespace APICatalogo.Controllers;
[Route("[controller]")]
[ApiController]
public class CategoriasController : ControllerBase
{
    //private readonly IRepository<Categoria> _repository; Após implementa o padraão Unit of Work, não utilizo mais diretamente o repository
    private readonly  IUnitOfWork _unitOfWork;
    private readonly IConfiguration _configuration;
    private readonly ILogger _logger;


    public CategoriasController(IConfiguration configuration, ILogger<CategoriasController> logger, IUnitOfWork unitOfWork)
    {
        _configuration = configuration;
        _logger = logger;
        _unitOfWork = unitOfWork;
    }

    [HttpGet("LerArquivoConfiguracao")]
    public string GetValoresConfig()
    {
        var valor1 = _configuration["chave1"];
        var valor2 = _configuration["chave2"];
        var secao1 = _configuration["secao1:chave2"];

        return valor1 + " " + valor2 + " " + secao1;
    }


    [HttpGet]
    [ServiceFilter(typeof(ApiLoggingFilter))]
    public ActionResult<IEnumerable<CategoriaDTO>> Get()
    {
        var categorias = _unitOfWork.CategoriaRepository.GetAll();

        if (categorias is null) return NotFound("Não existem categorias...");

        var categoriasDto = CategoriaDTOMappingExtensions.ToCategoriasDtoList(categorias);

        return Ok(categoriasDto);
    }


    [HttpGet("{id}", Name = "ObterCategoria")]
    public ActionResult<CategoriaDTO> Get(int id)
    {
        //throw new ArgumentException("Excecão para teste de Middleware");
        //throw new ArgumentException("Teste APIExceptionFilter - Ocorreu um erro no tratamento do request");

        _logger.LogInformation($"===============GET api/categorias/id = {id} ================");

        var categoria = _unitOfWork.CategoriaRepository.Get(c => c.CategoriaId == id);

        if (categoria is null)
        {
            _logger.LogWarning($"Cateoria com id = {id} não encontrada");
            return NotFound($"Cateoria com id = {id} não encontrada");
        }

        var categoriaDTO = CategoriaDTOMappingExtensions.ToCategoriaDto(categoria);

        return Ok(categoriaDTO);

    }

    [HttpPost]
    public ActionResult<CategoriaDTO> Post(CategoriaDTO categoriaDto)
    {
        if (categoriaDto is null)
        {
            _logger.LogWarning($"Dados invalidos: {categoriaDto.ToString()}");
            return BadRequest("Falta informações..");
        }

        var categoria = CategoriaDTOMappingExtensions.ToCategoria(categoriaDto);

        var categoriaCriada = _unitOfWork.CategoriaRepository.Create(categoria);
        _unitOfWork.Commit();

        var novaCategoriaDTO = CategoriaDTOMappingExtensions.ToCategoriaDto(categoria);

        return new CreatedAtRouteResult("ObterCategoria", new { id = novaCategoriaDTO.CategoriaId }, novaCategoriaDTO);
    }

    [HttpPut("/Categorias/{id}")]
    public ActionResult<CategoriaDTO> Put(int id, CategoriaDTO categoriaDto)
    {
        if (id != categoriaDto.CategoriaId)
        {
            _logger.LogWarning("Dados inválidos.");
            return BadRequest();
        }

        var categoria = CategoriaDTOMappingExtensions.ToCategoria(categoriaDto);

        var categoriaAtualizada = _unitOfWork.CategoriaRepository.Update(categoria);
        _unitOfWork.Commit();

        var categoriaAtualizadaDTO = CategoriaDTOMappingExtensions.ToCategoriaDto(categoria);

        return Ok(categoriaAtualizadaDTO);
    }

    [HttpDelete("/Categorias/{id}")]
    public ActionResult<CategoriaDTO> Delete(int id)
    {
        var categoria = _unitOfWork.CategoriaRepository.Get(c => c.CategoriaId == id);

        if (categoria is null)
        {
            _logger.LogWarning("Categoria não localizada");
            return NotFound($"Categoria com id = {id} não localizada");
        }

        var categoriaExcluida = _unitOfWork.CategoriaRepository.Delete(categoria);

        _unitOfWork.Commit();

        var categoriaExcluidaDTO = CategoriaDTOMappingExtensions.ToCategoriaDto(categoriaExcluida);

        return Ok(categoriaExcluidaDTO);
    }
}
```


## Implementação com Biblioteca AutoMapper


1- Instalo o pacote Nuget do Automapper

2- Crio O DTO tendo como base a entidade

```C#
namespace APICatalogo.DTOs;

public class ProdutoDTO
{
    public int ProdutoId { get; set; }

    [Required(ErrorMessage = "O nome é obrigatório")]
    [StringLength(80)]
    [PrimeiraLetraMaiuscula] //Validação é executada antes de entrar no Controller! 
    public string? Nome { get; set; }

    [Required]
    [StringLength(300)]
    public string? Descricao { get; set; }

    [Required]
    public decimal Preco { get; set; }

    [Required]
    [StringLength(300)]
    public string? ImagemUrl { get; set; }

    public int CategoriaId { get; set; }
}
```

3- Crio uma classe para mapear as informaçoes da Entidade para DTO, esta classe deve herda a classe Profile do AutoMapper

```C#
namespace APICatalogo.DTOs.Mappings;

public class ProdutoDTOMappingProfile : Profile
{
    public ProdutoDTOMappingProfile()
    {
        CreateMap<Produto, ProdutoDTO>().ReverseMap();
        CreateMap<Categoria, CategoriaDTO>().ReverseMap();
    }
}
```

4- Na classe `Program.cs` adiciono o AutoMapper indicando o tipo da classe que contem o mapeamento

```C#
builder.Services.AddAutoMapper(typeof(ProdutoDTOMappingProfile));
```


5- No controller, faço a injeção de dependência para utilizar uma instância de IMapper.

OBS.: O método para conversão recebe o tipo de dado de saída e como parâmetro o dado de origem -> `_mapper.Map<IEnumerable<ProdutoDTO>>(produtosPorCategoria);  //Map<Destino>(Origem)`


```C#
namespace APICatalogo.Controllers;
[Route("[controller]")]
[ApiController]
public class ProdutosController : ControllerBase
{
    //private readonly IProdutoRepository _produtoRepository; // Tem acesso aos métodos genéricos e específicos
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;

    public ProdutosController(IProdutoRepository produtoRepository, IUnitOfWork unitOfWork, IMapper mapper)
    {
        //_produtoRepository = produtoRepository;
        _unitOfWork = unitOfWork;
        _mapper = mapper;
    }

    [HttpGet("produtos/{id}")]
    public ActionResult<IEnumerable<ProdutoDTO>> GetProdutosCategoria(int id)
    {
        var produtosPorCategoria = _unitOfWork.ProdutoRepository.GetProdutoPorCategoria(id);

        if (produtosPorCategoria is null) return NotFound();

        var produtosDto = _mapper.Map<IEnumerable<ProdutoDTO>>(produtosPorCategoria);  //Map<Destino>(Origem)

        return Ok(produtosDto); 
    }

    [HttpGet]
    public ActionResult<IEnumerable<ProdutoDTO>> Get()
    {
        var produtos = _unitOfWork.ProdutoRepository.GetAll();
        if (produtos is null) return NotFound();

        var produtosDto = _mapper.Map<IEnumerable<ProdutoDTO>>(produtos);

        return Ok(produtosDto);
    }

    [HttpGet("{id}", Name= "ObterProduto")]
    public ActionResult<ProdutoDTO> Get(int id) //[FromQuery] int id -> Dessa forma não preciso passar o ID como parâmetro da Action.
    {
        var produto = _unitOfWork.ProdutoRepository.Get(p => p.ProdutoId == id);
        if (produto is null) return NotFound("Produto não encontrado");

        var produtoDto = _mapper.Map<ProdutoDTO>(produto);

        return Ok(produtoDto);
    }

    [HttpPost]
    public ActionResult<ProdutoDTO> Post([FromBody] ProdutoDTO produtoDto)
    {
        if (produtoDto is null) return BadRequest("Falta informações..");

        var produto = _mapper.Map<Produto>(produtoDto);

        var novoProduto = _unitOfWork.ProdutoRepository.Create(produto);
        _unitOfWork.Commit();

        var novoProdutoDto = _mapper.Map<ProdutoDTO>(novoProduto);

        //Retorna status 201 s um cabeçalho com o id e a rota para obter o produto criado
        return new CreatedAtRouteResult("ObterProduto", new { id = novoProdutoDto.ProdutoId }, novoProdutoDto); 
    }

    [HttpPut("/Produtos/{id}")]
    public ActionResult<ProdutoDTO> Put(int id, ProdutoDTO produtoDto) {
        if (id != produtoDto.ProdutoId) return BadRequest();

        var produto = _mapper.Map<Produto>(produtoDto);

        var atualizouProduto = _unitOfWork.ProdutoRepository.Update(produto);
        _unitOfWork.Commit();

        var novoProdutoDto = _mapper.Map<ProdutoDTO>(atualizouProduto);

        return atualizouProduto is not null ? Ok(novoProdutoDto) 
            : StatusCode(500, $"Falha ao atualizar o produto de id = {id}");
    }

    [HttpDelete("/Produtos/{id}")]
    public ActionResult<ProdutoDTO> Delete(int id)
    {
        var produto = _unitOfWork.ProdutoRepository.Get(p =>p.ProdutoId == id);

        if(produto is null)
            return NotFound($"Produto com id = {id} não localizado");

        var produtoExcluido = _unitOfWork.ProdutoRepository.Delete(produto);
        _unitOfWork.Commit();

        var produtoExcluidoDto = _mapper.Map<ProdutoDTO>(produtoExcluido);

        return Ok(produtoExcluidoDto);

    }
}

```

