# Programação Orientada a Objetos (OOP - Object-Oriented Programming)

## O que é
É um **paradigma de programação** baseado no conceito de "objetos", que podem conter dados na forma de campos (frequentemente conhecidos como atributos ou propriedades) e código, na forma de procedimentos (frequentemente conhecidos como métodos).

## Por que existe:
Surgiu para lidar com a crescente complexidade dos softwares. Paradigmas anteriores, como o procedural (baseado em funções ou procedimentos), podem se tornar difíceis de gerenciar em projetos grandes, pois dados e funções que operam sobre esses dados são geralmente separados. A OOP busca **organizar o código de forma mais intuitiva**, agrupando dados e comportamentos relacionados em unidades lógicas chamadas "objetos", espelhando melhor as entidades do mundo real ou do domínio do problema. Isso promove **reutilização de código, manutenibilidade, flexibilidade e escalabilidade**.

## Classes
Uma classe é um **molde**, um **plano** ou uma **especificação** para criar objetos. Ela define um conjunto de atributos (dados) e métodos (comportamentos) que os objetos criados a partir dela terão.

Serve como um **template**. Por exemplo, a classe Carro definiria atributos como cor, marca, velocidadeAtual e métodos como acelerar(), frear(), ligar(). A classe em si não é um carro, mas a ideia de um carro.

## Objeto (ou Instância)
Um objeto é uma **instância concreta** de uma classe. É o "carro de verdade" criado a partir do molde Carro. Cada objeto possui seu próprio estado (valores específicos para seus atributos), mas compartilha a estrutura de atributos e os comportamentos (métodos) definidos pela classe.

### Classe X Objeto
Se Carro é a classe (o molde), então meuFusca (com cor="Azul", marca="VW") e carroDaVizinha (com cor="Vermelho", marca="Fiat") são objetos ou instâncias da classe Carro

## Atributo (Propriedade/Campo)
São as **características** ou **dados** que um objeto de uma determinada classe possui. Representam o **estado** de um objeto.

Armazenar informações sobre o objeto. No exemplo da classe Carro, cor, marca, modelo, velocidadeAtual seriam atributos. Cada objeto Carro terá seus próprios valores para esses atributos (meu Fusca é azul, o carro da vizinha é vermelho).

## Métodos (Função/Comportamento)
São as **ações** ou **operações** que um objeto de uma determinada classe pode realizar. Representam o **comportamento** de um objeto.

Definir o que o objeto pode fazer, muitas vezes alterando seu próprio estado (seus atributos) ou interagindo com outros objetos. Na classe Carro, acelerar(), frear(), ligar(), obterVelocidadeAtual() seriam métodos. O método acelerar(), por exemplo, poderia modificar o valor do atributo velocidadeAtual.

# Pilares da OOP

## Abstração

É o processo de **identificar as características essenciais de um objeto e ignorar os detalhes irrelevantes ou complexos**. Foca no **"o quê"** um objeto faz, em vez de **"como"** ele faz. Ela expõe apenas as funcionalidades relevantes para o contexto, ocultando a complexidade da implementação interna.

### Importância
- **Simplificação:** Torna os objetos mais fáceis de entender e usar, apresentando uma visão simplificada. Pense no controle remoto da TV: você sabe que o botão "Power" liga/desliga, sem precisar saber como o circuito interno funciona.
    
- **Foco no Essencial:** Ajuda a modelar o domínio do problema de forma mais eficaz, concentrando-se nos aspectos importantes para a aplicação.
    
- **Gerenciamento de Complexidade:** Permite construir sistemas complexos dividindo-os em componentes menores e mais gerenciáveis, cada um com uma interface abstrata clara.
    
- **Redução do Impacto de Mudanças:** Mudanças na implementação interna (o "como") não afetam quem usa a abstração (o "quê"), desde que a interface abstrata seja mantida.

## Encapsulamento

É o princípio de **agrupar os dados (atributos) e os métodos (comportamentos) que operam nesses dados dentro de uma única unidade, a classe**. Além disso, o encapsulamento envolve **restringir o acesso direto ao estado interno (atributos) do objeto**, expondo apenas funcionalidades controladas (através de métodos públicos).

### Importância
- **Proteção de Dados:** Evita que o estado interno do objeto seja modificado de forma inesperada ou inválida por código externo, garantindo a **integridade dos dados**. (Ex: Impedir que a velocidadeAtual de um Carro seja definida como um valor negativo diretamente).

- **Controle de Acesso:** A classe decide o que é exposto (público) e o que é mantido interno (privado/protegido). Isso define uma "interface" clara para interagir com o objeto.
    
- **Redução de Complexidade:** Usuários da classe não precisam conhecer os detalhes internos de implementação, apenas como usar sua interface pública (seus métodos).
    
- **Manutenibilidade e Flexibilidade:** A implementação interna pode ser alterada sem afetar o código que usa a classe, desde que a interface pública permaneça a mesma. Isso facilita a evolução do software.


## Herança

É um mecanismo que permite que uma nova classe (chamada **subclasse** ou classe derivada) **herde atributos e métodos de uma classe existente** (chamada **superclasse** ou classe base). Isso estabelece uma relação **"é um(a)"** (Ex: um Cachorro é um Animal).

### Importância
- **Reutilização de Código:** Evita a duplicação de código. Atributos e métodos comuns definidos na superclasse estão automaticamente disponíveis nas subclasses.
    
- **Organização e Hierarquia:** Permite criar hierarquias de classes que refletem relacionamentos do mundo real, tornando o modelo de domínio mais claro e organizado (Ex: Animal -> Mamifero -> Cachorro).
    
- **Polimorfismo (Facilita):** A herança é uma das maneiras de implementar o polimorfismo (veremos a seguir).
    
- **Extensibilidade:** Subclasses podem adicionar novos atributos e métodos específicos, ou **sobrescrever** (modificar) métodos herdados para fornecer um comportamento especializado.

## Polimorfismo

 Significa "muitas formas". Na OOP, é a capacidade de **objetos de diferentes classes responderem à mesma mensagem (chamada de método) de maneiras diferentes e específicas**. Geralmente, isso é alcançado através da herança (sobrescrita de métodos) ou de interfaces/classes abstratas.

### Importância

- **Flexibilidade e Extensibilidade:** Permite tratar objetos de diferentes subclasses de forma uniforme através da interface da superclasse. Você pode adicionar novas subclasses que respondem à mesma mensagem sem modificar o código que as utiliza. (Ex: Uma função fazerBarulho(animal) pode funcionar com um Cachorro ou um Gato, e cada um fará seu próprio som).
    
- **Desacoplamento:** O código que utiliza os objetos polimórficos não precisa conhecer o tipo específico do objeto, apenas a interface comum (o método que está sendo chamado). Isso torna o sistema mais flexível a mudanças e adições.
    
- **Código Mais Genérico e Reutilizável:** Permite escrever código que opera sobre uma abstração (superclasse ou interface) em vez de tipos concretos, tornando-o aplicável a uma gama maior de objetos.


# Exemplo de aplicação do POO

```csharp
using System;
using System.Collections.Generic; // Necessário para usar List<T>
using System.Globalization; // Para formatação de número (opcional)

// --- Abstração ---
// Classe Abstrata: Define um contrato ("o quê"), não pode ser instanciada.
// Contém membros abstratos (sem implementação) e/ou virtuais (com implementação padrão, podem ser sobrescritos).
public abstract class FormaGeometrica
{
	// Atributo protegido (acessível na classe e subclasses)
	// Encapsulamento: Usaremos uma Property pública para acesso controlado.
	private string _nome;

	// Encapsulamento: Property pública para acessar o nome (somente leitura pública)
	public string Nome
	{
		get { return _nome; }
		// private set { _nome = value; } // Permitiria alteração apenas dentro da classe
	}

	// Construtor da classe base
	protected FormaGeometrica(string nome)
	{
		// Validação básica poderia ser adicionada aqui
		if (string.IsNullOrWhiteSpace(nome))
		{
			throw new ArgumentException("Nome não pode ser vazio.", nameof(nome));
		}
		this._nome = nome;
	}

	// Método Abstrato: DEVE ser implementado ("override") pelas subclasses concretas.
	// Define o "o quê" (toda forma DEVE poder calcular sua área), mas não o "como".
	public abstract double CalcularArea();

	// Método Virtual: Possui uma implementação padrão, MAS PODE ser sobrescrito ("override")
	// pelas subclasses se um comportamento diferente for necessário.
	public virtual void Descrever()
	{
		Console.WriteLine($"Eu sou uma forma geométrica chamada '{Nome}'.");
	}
}

// --- Herança e Implementação da Abstração ---

// Subclasse Retangulo HERDA de FormaGeometrica
public class Retangulo : FormaGeometrica
{
	// Atributos privados específicos de Retangulo (Encapsulamento)
	private double _largura;
	private double _altura;

	// Encapsulamento: Properties públicas para acesso controlado com validação
	public double Largura
	{
		get { return _largura; }
		set
		{
			if (value <= 0)
			{
				// Lançar exceção é mais robusto que imprimir erro em cenários reais
				throw new ArgumentOutOfRangeException(nameof(Largura), "Largura deve ser positiva.");
			}
			_largura = value;
		}
	}

	public double Altura
	{
		get { return _altura; }
		set
		{
			if (value <= 0)
			{
				throw new ArgumentOutOfRangeException(nameof(Altura), "Altura deve ser positiva.");
			}
			_altura = value;
		}
	}

	// Construtor de Retangulo
	// ": base(nome)" chama o construtor da superclasse (FormaGeometrica)
	public Retangulo(string nome, double largura, double altura) : base(nome)
	{
		// Usamos as properties no construtor para garantir que a validação seja executada
		Largura = largura;
		Altura = altura;
	}

	// Implementação OBRIGATÓRIA do método abstrato da superclasse
	// Usa a palavra-chave "override"
	public override double CalcularArea()
	{
		// Acessa os dados através das properties (encapsulamento)
		return Largura * Altura;
	}

	// Sobrescrita OPCIONAL do método virtual da superclasse (Polimorfismo)
	// Usa a palavra-chave "override"
	public override void Descrever()
	{
		// base.Descrever(); // Opcional: Chamar a implementação original da classe base
		Console.WriteLine($"Eu sou o retângulo '{Nome}' com largura {Largura} e altura {Altura}.");
	}
}

// Subclasse Circulo HERDA de FormaGeometrica
public class Circulo : FormaGeometrica
{
	// Atributo privado (Encapsulamento)
	private double _raio;

	// Property pública com validação (Encapsulamento)
	public double Raio
	{
		get { return _raio; }
		set
		{
			if (value <= 0)
			{
				throw new ArgumentOutOfRangeException(nameof(Raio), "Raio deve ser positivo.");
			}
			_raio = value;
		}
	}

	// Construtor de Circulo, chamando o construtor base
	public Circulo(string nome, double raio) : base(nome)
	{
		Raio = raio; // Usa a property para validação
	}

	// Implementação OBRIGATÓRIA do método abstrato
	public override double CalcularArea()
	{
		return Math.PI * Raio * Raio; // ou Math.Pow(Raio, 2)
	}

	// Sobrescrita OPCIONAL do método virtual (Polimorfismo)
	public override void Descrever()
	{
		Console.WriteLine($"Eu sou o círculo '{Nome}' com raio {Raio}.");
	}
}

 // Subclasse Triangulo HERDA de FormaGeometrica
public class Triangulo : FormaGeometrica
{
	private double _baseTriangulo;
	private double _alturaTriangulo;

	public double BaseTriangulo
	{
		get { return _baseTriangulo; }
		set
		{
			if (value <= 0) throw new ArgumentOutOfRangeException(nameof(BaseTriangulo), "Base deve ser positiva.");
			_baseTriangulo = value;
		}
	}

	public double AlturaTriangulo
	{
		get { return _alturaTriangulo; }
		 set
		{
			if (value <= 0) throw new ArgumentOutOfRangeException(nameof(AlturaTriangulo), "Altura deve ser positiva.");
			_alturaTriangulo = value;
		}
	}

	 public Triangulo(string nome, double baseTriangulo, double alturaTriangulo) : base(nome)
	 {
		 BaseTriangulo = baseTriangulo;
		 AlturaTriangulo = alturaTriangulo;
	 }

	// Implementação OBRIGATÓRIA do método abstrato
	public override double CalcularArea()
	{
		return (BaseTriangulo * AlturaTriangulo) / 2.0;
	}

	// Sobrescrita OPCIONAL do método virtual (Polimorfismo)
	public override void Descrever()
	{
		Console.WriteLine($"Eu sou o triângulo '{Nome}' com base {BaseTriangulo} e altura {AlturaTriangulo}.");
	}
}

// Classe principal para execução do programa
public class Program
{
	// --- Demonstração do Polimorfismo ---
	// Este método funciona com QUALQUER objeto que SEJA UMA FormaGeometrica
	// Não importa se é Retangulo, Circulo ou outra subclasse futura.
	public static void ImprimirDetalhesForma(FormaGeometrica forma)
	{
		forma.Descrever(); // Chama o método Descrever() específico do objeto real
		// :F2 formata o número para duas casas decimais
		Console.WriteLine($"Área: {forma.CalcularArea():F2}");
		Console.WriteLine("---");
	}

	// Ponto de entrada do programa
	public static void Main(string[] args)
	{
		 // Define a cultura para usar "." como separador decimal (opcional, mas bom para consistência)
		CultureInfo.CurrentCulture = CultureInfo.InvariantCulture;

		try // Boa prática: tratar possíveis exceções (ex: valores inválidos no construtor)
		{
			// Instanciando objetos
			// Note que usamos o tipo base (FormaGeometrica) para as variáveis,
			// o que facilita o polimorfismo.
			FormaGeometrica retanguloA = new Retangulo("Ret. A", 10.0, 5.0);
			FormaGeometrica circuloB = new Circulo("Circ. B", 7.0);
			FormaGeometrica trianguloC = new Triangulo("Tri. C", 6.0, 8.0);

			// Tentativa de instanciar classe abstrata (Geraria erro de compilação, como esperado)
			// FormaGeometrica formaGenerica = new FormaGeometrica("Genérica"); // Erro CS0144

			// Tentativa de definir valor inválido (lançaria ArgumentOutOfRangeException)
			// retanguloA.Largura = -5; // Erro em tempo de execução se não tratado

			// Criando uma lista de formas (usando o tipo base)
			List<FormaGeometrica> formas = new List<FormaGeometrica>();
			formas.Add(retanguloA);
			formas.Add(circuloB);
			formas.Add(trianguloC);

			// --- Polimorfismo em ação ---
			// O mesmo método ImprimirDetalhesForma se comporta diferente
			// dependendo do tipo REAL do objeto na lista (Retangulo, Circulo, Triangulo).
			Console.WriteLine("Demonstrando Polimorfismo:");
			foreach (FormaGeometrica f in formas)
			{
				ImprimirDetalhesForma(f); // A mágica do polimorfismo acontece aqui!
			}
		}
		catch (Exception ex)
		{
			 Console.ForegroundColor = ConsoleColor.Red;
			 Console.WriteLine($"\nOcorreu um erro: {ex.Message}");
			 Console.ResetColor();
		}

		Console.WriteLine("\nPressione qualquer tecla para sair...");
		Console.ReadKey();
	}
}
```
**Análise do Exemplo C#:**

1. **Encapsulamento:**
    
    - Os campos (_nome, _largura, _altura, _raio, etc.) são declarados como private, impedindo o acesso direto de fora da classe.
        
    - O acesso aos dados é feito através de Properties públicas (Nome, Largura, Altura, Raio). As properties Largura, Altura, Raio incluem lógica de validação nos seus set accessors, garantindo a integridade dos dados (valores devem ser positivos). A property Nome tem um get público, mas nenhum set público (ou um private set), tornando-a efetivamente somente leitura de fora da classe após a inicialização no construtor.
        
    - **O porquê:** Protege o estado interno do objeto, controla como os dados são modificados e expõe uma interface clara e segura.
        
2. **Abstração:**
    
    - FormaGeometrica é declarada como public abstract class. Isso significa que ela serve como um modelo, mas não pode ser instanciada diretamente.
        
    - O método public abstract double CalcularArea(); não tem implementação na classe base. Ele define um contrato: qualquer classe concreta que herdar de FormaGeometrica deve fornecer sua própria implementação de CalcularArea. Foca no "o quê" (calcular área) e não no "como".
        
    - O método Descrever() é virtual, oferecendo uma implementação padrão, mas permitindo que subclasses a substituam (override).
        
    - **O porquê:** Simplifica a complexidade ao definir um contrato essencial sem se prender aos detalhes de implementação, permitindo tratar diferentes formas de maneira uniforme (polimorfismo).
        
3. **Herança:**
    
    - As classes Retangulo, Circulo e Triangulo usam a sintaxe : FormaGeometrica para indicar que herdam da classe base FormaGeometrica.
        
    - Elas herdam a property Nome e o método Descrever() (que podem sobrescrever).
        
    - O construtor de cada subclasse usa : base(nome) para chamar o construtor da classe FormaGeometrica, garantindo que a inicialização da parte herdada seja feita corretamente.
        
    - **O porquê:** Reutiliza código (o Nome e sua lógica básica), estabelece uma relação "é um(a)" (um Retangulo é uma FormaGeometrica) e organiza o código em uma hierarquia lógica.
        
4. **Polimorfismo:**
    
    - A função ImprimirDetalhesForma(FormaGeometrica forma) aceita qualquer objeto cujo tipo seja FormaGeometrica ou uma de suas subclasses.
        
    - Dentro do loop foreach, quando f.Descrever() ou f.CalcularArea() é chamado, o C# determina em tempo de execução qual é o tipo real do objeto referenciado por f (Retangulo, Circulo ou Triangulo) e chama a versão sobrescrita (override) correspondente daquele método. A mesma chamada de método resulta em comportamentos diferentes dependendo do objeto.
        
    - **O porquê:** Permite escrever código mais genérico e flexível. A função ImprimirDetalhesForma não precisa saber sobre Retangulo ou Circulo especificamente; ela funciona com qualquer FormaGeometrica atual ou futura, desde que cumpra o contrato (implemente CalcularArea e, opcionalmente, sobrescreva Descrever).