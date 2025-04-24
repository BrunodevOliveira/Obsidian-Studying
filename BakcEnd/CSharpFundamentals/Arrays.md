
# Arrays em C\#: Fundamentos e Manipulação

No desenvolvimento back-end com C\#, arrays são estruturas fundamentais que você utilizará constantemente para armazenar e manipular coleções de dados. Como desenvolvedor front-end migrando para full-stack, entender arrays é essencial para implementar lógicas eficientes em suas APIs.

## O que são Arrays e seus principais aspectos

Arrays em C\# são estruturas de dados fortemente tipadas que armazenam uma coleção fixa de elementos do mesmo tipo. Diferente de JavaScript, onde arrays são mais flexíveis, em C\# eles possuem características específicas:

- **Tamanho fixo**: Uma vez criado, o tamanho do array não pode ser alterado
- **Indexação baseada em zero**: O primeiro elemento está na posição 0
- **Homogeneidade**: Todos os elementos devem ser do mesmo tipo
- **Acesso direto**: Elementos podem ser acessados diretamente pelo índice

**Declaração e inicialização de arrays:**

```csharp
// Declaração com tamanho definido
int[] numeros = new int[5]; // Array de inteiros com 5 posições

// Declaração com inicialização direta
string[] nomes = new string[] { "João", "Maria", "Pedro" };

// Sintaxe simplificada
double[] precos = { 19.99, 29.99, 39.99 };

// Array multidimensional
int[,] matriz = new int[3, 2]; // Matriz 3x3

// Array de arrays (jagged array)
int[][] arrayIrregular = new int[3][];
arrayIrregular[0] = new int[] { 1, 2 };
arrayIrregular[1] = new int[] { 3, 4, 5 };
arrayIrregular[2] = new int[] { 6 };
```


## Maneiras de percorrer Arrays

Existem várias formas de iterar sobre arrays em C\#, cada uma com seus casos de uso específicos:

**1. Loop for tradicional:**

```csharp
int[] numeros = { 10, 20, 30, 40, 50 };

for (int i = 0; i < numeros.Length; i++)
{
    Console.WriteLine($"Elemento {i}: {numeros[i]}");
}
```

**2. Loop foreach (mais limpo e seguro):**

```csharp
string[] tecnologias = { "C#", "ASP.NET Core", "Entity Framework" };

foreach (string tech in tecnologias)
{
    Console.WriteLine(tech);
}
```

**3. Métodos LINQ (Language Integrated Query):**

```csharp
int[] valores = { 5, 8, 12, 15, 20 };

// Usando LINQ para filtrar e processar
var valoresFiltrados = valores.Where(v => v > 10);
foreach (var valor in valoresFiltrados)
{
    Console.WriteLine(valor); // Imprime 12, 15, 20
}
```

**4. Métodos Array.ForEach:**

```csharp
string[] frameworks = { "Angular", "React", "Vue" };

Array.ForEach(frameworks, framework => {
    Console.WriteLine($"Framework: {framework}");
});
```


## Manipulação dos dados de um Array

A manipulação de arrays em C\# envolve diversas operações comuns que você precisará dominar:

**1. Preenchimento e modificação:**

```csharp
int[] numeros = new int[5];

// Preenchendo o array
for (int i = 0; i < numeros.Length; i++)
{
    numeros[i] = i * 10;
}

// Modificando um elemento específico
numeros[2] = 100;
```

**2. Ordenação e inversão:**

```csharp
string[] frutas = { "banana", "maçã", "laranja", "uva" };

// Ordenando o array
Array.Sort(frutas);
// Resultado: "banana", "laranja", "maçã", "uva"

// Invertendo a ordem
Array.Reverse(frutas);
// Resultado: "uva", "maçã", "laranja", "banana"
```

**3. Busca de elementos:**

```csharp
int[] valores = { 10, 20, 30, 40, 50 };

// Verificando se contém um valor
bool contem30 = Array.Exists(valores, v => v == 30); // true

// Encontrando o índice de um elemento
int indice = Array.IndexOf(valores, 40); // 3

// Encontrando todos os elementos que atendem a uma condição
int[] maioresQue25 = Array.FindAll(valores, v => v > 25);
// Resultado: { 30, 40, 50 }
```

**4. Copiando arrays:**

```csharp
int[] original = { 1, 2, 3, 4, 5 };

// Copiando para um novo array
int[] copia = new int[original.Length];
Array.Copy(original, copia, original.Length);

// Método alternativo de cópia
int[] outraCopia = (int[])original.Clone();
```

**5. Redimensionando arrays (criando um novo):**

```csharp
int[] numeros = { 1, 2, 3 };

// Redimensionando (na verdade, cria um novo array)
Array.Resize(ref numeros, 5);
// Resultado: { 1, 2, 3, 0, 0 }
```

**6. Utilizando métodos LINQ para manipulação avançada:**

```csharp
int[] valores = { 3, 9, 2, 8, 6, 5 };

// Filtrando, ordenando e transformando
var resultado = valores
    .Where(v => v % 2 == 0)     // Filtra pares: 2, 8, 6
    .OrderByDescending(v => v)  // Ordena decrescente: 8, 6, 2
    .Select(v => v * 2);        // Multiplica por 2: 16, 12, 4

// resultado contém: { 16, 12, 4 }
```


## Considerações de Desempenho

- Arrays têm acesso mais rápido aos elementos (O(1)) comparado a outras coleções
- Para coleções de tamanho variável, prefira `List<T>` em vez de redimensionar arrays
- Em operações de busca frequentes, considere usar `Dictionary<K,V>` ou `HashSet<T>`
- Para grandes volumes de dados, os métodos LINQ podem ser menos eficientes que loops tradicionais


# Arrays vs Lists: Entendendo as diferenças fundamentais

Quando falamos que um Array tem "tamanho fixo", isso significa que, uma vez criado, o número de elementos que ele pode armazenar não pode ser alterado. Vamos entender melhor essa diferença fundamental entre Arrays e Lists.

## Arrays e seu tamanho fixo

Se você tem um array como este:

```csharp
int[] numeros = new int[^5]; // Array com 5 posições
```

Este array foi criado com exatamente 5 posições (índices de 0 a 4). Não é possível adicionar um sexto elemento a este array. O tamanho fixo significa que:

1. A memória para armazenar os 5 elementos é alocada de forma contígua
2. Não existe uma operação nativa para adicionar mais elementos
3. Se você precisar de mais espaço, terá que criar um novo array maior e copiar os elementos

Quando tentamos adicionar um elemento além do tamanho definido, ocorre uma exceção `IndexOutOfRangeException`:

```csharp
numeros[^5] = 60; // Erro! O índice 5 não existe neste array
```

Para "adicionar" mais elementos, você precisaria criar um novo array maior e copiar os elementos:

```csharp
// Método manual para "redimensionar"
int[] arrayMaior = new int[^10];
Array.Copy(numeros, arrayMaior, numeros.Length);
numeros = arrayMaior; // numeros agora aponta para o novo array
```

Ou usar o método estático `Resize`:

```csharp
Array.Resize(ref numeros, 10); // Cria um novo array e transfere a referência
```

Mas é importante entender que o método `Resize` não está realmente redimensionando o array original - ele está criando um novo array e copiando os elementos, o que pode ser ineficiente para operações frequentes.

## List<T> e seu tamanho dinâmico

Em contraste, a classe `List&lt;T&gt;` foi projetada especificamente para ter tamanho dinâmico:

```csharp
List&lt;int&gt; numeros = new List&lt;int&gt;(); // Lista vazia
// Ou inicializada com valores
List&lt;int&gt; numeros = new List&lt;int&gt; { 10, 20, 30, 40, 50 };
```

Com uma `List&lt;T&gt;`, você pode:

1. Adicionar elementos facilmente:

```csharp
numeros.Add(60); // Adiciona um elemento ao final
numeros.Insert(2, 25); // Insere o valor 25 na posição 2
```

2. Remover elementos:

```csharp
numeros.Remove(30); // Remove o valor 30
numeros.RemoveAt(1); // Remove o elemento na posição 1
```

3. A lista gerencia automaticamente seu tamanho interno, alocando mais espaço quando necessário

## Principais diferenças entre Array e List<T>

| Característica | Array | List<T> |
| :-- | :-- | :-- |
| Tamanho | Fixo na criação | Dinâmico, cresce conforme necessário |
| Performance | Acesso mais rápido aos elementos | Ligeiramente mais lento no acesso |
| Memória | Uso mais eficiente | Pode alocar mais espaço que o necessário |
| Métodos | Limitados | Diversos métodos para manipulação |
| Flexibilidade | Baixa | Alta |

## Quando usar cada um

**Use Arrays quando:**

- O número de elementos é conhecido e não mudará
- Performance é crítica, especialmente para grandes volumes de dados
- Você precisa de acesso direto por índice com máxima eficiência

**Use List<T> quando:**

- O número de elementos pode variar durante a execução
- Você precisa adicionar ou remover elementos frequentemente
- A flexibilidade é mais importante que a performance absoluta

Na prática, a maioria dos desenvolvedores C\# prefere usar `List&lt;T&gt;` para a maioria dos cenários, devido à sua flexibilidade e facilidade de uso, recorrendo a arrays apenas em situações específicas onde a performance é crítica ou quando estão trabalhando com APIs que exigem arrays.