# Subdomínio
> [!question] Dividir
> Assim como no churrasco você não assa o boi inteiro, no DDD precisamos dividir o domínio (churrasco) em subdomínios (peças de carne) e nos concentrar no preparo de cada uma delas
- O DDD nos encoraja a dividir em partes o problema que temos que resolver com o software
- <mark style="background-color: #fff88f; color: black">Essas partes são chamadas de subdomínio</mark>
- Cada subdomínio terá sua responsabilidade
- Não devemos nos preocupar com nada que seja fora do domínio, por exemplo infraestrutura

# Core Domain
- Dominio principal
- Aquilo que a organização faz de melhor e não deve terceirizar
- O correto é começar pelo Core Domain


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
Bounded Contexts:

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

# Shared Kernel

# Ubiquotous Language

