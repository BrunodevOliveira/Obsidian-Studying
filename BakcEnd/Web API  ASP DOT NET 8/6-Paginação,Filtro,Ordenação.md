Aplicar a paginação usando parâmetros na URL para definir o número e tamanho da página:

`https://localhost:7014/produtos?pageNumber=2&pageSize=3`
pageNumber -> número da página
pageSize -> quantidade de registros por página


# [[Excalidraw/Paginação.md|Implementação]]
Em um método Action de ProdutosController podemos incluir parâmetro para definir o número e tamanho da página:

- `[FromQuery]` -> vincula os parâmetros a valores fornecidos na string de consulta (query string)
- `ProdutosParameters` -> Classe quie encapsula os parâmetro usados
- `GetProdutos` -> Método do repositório de produtos usado para obter os produtos com os parâmetros definidos
![[Pasted image 20241023211012.png]]




