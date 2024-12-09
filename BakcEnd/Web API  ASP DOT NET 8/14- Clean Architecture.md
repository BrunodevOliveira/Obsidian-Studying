# Criar solução com 5 projetos

Solution Catalogo
- `Catalogo.Domain` -> Modelo de domínio, Interfaces, regras de negócio
- `Catalogo.Application` -> Regras da aplicação, serviços, mapeamentos, DTOs
- `Catalogo.Infrastructure` -> Lógica acesso a dados, Contexto, Configurações, ORM
- `Catalogo.CrossCutting` -> IoC, Registro dos serviços e recursos, DI
- `Catalogo.API` -> Controladores, endpoints e filtros

Tipos de projeto
- `Catalogo.API `-> ASP .NET Core Web API
- Demais projetos -> Class Library (.Net 5.0)


## Catalogo.Domain

### Entities
- Devem conter comportamentos e propriedades
- Nelas devem estar definidas as regras de negócio do projeto
### Interfaces
- Define as interfaces responsáveis por implementar os repositórios
- Através dessas interfaces que conseguimos ter acesso ao banco de dados no projeto `Catalogo.Infraestructure`
- Interfaces também previne o acoplamento de dados


## Catalogo.Application
> [!NOTE]
> Projeto responsável por definir as <mark style="background-color: #fff88f; color: black">regras de negócio da aplicação</mark>, serviços, mapeamentos e DTO's 

### DTOs
- Servem para transferência de dados entre camadas
	- Nos DTOs posso utilizar os data anotations

### Interfaces
- Interfaces utilizadas para implementar os serviços
	-  serviços são responsáveis por implementar os casos de uso 

### Mappings
- Aqui definimos o mapeamento entre as entidades e DTOs

### Services
- Onde implementamos as interfaces descritas no diretório interfaces


## Catalogo.Infraestructure
> [!NOTE]
> Definimos a lógica de acesso aos dados. Contexto, as configurações e as dependências do ORM escolhido como o EF, Identity

### Context
- Aqui criamos as classes que irão fazer o mapeamento das minhas classes para o BD

### EntitiesConfiguration
- Aqui realizamos as configurações do EF utilizando a FluentAPI
- Essas configurações são feitas nessa camada para desaclopar meu domínio do framework utilizado para acesso ao banco
### Repositories
- Aqui implementamos os repositórios cujas interfaces são definidas na camada de domínio


## Catalogo.CrossCutting
> [!NOTE]
> Aqui registramos os serviços que serão utilizados através de Injeção de dependência

### Ioc
- Classes responsável por agrupar as dependências dos demais projetos
