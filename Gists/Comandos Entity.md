
 <center><h1>Create</h1></center>
 
| Metodo                             | Objetivo                               | Exemplo de uso                                                            |     |
| ---------------------------------- | -------------------------------------- | ------------------------------------------------------------------------- | --- |
| `Add` / `AddAsync`                 | Inserir uma nova entidade no contexto  | `await _context.AddAsync(entidade); await _context.SaveChangesAsync();`   |     |
| `AddRange` / `AddRangeAsync`       | Inserir múltiplas entidades de uma vez | `await _context.AddRangeAsync(lista); await _context.SaveChangesAsync();` |     |
| `SaveChanges` / `SaveChangesAsync` | Persistir inserções no banco           | `await _context.SaveChangesAsync();`                                      |     |
|                                    |                                        |                                                                           |     |


---

 <center><h1>Read</h1></center>

| Metodo                                     | Objetivo                                                           | Exemplo de uso                                                                                               |
| ------------------------------------------ | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| `Find` / `FindAsync`                       | Recuperar entidade por chave primária (usa cache do contexto)      | `var e = await _context.Produtos.FindAsync(id);`                                                             |
| `FirstOrDefault` / `FirstOrDefaultAsync`   | Retornar o primeiro elemento que satisfaça a condição              | `var p = await _context.Produtos.FirstOrDefaultAsync(p => p.Nome == nome);`                                  |
| `SingleOrDefault` / `SingleOrDefaultAsync` | Retornar exatamente um elemento (ou null) — garante unicidade      | `var u = await _context.Usuarios.SingleOrDefaultAsync(u => u.Email == email);`                               |
| `Where`                                    | Filtrar coleções (IQueryable)                                      | `var ativos = _context.Clientes.Where(c => c.Ativo);`                                                        |
| `ToList` / `ToListAsync`                   | Executar a consulta e materializar lista                           | `var lista = await query.ToListAsync();`                                                                     |
| `Include` / `ThenInclude`                  | Eager loading de relacionamentos                                   | `var p = await _context.Pedidos.Include(p => p.Itens).ThenInclude(i => i.Produto).FirstOrDefaultAsync(...);` |
| `AsNoTracking`                             | Consulta somente leitura sem change tracking (melhora performance) | `var read = await _context.Produtos.AsNoTracking().ToListAsync();`                                           |
| `AnyAsync` / `Count`                       | Verificar existência ou contagem de registros                      | `var existe = await _context.Usuarios.AnyAsync(u => u.Ativo);`                                               |
| `OrderBy` / `Skip` / `Take`                | Ordenação e paginação                                              | `var page = await query.OrderBy(x => x.Id).Skip(10).Take(10).ToListAsync();`                                 |


---

 <center><h1>Update</h1></center>

| Metodo                                                      | Objetivo                                                                     | Exemplo de uso                                                                                                      |
| ----------------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `Update`                                                    | Marcar toda a entidade como modificada                                       | `_context.Update(entidade); await _context.SaveChangesAsync();`                                                     |
| `Attach` + `Entry(...).Property(...).IsModified = true`     | Atualização parcial em entidade desanexada (marcar propriedades específicas) | `_context.Attach(p); _context.Entry(p).Property(x => x.Nome).IsModified = true; await _context.SaveChangesAsync();` |
| `Entry(entity).State = EntityState.Modified`                | Marcar explicitamente o estado da entidade como Modified                     | `_context.Entry(entidade).State = EntityState.Modified; await _context.SaveChangesAsync();`                         |
| `SaveChanges` / `SaveChangesAsync`                          | Persistir alterações no banco                                                | `await _context.SaveChangesAsync();`                                                                                |
| Tratamento de concorrência (`DbUpdateConcurrencyException`) | Detectar e resolver conflitos de concorrência otimista                       | `try { await _context.SaveChangesAsync(); } catch(DbUpdateConcurrencyException) { /* resolver conflito */ }`        |

 
---

 <center><h1>Delete</h1></center>

| Metodo                                      | Objetivo                                        | Exemplo de uso                                                                                                                      |
| ------------------------------------------- | ----------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `Remove` / `RemoveRange`                    | Remover entidade(s) do contexto                 | `_context.Produtos.Remove(produto); await _context.SaveChangesAsync();`                                                             |
| `Entry(entity).State = EntityState.Deleted` | Marcar entidade como deletada sem chamar Remove | `_context.Entry(entidade).State = EntityState.Deleted; await _context.SaveChangesAsync();`                                          |
| `Find` + `Remove` (padrão)                  | Buscar por id e remover com verificação nula    | `var p = await _context.Produtos.FindAsync(id); if (p != null) { _context.Produtos.Remove(p); await _context.SaveChangesAsync(); }` |
| `SaveChanges` / `SaveChangesAsync`          | Persistir remoções no banco                     | `await _context.SaveChangesAsync();`                                                                                                |
