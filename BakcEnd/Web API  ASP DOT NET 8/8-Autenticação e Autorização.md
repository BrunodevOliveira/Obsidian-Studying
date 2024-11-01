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

## 1-Configurando o projeto para utilizar Identity

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



## 2-Implementação JWT Token

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


1.1 - Criar a classe `ApplicationUser` e ajustar o código para utilizar RefresToken

Em Models Crio a classe `ApplicationUser` :
```C#
namespace APICatalogo.Models;  
  
public class ApplicationUser : IdentityUser  
{  
    public string? RefreshToken { get; set; }  
    public DateTime RefreshTokenExpiryTime { get; set; }  
}
```

Ajustar a classe AppDbContext para utilizar `ApplicationUser` como um tipo genérico de `IdenttityDbContext`:
```C#
public class  AppDbContext : IdentityDbContext<ApplicationUser> 
{   
	public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)  
	{    }  

    public DbSet<Categoria>? Categorias { get; set; }  
    public DbSet<Produto>? Produtos { get; set; }  
	
	protected override void OnModelCreating(ModelBuilder builder)  
	{        base.OnModelCreating(builder);    }
}
```

Ajustar o código da classe `program.cs` que configura o Identity no contexto de injeção de dependência
```C#
//Configuração do Indentity  
builder.Services.AddIdentity<ApplicationUser, IdentityRole>() //IdentityUser/ApplicationUser = Usuário | IdentityRole = funções do usuário  
    .AddEntityFrameworkStores<AppDbContext>() //Adiciono Indentity para armazenar os dados tendo como base meu contexto  
    .AddDefaultTokenProviders(); //tokens para lidar com a autenticação
```

Após essa mudanças nos arquivos, <span style="color:rgb(255, 255, 0)">aplicamos um Migrations</span>:
- `dotnet ef migrations add AjusteApplicationUser`
- `dotnet ef database update`

1.2 - Criando DTOs para gerenciar o login, o registro e o token
```C#
namespace APICatalogo.DTOs;
public class RegisterModelDTO  
{  
    [Required(ErrorMessage = "Nome do usuário é obrigatório")]  
    public string?  UserName { get; set; }  
        [EmailAddress]  
    [Required(ErrorMessage = "Email é obrigatório")]  
    public string? Email { get; set; }  
        [Required(ErrorMessage = "Senha é obrigatório")]  
    public string?  Password { get; set; }  
}
```
```c#
namespace APICatalogo.DTOs;
public class TokenModelDTO  
{  
    public string? AcessToken { get; set; }  
    public string? RefrehToken { get; set; }  
}
```
```C#
namespace APICatalogo.DTOs;
public class ResponseDTO  
{  
    public string? Status { get; set; }  
    public string? Message { get; set; }  
}
```


2- Criar um serviço `ItokenService`

Responsabilidades:
- Gerar um Token JWT
- Gerar o RefreshToken (permite obter um novo token sem fazer login)
- Revogar o Refresh Token

Criar `ItokenService` e class `TokenService`
```C#
namespace APICatalogo.Services;  
  
public interface ITokenService  
{  
    JwtSecurityToken GenerateAccessToken(IEnumerable<Claim> claims, IConfiguration _config);  
    string GenerateRefreshToken();  
    ClaimsPrincipal GetPrincipalFromExpiredToken(string token, IConfiguration _config);  
}
```

```C#
namespace APICatalogo.Services;  
  
public class TokenService : ITokenService  
{  
    public JwtSecurityToken GenerateAccessToken(IEnumerable<Claim> claims, IConfiguration _config)  
    {        //Obter a chave secreta definada no appsettings.json  
        var key = _config.GetSection("JWT").GetValue<string>("SecretKey") ??  
                  throw new NullReferenceException("SecretKey");  
        //Converto a chave para um array de bytes  
        var privateKey = Encoding.UTF8.GetBytes(key);  
  
        //Criar as credencias de assinatura  
        var signingCredentials = new SigningCredentials(new SymmetricSecurityKey(privateKey),  
            SecurityAlgorithms.HmacSha256Signature);  
  
        //Informações que o token terá ao ser gerado  
        var tokenDescriptor = new SecurityTokenDescriptor  
        {  
            Subject = new ClaimsIdentity(claims),  
  
            Expires = DateTime.UtcNow.AddMinutes(_config.GetSection("JWT")  
                .GetValue<double>("TokenValidityInMinutes")),  
  
            Audience = _config.GetSection("JWT").GetValue<string>("ValidAudience"),  
  
            Issuer = _config.GetSection("JWT").GetValue<string>("ValidIssuer"), //Valor do emissor  
  
            //atribuo as credencias criadas no passo acima            SigningCredentials = signingCredentials  
        };  
        var tokenHandler = new JwtSecurityTokenHandler();  
  
        var token = tokenHandler.CreateJwtSecurityToken(tokenDescriptor);  
  
        return token;  
    }  
    public string GenerateRefreshToken()  
    {        //Crio um array de bytes vazia  
        var secureRandomBytes = new byte[128];  
  
        //Intancia de  RandomNumberGenerator  
        using var randomNumberGenerator = RandomNumberGenerator.Create();  
  
        //Populo a variavel secureRandomBytes com bytes aleatórios  
        randomNumberGenerator.GetBytes(secureRandomBytes);  
  
        //Converto o bytes para uma representação em string no formato base64  
        var refreshToken = Convert.ToBase64String(secureRandomBytes);  
  
        return refreshToken;  
    }  
    //Retorna as Clains do token expirado  
    public ClaimsPrincipal GetPrincipalFromExpiredToken(string token, IConfiguration _config)  
    {        var secretKey = _config.GetSection("JWT").GetValue<string>("SecretKey") ??  
                        throw new NullReferenceException("SecretKey");  
  
        //Parâmetros de validação do token  
        var tokkenValidationParameters = new TokenValidationParameters  
        {  
            ValidateAudience = false,  
            ValidateIssuer = false,  
            ValidateIssuerSigningKey = true,  
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(secretKey)),  
            ValidateLifetime = false,  
        };  
  
        var tokenHandler = new JwtSecurityTokenHandler();  
  
        //Valida o token expirado com base nos parâmetros de validação configurados em tokkenValidationParameters  
        //out SecurityToken securityToken -> Parâmetro de saída        var principal = tokenHandler.ValidateToken(token, tokkenValidationParameters, out SecurityToken securityToken);  
  
        /*  
         * Se o parâmetro de saída securityToken não for uma instância de JwtSecurityToken         * ou o algoritmo não for o HmacSha256, executamos uma exceção         */        if (securityToken is not JwtSecurityToken jwtSecurityToken ||  
            !jwtSecurityToken.Header.Alg.Equals(SecurityAlgorithms.HmacSha256,  
                StringComparison.InvariantCultureIgnoreCase)  
           )        {            throw new SecurityTokenException("Invalid token");  
        }  
        return principal;  
    }}
```

Registrar o serviço no container DI na classe Program


3- Criar controller AuthController com os endpoints para:
- Login
- Register
- Refreshtoken
- Revoke
 