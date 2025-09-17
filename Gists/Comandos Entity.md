| Metodo | Objetivo | Exemplo de uso |
|---|---|---|
| `Add` / `AddAsync` | Inserir uma nova entidade no contexto (Create) | `await _context.AddAsync(novaEntidade); await _context.SaveChangesAsync();` |
| `AddRange` / `AddRangeAsync` | Inserir múltiplas entidades de uma vez | `await _context.AddRangeAsync(lista); await _context.SaveChangesAsync();` |
| `Find` / `FindAsync` | Recuperar entidade por chave primária (Read) | `var e = await _context.Produtos.FindAsync(id);` |
| `FirstOrDefault` / `FirstOrDefaultAsync` | Recuperar o primeiro elemento que satisfaça a condição | `var p = await _context.Produtos.FirstOrDefaultAsync(p => p.Nome == nome);` |
| `SingleOrDefault` / `SingleOrDefaultAsync` | Recuperar exatamente um elemento (ou null) — garante unicidade | `var u = await _context.Usuarios.SingleOrDefaultAsync(u => u.Email == email);` |
| `Where` | Filtrar coleções (consulta) | `var ativos = _context.Clientes.Where(c => c.Ativo);` |
| `ToList` / `ToListAsync` | Executar a consulta e materializar lista | `var lista = await query.ToListAsync();` |
| `Include` / `ThenInclude` | Eager loading de relacionamentos | `var p = await _context.Pedidos.Include(p => p.Itens).ThenInclude(i => i.Produto).FirstOrDefaultAsync(...);` |
| `AsNoTracking` | Consultas somente leitura sem change tracking (melhora performance) | `var read = await _context.Produtos.AsNoTracking().ToListAsync();` |
| `Update` | Marcar toda a entidade como modificada (Update) | `_context.Update(entidade); await _context.SaveChangesAsync();` |
| `Attach` + `Entry(...).Property(...).IsModified = true` | Atualização parcial (marcar apenas propriedades específicas) | `_context.Attach(p); _context.Entry(p).Property(x => x.Nome).IsModified = true; await _context.SaveChangesAsync();` |
| `Remove` / `RemoveRange` | Remover entidade(s) do contexto (Delete) | `_context.Produtos.Remove(produto); await _context.SaveChangesAsync();` |
| `Entry(entity).State = EntityState.Deleted` | Alternativa para marcar entidade como deletada | `_context.Entry(entidade).State = EntityState.Deleted; await _context.SaveChangesAsync();` |
| `SaveChanges` / `SaveChangesAsync` | Persistir todas as alterações feitas no contexto | `await _context.SaveChangesAsync();` |
| `Database.BeginTransaction` / `BeginTransactionAsync` | Executar operações em transação explícita | `using var tx = await _context.Database.BeginTransactionAsync(); /* ... */ await tx.CommitAsync();` |
| Tratamento de concorrência (`DbUpdateConcurrencyException`) | Detectar conflitos de concorrência otimista (RowVersion) | `try { await _context.SaveChangesAsync(); } catch(DbUpdateConcurrencyException) { /* resolver conflito */ }` |
