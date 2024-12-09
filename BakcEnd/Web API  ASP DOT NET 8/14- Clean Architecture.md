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

