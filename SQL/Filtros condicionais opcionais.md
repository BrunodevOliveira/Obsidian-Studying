> [!NOTE]
    > Essa estrutura permite que uma query funcione tanto quando um filtro é aplicado quanto quando ele deve ser ignorado.

# Formas de aplicação

## 1. Com NULL
```sql
-- Usando NULL como valor sentinela (mais limpo)
AND (
  '${filtro.hostname}' IS NULL 
  OR hostname = '${filtro.hostname}'
)
```

## 2. Com **CASE WHEN**
```sql
-- Mais explícita e legível
AND (
  CASE 
    WHEN '${filtro.hostname}' = '-1' THEN TRUE
    WHEN '${filtro.hostname}' IS NULL THEN TRUE
    ELSE hostname = '${filtro.hostname}'
  END
)
```

## 3. Com COALESCE
```sql
-- Usando COALESCE para tratar valores nulos
AND hostname = COALESCE(NULLIF('${filtro.hostname}', ''), hostname)

```

### Filtro de data
```sql
SELECT *
FROM sua_tabela
WHERE sua_coluna_data BETWEEN
  COALESCE(CAST('${dataInicio}' AS TIMESTAMP), '1900-01-01 00:00:00'::TIMESTAMP) -- Se dataInicio NULL, usa uma data bem antiga
  AND
  COALESCE(CAST('${dataFim}' AS TIMESTAMP), '2100-12-31 23:59:59'::TIMESTAMP); -- Se dataFim NULL, usa uma data bem futura

```
- `COALESCE(valor1, valor2)` retorna o primeiro valor não `NULL` da lista [6](https://www.coginiti.co/tutorials/analysis/compare-null-to-value/).
    
- Aqui, se `dataInicio` for `NULL`, ele usará `'1900-01-01 00:00:00'`. Se `dataFim` for `NULL`, ele usará `'2100-12-31 23:59:59'`. Isso permite que o `BETWEEN` funcione sempre com dois valores concretos, simulando o "sem limite".
    
- <span style="background:rgba(240, 200, 0, 0.2)">A desvantagem é que você precisa escolher datas de "min" e "max" que realmente englobem todos os dados possíveis na sua tabela.</span>

## Exemplo de aplicação
```sql
SELECT *
FROM servidores s
WHERE 1=1
  -- Filtro por hostname (opcional)
  AND (
    COALESCE(NULLIF('${filtro.hostname}', ''), hostname) = hostname
  )
  -- Filtro por status (opcional)
  AND (
    '${filtro.status}' IS NULL 
    OR s.status = '${filtro.status}'
  )
  -- Filtro por data (opcional)
  AND (
    '${filtro.data_inicio}' IS NULL 
    OR s.data_criacao >= CAST('${filtro.data_inicio}' AS DATE)
  )
  -- Filtro por range numérico (opcional)
  AND (
    '${filtro.cpu_min}' IS NULL 
    OR s.cpu_percent >= CAST('${filtro.cpu_min}' AS DECIMAL)
  );
```


# Função auxiliar para filtros
```sql
CREATE OR REPLACE FUNCTION filtro_opcional(
  valor_filtro TEXT,         -- Parâmetro 1: o valor recebido para filtrar
  valor_coluna TEXT,         -- Parâmetro 2: o valor da coluna da tabela
  valor_sentinela TEXT DEFAULT '-1' -- Parâmetro 3 (opcional): o valor que indica "ignorar filtro"
) RETURNS BOOLEAN AS $$      -- Tipo de retorno da função: um booleano (TRUE/FALSE)
BEGIN
  -- Lógica da função
  RETURN (
    valor_filtro = valor_sentinela  -- Se o valor_filtro for igual ao valor_sentinela
    OR valor_filtro IS NULL         -- OU se o valor_filtro for NULL (ignora filtro)
    OR valor_filtro = valor_coluna  -- OU se o valor_filtro for igual ao valor da coluna (aplica filtro)
  );
END;
$$ LANGUAGE plpgsql;        -- Linguagem em que a função está escrita (PL/pgSQL)

```

- `CREATE OR REPLACE FUNCTION filtro_opcional(...)`:
    
    - `CREATE FUNCTION`: Comando para criar uma nova função [11](https://www.postgresql.org/docs/current/sql-createfunction.html)[9](https://www.pgtutorial.com/plpgsql/plpgsql-functions/)[12](https://neon.com/postgresql/postgresql-plpgsql/postgresql-create-function)[10](https://www.beekeeperstudio.io/blog/postgresql-create-function).
        
    - `OR REPLACE`: Se uma função com o mesmo nome já existir, ela será substituída pela nova definição. Isso é útil para atualizar a função sem ter que excluí-la e recriá-la [11](https://www.postgresql.org/docs/current/sql-createfunction.html)[9](https://www.pgtutorial.com/plpgsql/plpgsql-functions/)[12](https://neon.com/postgresql/postgresql-plpgsql/postgresql-create-function).
        
    - `filtro_opcional`: É o nome que você dá à sua função. Use nomes descritivos [11](https://www.postgresql.org/docs/current/sql-createfunction.html)[9](https://www.pgtutorial.com/plpgsql/plpgsql-functions/)[12](https://neon.com/postgresql/postgresql-plpgsql/postgresql-create-function).
        
- `(valor_filtro TEXT, valor_coluna TEXT, valor_sentinela TEXT DEFAULT '-1')`
    
    - Define os **parâmetros de entrada** da função [11](https://www.postgresql.org/docs/current/sql-createfunction.html)[9](https://www.pgtutorial.com/plpgsql/plpgsql-functions/)[12](https://neon.com/postgresql/postgresql-plpgsql/postgresql-create-function).
        
    - `valor_filtro TEXT`: É o valor que viria da sua aplicação (ex: `'${filtro.hostname}'`). Declaro como `TEXT` para ser genérico, mas pode ser `INTEGER`, `DATE`, etc., dependendo do uso.
        
    - `valor_coluna TEXT`: É o valor da coluna da sua tabela que você quer comparar (ex: `hostname`). Também `TEXT`.
        
    - `valor_sentinela TEXT DEFAULT '-1'`: Este é um parâmetro **opcional** com um valor padrão. Se você não passá-lo ao chamar a função, ele usará `'-1'`. Isso permite flexibilidade no valor que indica "ignorar o filtro".
        
- `RETURNS BOOLEAN`:
    
    - Especifica o **tipo de dado que a função irá retornar** [11](https://www.postgresql.org/docs/current/sql-createfunction.html)[9](https://www.pgtutorial.com/plpgsql/plpgsql-functions/)[12](https://neon.com/postgresql/postgresql-plpgsql/postgresql-create-function)[10](https://www.beekeeperstudio.io/blog/postgresql-create-function).
        
    - Neste caso, ela retorna `TRUE` ou `FALSE`, que são ideais para uso em cláusulas `WHERE`.
        
- `AS $$ ... $$ LANGUAGE plpgsql;`:
    
    - `AS $$ ... $$`: Delimita o **corpo da função**. O código SQL ou PL/pgSQL da função fica entre esses dois `$$` (isso é chamado de "dollar-quoted string", e é uma forma segura de escrever blocos de código sem se preocupar com aspas) [9](https://www.pgtutorial.com/plpgsql/plpgsql-functions/)[12](https://neon.com/postgresql/postgresql-plpgsql/postgresql-create-function)[10](https://www.beekeeperstudio.io/blog/postgresql-create-function).
        
    - `LANGUAGE plpgsql`: Indica a linguagem em que o corpo da função está escrito. `PL/pgSQL` é uma linguagem procedural para PostgreSQL, que permite lógica mais complexa (como `IF/ELSE`, loops, variáveis) do que o SQL puro [11](https://www.postgresql.org/docs/current/sql-createfunction.html)[9](https://www.pgtutorial.com/plpgsql/plpgsql-functions/)[12](https://neon.com/postgresql/postgresql-plpgsql/postgresql-create-function)[10](https://www.beekeeperstudio.io/blog/postgresql-create-function).
        
- `BEGIN ... END;`:
    
    - Definem um bloco de código dentro da função PL/pgSQL [9](https://www.pgtutorial.com/plpgsql/plpgsql-functions/)[10](https://www.beekeeperstudio.io/blog/postgresql-create-function).
        
- `RETURN (...)`:
    
    - A instrução `RETURN` especifica o valor que a função deve retornar. A lógica aqui é a mesma que vimos para filtros opcionais: se o `valor_filtro` for o sentinela, ou se for `NULL`, ou se ele realmente corresponder ao `valor_coluna`, a condição é `TRUE`.


## Uso numa query
```sql
SELECT *
FROM sua_tabela
WHERE filtro_opcional('${filtro.nome}', nome_da_coluna);

-- Ou com um sentinela diferente, se necessário
SELECT *
FROM outra_tabela
WHERE filtro_opcional('${filtro.id}', id_coluna::TEXT, '0'); -- Convertendo id_coluna para TEXT se for INTEGER
```
<span style="background:rgba(240, 200, 0, 0.2)">Note que, como a função foi definida para `TEXT`, você pode precisar fazer um `CAST` da sua coluna para `TEXT` se ela for de outro tipo (ex: `id_coluna::TEXT`), ou criar sobrecargas da função para diferentes tipos de dados.</span>
