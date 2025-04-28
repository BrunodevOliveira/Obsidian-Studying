# Trabalhando com Moedas em C\#

No desenvolvimento de aplicações que lidam com valores monetários, é fundamental entender como manipular corretamente moedas em C\#. Vamos explorar os principais aspectos para garantir precisão e formatação adequada nos seus projetos.

## Contextualização

Como desenvolvedor full-stack, você frequentemente precisará lidar com valores monetários em diversas partes da aplicação - desde o cálculo de preços no back-end até a exibição formatada para o usuário final. Manipular moedas incorretamente pode levar a erros críticos de arredondamento e problemas financeiros sérios em aplicações comerciais.

## Tipo para Moedas

### Decimal: O Tipo Ideal

Para qualquer operação que envolva valores monetários, o tipo `decimal` é a escolha mais adequada em C\#. Diferentemente de `float` ou `double`, o tipo `decimal` oferece a precisão necessária para evitar erros de arredondamento em cálculos financeiros.

**Por que usar decimal?**

Vamos considerar um exemplo prático: calcular o valor total de 17 itens com preço unitário de R\$ 4,99.

```csharp
// Usando float (INCORRETO para valores monetários)
float precoFloat = 4.99f;
float totalFloat = precoFloat * 17; // Resultado: 84,82999

// Usando decimal (CORRETO para valores monetários)
decimal precoDecimal = 4.99m; // Note o sufixo 'm' para decimal
decimal totalDecimal = precoDecimal * 17; // Resultado: 84,83
```

Observe que ao usar `float`, obtemos um resultado impreciso (84,82999), enquanto com `decimal` temos o valor correto (84,83).

**Declarando valores decimais:**

```csharp
// Use o sufixo 'm' ou 'M' para declarar literais decimais
decimal preco = 199.99m;
decimal desconto = 0.15M;
```


## Formatando Moedas

### Usando o Especificador de Formato "C"

O C\# oferece uma maneira simples de formatar valores como moeda usando o especificador de formato "C" com o método `ToString()`:

```csharp
decimal valor = 1234.56m;

// Formatação básica de moeda (usa a cultura atual do sistema)
string valorFormatado = valor.ToString("C"); // R$ 1.234,56 (assumindo cultura pt-BR)
```


### Especificando a Precisão Decimal

Você pode controlar o número de casas decimais adicionando um número após o "C":

```csharp
decimal valor = 1234.56789m;

string valorC2 = valor.ToString("C2"); // R$ 1.234,57 (2 casas decimais - padrão)
string valorC3 = valor.ToString("C3"); // R$ 1.234,568 (3 casas decimais)
string valorC4 = valor.ToString("C4"); // R$ 1.234,5679 (4 casas decimais)
```


### Formatação com Diferentes Culturas

Para formatar valores monetários de acordo com diferentes culturas e países:

```csharp
using System.Globalization;

decimal valor = 1234.56m;

// Formatando para a cultura brasileira
string valorBR = valor.ToString("C", new CultureInfo("pt-BR")); // R$ 1.234,56

// Formatando para a cultura dos EUA
string valorUS = valor.ToString("C", new CultureInfo("en-US")); // $1,234.56

// Formatando para a cultura japonesa
string valorJP = valor.ToString("C", new CultureInfo("ja-JP")); // ¥1,235
```


### Personalizando o Formato da Moeda

Você pode personalizar completamente o formato da moeda usando a classe `NumberFormatInfo`:

```csharp
using System.Globalization;

decimal valor = 1234.56m;

// Criando um formato personalizado baseado na cultura atual
NumberFormatInfo nfi = (NumberFormatInfo)CultureInfo.CurrentCulture.NumberFormat.Clone();

// Alterando o símbolo da moeda
nfi.CurrencySymbol = "BRL";

// Alterando o padrão de exibição para valores positivos
nfi.CurrencyPositivePattern = 2; // Formato: Símbolo Espaço Valor

// Formatando com as configurações personalizadas
string valorPersonalizado = valor.ToString("C", nfi); // BRL 1.234,56
```


## Operações Matemáticas com Moedas

### Arredondamento Correto

Ao realizar cálculos com valores monetários, é importante aplicar o arredondamento adequado:

```csharp
decimal preco = 10.345m;

// Arredondando para 2 casas decimais
decimal precoArredondado = Math.Round(preco, 2); // 10,35

// Arredondando para baixo (truncando)
decimal precoTruncado = Math.Floor(preco * 100) / 100; // 10,34

// Arredondando para cima
decimal precoParaCima = Math.Ceiling(preco * 100) / 100; // 10,35
```


### Evitando Erros de Precisão em Cálculos

Quando trabalhamos com divisões ou multiplicações que podem gerar muitas casas decimais, é importante arredondar os resultados intermediários:

```csharp
decimal valor = 576.43m;
decimal resto;

// Calculando quantas notas de R$ 100
int notas100 = (int)(valor / 100);
resto = valor % 100;

// Calculando quantas notas de R$ 50
int notas50 = (int)(resto / 50);
resto = resto % 50;

// Continuando o cálculo para outras denominações...

// Para moedas de centavos, é importante arredondar para evitar erros de precisão
resto = Math.Round(resto, 2);
int moedas1Cent = (int)Math.Round(resto * 100) % 5;
```


## Exemplo Prático: Sistema de Caixa

Vamos implementar um exemplo completo de um sistema que calcula o troco em diferentes denominações:

```csharp
using System;
using System.Globalization;

public class CaixaRegistradora
{
    public static void CalcularTroco(decimal valor)
    {
        // Formatando o valor de entrada
        Console.WriteLine($"Valor total: {valor.ToString("C", new CultureInfo("pt-BR"))}");
        
        // Calculando notas
        int notas100 = (int)(valor / 100);
        valor %= 100;
        
        int notas50 = (int)(valor / 50);
        valor %= 50;
        
        int notas20 = (int)(valor / 20);
        valor %= 20;
        
        int notas10 = (int)(valor / 10);
        valor %= 10;
        
        int notas5 = (int)(valor / 5);
        valor %= 5;
        
        int notas2 = (int)(valor / 2);
        valor %= 2;
        
        // Calculando moedas (com arredondamento para evitar erros de precisão)
        valor = Math.Round(valor, 2);
        
        int moedas1 = (int)valor;
        valor %= 1;
        
        int moedas50 = (int)(valor / 0.5m);
        valor %= 0.5m;
        
        int moedas25 = (int)(valor / 0.25m);
        valor %= 0.25m;
        
        int moedas10 = (int)(valor / 0.1m);
        valor %= 0.1m;
        
        int moedas5 = (int)(valor / 0.05m);
        valor %= 0.05m;
        
        // Multiplicamos por 100 e arredondamos para evitar erros de ponto flutuante
        int moedas1cent = (int)Math.Round(valor * 100);
        
        // Exibindo o resultado
        Console.WriteLine("NOTAS:");
        Console.WriteLine($"{notas100} nota(s) de R$ 100.00");
        Console.WriteLine($"{notas50} nota(s) de R$ 50.00");
        Console.WriteLine($"{notas20} nota(s) de R$ 20.00");
        Console.WriteLine($"{notas10} nota(s) de R$ 10.00");
        Console.WriteLine($"{notas5} nota(s) de R$ 5.00");
        Console.WriteLine($"{notas2} nota(s) de R$ 2.00");
        
        Console.WriteLine("MOEDAS:");
        Console.WriteLine($"{moedas1} moeda(s) de R$ 1.00");
        Console.WriteLine($"{moedas50} moeda(s) de R$ 0.50");
        Console.WriteLine($"{moedas25} moeda(s) de R$ 0.25");
        Console.WriteLine($"{moedas10} moeda(s) de R$ 0.10");
        Console.WriteLine($"{moedas5} moeda(s) de R$ 0.05");
        Console.WriteLine($"{moedas1cent} moeda(s) de R$ 0.01");
    }
}
```

# Sufixo `M` ou `m`
> [!NOTE]
> O sufixo 'm' ou 'M' em C# é utilizado para indicar explicitamente que um número literal deve ser tratado como um valor do tipo `decimal`, e não como outro tipo numérico como `double` ou `float`
> 

## Diferenciação de tipos
Sem o sufixo, o compilador C# interpreta literais numéricos com ponto decimal como sendo do tipo `double` por padrão. Ao adicionar o sufixo 'm' ou 'M', você instrui o compilador a tratar o valor como `decimal`


```csharp
// Sem sufixo - requer conversão explícita
decimal precoErrado = (decimal)199.99; // Conversão de double para decimal

// Com sufixo - já é decimal diretamente
decimal precoCorreto = 199.99m;
```

## Precisão em cálculos financeiros
O uso do sufixo é especialmente importante em cálculos financeiros, onde a precisão é crucial. O tipo `decimal` foi projetado especificamente para cálculos monetários e financeiros, oferecendo maior precisão do que `float` ou `double`[5](https://www.macoratti.net/12/12/c_num1.htm).


Aja como um professor exigente e altamente qualificado, especialista em C# e .NET, com sólida experiência prática no desenvolvimento de projetos reais aplicando Clean Architecture, SOLID, DDD, ASP.NET Core, Entity Framework, entre outras ferramentas e padrões utilizados no mercado de trabalho.

