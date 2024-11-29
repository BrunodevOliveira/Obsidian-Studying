# Json Web token

Usa **tokens criptografados** para permitir o acesso aos recursos de uma Web API - essa abordagem é conhecida como **Bearer Authentication**
> [!NOTE]
> Para gerar um token JWT não precisamos de um banco de dados, mas apenas de informações do usuário para criação do token

# Token JWT
É formado por 3 partes
- Header -> Contém o tipo de token e a criptografias usada (HMAC ou RSA)
- Payload -> Contém os 'pedidos'. São declarações ou claims associadas ao usuário do token
- Signature -> Assinatura usada na validação do token usando uma **chave secreta**

> [!NOTE]
> Para gerar a ssinatura usamos o Header e o Payload codificando-os e a seguir assinamos usando o algoritmo definido

# Gerenciando tokens JWT no ambiente de Dev
> [!NOTE]
> Aqui é ensinado a utilizar token JWT sem a necessidade de consulta a um DB para gerar o token.
> É uma maneira de testar tokens no ambiente de desenvolvimento
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

- Após a execução do comando será gerado uma propriedade `Authentication` no arquivo `appsettings.Development.json`

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


## Comandos básicos
`dotnet user-jwts create` -> Gera um token JWT com um Id Definido
- Registra automaticamente um conjunto de números secretos `<UserSecretId>` exigidos pelo **Secret Manager** no arquivo de projeto `.csproj` e define as configurações de autenticação no arquivo appsettings.json para o desenvolvimento.

`dotnet user-jwts list` -> Lista todos os tokens JWT emitidos

`dotnet user-jwts key` -> Obtem a chave de emissão do token JWT atual

`dotnet user-jwts remove ID` -> Exclui o token JWT emitido pelo seu ID

`dotnet user-jwts clear` -> Limpa todos os tokens emitidos

`dotnet user-jwts key --reset -force` -> Redefine a chave de emissão do token JWT atualmente em uso

`dotnet user-jwts print <id-token> --show-all` -> Exibe todas as informações do token





# Identity

- Usar o Identity para criar as tabelas para persistência das informações do usuário com as informações de login

- Configurar o Identity usando o MySQL para armazenar nome, senha e dados do usuário

- O Identity vai ser usado para poder <mark style="background-color: #fff88f; color: black">autenticar o usuário e obter informações deste usuário para gerar o token</mark>

Tabelas geradas pelo Identity:

**AspNetUsers**

- Armazena informações dos usuários como username, email, senha (hash), dados de perfil
- Mantém status de confirmação de email e telefone
- Controla bloqueios de conta

**AspNetRoles**

- Armazena as funções/papéis disponíveis no sistema
- Permite criar hierarquias de permissões
- Facilita o controle de acesso baseado em roles

**AspNetUserRoles**

- Tabela de relacionamento entre usuários e roles
- Estabelece quais funções cada usuário possui

**AspNetUserClaims**

- Armazena claims (declarações) adicionais dos usuários
- Permite incluir informações customizadas como preferências

**AspNetUserLogins**

- Registra logins externos (Google, Facebook, etc)
- Mantém vínculo entre contas locais e externas

**AspNetUserTokens**

- Armazena tokens para reset de senha, confirmação de email
- Mantém tokens de autenticação two-factor

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

### 1- Habilitar e configurar a autenticação JWT Bearer
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


### 2- Criar um serviço `ItokenService`

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
```C#
builder.Services.AddScoped<ITokenService, TokenService>();
```



### 3- Criar controller AuthController com os endpoints para:
- Login
- Register
- Refreshtoken
- Revoke
Na pasta controller criamos o arquivo `AuthController`
```C#
[Route("api/[controller]")]  
[ApiController]  
public class AuthController : ControllerBase  
{  
    private readonly ITokenService _tokenService;  
    private readonly UserManager<ApplicationUser> _userManager;  
    private readonly RoleManager<IdentityRole> _roleManager;  
    private readonly IConfiguration _configuration;  
  
    public AuthController(ITokenService tokenService, UserManager<ApplicationUser> userManager,  
        RoleManager<IdentityRole> roleManager, IConfiguration configuration)  
    {        _tokenService = tokenService;  
        _userManager = userManager;  
        _roleManager = roleManager;  
        _configuration = configuration;  
    }  
    //Gera o REfreshToken e Token para a sessão  
    [HttpPost]  
    [Route("login")]  
    public async Task<IActionResult> Login([FromBody] LoginModelDTO model)  
    {        var user = await _userManager.FindByNameAsync(model.UserName!); //Verifica se o usuário existe  
        var senhaUserValida = await _userManager.CheckPasswordAsync(user, model.Password!);  
  
        if (user is not null && senhaUserValida)  
        {            var userRoles = await _userManager.GetRolesAsync(user); //Perfis do usuário logado  
                        //Informações do usuário que serão incluidas no token  
            var authClaims = new List<Claim>  
            {                new Claim(ClaimTypes.Name, user.UserName!),  
                new Claim(ClaimTypes.Email, user.Email!),  
                new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),//Gera um Id para o token  
            };  
            //Inclui os perfis na lista de claims do usuário  
            foreach (var userRole in userRoles)  
            {                authClaims.Add(new Claim(ClaimTypes.Role, userRole));  
            }            var token  = _tokenService.GenerateAccessToken(authClaims, _configuration);//Gera o token  
            var refreshToken = _tokenService.GenerateRefreshToken();  
            //Converte o valor atribuido ao REfreshToken em appsettings.json para inteiro e atribui a variável refreshTokenValidityInMinutes  
            // _ -> é uma variável que usamos quando não estamos interessados no retorno da operação            _ = int.TryParse(_configuration["JWT:RefreshTokenValidityInMinutes"], out int refreshTokenValidityInMinutes);  
                          
            user.RefreshTokenExpiryTime = DateTime.Now.AddMinutes(refreshTokenValidityInMinutes);  
                        user.RefreshToken = refreshToken;  
            await _userManager.UpdateAsync(user); //Armazena o RefreshToken e RefreshTokenExpiryTime na tabela ASPNetUser do Identity  
            return Ok(new  
            {  
                Token = new JwtSecurityTokenHandler().WriteToken(token),  
                RefreshToken = refreshToken,  
                Expiration = token.ValidTo  
            });   
        }  
        return Unauthorized();  
    }  
	[HttpPost]  
    [Route("register")]  
    public async Task<IActionResult> Register([FromBody] RegisterModelDTO model)  
    {        var userExist = await _userManager.FindByNameAsync(model.UserName!);  
  
        if (userExist is not null)  
        {            return StatusCode(StatusCodes.Status500InternalServerError,  
                new ResponseDTO() {Status = "Error", Message = "User already exists!"});  
        }  
        ApplicationUser user = new()  
        {            UserName = model.UserName,  
            Email = model.Email,  
            SecurityStamp = Guid.NewGuid().ToString(),  
        };  
        var result = await _userManager.CreateAsync(user, model.Password!);  
  
        if (!result.Succeeded)  
        {            return StatusCode(StatusCodes.Status500InternalServerError,  
                new ResponseDTO() {Status = "Error", Message = "User creation failed!"});  
        }        return Ok(new ResponseDTO() {Status = "Success", Message = "User created successfully!"});   
    }  
  
    // Retorna um novo token JWT a partir do Token expiradoe e do  RefreshToken  
    [HttpPost]  
    [Route("refresh-token")]  
    public async Task<IActionResult> RefreshToken(TokenModelDTO tokenModel) //Recebe o RefreshToken e o token JWT expirado  
    {  
        if (tokenModel is null)  
        {            return BadRequest("Invalid client request");  
        }        //token JWT expirado  
        string? accessToken = tokenModel.AcessToken ?? throw new ArgumentNullException(nameof(tokenModel.AcessToken));  
        string? refreshToken = tokenModel.RefrehToken ?? throw new ArgumentNullException(nameof(tokenModel.RefrehToken));  
        //Extrai as Claims do token JWT expirado para gerar um novo token  
        var principal = _tokenService.GetPrincipalFromExpiredToken(accessToken!, _configuration);  
  
        if (principal == null)  
        {            return BadRequest("Invalid access token/refresh token");  
        }        string username = principal.Identity.Name;  
        //Busca o usuário no banco  
        var user = await _userManager.FindByNameAsync(username!);  
        /*  
         * Verifica se o usuário existe;         * Compara o refreshToken armazenado na tabela para esse usuário com o  informado no request;         * Compara a data do refreshToken com a data atual         */        
         if (user == null || user.RefreshToken != refreshToken || user.RefreshTokenExpiryTime < DateTime.Now)  
        {            return BadRequest("Invalid access token/refresh token");  
        }        //Caso o refreshToken for valid, gero um novo token de acesso  
        var newAccessToken = _tokenService.GenerateAccessToken(principal.Claims.ToList(), _configuration);  
        //Crio um novo refreshToken  
        var newRefreshToken = _tokenService.GenerateRefreshToken();  
        //Atualizo o valor do RefreshToken do usuário na tabela  
        user.RefreshToken = newRefreshToken;  
        await _userManager.UpdateAsync(user);  
  
        //Retorna um novo Token de acesso e um novo refreshToken que foi armazenado na tabela para o usuário   
		return new ObjectResult(new  
        {  
            accessToken = new JwtSecurityTokenHandler().WriteToken(newAccessToken),  
            refreshToken = newRefreshToken,  
        });  
    }  
    [Authorize] //Somente um usuário autenticado poderá acessar essa rota  
    [HttpPost]  
    [Route("revoke/{username}")]  
    public async Task<IActionResult> Revoke(string username)  
    {        var user = await _userManager.FindByNameAsync(username);  
        if(user is null) return BadRequest("Invalid user name");  
                user.RefreshToken = null;  
        await _userManager.UpdateAsync(user);  
        return NoContent();  
    }}
```

### 4- Configuração do `Swagger` para acesso via token
Em `Program.cs` substituo a configuração padrão do swagger por esta que possibilita testar os endpoints protegidos por Token
```C#
//Configuração antiga a ser substituida -> builder.Services.AddSwaggerGen()
builder.Services.AddSwaggerGen(c =>  
{  
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "APICatalogo", Version = "v1" });  
        c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme()  
    {  
        Name = "Authorization",  
        Type = SecuritySchemeType.ApiKey,  
        Scheme = "Bearer",  
        BearerFormat = "JWT",  
        In = ParameterLocation.Header,  
        Description = "Bearer JWT."  
    });  
  
    c.AddSecurityRequirement(new OpenApiSecurityRequirement()  
    {  
        {            new OpenApiSecurityScheme  
            {  
                Reference = new OpenApiReference  
                {  
                    Type = ReferenceType.SecurityScheme,  
                    Id = "Bearer"  
                }  
            },            new string[] {}  
        }    });});
```

## 3-Autorização baseada em políticas com JWT

- Permite definir regras personalizadas para controlar o acesso a recursos
- Podem ser aplicadas em controladores, métodos Action ou a nível de aplicativo
- A ideia por trás das políticas é centralizar e organizar a lógica de autorização, tornando-a mais fácil de gerenciar e entender

### Funcionamento
- Ao receber o request, a ASP.NET Core verifica o token JWT
- Com base nas informações do token, as políticas de autorização são aplicadas
- Se o usuário atender aos requisitos da política, ele terá acesso ao recurso solicitado

### Configuração e aplicação de politica

- Usar `Claims` do usuário atribuídas na geração do token JWT
- Usar a informações como e-mail, nome e perfil ou função (Role) do usuário.
- Aproveitar as tabelas do Identity `aspnetroles` e `aspnetuerroles`
- Permitir a criação de Role e também atribuir um usuário a uma Role
	- Adicionar dois endpoints: `CreateRole` e `AddUserToRole`
- Criar e configurar políticas com base nas Claims atribuídas ao token JWT
- Aplicar essas políticas aos endpoints da nossa API

#### Criação dos Endpoints `CreateRole` e `AddUserToRole`:

```C#
[HttpPost]  
[Route("CreateRole")]  
public async Task<IActionResult> CreateRole(string roleName)  
{  
    var roleExist = await _roleManager.RoleExistsAsync(roleName);  
  
    if (!roleExist)  
    {        var roleResult = await _roleManager.CreateAsync(new IdentityRole(roleName));  
  
        if (roleResult.Succeeded)  
        {            _logger.LogInformation(1, "Role created successfully");  
            return StatusCode(StatusCodes.Status200OK,   
			new ResponseDTO() {Status = "Success", Message = $"Role {roleName} created successfully!"});  
		}else  
        {  
            _logger.LogInformation(2, "Error");  
            return StatusCode(StatusCodes.Status400BadRequest,  
                new ResponseDTO() {Status = "Error", Message = $"Issue adding the new {roleName} role"});  
        }    }   
         return StatusCode(StatusCodes.Status400BadRequest,  
        new ResponseDTO() {Status = "Error", Message = $"Role {roleName} already exists!"});  
}
```

```C#
[HttpPost]  
[Route("AddUserToRole")]  
public async Task<IActionResult> AddUserToRole(string email, string roleName)  
{  
    var user = await _userManager.FindByEmailAsync(email);  
  
    if (user != null)  
    {        var result = await _userManager.AddToRoleAsync(user, roleName);  
  
        if (result.Succeeded)  
        {            _logger.LogInformation(1, $"User {user.Email} added to role {roleName}");  
            return StatusCode(StatusCodes.Status200OK,   
			new ResponseDTO() {Status = "Success", Message = $"User {user.Email} added to role {roleName}"});  
        } else  
        {  
            _logger.LogInformation(2, $"Error: Unable to add user {user.Email} to role {roleName}");  
            return StatusCode(StatusCodes.Status400BadRequest, new ResponseDTO() { Status = "Error", Message =   
				$"Error: Unable to add user {user.Email} to role {roleName}"});  
        }    }    
        return BadRequest(new { error = "Unable to find user" });  
}
```

#### Configurando Autorização
- Aqui definimos regras para controlar o acesso a recursos
- As políticas são declarativas e podem ser aplicadas em `controladores` `métodos actions` ou a `nível de aplicativo`
```C#
//Program.cs
builder.Services.AddAuthorization(options =>  
{     
//RequireRole -> Exige que o usuário tenha uma determinada Role para acessar o recurso  
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));  
    //RequireClaim -> Exige que o usuário tenha uma Claim específica para acessar um recurso protegido   
	options.AddPolicy("SuperAdminOnly", policy =>   
        policy.RequireRole("Admin").RequireClaim("id", "bruno"));  
        options.AddPolicy("UserOnly", policy => policy.RequireRole("User"));  
    //RequireAssertion -> Permite definir uma expressão lambda com uma condição customizada para autorização  
    options.AddPolicy("ExclusivePolicyOnly", policy =>   
        policy.RequireAssertion(context =>   
            context.User.HasClaim(claim => claim.Type == "id" && claim.Value == "bruno")   
                                           || context.User.IsInRole("SuperAdmin")));  
});
```

```C#
namespace APICatalogo.Controllers;
Login() {
	var authClaims = new List<Claim>  
	{  
	    new Claim(ClaimTypes.Name, user.UserName!),  
	    new Claim(ClaimTypes.Email, user.Email!),  
	    new Claim("id", user.UserName!), //Adicionado para política "ExclusivePolicyOnly"
	    new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),//Gera um Id para o token  
	};
}

```


#### Utilização
```C#
[HttpPost]  
[Route("revoke/{username}")]  
[Authorize(Policy = "ExclusiveOnly")] //Somente um usuário autenticado e autorizado poderá acessar essa rota  
public async Task<IActionResult> Revoke(string username)  
{  
    var user = await _userManager.FindByNameAsync(username);  
    if(user is null) return BadRequest("Invalid user name");  
        user.RefreshToken = null;  
    await _userManager.UpdateAsync(user);  
    return NoContent();  
}
```



