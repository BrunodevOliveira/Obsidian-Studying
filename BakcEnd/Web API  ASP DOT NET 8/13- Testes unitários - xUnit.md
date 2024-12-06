Arrange -> preparação
Act -> Execução
Assert -> Verificação

# Roteiro 
- Incluir o projeto de testes de unidade ApiCatalogoxUnitTests no projeto APICatalogo usando o template xUnit Test Project
- Incluir no projeto de testes o pacote nuget -> FluentAssertions
- Incluir uma referência ao projeto APICatalogo no projeto de testes
- Criar a classe `ProdutosUnitTestController` e configurar o ambiente para testar os endpoints do controlador `ProdutosController` da `APICatalogo`
- Criar as classes para testar os endpoints Get, Post, Put e DElete que implementa a interface `IClassFixture<ProdutosUnitTestController>`

```C#
namespace ApiCatalogoxUnitTests.UnitTests;  
  
public class ProdutosUnitTestController //Usada para configurar os teste {  
    public IUnitOfWork repository;  
    public IMapper mapper;  
    public static DbContextOptions<AppDbContext> dbContextOptions;  
  
    public static string connextionString = "Server=localhost;DataBase=CatalogoDB;Uid=root;Pwd=3453";  
  
    //Configura a variável  dbContextOptions. é executado apenas uma vez ao instanciar a classe.  
    static ProdutosUnitTestController()  
    {        dbContextOptions = new DbContextOptionsBuilder<AppDbContext>()  
	            .UseMySql(connextionString, ServerVersion.AutoDetect(connextionString))  
	            .Options;  
    }  
    public ProdutosUnitTestController()  
    {        
	    var config = new MapperConfiguration(cfg =>  
        {  
            cfg.AddProfile(new ProdutoDTOMappingProfile());  
        });        
        
        mapper = config.CreateMapper();  
        var context = new AppDbContext(dbContextOptions);  
        repository = new UnitOfWork(context);  
    }}
```

```C#
namespace ApiCatalogoxUnitTests.UnitTests.ProdutoController;  
  
//IClassFixture -> Permite instanciar ProdutosUnitTestController apenas uma vez e compartilhar essa instancia com todos os testes  
public class GetProdutoUnitTests : IClassFixture<ProdutosUnitTestController>  
{  
    private readonly ProdutosController _controller;  
  
    public GetProdutoUnitTests(ProdutosUnitTestController controller)  
    {        
	    _controller = new ProdutosController(controller.repository, controller.mapper);  
    }  
    
    [Fact]  
    public async Task GetProdutoById_OkResult()  
    {   
	    //Arrange  
        var prodId = 2;  
        
        //Act  
        var data = await _controller.Get(prodId);  
        
        //Assert (xunit)  
        // var okResult = Assert.IsType<OkObjectResult>(data);        
        // Assert.Equal(200, okResult.StatusCode);                
        
        //Assert (Fluentassertions)  
        //Verifica se o resultado é do tipo OkObjectResult é 200         data.Result.Should().BeOfType<OkObjectResult>().Which.StatusCode.Should().Be(200);  
    }  
    
    [Fact]  
    public async Task GetProdutoById_NotFound()  
    {        
	    //Arrange  
        var prodId = 999;  
        
        // Act  
        var data = await _controller.Get(prodId);  
        
        //Assert  
        data.Result.Should().BeOfType<NotFoundObjectResult>().Which.StatusCode.Should().Be(404);  
	}  
        
	[Fact]  
    public async Task GetProdutoById_BadRequest()  
    {        
	    //Arrange  
        var prodId = -1;  
        
        // Act  
        var data = await _controller.Get(prodId);  
        
        //Assert  
        data.Result.Should().BeOfType<BadRequestObjectResult>().Which.StatusCode.Should().Be(400);  
	}  
  
    [Fact]  
    public async Task GetProdutos_ListOfProdutoDTO()  
    {        
    
	    //Act  
        var data = await _controller.Get();  
        
        //Assert  
        data.Result.Should().BeOfType<OkObjectResult()
	        .Which.Value.Should().BeAssignableTo<IEnumerable<ProdutoDTO>>()  
            .And.NotBeNull();  
    }
      
    [Fact]  
    public async Task GetProdutos_BadRequest()  
    {        
	    var data = await _controller.Get();  
        data.Result.Should().BeOfType<BadRequestResult>().Which.StatusCode.Should().Be(400);  
    }}
```

