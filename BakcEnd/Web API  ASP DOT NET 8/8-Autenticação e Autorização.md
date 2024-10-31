# Json Web token

Usa **tokens criptografados** para permitir o acesso aos recursos de uma Web API - essa abordagem é conhecida como **Bearer Authentication**
> [!NOTE]
> Para gerar um token JWT não precisamos de um banco de dados, mas apenas de informações do usuário para criação do token

## Token JWT
É formado por 3 partes
- Header -> Contém o tipo de token e a criptografias usada (HMAC ou RSA)
- Payload -> Contém os 'pedidos'. São declarações ou claims associadas ao usuário do token
- Signature -> Assinatura usada na validação do token usando uma **chave secreta**

> [!NOTE]
> Para gerar a ssinatura usamos o Header e o Payload codificando-os e a seguir assinamos usando o algoritmo definido

## Gerenciando tokens JWT no ambiente de Dev
`dotnet user-jwts create` -> Gera um token JWT com um Id Definido
- Registra automaticamente um conjunto de números secretos `<UserSecretId>` exigidos pelo **Secret Manager** no arquivo de projeto `.csproj` e define as configurações de autenticação no arquivo appsettings.json para o desenvolvimento.

`dotnet user-jwts list` -> Lista todos os tokens JWT emitidos

`dotnet user-jwts key` -> Obtem a chave de emissão do token JWT atual

`dotnet user-jwts remove ID` -> Exclui o token JWT emitido pelo seu ID

`dotnet user-jwts clear` -> Limpa todos os tokens emitidos

`dotnet user-jwts key --reset -force` -> Redefine a chave de emissão do token JWT atualmente em uso

`dotnet user-jwts print <id-token> --show-all` -> Exibe todas as informações do token


## Instalação
1- Instalação do pacote nuget `Microsoft.AspNetCore.Authentication.JwtBearer`

2- Configuração no `program.cs`

```C#
builder.Services.AddAuthorization();  
builder.Services.AddAuthentication("Bearer").AddJwtBearer();
```

3- Atribuir o atributo `[Authorize]` no endpoit que será protegido

```C#
[HttpGet]  
[Authorize]  
[ServiceFilter(typeof(ApiLoggingFilter))]  
public async Task<ActionResult<IEnumerable<CategoriaDTO>>> Get()  
{  
    var categorias = await _unitOfWork.CategoriaRepository.GetAllAsync();  
  
    if (categorias is null) return NotFound("Não existem categorias...");  
  
    var categoriasDto = CategoriaDTOMappingExtensions.ToCategoriasDtoList(categorias);  
  
    return Ok(categoriasDto);  
}
```

4- Abro o terminal dentro do diretório do projeto e executo o comando `dotnet user-jwts create`

Após a execução do comando será gerado uma propriedade `Authentication` no arquivo `appsettings.Development.json`

```C#
"Authentication": {  
  "Schemes": {  
    "Bearer": {  
      "ValidAudiences": [  
        "http://localhost:24392",  
        "https://localhost:44329",  
        "http://localhost:5180",  
        "https://localhost:7014"  
      ],  
      "ValidIssuer": "dotnet-user-jwts"  
    }  
}}
```

No arquivo do projeto `APICatalogo.csproj` é adicionado o ID (UserSecretsId):

```C#
<PropertyGroup>  
  <TargetFramework>net8.0</TargetFramework>  
  <Nullable>enable</Nullable>  
  <ImplicitUsings>enable</ImplicitUsings>  
  <UserSecretsId>a0496c99-48a4-493c-8ec5-1aa0ce6afebb</UserSecretsId>  
</PropertyGroup>
```


# Identity

Usar o Identity para criar as tabelas para persistência das informações do usuário com as informações de login

Configurar o Identity usando o MySQL para armazenar nome, senha e dados do usuário

O Identity vai ser usado para poder <mark style="background-color: #fff88f; color: black">autenticar o usuário e obter informações deste usuário para gerar o token</mark>

## Configurando o projeto para utilizar Identity

1- Alterar a classe de contexto `AppDbContext.cs` para herdar de `IdentityDbContext`
- Antes de herdar, precisamos instalar o seguinte pacote -> `Microsoft.AspNetCore.Identity.EntityFrameworkCore`
```C#
public class AppDbContext : IdentityDbContext
```

2- Configurar o Identity na classe `programs.cs` 

```C#
builder.Services.AddIdentity<IdentityUser, IdentityRole>() //IdentityUser = Usuário | IdentityRole = funções do usuário  
    .AddEntityFrameworkStores<AppDbContext>() //Adiciono Indentity para armazenar os dados tendo como base meu contexto  
    .AddDefaultTokenProviders(); //tokens para lidar com a autenticação
```

3- Aplicar o Migrations do EF Core para gerar as tabelas do Identity no banco de dados MySQL:
- Gerar o script de migração -><span style="color:rgb(255, 255, 0)"><span style="color:rgb(255, 255, 0)"> <span style="color:rgb(255, 255, 0)">(Dentro da pasta do projeto)</span></span> </span>`dotnet ef migrations add CriaTabelasIdentity --verbose`
- Aplicar o script de migração gerado -><span style="color:rgb(255, 255, 0)"><span style="color:rgb(255, 255, 0)"> <span style="color:rgb(255, 255, 0)">(Dentro da pasta do projeto)</span></span> </span>`dotnet ef database update --verbose`



## Implementação JWT Token

1- Habilitar e configurar a autenticação JWT Bearer
- Verificar se já está Adicionado o pacote `Microsoft.AspNetCore.Authentication.JwtBearer`
- Definir em um locar seguro (appsettings.json) -> chave secreta, a audiência, o emissor e o tempo de vida
```json
"JWT": {  
  "ValidAudience": "http://localhost:7066",  
  "ValidIssuer": "http://localhost:5066",  
  "SecretKey": "Minha@Super#Secreta&Chave*Privada!2023%",  
  "TokenValidityInMinutes": 30,  
  "RefreshTokenValidityInMinutes": 60  
}
```
-  Na classe `program.cs` Habilito e configuro o JWT Bearer na aplicação
```C#
var secretKey = builder.Configuration["JWTSettings:SecretKey"] ??   
    throw new ArgumentNullException("Invalid secret Key!");

builder.Services.AddAuthentication(options =>  
{  
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;  
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;  
}).AddJwtBearer(options =>  
{  
	//Configuração de validação do token
    options.SaveToken = true;  
    options.RequireHttpsMetadata = false;  
    options.TokenValidationParameters = new TokenValidationParameters()  
    {  
        ValidateIssuer = true,  
        ValidateAudience = true,  
        ValidateLifetime = true,  
        ValidateIssuerSigningKey = true,  
        ClockSkew = TimeSpan.Zero,  
        ValidAudience = builder.Configuration["JWTSettings:Audience"],  
        ValidIssuer = builder.Configuration["JWTSettings:Issuer"],  
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(secretKey)),  
  
    };  
  
});
```


2- Criar um serviço `ItokenService`
- Gerar um Token JWT
- Gerar o RefreshToken (permite obter um novo token sem fazer login)
- Revogar o Refresh Token

3- Criar controller AuthController com os endpoints para:
- Login
- Register
- Refreshtoken
- Revoke
