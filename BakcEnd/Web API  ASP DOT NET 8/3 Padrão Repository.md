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


# Repositório Genérico

Cria uma `interface genérica` contendo o contrato que define métodos genéricos que a `classe concreta` deverá implementar

## Abordagem hibrida
<mark style="background-color: #fff88f; color: black">Combina repositórios genéricos</mark> para operações de <mark style="background-color: #fff88f; color: black">acesso a dados comuns</mark> e <mark style="background-color: #fff88f; color: black">repositórios específicos</mark> quando <mark style="background-color: #fff88f; color: black">funcionalidades personalizadas</mark> são necessárias para entidades específicas.

Lógica de implementação:
- Criar um <span style="color:rgb(107, 255, 174)">Repositório Genérico </span>para <span style="color:rgb(107, 255, 174)">operações</span> CRUD <span style="color:rgb(107, 255, 174)">comuns</span> usadas
- Criar um <span style="color:rgb(255, 255, 0)">Repositório Específico</span> para <span style="color:rgb(255, 255, 0)">operações</span> <span style="color:rgb(255, 255, 0)">específicas</span> para cada entidade
- <span style="color:rgb(254, 0, 65)">Herdar o Repositório Específico do Repositório Genérico</span> 

## Implementação

### 1-Interface genérica (`IRepository<T>`)
A interface genérica é a base do padrão, definindo operações comuns que serão compartilhadas por todos os repositórios:

```C#
using System.Linq.Expressions;

namespace APICatalogo.Repositories;

//T representa o tipo do repositório(classe) a ser implementado
//Essa interface é herdada pelos repositórios específicos
public interface IRepository<T>
{
    IEnumerable<T> GetAll();

    //Expressio -> Representa uma função Lambda
    //Func<T, bool> -> Delegate que representa uma função lambda que recebe um obj do tipo T e reotrna um bool 
    //predicate -> O critério que será usado para filtrar
    T? Get(Expression<Func<T, bool>> predicate);

    T Create(T entity);

    T Put(T entity);
    T Delete(T entity);
}
```

**Pontos importantes:**
- `T` é um tipo genérico que representa a entidade
- `Expression<Func<T, bool>>` permite passar expressões lambda para filtrar dados
- Métodos definem operações CRUD básicas

### 2-Implementação genérica(`Repository<T>`)
A classe genérica implementa a interface e fornece a lógica comum

```C#
namespace APICatalogo.Repositories;

/*
    - Repository é uma classe genérica que implementa uma interface Genérica
    - where T : class -> restrição para garantir que o Tipo T seja uma classe
 */
public class Repository<T> : IRepository<T> where T : class
{
    protected readonly AppDbContext _context;

    public Repository(AppDbContext contex)
    {
        _context = contex;
    }

    public IEnumerable<T> GetAll()
    {
        //Método Set é utilizado para acessar uma tabela ou coleção
        return _context.Set<T>().ToList();
    }

    public T? Get(Expression<Func<T, bool>> predicate)
    {
        return _context.Set<T>().FirstOrDefault(predicate);
    }

    public T Create(T entity)
    {
        _context.Set<T>().Add(entity);
        _context.SaveChanges();
        return entity; 
    }

    public T Put(T entity)
    {
        _context.Set<T>().Update(entity);
        _context.SaveChanges(); 

        return entity;
    }

    public T Delete(T entity)
    {
        _context.Set<T>().Remove(entity);

        _context.SaveChanges();

        return entity;
    }
}
```


### 3-Interface específica
Na classe `Program.cs` adiciono no container de injeção de dependência a classe `Repository`:

```C#
public interface IProdutoRepository : IRepository<Produto> 
{ 
	IEnumerable<Produto> GetProdutoPorCategoria(int id); 
}
```

**Características:**
- Herda todos os métodos genéricos
- Adiciona métodos específicos da entidade
- Aumenta a flexibilidade do repositório

Na interface especifica do repositório, herdo a interface genérica `IRepository` passando como tipo a classe que representa o Modelo dos dados.

Obs.: A interface específica herda todas as assinaturas dos métodos que estão em `IRepository`. Também podemos implementar novos métodos que serão exibidos apenas nessa interface específica.


### 4-Implementação específica
Implementa a lógica específica para cada entidade

```C#
public class ProdutoRepository : Repository<Produto>, IProdutoRepository
{
    public ProdutoRepository(AppDbContext context) : base(context) { }
    
    public IEnumerable<Produto> GetProdutoPorCategoria(int id)
    {
        return GetAll().Where(c => c.CategoriaId == id);
    }
}
```
**Pontos-chave:**
- Herda a implementação genérica
- Implementa a interface específica
- Pode acessar o contexto através de `_context`

Na classe concreta que irá representar o repositório devemos herdar da classe genérica `Repository` e implementar a interface específica:

### 5-configuração no `Program.cs`

```C#
//Registro do Repository genérico para podet acessar o banco por ele
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
```


