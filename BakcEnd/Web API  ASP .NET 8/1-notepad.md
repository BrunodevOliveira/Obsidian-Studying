# Configurações DB na abordagem code first

- Instalo EF
- Crio as Classes Modelos que irão representar as tabelas
- Context > AppDbContext (Classe de cotexto) - Mapeio as entidades 
- Crio ConnectionString em appsettings.json
- Program.cs incluo o contexto do EF
- Crio o  relacionamento entre as classes antes de executar a Migration
- 

## Migrations
Responsável por atualizar de forma incremental o esquema de banco de dados para mantê-lo em sincronia com o modelo de dados do aplicativo, preservando os dados existentes no banco de dados.

Sempre que alterar as classes de modelo  é preciso executar o Migrations para manter o esquema do banco de dados atualizado

Comandos utilizados
- Criar o script de migração -> dotnet ef migrations add 'nome'
- Remove o script de migração criado -> dotnet ef migrations remove 'nome'
- Gera o banco de dados e as tabelas com base no script -> dotnet ef database update

Comandos executados na janela Package Manager Console
OBS.: Para executar instalar o pacote nuget Microsoft.EntityFrameworkCore.Tools
- Cria script de migração -> add-migration 'nome'
- Remove o script de migração -> remove-migration 'nome'
- Gera o banco de dados e as tabelas com base no script -> update-database

## Data Annotations
Conjunto de atributos que podem ser aplicados a classes e seus membros para fornecer metadados sobre como esses recursos devem ser tratados pelo sistema.
Ex.:
[Required]
[StringLength(80)]
OBS.: Fluent API substitui o uso de data annotations

## Populando tabelas com Migrations
1- Criar uma nova migração
	dotnet ef migrations add PopulaCategorias
OBS.: Quando não existe nenhuma mudança nos Models, o comando gera uma migration vazia.
2- Definir os comandos SQL no método **Up()** para incluir dados -> 
	insert into categorias(Nome, ImagemUrl) Values('Bebidas', 'bebidas.jpg')
3- Definir os comandos SQL no método **Down()** para reverter a migração
	delete from categorias
4- Aplicar a migração
	dotnet ef database update


# Controllers

São classes derivadas da classe *ControllerBase*

## Estrutura dos decoradores:
[ApiController] //Habilita recursos referentes a API
[Route("[controller]/{action}")] //Define as rotas e se existir mais de uma rota com o mesmo verbo, utilizamos uma variável para especificar a action que deve ser executada
public class NomeRotaController : ControllerBase {}

## Configuração inicial:
Program.cs habilitamos o serviço de controladores
	builder.Services.AddControllers();
	app.MapControllers()


## Criação

1- clique direito na pasta Controller > Adicionar > Controlador > API > Controlador vazio

2- Injetar o contexto no construtor para que o EF acesse o BD

3- Criar os métodos Actions


# Serialização
Importante configuração para evitar erro de referência cíclica 

## Princiapis métodos
ReferenceHandler -> define como o JsonSerialize lida com referências sobre serialização e desserialização

IgnoreCycles -> Ignora o objeto quando um ciclo de referência é detectado durante a serialização

## Program.cs
- Chamo o método AddJsonOptions
builder.Services.AddControllers().AddJsonOptions(options => options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles);


## Desserialização
- Posso indicar partes do JSON que devem ser ignorar (JsonIgnore)
- Nos modelos posso indicar que seja ignorada as propriedades de navegação
class Produto {
	[JsonIgnore]
	public Categoria? Categoria { get; set; }
}

# Otimização nas consultas

## Utilizar AsNoTRackking( em consultas GET
O EF armazena as entidades no contexto (em cache) realizando o tracking ou rastreamento das entidades para acompanhar o estado das entidades

Este recurso adiciona uma sobrecarga que afetas  o desempenho das consultas rastreadas

Para melhorar o desempenho podemos usar o método -> AsNoTRackking()
Ex.: var produtos = _context.Produtos.AsNoTracking().ToList()

Usar AsNoTRackking() apenas para consultas somente leitura (Verbo GET) sem a necessidade de alterar os dados.

## Nunca retornar todos os registros em uma consulta
Ex.: _context.Produtos.Take(10).ToList();

## Nunca retornar objetos relacionados sem aplicar filtro
Ex.: _context.Categorias.Include(p => p.Produtos).Where(c => c.CategoriaId <= 5).ToList();
