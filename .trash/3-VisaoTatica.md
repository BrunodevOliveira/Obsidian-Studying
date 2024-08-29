Elementos que irão compor um bounded context

Tópicos abordados
- Arquitetura em camadas
- Shared Kernel
- Foco no Domínio
- Entities(Entidades)
- Associações ente entidades
- Enum
- Comportamentos em entidades
- Comportamente em Objetos de valor
- Aggregates
- Repositories
- ORM
- Injeção de dependência
- Domain Services

# Arquitetura em Camadas
A solution representa o Subdomínio
As pastas representam os bounded Contexts, principal e que se relacionam com o principal
Os projetos irão representar as camadas
Namespace: Dominio.Subdominio.Camada

# Entidades
- Utilizada para armazenar, localizar e recuperar
- Deve possuir uma chave de identidade "ID"
- São objetos mutáveis. Suas propriedades podem mudar e portanto não podem ser utilizadas para identificar o objeto

## Como definir as chaves 
A maioria dos bancos já implementa uma forma de gerar uma key/Id de forma automática para cada registro
Caso preferir gerar o ID pelo C#, podemos utilizar o `System.Guid` 

## Estrutura inicial 
![[DDD-Flow para criação de uma entidade.png]]
IEntity -> Interface que identifica o que é uma Entidade no domínio

Entity<Tkey> -> Classe que define o tipo do KEY e o comportamento para comparar a igualdade entre classes

EntityKeySeq -> Classe que define que a Entidade terá uma Key do tipo long e que será gerada por uma "Sequence"



# Asso