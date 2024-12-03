# Abordagens e configuração

## 1- Querystring
Inclui o número da versão como um parâmetro de consulta na URL
`https://api.exemplo.com/resource?version=1`
`https://api.exemplo.com/resource?version=2`

## 2- URI
Inclui a versão diretamente na URL da API
`https://api.exemplo.com/v1/resource`
`https://api.exemplo.com/v2/resource`

## 3- Headers
Especifica a versão desejada no cabeçalho(header) do request HTTP
```
GET/resource HTTP/1.1
Host: api.exemplo.com
Accept: application/json
X-API-Version: 1
```

## 4- Media Type
Usar diferentes tipos de mídia para representar versões diferentes de API
`Accept: application/vnd.exemplo.v1+json`

# Configuração

## Pacotes 
- `Asp.Versioning.Mvc.ApiExplorer`
- `Asp.Versioning.Http`

## Configuração em `Program.cs`
