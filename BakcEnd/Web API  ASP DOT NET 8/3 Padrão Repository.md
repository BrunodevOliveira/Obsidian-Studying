O **Padrão Repository** é uma abordagem de design que visa <mark style="background-color: #fff88f; color: black">separar a lógica de acesso a dados da lógica de negócios em aplicações</mark>, como as desenvolvidas com **ASP.NET Core**. Este padrão facilita a manutenção e a testabilidade do código, permitindo que as camadas superiores da aplicação não dependam diretamente da tecnologia de persistência utilizada, como bancos de dados ou serviços externos.

![[Excalidraw/Padrão Repository.md#^clippedframe=3J8Q5VDlycR_GnZXIveBy]]
## Objetivos do Padrão Repository

1. **Abstração do Acesso a Dados**: O repositório atua como uma interface entre a aplicação e a fonte de dados, encapsulando a lógica necessária para acessar e manipular os dados. Isso permite que as camadas superiores interajam com os dados sem se preocupar com detalhes de implementação específicos[

2. **Separação de Responsabilidades**: Ao isolar o acesso a dados em repositórios, o padrão promove uma melhor organização do código, facilitando a manutenção e a evolução da aplicação. As classes de negócios podem se concentrar em suas responsabilidades sem se misturar com o código de acesso a dados[

3. **Facilidade de Testes**: Com o uso de interfaces e injeção de dependência, é possível simular repositórios em testes unitários, permitindo que os desenvolvedores testem suas classes de negócios sem depender de um banco de dados real

## Passos para implementação

Criar a interface  que será o contrato que a classe concreta deve implementar

```C#
namespace APICatalogo.Repositories;

public interface ICategoriaRepository
{
    IEnumerable<Categoria> GetCategorias();

    Categoria GetCategoria(int id);

    Categoria Create(Categoria categoria);

    Categoria Update(Categoria categoria);
    
    Categoria Delete(int id);
}
```


Criar a classe concreta que implemente a interface

```C#
namespace APICatalogo.Repositories;

public class CategoriaRepository : ICategoriaRepository
{
    private readonly AppDbContext _context;

    public CategoriaRepository(AppDbContext context)
    {
        _context = context;
    }

    public IEnumerable<Categoria> GetCategorias()
    {
        return _context.Categorias.ToList();
    }

    public Categoria GetCategoria(int id)
    {
        //FirstOrDefault -> executa a consulta diretamente no banco de dados
        return _context.Categorias.FirstOrDefault(c => c.CategoriaId == id);
    }

    public Categoria Create(Categoria categoria)
    {
        if(categoria is null) throw new ArgumentNullException(nameof(categoria));

        _context.Categorias.Add(categoria);
        _context.SaveChanges();
        return categoria;
    }

    public Categoria Update(Categoria categoria)
    {
        if (categoria is null) throw new ArgumentNullException(nameof(categoria));
        _context.Entry(categoria).State = EntityState.Modified; //Informa ao contexto do EF Core que o objeto categoria foi modificado e precisa ser atualizado no BD

        _context.SaveChanges();

        return categoria;
    }

    public Categoria Delete(int id)
    {
        //Find -> Busca primeiro na memoria, caso não encontre faz a consulta ao banco de dados.
        var categoria =  _context.Categorias.Find(id);
        
        if (categoria is null) throw new ArgumentNullException(nameof(categoria));

        _context.Categorias.Remove(categoria);
        _context.SaveChanges();
        
        return categoria;
    }
}
```


Adicionar no `Container DI` em `Program.cs` o ciclo de vida dessa injeção 

```C#
// uma instância de CategoriaRepository será criada uma vez para cada escopo de request.
builder.Services.AddScoped<ICategoriaRepository, CategoriaRepository>();
```


