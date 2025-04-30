# Tipos de Classes
## Classes e Membros Estáticos (static)

### O que são

Um membro **não estático** (o padrão) pertence a uma instância específica (objeto) da classe. Cada objeto tem sua própria cópia desses dados (campos não estáticos) e os métodos não estáticos operam no estado desse objeto específico (usando this).

Um membro **estático** (static) pertence à própria classe, não a qualquer instância individual. Existe apenas uma cópia de um campo estático, compartilhada por todas as instâncias (se houver) da classe. Métodos estáticos são chamados diretamente na classe e não têm acesso à palavra-chave this (pois não operam em uma instância específica).
> [!NOTE]
> Uma **classe estática** (static class) é uma classe que só pode conter membros estáticos e não pode ser instanciada (não se pode criar objetos dela com new) nem herdada.

### Por que usar
### Métodos Utilitários 
 Ideal para agrupar funções que não dependem do estado de um objeto específico (ex: métodos matemáticos em System.Math, métodos de manipulação de string, conversões).
### Estado Compartilhado 

Para manter dados que devem ser comuns a todas as instâncias de uma classe (ex: um contador de quantas instâncias foram criadas, embora haja padrões melhores como Singleton para gerenciamento de instância única). Use com cuidado para evitar problemas de concorrência em ambientes multi-thread.
- **Constantes e Fábricas:** Definir constantes (const ou static readonly) ou métodos fábrica (static FactoryMethod()) que criam instâncias da classe.
            
- **Classes Utilitárias Puras:** Classes estáticas são perfeitas quando você só quer um "container" para métodos utilitários, sem a necessidade ou possibilidade de criar instâncias (ex: File, Directory, Math).
            

## Classes Seladas (sealed)

### O que são
Uma classe declarada com a palavra-chave sealed não pode ser usada como classe base para outras classes. Ou seja, nenhuma outra classe pode herdar dela.

### Por que usar

- **Prevenir Extensão Indesejada:** Quando o design da sua classe está "completo" e você quer garantir que seu comportamento não seja modificado ou estendido de maneira imprevista por subclasses.

- **Segurança:** Pode impedir que código malicioso herde de uma classe sua para obter acesso a membros protegidos ou para alterar seu comportamento.

- **Otimização de Performance (Pequena):** O compilador e o runtime sabem que não precisam procurar por métodos sobrescritos em subclasses (porque não podem existir), permitindo algumas otimizações de chamadas de método (devirtualização). A classe string em C# é um exemplo clássico de classe selada.

## Classes Parciais (partial)
## O que são

Permitem que a definição de uma única classe, struct, interface ou método seja dividida em dois ou mais arquivos de código-fonte (.cs). O compilador junta todas as partes para formar a definição completa antes da compilação.

## Por que usar

- **Trabalho com Código Gerado Automaticamente:** Muito comum em frameworks de UI (Windows Forms, WPF, ASP.NET). Uma parte da classe é gerada pelo designer visual (controles de UI, etc.), e a outra parte é onde você escreve sua lógica de negócio, evitando misturar os dois e prevenindo que seu código seja sobrescrito pelo gerador.

- **Organização de Classes Grandes:** Se uma classe se torna muito extensa, dividi-la em partes lógicas (cada uma em seu arquivo partial class NomeDaClasse_ParteLogica.cs) pode melhorar a organização e a manutenibilidade, embora classes muito grandes possam ser um sinal de que a classe tem responsabilidades demais (violação do Princípio da Responsabilidade Única - SRP).

- **Colaboração em Equipe:** Permite que diferentes desenvolvedores trabalhem em partes diferentes da mesma classe simultaneamente com menos conflitos de merge.

## Classes Abstratas (abstract class)

### O que são
Uma classe marcada como abstract não pode ser instanciada diretamente (new MinhaClasseAbstrata() é proibido). Ela serve como uma classe base para outras classes. Pode conter tanto membros abstratos (métodos, propriedades, eventos, indexadores sem implementação, marcados com abstract) quanto membros concretos (com implementação, que podem ser virtual para permitir sobrescrita ou não).
        
### Por que usar

-  **Definir um Modelo Base Comum:** Quando você quer fornecer uma implementação padrão ou comportamento comum para um grupo de classes relacionadas, mas também quer forçar que certas partes sejam implementadas especificamente por cada subclasse.

-  **Reutilização de Código + Contrato Parcial:** Permite reutilizar a lógica implementada na classe abstrata e, ao mesmo tempo, define um contrato (através dos membros abstratos) que as subclasses devem seguir.

-  **Relação "É UM(A)":** Modela uma hierarquia de herança clara onde as subclasses são um tipo da classe abstrata (um Retangulo é uma FormaGeometrica). Uma classe só pode herdar de uma classe abstrata (ou concreta).

# **Interfaces (interface)**

Uma interface define um **contrato** puramente abstrato. Ela especifica um conjunto de membros públicos (métodos, propriedades, eventos, indexadores) que uma classe ou struct deve implementar se "assinar" essa interface. Tradicionalmente (antes do C# 8), interfaces não continham nenhuma implementação. C# 8+ permite implementações padrão, mas o uso primário ainda é definir contratos. Interfaces não podem ser instanciadas.
## Por que usar

- **Definir Capacidades ("PODE FAZER"):** Enquanto a herança de classe define o que um objeto é, interfaces definem o que um objeto pode fazer. Uma classe pode implementar múltiplas interfaces, permitindo modelar diferentes capacidades ortogonais (ex: uma Ave pode implementar IVoador e IOviparo). Isso contorna a limitação de herança única de classes em C#.

- **Desacoplamento Total:** Código que depende de interfaces só conhece o contrato, não a implementação concreta. Isso promove baixo acoplamento, facilitando a substituição de implementações (ex: usar um DatabaseLogger ou FileLogger onde um ILogger é esperado) e testes unitários (usando Mocks).

- **Polimorfismo:** Permite tratar objetos de classes completamente diferentes de forma uniforme, desde que implementem a mesma interface.

 - **Princípio de Inversão de Dependência (DIP):** Fundamental para arquiteturas limpas e testáveis, onde módulos de alto nível não dependem de módulos de baixo nível, mas ambos dependem de abstrações (interfaces).
            
## Abstract Class vs. Interface (Quando usar qual?)

 - Use **Abstract Class** quando:
	-  Você quer fornecer implementação comum ou estado base para subclasses.
		
	- Você está modelando uma relação forte de "é um(a)" e prevê que a hierarquia não mudará drasticamente.
		
	- Você precisa de membros não-públicos (protected, internal, private).
		
	- Você precisa adicionar novos membros com implementação no futuro sem quebrar todas as classes que a herdam.

- Use **Interface** quando:
	- Você quer definir uma capacidade que pode ser compartilhada por classes de hierarquias completamente diferentes.
		
	- Você quer forçar um contrato estrito sem fornecer nenhuma implementação (visão tradicional).
		
	- Você precisa de "herança múltipla" de tipos (uma classe implementa várias interfaces).
		
	- Você está projetando para baixo acoplamento e alta testabilidade (Inversão de Dependência).


## Exemplo

``` csharp
using System;
using System.Collections.Generic;
using System.IO; // Para exemplo de interface

// --- 1. Estáticos ---
public static class MathUtils // Classe estática: só membros estáticos, não pode instanciar
{
	public static readonly double PI = 3.14159; // Campo estático constante (ou quase)

	public static int Somar(int a, int b) // Método estático
	{
		return a + b;
	}
}

public class ContadorInstancias
{
	private static int _contadorGlobal = 0; // Campo estático compartilhado
	public int IdInstancia { get; private set; } // Campo de instância

	public ContadorInstancias()
	{
		_contadorGlobal++;
		IdInstancia = _contadorGlobal; // Cada instância tem seu ID
	}

	public static int ObterTotalInstancias() // Método estático para acessar dado estático
	{
		return _contadorGlobal;
	}
}

// --- 2. Seladas ---
public class VeiculoBase
{
	public virtual void Ligar() { Console.WriteLine("Veículo base ligado."); }
}

public sealed class Carro : VeiculoBase // Carro é selado, não pode ser herdado
{
	public override void Ligar() { Console.WriteLine("Carro ligado."); }
	public void AbrirPortaMalas() { Console.WriteLine("Porta-malas aberto."); }
}

// public class CarroDeCorrida : Carro { } // ERRO DE COMPILAÇÃO: 'CarroDeCorrida': cannot derive from sealed type 'Carro'

// --- 3. Parciais (Exemplo Conceitual) ---
// Arquivo: Pessoa_Dados.cs
public partial class Pessoa
{
	public string Nome { get; set; }
	public int Idade { get; set; }
}

// Arquivo: Pessoa_Metodos.cs
public partial class Pessoa
{
	public void Apresentar()
	{
		Console.WriteLine($"Olá, meu nome é {Nome} e tenho {Idade} anos.");
	}
}
// O compilador junta as duas partes em uma única classe Pessoa

// --- 4. Sobrecarga ---
public class Calculadora
{
	public int Somar(int a, int b) // Sobrecarga 1
	{
		Console.WriteLine("Somando 2 inteiros");
		return a + b;
	}

	public double Somar(double a, double b) // Sobrecarga 2 (mesmo nome, tipos diferentes)
	{
		Console.WriteLine("Somando 2 doubles");
		return a + b;
	}

	public int Somar(int a, int b, int c) // Sobrecarga 3 (mesmo nome, quantidade diferente)
	{
		Console.WriteLine("Somando 3 inteiros");
		return a + b + c;
	}
	// public string Somar(int a, int b) // ERRO: Já existe método com essa assinatura (tipo de retorno não conta)
}

// --- 5. Classe Abstrata ---
public abstract class Animal
{
	public string Nome { get; protected set; } // Propriedade com implementação

	protected Animal(string nome) { Nome = nome; } // Construtor (será chamado por subclasses)

	public abstract void EmitirSom(); // Método abstrato - FORÇA implementação na subclasse

	public virtual void Dormir() // Método virtual - fornece implementação, permite override
	{
		Console.WriteLine($"{Nome} está dormindo (Zzzzz)...");
	}
}

public class Cachorro : Animal
{
	public Cachorro(string nome) : base(nome) { } // Chama construtor base

	public override void EmitirSom() // Implementação OBRIGATÓRIA do abstrato
	{
		Console.WriteLine($"{Nome} diz: Au au!");
	}

	public override void Dormir() // Sobrescrita OPCIONAL do virtual
	{
		 Console.WriteLine($"{Nome} está dormindo como um anjo (ou roncando).");
	}
}

// --- 6. Interface ---
public interface ILogger // Define o contrato: "Quem implementar DEVE ter um método Log"
{
	void Log(string message); // Assinatura do método, sem implementação
	// string LogLevel { get; set; } // Interfaces também podem definir propriedades, eventos, etc.
}

// Classe que implementa a interface
public class ConsoleLogger : ILogger
{
	public void Log(string message) // Implementação OBRIGATÓRIA do método da interface
	{
		Console.WriteLine($"[CONSOLE LOG]: {message}");
	}
}

// Outra classe, completamente diferente, que também implementa a interface
public class FileLogger : ILogger
{
	private readonly string _filePath;
	public FileLogger(string filePath) { _filePath = filePath; }

	public void Log(string message) // Implementação OBRIGATÓRIA
	{
		try
		{
			 // AppendAllText é estático na classe File
			File.AppendAllText(_filePath, $"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] {message}{Environment.NewLine}");
		}
		catch (Exception ex)
		{
			Console.WriteLine($"Erro ao logar em arquivo {_filePath}: {ex.Message}");
		}
	}
}

// Serviço que DEPENDE da abstração (Interface), não da implementação concreta
public class ProcessadorDeTarefas
{
	private readonly ILogger _logger; // Depende da INTERFACE

	// Injeção de Dependência: Recebe a implementação concreta no construtor
	public ProcessadorDeTarefas(ILogger logger)
	{
		_logger = logger ?? throw new ArgumentNullException(nameof(logger));
	}

	public void Processar(string tarefa)
	{
		_logger.Log($"Iniciando tarefa: {tarefa}");
		// ... lógica da tarefa ...
		System.Threading.Thread.Sleep(500); // Simula trabalho
		_logger.Log($"Tarefa concluída: {tarefa}");
	}
}


// --- Programa Principal para Demonstração ---
public class Program
{
	public static void Main(string[] args)
	{
		Console.WriteLine("--- Estáticos ---");
		Console.WriteLine($"PI: {MathUtils.PI}"); // Chamada direta na classe estática
		Console.WriteLine($"Soma estática: {MathUtils.Somar(5, 3)}");

		ContadorInstancias c1 = new ContadorInstancias();
		ContadorInstancias c2 = new ContadorInstancias();
		Console.WriteLine($"ID c1: {c1.IdInstancia}, ID c2: {c2.IdInstancia}");
		Console.WriteLine($"Total de instâncias: {ContadorInstancias.ObterTotalInstancias()}"); // Chamada método estático

		Console.WriteLine("\n--- Seladas ---");
		Carro meuCarro = new Carro();
		meuCarro.Ligar();
		meuCarro.AbrirPortaMalas();

		Console.WriteLine("\n--- Parciais ---");
		Pessoa p1 = new Pessoa { Nome = "Ana", Idade = 30 }; // As partes foram combinadas
		p1.Apresentar();

		Console.WriteLine("\n--- Sobrecarga ---");
		Calculadora calc = new Calculadora();
		Console.WriteLine(calc.Somar(10, 20));
		Console.WriteLine(calc.Somar(10.5, 20.3));
		Console.WriteLine(calc.Somar(10, 20, 30));

		Console.WriteLine("\n--- Classe Abstrata e Interface ---");
		Animal meuCachorro = new Cachorro("Rex"); // Objeto da classe concreta
		meuCachorro.EmitirSom();
		meuCachorro.Dormir();
		// Animal a = new Animal("Generico"); // ERRO DE COMPILAÇÃO: Cannot create an instance of the abstract type or interface 'Animal'

		Console.WriteLine("\n--- Interfaces e Injeção de Dependência ---");
		ILogger loggerConsole = new ConsoleLogger(); // Instância concreta
		ILogger loggerArquivo = new FileLogger("meu_log.txt"); // Outra instância concreta

		// Processador usa o ConsoleLogger
		ProcessadorDeTarefas proc1 = new ProcessadorDeTarefas(loggerConsole);
		proc1.Processar("Backup de Dados");

		// Processador usa o FileLogger (sem alterar ProcessadorDeTarefas!)
		ProcessadorDeTarefas proc2 = new ProcessadorDeTarefas(loggerArquivo);
		proc2.Processar("Envio de Email Marketing");

		Console.WriteLine("\nVerifique o arquivo 'meu_log.txt' se foi criado.");

		Console.WriteLine("\nPressione qualquer tecla para sair...");
		Console.ReadKey();
	}
}
```

### 4. Exercícios Práticos:

1. **Static Helper:** Crie uma classe estática ArrayUtils com métodos estáticos como EncontrarMaior(int[] array), CalcularMedia(double[] array) e Inverter(object[] array). Teste esses métodos chamando-os diretamente na classe.
    
2. **Sealed Model:** Crie uma classe base Pagamento com um método virtual Processar(). Crie uma subclasse PagamentoCartaoCredito que herda de Pagamento. Agora, marque PagamentoCartaoCredito como sealed. Tente criar uma outra classe PagamentoCartaoEmpresarial que herde de PagamentoCartaoCredito e observe o erro do compilador. Explique por que selar PagamentoCartaoCredito pode ser útil.
    
3. **Partial Refactoring:** Pegue uma classe relativamente grande que você tenha escrito (ou crie uma Cliente com muitas propriedades e métodos - Nome, Endereco, HistoricoPedidos, Validar(), Salvar(), Carregar(), etc.) e divida sua definição em dois arquivos usando partial: um para propriedades e outro para métodos.
    
4. **Overload Design:** Crie uma classe Mensagem com métodos sobrecarregados Enviar(). Uma sobrecarga deve aceitar um string (mensagem de texto simples). Outra deve aceitar um string (destinatário) e um string (assunto). Uma terceira pode aceitar um object (payload complexo) e um enum Prioridade.
    
5. **Abstract vs. Interface:** Modele um sistema de diferentes tipos de armazenamento de dados (DatabaseStorage, FileStorage, CloudStorage).
    
    - Primeiro, tente usar uma **classe abstrata** DataStorageBase com um método concreto GetStatus() e métodos abstratos Save(object data) e Load(string id).
        
    - Depois, refaça usando uma **interface** IDataStorage com os métodos Save e Load. Crie as classes concretas implementando a interface.
        
    - Compare as duas abordagens: Qual é mais flexível? Qual permite compartilhar código base? Em qual cenário cada uma seria melhor?
        
