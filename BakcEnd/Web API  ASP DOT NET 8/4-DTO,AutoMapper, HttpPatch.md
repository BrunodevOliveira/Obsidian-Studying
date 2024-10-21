# Data transfer Object

é uma estratégia que visa não expor todo o modelo de domínio  da nossa API. Devemos evitar retornar entidades de domínio a partir dos endpoints de uma API. 

Entidades de domínio não devem depender de lógica externa e devem ser isoladas das camadas externas da aplicação

Um DTO é um contêiner de dados usado para transportar dados entre diferentes partes ou camadas e um aplicação que define como os dados serão enviados pela rede

# Implementação

1- Criar uma classe/record para o DTO que contenha os membros que representam os dados que você deseja transferir entre diferentes partes de sua aplicação.

2- A nomenclatura usada para os DTO's geralmente usa o sufixo DTO no final do nome da entidade para a qual estamos criando o DTO


Obs. DTO's podem conter validações, formatações ou outras anotações aos membros de um DTO, por exemplo, Data Annotations 

## DTO de entrada

São usados para recebere dados nos requests HTTP


## DTO no retorno
São usados para modelar os dados que os controladores retornam aos clientes


# AutoMapper Biblioteca

Realiza o mapeamento entre os objetos que representam nossas entidades e os objetos que representam os nossos DTOs filtrando as propriedades que desejamos expor

**Pacotes necessários:**
- `Automapper`
- `Automapper.Extensions.Microsoft.DependencyInjection`

## Implementação

1- Criar classes de perfil que deve herdar de Profile
Criar perfis de mapeamento e definir como o AutoMapper deve mapear as entidades para os DTOs e vice-versa (Criar classes de perfil)

2- Configurar o serviço do AutoMapper

3- Para utilizar basta injetar a interface IMapper como uma dependência no construtor do Controller ou Serviço
