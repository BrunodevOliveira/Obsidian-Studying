---
tags: entity
---
## Passo 1: Reverter o Banco de Dados ao Estado Inicial


  Este comando aplica a "migration zero", que efetivamente desfaz todas as migrations aplicadas, rodando o método Down() da sua última migration e apagando as tabelas.
```bash
  dotnet ef database update 0 -p ../Base.Infrastructure
```
 ---

## Passo 2: Remover o Arquivo da Migration Incorreta


  Agora que o banco foi revertido, este comando irá apagar com segurança os arquivos da pasta Migrations.
```bash
  dotnet ef migrations remove -p ../Base.Infrastructure
```
  ---

## Passo 3: Criar a Nova Migration Correta


  Este comando irá criar os novos arquivos de migration, agora com a estrutura correta do relacionamento N-para-N.

  
  ```bash
  dotnet ef migrations add InitialModel -p ../Base.Infrastructure
  ```

---

## Passo 4: Aplicar a Nova Migration ao Banco de Dados


  Este comando final irá ler a nova migration InitialModel e criar todas as tabelas no seu banco de dados com o esquema correto.
```bash
  dotnet ef database update -p ../Base.Infrastructure
```
