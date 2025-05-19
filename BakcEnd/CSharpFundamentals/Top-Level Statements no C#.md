# Top-Level Statements no C\#

Os Top-Level Statements (Instruções de Nível Superior) foram ==introduzidos no C\# 9.0== como uma forma de simplificar a estrutura básica de programas C\#, permitindo que você <span style="background:#40a9ff">escreva código diretamente no nível superior de um arquivo</span>, ==sem a necessidade== de encapsulá-lo dentro de ==namespaces==, ==classes ou um método Main.==

## O que são Top-Level Statements?

Antes do C\# 9.0, todo programa C\# exigia uma estrutura padrão que incluía:

- Diretivas using
- Declaração de namespace
- Declaração de classe
- Método Main estático

Por exemplo:

```csharp
using System;

namespace MeuPrograma
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Olá, mundo!");
        }
    }
}
```

Com os Top-Level Statements, você pode simplificar isso para:

```csharp
Console.WriteLine("Olá, mundo!");
```


## Regras e Limitações

1. **Apenas um arquivo com Top-Level Statements**: Você só pode ter um arquivo com instruções de nível superior em todo o projeto[^1][^5]. Isso ocorre porque o compilador gera um método Main implícito que serve como ponto de entrada do programa[^6].
2. **Ordem das declarações**: As instruções de nível superior devem vir antes de qualquer declaração de namespace ou tipo[^1][^4]. Se você tentar colocar namespaces ou classes antes das instruções de nível superior, receberá o erro: "Top-Level statements must precede namespace and type declarations"[^4].
3. **Acesso aos argumentos**: Os argumentos da linha de comando estão disponíveis através de uma variável implícita chamada `args`, que é um array de strings[^8].
4. **Métodos locais**: Você pode definir métodos locais dentro das instruções de nível superior[^2]:
```csharp
Console.WriteLine("Olá, mundo!");

int resultado = Somar(5, 10);
Console.WriteLine($"O resultado da soma é: {resultado}");

int Somar(int a, int b)
{
    return a + b;
}
```


## Como o Compilador Processa Top-Level Statements

Nos bastidores, o compilador C\# gera automaticamente:

- Uma classe Program implícita
- Um método Main estático dentro dessa classe
- Coloca suas instruções de nível superior dentro desse método Main[^6]

A assinatura do método Main gerado depende do uso de operações assíncronas e instruções return[^8]:

| Operações assíncronas \ Return com expressão | Presente | Ausente |
| :-- | :-- | :-- |
| Presente | `static Task<int> Main(string[] args)` | `static Task Main(string[] args)` |
| Ausente | `static int Main(string[] args)` | `static void Main(string[] args)` |

## Vantagens dos Top-Level Statements

1. **Código mais conciso**: Reduz a verbosidade, especialmente para programas pequenos ou exemplos de código[^2].
2. **Facilita o aprendizado**: Ideal para iniciantes, pois remove conceitos complexos como namespaces e classes do código inicial[^4].
3. **Prototipagem rápida**: Permite experimentar ideias rapidamente sem a necessidade de criar toda a estrutura de um programa completo[^1].
4. **Transição suave**: Você pode começar com instruções de nível superior e, à medida que seu programa cresce, refatorar para uma estrutura mais organizada com classes e métodos[^1].



[^1]: https://learn.microsoft.com/pt-br/dotnet/csharp/tutorials/top-level-statements

[^2]: https://dev.to/develop4us/dica-c-top-level-statements-2be2

[^3]: https://www.youtube.com/watch?v=tzV3HzXytZM

[^4]: https://www.macoratti.net/20/11/c9_instsup1.htm

[^5]: https://pt.stackoverflow.com/questions/556831/existe-alguma-forma-de-mudar-o-novo-layout-da-ide-vs

[^6]: https://www.youtube.com/watch?v=VjFgZtnHNjo

[^7]: https://www.macoratti.net/20/10/vda151020.htm

[^8]: https://learn.microsoft.com/pt-br/dotnet/csharp/language-reference/proposals/csharp-9.0/top-level-statements

[^9]: https://www.youtube.com/watch?v=tpf4LVOYELQ

[^10]: https://www.dio.me/articles/revolucao-no-c-da-verbosidade-a-elegancia

