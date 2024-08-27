# Domain
O problema que iremos resolver com o software que será construido. Geralmente se refere ao contexto geral de uma organização


# Core Domain
- Dominio principal
- Aquilo que a organização faz de melhor e não deve terceirizar
- O correto é começar pelo Core Domain


# Subdomínio

> [!question] Dividir
> Assim como no churrasco você não assa o boi inteiro, no DDD precisamos dividir o domínio (churrasco) em subdomínios (peças de carne) e nos concentrar no preparo de cada uma delas
- O DDD nos encoraja a dividir em partes o problema que temos que resolver com o software
- <mark style="background-color: #fff88f; color: black">Essas partes são chamadas de subdomínio</mark>
- Cada subdomínio terá sua responsabilidade
- Não devemos nos preocupar com nada que seja fora do domínio, por exemplo infraestrutura


# Bounded Context
> [!info] 
> é uma delimitação conceitual dentro do domínio do negócio. Ele define uma fronteira onde um modelo de domínio específico é consistente e aplicável.

- É onde como colocamos <span style="color:rgb(254, 0, 65)">limites no modelo do subdomínio</span>
- Conceitos que fazem sentido em uma parte do domínio, podem não ter o mesmo sentido em outras partes, mesmo que tenham o mesmo nome, ou até se refiram à mesma coisa
> [!attention] Subdominio X Bounded Context
> Subdominio é o problema, enqaunto o Bounded Context é a solução do problema.
> Subdominio é o chão da sala e o Bounded context é o tapete

## Limites de um Bounded Context
- Devemos  delimitar quais ais informações que um Bounded context deve ter acesso e não deixar que ele enxergue coisas que são de outros Bounded context ou que não faça parte de seu escopo
![[LimitesDoBoundedContext.png]]

## Bounded Context X Microserviços
> [!NOTE]
> Bounded Contexts são uma ferramenta de modelagem de domínio que ajuda a organizar a complexidade do negócio, enquanto microserviços são uma escolha arquitetural que influencia como o sistema é construído e implantado. Embora frequentemente alinhados, eles abordam diferentes aspectos do design de software.

**Bounded Context:**

1. Conceito: É um padrão de design do Domain-Driven Design (DDD) que foca na organização lógica do domínio do negócio.
2. Escopo: Define limites conceituais dentro do domínio da aplicação.
3. Implementação: Pode ser implementado em um monolito ou em microserviços.
4. Objetivo: Manter a consistência do modelo e da linguagem ubíqua dentro de um contexto específico.
5. Granularidade: Pode variar em tamanho, desde pequenas partes de um aplicativo até sistemas inteiros.

**Microserviços:**

1. Conceito: É um estilo arquitetural para desenvolver aplicações como um conjunto de serviços pequenos e independentes.
2. Escopo: Define limites físicos e operacionais na arquitetura do sistema.
3. Implementação: Sempre distribuído, com serviços separados e independentes.
4. Objetivo: Melhorar a escalabilidade, a manutenibilidade e a implantação independente de partes do sistema.
5. Granularidade: Geralmente pequenos serviços focados em capacidades específicas do negócio.

**Principais diferenças:**

1. Nível de abstração:
    - Bounded Contexts: Abstração de domínio e design.
    - Microserviços: Abstração arquitetural e de implantação.
2. Independência:
    - Bounded Contexts: Podem compartilhar a mesma base de código ou banco de dados.
    - Microserviços: São completamente independentes, com seu próprio código e armazenamento de dados.
3. Implantação:
    - Bounded Contexts: Podem ser implantados juntos ou separadamente.
    - Microserviços: Sempre implantados separadamente.
4. Comunicação:
    - Bounded Contexts: Podem se comunicar diretamente através de chamadas de método in-process.
    - Microserviços: Comunicam-se através de protocolos de rede (HTTP, gRPC, mensageria, etc.).
5. Alinhamento:
    - Bounded Contexts: Alinhados com o modelo de domínio e a organização do negócio.
    - Microserviços: Alinhados com capacidades técnicas e de negócios, mas também considerando aspectos operacionais.

**Relação entre os conceitos:**
- Um microserviço bem projetado geralmente encapsula um Bounded Context.
- Um Bounded Context pode ser implementado como um ou mais microserviços.
- É possível ter múltiplos Bounded Contexts em um único microserviço (embora não seja ideal).


# Context Map
Se um domínio possui muitos Bounded Contexts, o DDD sugere o uso de Mapas de Contexto como uma forma de documentar e comunicar os limites de cada contexto.
- é uma documentação para destacar os elementos que podem gerar conflitos entre os bounded contexts
- Nesse documento apontamos as palavras em comum entre bounded contexts e o seu significado em cada contexto, os serviços compartilhados que cada um acessa (ex.: login)

# Shared Kernel
É o Bounded Context que será compartilhado por todos os outros subdomínios do nosso domínio.

# Ubiquotous Language
> [!NOTE]
> É o vocabulário do seu modelo

Não devemos ter "dicionários" para traduzir os termos do usuário para os nossos termos.
O ideal é ter uma linguagem única ou comum para todos os envolvidos

Linhagem única:
- Para um único bounded context
- Utilizada nesse contexto em qualquer conversa ou codigo
- Documento, Código, Database, Emails, Pendências, etc


# Resumo
> [!NOTE]
> Imaginemos que estamos construindo uma cidade chamada "DDDville"
> 
- Domain (Domínio):
    - Analogia: É toda a cidade de DDDville. Representa todo o problema que estamos tentando resolver.
    - Técnico: O problema geral que o software resolve, incluindo todas as regras de negócio, processos e entidades.
- Core Domain (Domínio Principal):
    - Analogia: É o centro da cidade, onde está a prefeitura. É a parte mais importante e que diferencia DDDville de outras cidades.
    - Técnico: A parte mais crítica e valiosa do sistema, que oferece vantagem competitiva e não deve ser terceirizada.
- Subdomain (Subdomínio):
    - Analogia: São os diferentes bairros da cidade. Cada bairro tem sua função específica (residencial, comercial, industrial).
    - Técnico: Áreas distintas dentro do domínio principal, cada uma com seu próprio conjunto de problemas e soluções.
- Bounded Context (Contexto Delimitado):
    - Analogia: São as regras e leis específicas de cada bairro. Por exemplo, no bairro industrial, as regras são diferentes do bairro residencial.
    - Técnico: Fronteira explícita dentro da qual um modelo de domínio específico se aplica, com sua própria linguagem ubíqua e implementação.
- Context Map (Mapa de Contexto):
    - Analogia: É o mapa da cidade que mostra como os bairros se conectam e interagem entre si.
    - Técnico: Documento ou diagrama que ilustra as relações entre diferentes Bounded Contexts, incluindo padrões de integração como Shared Kernel, Customer-Supplier, etc.
- Shared Kernel (Núcleo Compartilhado):
    - Analogia: É como o sistema de água e esgoto da cidade, compartilhado entre diferentes bairros.
    - Técnico: Código ou modelo de domínio compartilhado entre dois ou mais Bounded Contexts, exigindo coordenação para mudanças.
- Ubiquitous Language (Linguagem Ubíqua):
    - Analogia: É o dialeto local de DDDville. Todos na cidade usam os mesmos termos para se referirem às coisas, evitando mal-entendidos.
    - Técnico: Vocabulário comum usado por desenvolvedores e especialistas do domínio para descrever o sistema, refletido no código e na modelagem.