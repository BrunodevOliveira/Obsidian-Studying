---
aliases:
  - "Tutorial: Integrando MySQL com Docker em ASP.NET Core 8 + DDD1"
---
# 🏗️ PASSO 1: Estrutura do projeto
📁 MeuProjeto.sln
├── 📁 src/
│   ├── 📁 MeuProjeto.Domain/           # Entidades e regras de negócio
│   ├── 📁 MeuProjeto.Application/      # Casos de uso e serviços
│   ├── 📁 MeuProjeto.Infrastructure/   # EF Core, repositórios
│   ├── 📁 MeuProjeto.API/             # Controllers e configurações
│   └── 📁 MeuProjeto.Communication/    # DTOs e contratos
└── 📁 docker/
    └── 📄 docker-compose.yml

# 🐳 PASSO 2: Configurando o MySQL no Docker
Crie o arquivo `docker/docker-compose.yml`
```YAML
version: '3.8'

services:
  # Serviço do MySQL
  mysql-db:
    image: mysql:8.0
    container_name: meu-projeto-mysql
    restart: always
    environment:
      # Configurações essenciais do MySQL
      MYSQL_ROOT_PASSWORD: Minhasenha123!
      MYSQL_DATABASE: MeuProjetoDB
      MYSQL_USER: meuusuario
      MYSQL_PASSWORD: MinhasenhaUser123!
    ports:
      # Porta externa:interna
      - "3306:3306"
    volumes:
      # Persiste os dados do banco
      - mysql_data:/var/lib/mysql
      # Scripts de inicialização (opcional)
      - ./init-scripts:/docker-entrypoint-initdb.d
    networks:
      - meu-projeto-network
    
  # Opcional: PHPMyAdmin para administração visual
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: meu-projeto-phpmyadmin
    restart: always
    environment:
      PMA_HOST: mysql-db
      PMA_PORT: 3306
      MYSQL_ROOT_PASSWORD: MinhasSenha123!
    ports:
      - "8080:80"
    depends_on:
      - mysql-db
    networks:
      - meu-projeto-network

# Volumes para persistência de dados
volumes:
  mysql_data:
    driver: local

# Rede para comunicação entre containers
networks:
  meu-projeto-network:
    driver: bridge
```

## 2.2 - Comandos Docker
```bash
# Subir os containers
docker-compose up -d

# Verificar se estão rodando
docker ps

# Para parar os containers (quando necessário)
docker-compose down

# Para parar E remover os dados
docker-compose down -v
```

# 📦 PASSO 3: Instalando Pacotes NuGet
## 3.1 - Na camada Infrastructure
```bash
# Entity Framework Core para MySQL
dotnet add package Pomelo.EntityFrameworkCore.MySql

# Ferramentas de migração
dotnet add package Microsoft.EntityFrameworkCore.Tools

# Para injeção de dependência
dotnet add package Microsoft.Extensions.DependencyInjection
```

## 3.2 - Na camada API

```bash
# Para configuração
dotnet add package Microsoft.Extensions.Configuration
dotnet add package Microsoft.Extensions.Configuration.Json
```

# 🏗️ PASSO 4: Implementando a Camada Domain
```cs
// MeuProjeto.Domain/Entities/BaseEntity.cs
using System;

namespace MeuProjeto.Domain.Entities
{
    public abstract class BaseEntity
    {
        public Guid Id { get; protected set; }
        public DateTime CreatedAt { get; protected set; }
        public DateTime UpdatedAt { get; protected set; }
        public bool IsDeleted { get; protected set; }

        protected BaseEntity()
        {
            Id = Guid.NewGuid();
            CreatedAt = DateTime.UtcNow;
            UpdatedAt = DateTime.UtcNow;
            IsDeleted = false;
        }

        public virtual void Update()
        {
            UpdatedAt = DateTime.UtcNow;
        }

        public virtual void Delete()
        {
            IsDeleted = true;
            UpdatedAt = DateTime.UtcNow;
        }
    }
}

// MeuProjeto.Domain/Entities/Usuario.cs
using System.ComponentModel.DataAnnotations;

namespace MeuProjeto.Domain.Entities
{
    public class Usuario : BaseEntity
    {
        public string Nome { get; private set; }
        public string Email { get; private set; }
        public string Telefone { get; private set; }

        // Construtor para Entity Framework
        protected Usuario() : base() { }

        // Construtor para criação via domínio
        public Usuario(string nome, string email, string telefone) : base()
        {
            SetNome(nome);
            SetEmail(email);
            SetTelefone(telefone);
        }

        // Métodos de domínio com validação
        public void SetNome(string nome)
        {
            if (string.IsNullOrWhiteSpace(nome))
                throw new ArgumentException("Nome não pode ser vazio");
            
            if (nome.Length < 2)
                throw new ArgumentException("Nome deve ter pelo menos 2 caracteres");

            Nome = nome.Trim();
            Update();
        }

        public void SetEmail(string email)
        {
            if (string.IsNullOrWhiteSpace(email))
                throw new ArgumentException("Email não pode ser vazio");

            if (!IsValidEmail(email))
                throw new ArgumentException("Email inválido");

            Email = email.Trim().ToLower();
            Update();
        }

        public void SetTelefone(string telefone)
        {
            if (string.IsNullOrWhiteSpace(telefone))
                throw new ArgumentException("Telefone não pode ser vazio");

            Telefone = telefone.Trim();
            Update();
        }

        private static bool IsValidEmail(string email)
        {
            try
            {
                var addr = new System.Net.Mail.MailAddress(email);
                return addr.Address == email;
            }
            catch
            {
                return false;
            }
        }
    }
}

// MeuProjeto.Domain/Interfaces/IUsuarioRepository.cs
using MeuProjeto.Domain.Entities;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace MeuProjeto.Domain.Interfaces
{
    public interface IUsuarioRepository
    {
        Task<Usuario?> GetByIdAsync(Guid id);
        Task<Usuario?> GetByEmailAsync(string email);
        Task<IEnumerable<Usuario>> GetAllAsync();
        Task<IEnumerable<Usuario>> GetByNameContainsAsync(string nome);
        Task AddAsync(Usuario usuario);
        Task UpdateAsync(Usuario usuario);
        Task DeleteAsync(Guid id);
        Task<bool> ExistsAsync(Guid id);
        Task<bool> EmailExistsAsync(string email);
        Task SaveChangesAsync();
    }
}
```

# 🗄️ PASSO 5: Implementando a Camada Infrastructure

## `ApplicationDbContext.cs`

```cs
// MeuProjeto.Infrastructure/Data/ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;
using MeuProjeto.Domain.Entities;
using MeuProjeto.Infrastructure.Data.Configurations;

namespace MeuProjeto.Infrastructure.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
        {
        }

        // DbSets - Tabelas do banco
        public DbSet<Usuario> Usuarios { get; set; } = null!;

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // Aplicar configurações de entidades
            modelBuilder.ApplyConfiguration(new UsuarioConfiguration());

            // Configurações globais
            ConfigureGlobalFilters(modelBuilder);
        }

        private void ConfigureGlobalFilters(ModelBuilder modelBuilder)
        {
            // Filtro global para soft delete
            foreach (var entityType in modelBuilder.Model.GetEntityTypes())
            {
                var isDeletedProperty = entityType.FindProperty("IsDeleted");
                if (isDeletedProperty != null && isDeletedProperty.ClrType == typeof(bool))
                {
                    var parameter = Expression.Parameter(entityType.ClrType, "e");
                    var propertyMethodInfo = typeof(EF).GetMethod(nameof(EF.Property))?.MakeGenericMethod(typeof(bool));
                    var isDeletedProperty2 = Expression.Call(propertyMethodInfo, parameter, Expression.Constant("IsDeleted"));
                    var compareExpression = Expression.MakeBinary(ExpressionType.Equal, isDeletedProperty2, Expression.Constant(false));
                    var lambda = Expression.Lambda(compareExpression, parameter);

                    modelBuilder.Entity(entityType.ClrType).HasQueryFilter(lambda);
                }
            }
        }

        public override Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
        {
            // Atualizar timestamps automaticamente
            foreach (var entry in ChangeTracker.Entries<BaseEntity>())
            {
                switch (entry.State)
                {
                    case EntityState.Added:
                        entry.Entity.GetType().GetProperty("CreatedAt")?.SetValue(entry.Entity, DateTime.UtcNow);
                        entry.Entity.GetType().GetProperty("UpdatedAt")?.SetValue(entry.Entity, DateTime.UtcNow);
                        break;

                    case EntityState.Modified:
                        entry.Entity.GetType().GetProperty("UpdatedAt")?.SetValue(entry.Entity, DateTime.UtcNow);
                        entry.Property("CreatedAt").IsModified = false; // Não alterar data de criação
                        break;
                }
            }

            return base.SaveChangesAsync(cancellationToken);
        }
    }
}

```

## `UsuarioConfiguration`

```cs
// MeuProjeto.Infrastructure/Data/Configurations/UsuarioConfiguration.cs
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using MeuProjeto.Domain.Entities;

namespace MeuProjeto.Infrastructure.Data.Configurations
{
    public class UsuarioConfiguration : IEntityTypeConfiguration<Usuario>
    {
        public void Configure(EntityTypeBuilder<Usuario> builder)
        {
            // Nome da tabela
            builder.ToTable("Usuarios");

            // Chave primária
            builder.HasKey(u => u.Id);
            builder.Property(u => u.Id)
                .HasColumnType("CHAR(36)")
                .IsRequired();

            // Propriedades
            builder.Property(u => u.Nome)
                .HasColumnType("VARCHAR(100)")
                .HasMaxLength(100)
                .IsRequired();

            builder.Property(u => u.Email)
                .HasColumnType("VARCHAR(150)")
                .HasMaxLength(150)
                .IsRequired();

            builder.Property(u => u.Telefone)
                .HasColumnType("VARCHAR(20)")
                .HasMaxLength(20)
                .IsRequired();

            // Campos de auditoria
            builder.Property(u => u.CreatedAt)
                .HasColumnType("DATETIME")
                .IsRequired();

            builder.Property(u => u.UpdatedAt)
                .HasColumnType("DATETIME")
                .IsRequired();

            builder.Property(u => u.IsDeleted)
                .HasColumnType("BOOLEAN")
                .HasDefaultValue(false)
                .IsRequired();

            // Índices
            builder.HasIndex(u => u.Email)
                .IsUnique()
                .HasDatabaseName("IX_Usuarios_Email");

            builder.HasIndex(u => u.Nome)
                .HasDatabaseName("IX_Usuarios_Nome");

            builder.HasIndex(u => u.IsDeleted)
                .HasDatabaseName("IX_Usuarios_IsDeleted");
        }
    }
}
```

## UsuarioRepository
```cs
// MeuProjeto.Infrastructure/Repositories/UsuarioRepository.cs
using Microsoft.EntityFrameworkCore;
using MeuProjeto.Domain.Entities;
using MeuProjeto.Domain.Interfaces;
using MeuProjeto.Infrastructure.Data;

namespace MeuProjeto.Infrastructure.Repositories
{
    public class UsuarioRepository : IUsuarioRepository
    {
        private readonly ApplicationDbContext _context;

        public UsuarioRepository(ApplicationDbContext context)
        {
            _context = context ?? throw new ArgumentNullException(nameof(context));
        }

        public async Task<Usuario?> GetByIdAsync(Guid id)
        {
            return await _context.Usuarios
                .FirstOrDefaultAsync(u => u.Id == id);
        }

        public async Task<Usuario?> GetByEmailAsync(string email)
        {
            if (string.IsNullOrWhiteSpace(email))
                return null;

            return await _context.Usuarios
                .FirstOrDefaultAsync(u => u.Email.ToLower() == email.ToLower().Trim());
        }

        public async Task<IEnumerable<Usuario>> GetAllAsync()
        {
            return await _context.Usuarios
                .OrderBy(u => u.Nome)
                .ToListAsync();
        }

        public async Task<IEnumerable<Usuario>> GetByNameContainsAsync(string nome)
        {
            if (string.IsNullOrWhiteSpace(nome))
                return new List<Usuario>();

            return await _context.Usuarios
                .Where(u => u.Nome.Contains(nome))
                .OrderBy(u => u.Nome)
                .ToListAsync();
        }

        public async Task AddAsync(Usuario usuario)
        {
            if (usuario == null)
                throw new ArgumentNullException(nameof(usuario));

            await _context.Usuarios.AddAsync(usuario);
        }

        public async Task UpdateAsync(Usuario usuario)
        {
            if (usuario == null)
                throw new ArgumentNullException(nameof(usuario));

            _context.Usuarios.Update(usuario);
            await Task.CompletedTask;
        }

        public async Task DeleteAsync(Guid id)
        {
            var usuario = await GetByIdAsync(id);
            if (usuario != null)
            {
                usuario.Delete(); // Soft delete
                await UpdateAsync(usuario);
            }
        }

        public async Task<bool> ExistsAsync(Guid id)
        {
            return await _context.Usuarios
                .AnyAsync(u => u.Id == id);
        }

        public async Task<bool> EmailExistsAsync(string email)
        {
            if (string.IsNullOrWhiteSpace(email))
                return false;

            return await _context.Usuarios
                .AnyAsync(u => u.Email.ToLower() == email.ToLower().Trim());
        }

        public async Task SaveChangesAsync()
        {
            await _context.SaveChangesAsync();
        }
    }
}
```

# 📱 PASSO 6: Implementando a Camada Communication

## `CriarUsuarioRequest`
```cs
// MeuProjeto.Communication/Requests/Usuario/CriarUsuarioRequest.cs
using System.ComponentModel.DataAnnotations;

namespace MeuProjeto.Communication.Requests.Usuario
{
    public class CriarUsuarioRequest
    {
        [Required(ErrorMessage = "Nome é obrigatório")]
        [StringLength(100, MinimumLength = 2, ErrorMessage = "Nome deve ter entre 2 e 100 caracteres")]
        public string Nome { get; set; } = string.Empty;

        [Required(ErrorMessage = "Email é obrigatório")]
        [EmailAddress(ErrorMessage = "Email deve ter um formato válido")]
        [StringLength(150, ErrorMessage = "Email deve ter no máximo 150 caracteres")]
        public string Email { get; set; } = string.Empty;

        [Required(ErrorMessage = "Telefone é obrigatório")]
        [StringLength(20, ErrorMessage = "Telefone deve ter no máximo 20 caracteres")]
        [RegularExpression(@"^\(\d{2}\)\s\d{4,5}-\d{4}$", ErrorMessage = "Telefone deve estar no formato (xx) xxxxx-xxxx")]
        public string Telefone { get; set; } = string.Empty;
    }
}
```

## AtualizaUsuarioRequest
```cs

// MeuProjeto.Communication/Requests/Usuario/AtualizarUsuarioRequest.cs
using System.ComponentModel.DataAnnotations;

namespace MeuProjeto.Communication.Requests.Usuario
{
    public class AtualizarUsuarioRequest
    {
        [Required(ErrorMessage = "Nome é obrigatório")]
        [StringLength(100, MinimumLength = 2, ErrorMessage = "Nome deve ter entre 2 e 100 caracteres")]
        public string Nome { get; set; } = string.Empty;

        [Required(ErrorMessage = "Email é obrigatório")]
        [EmailAddress(ErrorMessage = "Email deve ter um formato válido")]
        [StringLength(150, ErrorMessage = "Email deve ter no máximo 150 caracteres")]
        public string Email { get; set; } = string.Empty;

        [Required(ErrorMessage = "Telefone é obrigatório")]
        [StringLength(20, ErrorMessage = "Telefone deve ter no máximo 20 caracteres")]
        public string Telefone { get; set; } = string.Empty;
    }
}
```


## BaseResponse
```cs
// MeuProjeto.Communication/Responses/BaseResponse.cs
using System.Text.Json.Serialization;

namespace MeuProjeto.Communication.Responses
{
    public class BaseResponse<T>
    {
        public bool Success { get; set; }
        public string Message { get; set; } = string.Empty;
        public T? Data { get; set; }
        public List<string> Errors { get; set; } = new();

        [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]
        public object? Metadata { get; set; }

        public BaseResponse()
        {
            Success = true;
        }

        public BaseResponse(T data, string message = "")
        {
            Success = true;
            Data = data;
            Message = message;
        }

        public BaseResponse(string error)
        {
            Success = false;
            Message = error;
            Errors.Add(error);
        }

        public BaseResponse(List<string> errors)
        {
            Success = false;
            Errors = errors;
            Message = string.Join("; ", errors);
        }

        public static BaseResponse<T> SuccessResult(T data, string message = "")
        {
            return new BaseResponse<T>(data, message);
        }

        public static BaseResponse<T> ErrorResult(string error)
        {
            return new BaseResponse<T>(error);
        }

        public static BaseResponse<T> ErrorResult(List<string> errors)
        {
            return new BaseResponse<T>(errors);
        }
    }

    // Para respostas sem dados específicos
    public class BaseResponse : BaseResponse<object>
    {
        public BaseResponse() : base() { }
        public BaseResponse(string message) : base()
        {
            Message = message;
        }
        
        public static BaseResponse Success(string message = "")
        {
            return new BaseResponse(message);
        }
        
        public static BaseResponse Error(string error)
        {
            return new BaseResponse { Success = false, Message = error, Errors = { error } };
        }
    }
}
```

## UsuarioResponse
```cs
// MeuProjeto.Communication/Responses/Usuario/UsuarioResponse.cs
namespace MeuProjeto.Communication.Responses.Usuario
{
    public class UsuarioResponse
    {
        public Guid Id { get; set; }
        public string Nome { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public string Telefone { get; set; } = string.Empty;
        public DateTime CreatedAt { get; set; }
        public DateTime UpdatedAt { get; set; }
    }
}
```


# 🚀 PASSO 7: Implementando a Camada Application

## `IUsuarioService`

```cs
// MeuProjeto.Application/Interfaces/IUsuarioService.cs
using MeuProjeto.Communication.Requests.Usuario;
using MeuProjeto.Communication.Responses;
using MeuProjeto.Communication.Responses.Usuario;

namespace MeuProjeto.Application.Interfaces
{
    public interface IUsuarioService
    {
        Task<BaseResponse<UsuarioResponse>> CriarUsuarioAsync(CriarUsuarioRequest request);
        Task<BaseResponse<UsuarioResponse>> ObterUsuarioPorIdAsync(Guid id);
        Task<BaseResponse<UsuarioResponse>> ObterUsuarioPorEmailAsync(string email);
        Task<BaseResponse<List<UsuarioResponse>>> ObterTodosUsuariosAsync();
        Task<BaseResponse<List<UsuarioResponse>>> BuscarUsuariosPorNomeAsync(string nome);
        Task<BaseResponse<UsuarioResponse>> AtualizarUsuarioAsync(Guid id, AtualizarUsuarioRequest request);
        Task<BaseResponse> DeletarUsuarioAsync(Guid id);
    }
}
```

## `UsuarioService`

```cs
// MeuProjeto.Application/Services/UsuarioService.cs
using MeuProjeto.Application.Interfaces;
using MeuProjeto.Communication.Requests.Usuario;
using MeuProjeto.Communication.Responses;
using MeuProjeto.Communication.Responses.Usuario;
using MeuProjeto.Domain.Entities;
using MeuProjeto.Domain.Interfaces;
using Microsoft.Extensions.Logging;

namespace MeuProjeto.Application.Services
{
    public class UsuarioService : IUsuarioService
    {
        private readonly IUsuarioRepository _usuarioRepository;
        private readonly ILogger<UsuarioService> _logger;

        public UsuarioService(
            IUsuarioRepository usuarioRepository,
            ILogger<UsuarioService> logger)
        {
            _usuarioRepository = usuarioRepository ?? throw new ArgumentNullException(nameof(usuarioRepository));
            _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        }

        public async Task<BaseResponse<UsuarioResponse>> CriarUsuarioAsync(CriarUsuarioRequest request)
        {
            try
            {
                _logger.LogInformation("Iniciando criação de usuário com email: {Email}", request.Email);

                // Validar se email já existe
                if (await _usuarioRepository.EmailExistsAsync(request.Email))
                {
                    _logger.LogWarning("Tentativa de criar usuário com email já existente: {Email}", request.Email);
                    return BaseResponse<UsuarioResponse>.ErrorResult("Email já está em uso por outro usuário");
                }

                // Criar entidade de domínio
                var usuario = new Usuario(request.Nome, request.Email, request.Telefone);

                // Salvar no repositório
                await _usuarioRepository.AddAsync(usuario);
                await _usuarioRepository.SaveChangesAsync();

                _logger.LogInformation("Usuário criado com sucesso. ID: {UsuarioId}", usuario.Id);

                // Retornar response
                var response = new UsuarioResponse
                {
                    Id = usuario.Id,
                    Nome = usuario.Nome,
                    Email = usuario.Email,
                    Telefone = usuario.Telefone,
                    CreatedAt = usuario.CreatedAt,
                    UpdatedAt = usuario.UpdatedAt
                };

                return BaseResponse<UsuarioResponse>.SuccessResult(response, "Usuário criado com sucesso");
            }
            catch (ArgumentException ex)
            {
                _logger.LogWarning(ex, "Erro de validação ao criar usuário");
                return BaseResponse<UsuarioResponse>.ErrorResult(ex.Message);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Erro inesperado ao criar usuário");
                return BaseResponse<UsuarioResponse>.ErrorResult("Erro interno do servidor");
            }
        }

        public async Task<BaseResponse<UsuarioResponse>> ObterUsuarioPorIdAsync(Guid id)
        {
            try
            {
                _logger.LogInformation("Buscando usuário por ID: {UsuarioId}", id);

                if (id == Guid.Empty)
                {
                    return BaseResponse<UsuarioResponse>.ErrorResult("ID do usuário é obrigatório");
                }

                var usuario = await _usuarioRepository.GetByIdAsync(id);
                
                if (usuario == null)
                {
                    _logger.LogWarning("Usuário não encontrado. ID: {UsuarioId}", id);
                    return BaseResponse<UsuarioResponse>.ErrorResult("Usuário não encontrado");
                }

                var response = new UsuarioResponse
                {
                    Id = usuario.Id,
                    Nome = usuario.Nome,
                    Email = usuario.Email,
                    Telefone = usuario.Telefone,
                    CreatedAt = usuario.CreatedAt,
                    UpdatedAt = usuario.UpdatedAt
                };

                return BaseResponse<UsuarioResponse>.SuccessResult(response);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Erro ao buscar usuário por ID: {UsuarioId}", id);
                return BaseResponse<UsuarioResponse>.ErrorResult("Erro interno do servidor");
            }
        }

        public async Task<BaseResponse<UsuarioResponse>> ObterUsuarioPorEmailAsync(string email)
        {
            try
            {
                if (string.IsNullOrWhiteSpace(email))
                {
                    return BaseResponse<UsuarioResponse>.ErrorResult("Email é obrigatório");
                }

                var usuario = await _usuarioRepository.GetByEmailAsync(email);
                
                if (usuario == null)
                {
                    return BaseResponse<UsuarioResponse>.ErrorResult("Usuário não encontrado");
                }

                var response = new UsuarioResponse
                {
                    Id = usuario.Id,
                    Nome = usuario.Nome,
                    Email = usuario.Email,
                    Telefone = usuario.Telefone,
                    CreatedAt = usuario.CreatedAt,
                    UpdatedAt = usuario.UpdatedAt
                };

                return BaseResponse<UsuarioResponse>.SuccessResult(response);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Erro ao buscar usuário por email: {Email}", email);
                return BaseResponse<UsuarioResponse>.ErrorResult("Erro interno do servidor");
            }
        }

        public async Task<BaseResponse<List<UsuarioResponse>>> ObterTodosUsuariosAsync()
        {
            try
            {
                _logger.LogInformation("Buscando todos os usuários");

                var usuarios = await _usuarioRepository.GetAllAsync();
                
                var response = usuarios.Select(u => new UsuarioResponse
                {
                    Id = u.Id,
                    Nome = u.Nome,
                    Email = u.Email,
                    Telefone = u.Telefone,
                    CreatedAt = u.CreatedAt,
                    UpdatedAt = u.UpdatedAt
                }).ToList();

                return BaseResponse<List<UsuarioResponse>>.SuccessResult(response);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Erro ao buscar usuários por nome: {Nome}", nome);
                return BaseResponse<List<UsuarioResponse>>.ErrorResult("Erro interno do servidor");
            }
        }

        public async Task<BaseResponse<UsuarioResponse>> AtualizarUsuarioAsync(Guid id, AtualizarUsuarioRequest request)
        {
            try
            {
                _logger.LogInformation("Atualizando usuário. ID: {UsuarioId}", id);

                if (id == Guid.Empty)
                {
                    return BaseResponse<UsuarioResponse>.ErrorResult("ID do usuário é obrigatório");
                }

                var usuario = await _usuarioRepository.GetByIdAsync(id);
                if (usuario == null)
                {
                    _logger.LogWarning("Usuário não encontrado para atualização. ID: {UsuarioId}", id);
                    return BaseResponse<UsuarioResponse>.ErrorResult("Usuário não encontrado");
                }

                // Verificar se o email já está em uso por outro usuário
                var usuarioComMesmoEmail = await _usuarioRepository.GetByEmailAsync(request.Email);
                if (usuarioComMesmoEmail != null && usuarioComMesmoEmail.Id != id)
                {
                    _logger.LogWarning("Tentativa de atualizar usuário com email já existente: {Email}", request.Email);
                    return BaseResponse<UsuarioResponse>.ErrorResult("Email já está em uso por outro usuário");
                }

                // Atualizar dados do usuário
                usuario.SetNome(request.Nome);
                usuario.SetEmail(request.Email);
                usuario.SetTelefone(request.Telefone);

                await _usuarioRepository.UpdateAsync(usuario);
                await _usuarioRepository.SaveChangesAsync();

                _logger.LogInformation("Usuário atualizado com sucesso. ID: {UsuarioId}", id);

                var response = new UsuarioResponse
                {
                    Id = usuario.Id,
                    Nome = usuario.Nome,
                    Email = usuario.Email,
                    Telefone = usuario.Telefone,
                    CreatedAt = usuario.CreatedAt,
                    UpdatedAt = usuario.UpdatedAt
                };

                return BaseResponse<UsuarioResponse>.SuccessResult(response, "Usuário atualizado com sucesso");
            }
            catch (ArgumentException ex)
            {
                _logger.LogWarning(ex, "Erro de validação ao atualizar usuário. ID: {UsuarioId}", id);
                return BaseResponse<UsuarioResponse>.ErrorResult(ex.Message);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Erro inesperado ao atualizar usuário. ID: {UsuarioId}", id);
                return BaseResponse<UsuarioResponse>.ErrorResult("Erro interno do servidor");
            }
        }

        public async Task<BaseResponse> DeletarUsuarioAsync(Guid id)
        {
            try
            {
                _logger.LogInformation("Deletando usuário. ID: {UsuarioId}", id);

                if (id == Guid.Empty)
                {
                    return BaseResponse.Error("ID do usuário é obrigatório");
                }

                if (!await _usuarioRepository.ExistsAsync(id))
                {
                    _logger.LogWarning("Usuário não encontrado para deleção. ID: {UsuarioId}", id);
                    return BaseResponse.Error("Usuário não encontrado");
                }

                await _usuarioRepository.DeleteAsync(id);
                await _usuarioRepository.SaveChangesAsync();

                _logger.LogInformation("Usuário deletado com sucesso. ID: {UsuarioId}", id);

                return BaseResponse.Success("Usuário deletado com sucesso");
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Erro inesperado ao deletar usuário. ID: {UsuarioId}", id);
                return BaseResponse.Error("Erro interno do servidor");
            }
        }
    }
}dAt
                }).ToList();

                return BaseResponse<List<UsuarioResponse>>.SuccessResult(response);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Erro ao buscar todos os usuários");
                return BaseResponse<List<UsuarioResponse>>.ErrorResult("Erro interno do servidor");
            }
        }

        public async Task<BaseResponse<List<UsuarioResponse>>> BuscarUsuariosPorNomeAsync(string nome)
        {
            try
            {
                if (string.IsNullOrWhiteSpace(nome))
                {
                    return BaseResponse<List<UsuarioResponse>>.ErrorResult("Nome para busca é obrigatório");
                }

                var usuarios = await _usuarioRepository.GetByNameContainsAsync(nome);
                
                var response = usuarios.Select(u => new UsuarioResponse
                {
                    Id = u.Id,
                    Nome = u.Nome,
                    Email = u.Email,
                    Telefone = u.Telefone,
                    CreatedAt = u.CreatedAt,
                    UpdatedAt = u.Update
```


# 🌐 PASSO 8: Implementando a Camada API

## appsettings.json`

```json
// MeuProjeto.API/appsettings.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Port=3306;Database=MeuProjetoDB;Uid=meuusuario;Pwd=MinhasenhaUser123!;SslMode=none;AllowUserVariables=true;UseAffectedRows=false;"
  },
  "AllowedHosts": "*"
}
```

## Development
```json
// MeuProjeto.API/appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Port=3306;Database=MeuProjetoDB;Uid=meuusuario;Pwd=MinhasenhaUser123!;SslMode=none;AllowUserVariables=true;UseAffectedRows=false;"
  }
}
```

## Program.cs
```cs
// MeuProjeto.API/Program.cs
using Microsoft.EntityFrameworkCore;
using MeuProjeto.Application.Interfaces;
using MeuProjeto.Application.Services;
using MeuProjeto.Domain.Interfaces;
using MeuProjeto.Infrastructure.Data;
using MeuProjeto.Infrastructure.Repositories;
using System.Text.Json.Serialization;

var builder = WebApplication.CreateBuilder(args);

// Configuração de serviços
ConfigureServices(builder.Services, builder.Configuration);

var app = builder.Build();

// Configuração do pipeline de requisições
ConfigurePipeline(app);

// Executar migrações automaticamente em desenvolvimento
if (app.Environment.IsDevelopment())
{
    using var scope = app.Services.CreateScope();
    var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    await context.Database.MigrateAsync();
}

app.Run();

static void ConfigureServices(IServiceCollection services, IConfiguration configuration)
{
    // Entity Framework e MySQL
    services.AddDbContext<ApplicationDbContext>(options =>
    {
        options.UseMySql(
            configuration.GetConnectionString("DefaultConnection"),
            ServerVersion.AutoDetect(configuration.GetConnectionString("DefaultConnection")),
            mySqlOptions =>
            {
                mySqlOptions.EnableRetryOnFailure(
                    maxRetryCount: 3,
                    maxRetryDelay: TimeSpan.FromSeconds(5),
                    errorNumbersToAdd: null);
            });
    });

    // Injeção de dependência - Repositories
    services.AddScoped<IUsuarioRepository, UsuarioRepository>();

    // Injeção de dependência - Services
    services.AddScoped<IUsuarioService, UsuarioService>();

    // Controllers
    services.AddControllers()
        .AddJsonOptions(options =>
        {
            options.JsonSerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;
            options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
            options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
        });

    // Swagger/OpenAPI
    services.AddEndpointsApiExplorer();
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new() { 
            Title = "Meu Projeto API", 
            Version = "v1",
            Description = "API para gerenciamento de usuários com arquitetura DDD"
        });
    });

    // CORS
    services.AddCors(options =>
    {
        options.AddPolicy("AllowAll", policy =>
        {
            policy.AllowAnyOrigin()
                  .AllowAnyMethod()
                  .AllowAnyHeader();
        });
    });

    // Health Checks
    services.AddHealthChecks()
        .AddDbContextCheck<ApplicationDbContext>();

    // Logging
    services.AddLogging(builder =>
    {
        builder.AddConsole();
        builder.AddDebug();
    });
}

static void ConfigurePipeline(WebApplication app)
{
    // Desenvolvimento
    if (app.Environment.IsDevelopment())
    {
        app.UseSwagger();
        app.UseSwaggerUI(c =>
        {
            c.SwaggerEndpoint("/swagger/v1/swagger.json", "Meu Projeto API v1");
            c.RoutePrefix = string.Empty; // Swagger na raiz
        });
    }

    // Middleware de tratamento de exceções
    app.UseExceptionHandler("/error");

    // CORS
    app.UseCors("AllowAll");

    // Roteamento
    app.UseRouting();

    // Autorização (quando implementar)
    // app.UseAuthentication();
    // app.UseAuthorization();

    // Controllers
    app.MapControllers();

    // Health Check
    app.MapHealthChecks("/health");

    // Endpoint de erro global
    app.Map("/error", () => Results.Problem("Ocorreu um erro interno no servidor"));
}
```

## UsuariosController.cs

```cs
// MeuProjeto.API/Controllers/UsuariosController.cs
using Microsoft.AspNetCore.Mvc;
using MeuProjeto.Application.Interfaces;
using MeuProjeto.Communication.Requests.Usuario;

namespace MeuProjeto.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    [Produces("application/json")]
    public class UsuariosController : ControllerBase
    {
        private readonly IUsuarioService _usuarioService;
        private readonly ILogger<UsuariosController> _logger;

        public UsuariosController(
            IUsuarioService usuarioService,
            ILogger<UsuariosController> logger)
        {
            _usuarioService = usuarioService ?? throw new ArgumentNullException(nameof(usuarioService));
            _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        }

        /// <summary>
        /// Criar um novo usuário
        /// </summary>
        /// <param name="request">Dados do usuário a ser criado</param>
        /// <returns>Usuário criado</returns>
        [HttpPost]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status201Created)]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status400BadRequest)]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status500InternalServerError)]
        public async Task<IActionResult> CriarUsuario([FromBody] CriarUsuarioRequest request)
        {
            if (!ModelState.IsValid)
            {
                var errors = ModelState
                    .SelectMany(x => x.Value?.Errors ?? new List<ModelError>())
                    .Select(x => x.ErrorMessage)
                    .ToList();

                return BadRequest(BaseResponse<object>.ErrorResult(errors));
            }

            var result = await _usuarioService.CriarUsuarioAsync(request);

            if (result.Success)
            {
                return CreatedAtAction(
                    nameof(ObterUsuarioPorId),
                    new { id = result.Data?.Id },
                    result);
            }

            return BadRequest(result);
        }

        /// <summary>
        /// Obter usuário por ID
        /// </summary>
        /// <param name="id">ID do usuário</param>
        /// <returns>Dados do usuário</returns>
        [HttpGet("{id:guid}")]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status404NotFound)]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status500InternalServerError)]
        public async Task<IActionResult> ObterUsuarioPorId(Guid id)
        {
            var result = await _usuarioService.ObterUsuarioPorIdAsync(id);

            if (result.Success)
            {
                return Ok(result);
            }

            return NotFound(result);
        }

        /// <summary>
        /// Obter usuário por email
        /// </summary>
        /// <param name="email">Email do usuário</param>
        /// <returns>Dados do usuário</returns>
        [HttpGet("por-email")]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status404NotFound)]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status500InternalServerError)]
        public async Task<IActionResult> ObterUsuarioPorEmail([FromQuery] string email)
        {
            if (string.IsNullOrWhiteSpace(email))
            {
                return BadRequest(BaseResponse<object>.ErrorResult("Email é obrigatório"));
            }

            var result = await _usuarioService.ObterUsuarioPorEmailAsync(email);

            if (result.Success)
            {
                return Ok(result);
            }

            return NotFound(result);
        }

        /// <summary>
        /// Obter todos os usuários
        /// </summary>
        /// <returns>Lista de usuários</returns>
        [HttpGet]
        [ProducesResponseType(typeof(BaseResponse<List<UsuarioResponse>>), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(BaseResponse<List<UsuarioResponse>>), StatusCodes.Status500InternalServerError)]
        public async Task<IActionResult> ObterTodosUsuarios()
        {
            var result = await _usuarioService.ObterTodosUsuariosAsync();
            return Ok(result);
        }

        /// <summary>
        /// Buscar usuários por nome
        /// </summary>
        /// <param name="nome">Nome ou parte do nome para buscar</param>
        /// <returns>Lista de usuários encontrados</returns>
        [HttpGet("buscar")]
        [ProducesResponseType(typeof(BaseResponse<List<UsuarioResponse>>), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(BaseResponse<List<UsuarioResponse>>), StatusCodes.Status400BadRequest)]
        [ProducesResponseType(typeof(BaseResponse<List<UsuarioResponse>>), StatusCodes.Status500InternalServerError)]
        public async Task<IActionResult> BuscarUsuariosPorNome([FromQuery] string nome)
        {
            if (string.IsNullOrWhiteSpace(nome))
            {
                return BadRequest(BaseResponse<object>.ErrorResult("Nome para busca é obrigatório"));
            }

            var result = await _usuarioService.BuscarUsuariosPorNomeAsync(nome);
            return Ok(result);
        }

        /// <summary>
        /// Atualizar dados do usuário
        /// </summary>
        /// <param name="id">ID do usuário</param>
        /// <param name="request">Novos dados do usuário</param>
        /// <returns>Usuário atualizado</returns>
        [HttpPut("{id:guid}")]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status400BadRequest)]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status404NotFound)]
        [ProducesResponseType(typeof(BaseResponse<UsuarioResponse>), StatusCodes.Status500InternalServerError)]
        public async Task<IActionResult> AtualizarUsuario(Guid id, [FromBody] AtualizarUsuarioRequest request)
        {
            if (!ModelState.IsValid)
            {
                var errors = ModelState
                    .SelectMany(x => x.Value?.Errors ?? new List<ModelError>())
                    .Select(x => x.ErrorMessage)
                    .ToList();

                return BadRequest(BaseResponse<object>.ErrorResult(errors));
            }

            var result = await _usuarioService.AtualizarUsuarioAsync(id, request);

            if (result.Success)
            {
                return Ok(result);
            }

            return result.Message.Contains("não encontrado") ? NotFound(result) : BadRequest(result);
        }

        /// <summary>
        /// Deletar usuário
        /// </summary>
        /// <param name="id">ID do usuário</param>
        /// <returns>Confirmação da deleção</returns>
        [HttpDelete("{id:guid}")]
        [ProducesResponseType(typeof(BaseResponse), StatusCodes.Status200OK)]
        [ProducesResponseType(typeof(BaseResponse), StatusCodes.Status404NotFound)]
        [ProducesResponseType(typeof(BaseResponse), StatusCodes.Status500InternalServerError)]
        public async Task<IActionResult> DeletarUsuario(Guid id)
        {
            var result = await _usuarioService.DeletarUsuarioAsync(id);

            if (result.Success)
            {
                return Ok(result);
            }

            return result.Message.Contains("não encontrado") ? NotFound(result) : BadRequest(result);
        }
    }
}
```


# 📊 PASSO 9: Criando e Executando Migrações
Agora vamos gerar e executar as migrações do Entity Framework:


``` shell
# ===============================
# COMANDOS PARA MIGRAÇÕES
# ===============================

# 1. Primeiro, certifique-se de que o Docker está rodando
cd docker/
docker-compose up -d

# Verificar se MySQL está rodando
docker ps

# 2. Instalar ferramenta global do EF Core (se não tiver)
dotnet tool install --global dotnet-ef
# ou atualizar se já tiver
dotnet tool update --global dotnet-ef

# 3. Navegar para a pasta do projeto API
cd ../src/MeuProjeto.API/

# 4. Adicionar primeira migração (criar tabelas)
dotnet ef migrations add InitialCreate --project ../MeuProjeto.Infrastructure --startup-project .

# 5. Aplicar migrações no banco
dotnet ef database update --project ../MeuProjeto.Infrastructure --startup-project .

# 6. (Opcional) Ver o SQL que será executado
dotnet ef migrations script --project ../MeuProjeto.Infrastructure --startup-project .

# ===============================
# COMANDOS ÚTEIS DE MANUTENÇÃO
# ===============================

# Remover última migração (se houver erro)
dotnet ef migrations remove --project ../MeuProjeto.Infrastructure --startup-project .

# Ver lista de migrações
dotnet ef migrations list --project ../MeuProjeto.Infrastructure --startup-project .

# Reverter banco para migração específica
dotnet ef database update NomeDaMigracao --project ../MeuProjeto.Infrastructure --startup-project .

# Recriar banco do zero (CUIDADO: apaga todos os dados!)
dotnet ef database drop --project ../MeuProjeto.Infrastructure --startup-project .
dotnet ef database update --project ../MeuProjeto.Infrastructure --startup-project .

# ===============================
# EXECUTAR A API
# ===============================

# Executar em modo desenvolvimento
dotnet run --project src/MeuProjeto.API/

# Executar com hot reload
dotnet watch run --project src/MeuProjeto.API/

# ===============================
# COMANDOS DOCKER ÚTEIS
# ===============================

# Ver logs do MySQL
docker logs meu-projeto-mysql

# Entrar no container do MySQL
docker exec -it meu-projeto-mysql mysql -u meuusuario -p

# Parar containers
docker-compose down

# Parar e remover volumes (apaga dados!)
docker-compose down -v

# Recriar containers
docker-compose up -d --force-recreate
```

# 🧪 PASSO 10: Scripts de Teste da API

```shell
# ===============================
# SCRIPTS DE TESTE DA API
# ===============================

# Base URL da API (ajuste se necessário)
BASE_URL="https://localhost:7000/api"

# ===============================
# 1. CRIAR USUÁRIO
# ===============================
curl -X POST "$BASE_URL/usuarios" \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "João Silva",
    "email": "joao.silva@email.com",
    "telefone": "(11) 99999-1234"
  }'

# ===============================
# 2. CRIAR OUTRO USUÁRIO
# ===============================
curl -X POST "$BASE_URL/usuarios" \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "Maria Santos",
    "email": "maria.santos@email.com",
    "telefone": "(11) 88888-5678"
  }'

# ===============================
# 3. LISTAR TODOS USUÁRIOS
# ===============================
curl -X GET "$BASE_URL/usuarios"

# ===============================
# 4. BUSCAR USUÁRIO POR ID
# ===============================
# Substitua GUID_DO_USUARIO pelo ID retornado na criação
curl -X GET "$BASE_URL/usuarios/GUID_DO_USUARIO"

# ===============================
# 5. BUSCAR USUÁRIO POR EMAIL
# ===============================
curl -X GET "$BASE_URL/usuarios/por-email?email=joao.silva@email.com"

# ===============================
# 6. BUSCAR USUÁRIOS POR NOME
# ===============================
curl -X GET "$BASE_URL/usuarios/buscar?nome=João"

# ===============================
# 7. ATUALIZAR USUÁRIO
# ===============================
# Substitua GUID_DO_USUARIO pelo ID do usuário
curl -X PUT "$BASE_URL/usuarios/GUID_DO_USUARIO" \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "João Silva Santos",
    "email": "joao.silva.santos@email.com",
    "telefone": "(11) 99999-9999"
  }'

# ===============================
# 8. DELETAR USUÁRIO
# ===============================
# Substitua GUID_DO_USUARIO pelo ID do usuário
curl -X DELETE "$BASE_URL/usuarios/GUID_DO_USUARIO"

# ===============================
# 9. VERIFICAR HEALTH CHECK
# ===============================
curl -X GET "$BASE_URL/../health"

# ===============================
# TESTES DE VALIDAÇÃO (devem retornar erro)
# ===============================

# Teste 1: Criar usuário sem nome
curl -X POST "$BASE_URL/usuarios" \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "",
    "email": "teste@email.com",
    "telefone": "(11) 99999-1234"
  }'

# Teste 2: Criar usuário com email inválido
curl -X POST "$BASE_URL/usuarios" \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "Teste",
    "email": "email-invalido",
    "telefone": "(11) 99999-1234"
  }'

# Teste 3: Criar usuário com email duplicado
curl -X POST "$BASE_URL/usuarios" \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "Outro João",
    "email": "joao.silva@email.com",
    "telefone": "(11) 77777-7777"
  }'

# Teste 4: Buscar usuário inexistente
curl -X GET "$BASE_URL/usuarios/00000000-0000-0000-0000-000000000000"
```

# Melhorias
> [!NOTE]
    > https://claude.ai/share/e5f65754-4887-413c-960f-80d8f088e05e
    > 
