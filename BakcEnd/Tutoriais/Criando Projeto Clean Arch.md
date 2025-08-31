---
sticker: emoji//1f4d1
---
 Execute os seguintes comandos no seu terminal, na pasta onde deseja que o projeto seja criado (ex: C:\Users\cnrbr\projects\).

# Passo 1: Estrutura de Pastas e Solução

Vamos criar o diretório raiz, o arquivo de solução (.sln) e as pastas src e docker.
```shell
    # 1. Cria o diretório raiz do projeto e entra nele
     mkdir Base
     cd Base
     
     # 2. Cria o arquivo de solução (.sln)
     dotnet new sln --name Base

     3. Cria os diretórios para o código-fonte e para o docker
     mkdir src
     mkdir docker

```
  
# Passo 2: Criação dos Projetos

  Agora, vamos criar cada projeto dentro da pasta src, usando o template apropriado.

```shell
    # 1Base.Domain: Biblioteca de classes para suas entidades e regras de negócio puras.
    dotnet new classlib --name Base.Domain --output src/Base.Domain

    # 2. Base.Application: Biblioteca de classes para os casos de uso e lógica de aplicação.
    dotnet new classlib --name Base.Application --output src/Base.Application
    
    # 3. Base.Infrastructure: Biblioteca de classes para implementações de acesso a dados (EF Core), etc.
    dotnet new classlib --name Base.Infrastructure --output src/Base.Infrastructure

    # 4. Base.Communication: Biblioteca de classes para DTOs (Data Transfer Objects).
    dotnet new classlib --name Base.Communication --output src/Base.Communication
   
    # 5. Base.API: Projeto de API Web que servirá como ponto de entrada.
    dotnet new webapi --name Base.API --output src/Base.API
```

# Passo 3: Adicionar Projetos à Solução

  Os projetos foram criados, mas a solução ainda não os conhece. Vamos adicioná-los.
 ```shell
  dotnet sln Base.sln add src/Base.API/Base.API.csproj
  
  dotnet sln Base.sln add src/Base.Application/Base.Application.csproj
  
  dotnet sln Base.sln add src/Base.Domain/Base.Domain.csproj
  
  dotnet sln Base.sln add src/Base.Infrastructure/Base.Infrastructure.csproj
  
  dotnet sln Base.sln add src/Base.Communication/Base.Communication.csproj
```

# Passo 4: Referência entre projetos
Este é o passo mais crítico para garantir a Clean Architecture. A regra principal é: as dependências apontam para dentro. O Domain é o centro, e nada fora dele deve ser referenciado por ele.

  API → Application → Domain  → Infrastructure → Application
- A API depende da Application (para invocar os casos de uso) e da Communication (para os DTOs) e Infraestructure (Para registrar as implementações no container de DI) e Exception para gerenciamento de Exceções.

```shell
dotnet add src/Base.API/Base.API.csproj reference src/Base.Application/Base.Application.csproj

dotnet add src/Base.API/Base.API.csproj reference src/Base.Communication/Base.Communication.csproj

dotnet add src/Base.API/Base.API.csproj reference src/Base.Infrastructure/Base.Infrastructure.csproj

dotnet add src/Base.API/Base.API.csproj reference src/Base.Exception/Base.Exception.csproj
```

- A Application depende do Domain (para usar as entidades) e da Communication (para os contratos) e Exception para gerenciamento de Exceções.
```shell
dotnet add src/Base.Application/Base.Application.csproj reference src/Base.Domain/Base.Domain.csproj

dotnet add src/Base.Application/Base.Application.csproj reference src/Base.Communication/Base.Communication.csproj

dotnet add src/Base.Application/Base.Application.csproj reference src/Base.Exception/Base.Exception.csproj
```

- A Infrastructure depende do Domain
```shell
dotnet add src/Base.Infrastructure/Base.Infrastructure.csproj reference src/Base.Domain/Base.Domain.csproj
```

> [!caution] Importante
>  Os projetos Base.Domain e Base.Communication não devem ter nenhuma referência aos outros projetos da solução. Eles são o núcleo e os contratos, respectivamente.

