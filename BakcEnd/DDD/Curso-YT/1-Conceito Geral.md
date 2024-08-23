![[Introdução.excalidraw]]

# Visão estratégica

## Dominio
- Core Domain -> O principal objetivo do Software


## SubDomínio
- Shared Kernel -> Conjunto de funcionalidades que podem ser compartilhadas em outras partes da aplicação


# Visão tática
> [!note]+ Bounded Context
> Um Bounded Context é uma delimitação conceitual dentro do domínio do negócio. Ele define uma fronteira onde um modelo de domínio específico é consistente e aplicável.

## Arquitetura em camadas
> [!attention] Estrutura
> MeuSistema.sln
├── MeuSistema.Vendas (Bounded Context)
│   ├── Domain
│   ├── Application
│   └── Infrastructure
├── MeuSistema.Estoque (Bounded Context)
│   ├── Domain
│   ├── Application
│   └── Infrastructure
└── MeuSistema.Financeiro (Bounded Context)
- Cada Bounded Context deve implementar uma arquitetura em camadas
- O principal objetivo deve ser <span style="color:rgb(254, 0, 65)">proteger a camada de Domínio</span>
![[Pasted image 20240822212824.png]]
- Unbiquitous Language -> Utilizar os mesmos termos em todas as camadas
## Interface do usuário
- Forma dos usuários acessarem funcionalidades da aplicação que são expostas através da camada de aplicação

## Camada de aplicação
-  Camada onde implementamos uma API
## Camada de Domínio
![[Pasted image 20240822210252.png]]
- Entidade -> Conjunto de dados que pode ser modificado ao longo do tempo de vida da aplicação
	- Características -> Identidade(ID) | Atributos | Comportamento
- Objeto de Valor-> Mede, quantifica, ou descreve algum elemento no domínio  
	- Não possui um campo de identidade(ID). Sua identidade é baseada na composição dos valores dos atributos
	- Muito utilizado para armazenar comportamentos (Métodos)
-  Agregação -> Conjunto de objetos associados, que tratamos como um unidade para propósitos de modificação de dados
- Serviços-> Comportamentos que não cabem nas Entidades e Objetos de Valor
- Factories-> Construtores de entidades complexas
- Repositório -> Responsável pelo acesso e persistência de dados na camada de infraestrutura
	- Para isso utilizamos a <mark style="background-color: #fff88f; color: black">geralmente através de uma ORM</mark> 
	-  Utilizamos uma Interface para determinar o que deve ser implementado na camada de infraestrutura

> [!danger] Regra de outro!!
> O domínio não pode depender de nenhuma implementação externa!



 