# Delegates

Um delegate é essencialmente um **<font color="#00b0f0">ponteiro para um método seguro em termos de tipo</font>** (type-safe function pointer). Ele define uma "assinatura" de método (tipo de retorno e tipos de parâmetros). **<span style="background:#fff88f">Qualquer método que corresponda a essa assinatura pode ser atribuído a uma variável desse tipo de delegate</span>** e, posteriormente, invocado.

## Principal função

- Permitir que métodos sejam passados como parâmetros para outros métodos.

- Implementar callbacks (chamar um método de volta quando uma determinada ação ocorre).

- Servir como base para o mecanismo de eventos.

## Quando utilizar

- Você precisa executar um método que <span style="background:#fff88f">só será conhecido em tempo de execução</span>.

- Você quer que <span style="background:#fff88f">uma classe chame um método de outra classe sem ter uma dependência direta dela</span> (callback).

- Em LINQ, muitos métodos de extensão (como Where, Select) aceitam delegates (geralmente na forma de expressões lambda) para definir a lógica de filtragem ou projeção.

    ```Csharp
// 1. Declarar o delegate (define a assinatura)
public delegate void MeuDelegateSimples(string mensagem);
public delegate int DelegateComRetorno(int a, int b);

public class UsandoDelegates
{
    public void MetodoParaDelegate(string msg)
    {
        Console.WriteLine($"Mensagem recebida: {msg}");
    }

    public int Somar(int x, int y)
    {
        return x + y;
    }

    public void Executar()
    {
        // 2. Instanciar o delegate e apontar para um método
        MeuDelegateSimples del1 = MetodoParaDelegate;
        // ou
        MeuDelegateSimples del2 = new MeuDelegateSimples(MetodoParaDelegate);
        // ou com expressão lambda
        MeuDelegateSimples del3 = (texto) => Console.WriteLine($"Lambda diz: {texto}");

        // 3. Invocar o delegate (que executa o método apontado)
        del1("Olá do delegate!"); // Saída: Mensagem recebida: Olá do delegate!
        del2.Invoke("Outra forma de invocar!"); // Saída: Mensagem recebida: Outra forma de invocar!
        del3("Teste"); // Saída: Lambda diz: Teste

        DelegateComRetorno delSoma = Somar;
        int resultado = delSoma(5, 3);
        Console.WriteLine($"Soma: {resultado}"); // Saída: Soma: 8

        // Delegates podem ser "multicast" - apontar para múltiplos métodos
        MeuDelegateSimples delMulticast = MetodoParaDelegate;
        delMulticast += (texto) => Console.WriteLine($"Segundo assinante: {texto}"); // Adiciona outro método

        delMulticast("Multicast!");
        // Saída:
        // Mensagem recebida: Multicast!
        // Segundo assinante: Multicast!
    }
}
    ```

# Eventos
Um event é um mecanismo que permite que uma classe (o **publicador** ou publisher) notifique outras classes (os **assinantes** ou subscribers) quando algo de interesse acontece (o "evento"). Eventos são construídos sobre delegates. <span style="background:#fff88f">Um evento é, na verdade, um delegate encapsulado que só permite que os assinantes se registrem (+=) ou cancelem o registro (-=), e apenas a classe publicadora pode "disparar" (invocar) o evento</span>.

## Principal função

- **Desacoplamento:** O publicador não precisa conhecer os assinantes. Ele apenas dispara o evento, e quem estiver "ouvindo" será notificado. Os assinantes também não precisam se conhecer.

- **Notificação:** Fornecer um mecanismo padronizado para classes sinalizarem que uma ação ocorreu ou que um estado mudou.

- **Extensibilidade:** Novas classes podem se inscrever para receber notificações sem modificar a classe publicadora.

## Quando utilizar

- Uma classe precisa informar outras partes do sistema sobre mudanças de estado ou ações importantes (ex: clique de botão, download concluído, valor alterado, pedido processado).

- Para implementar o padrão de design Observer.

- Em interfaces gráficas (GUI), onde interações do usuário (cliques, movimentos do mouse) disparam eventos.


    ```Csharp
    // --- Definindo o Evento (na classe Publicadora) ---

// 1. (Opcional, mas comum) Definir uma classe para os dados do evento (herda de EventArgs)
public class MensagemEventArgs : EventArgs
{
    public string Mensagem { get; }
    public DateTime Timestamp { get; }

    public MensagemEventArgs(string mensagem)
    {
        Mensagem = mensagem;
        Timestamp = DateTime.Now;
    }
}

public class Publicador
{
    // 2. Definir o delegate para o evento (se não usar um genérico como EventHandler)
    //    Pode usar um delegate existente ou criar um novo.
    //    Convenção: nome do delegate termina com "EventHandler".
    //    Convenção: primeiro parâmetro é 'object sender', segundo é 'EventArgs' ou derivado.
    public delegate void MensagemEnviadaEventHandler(object sender, MensagemEventArgs e);

    // 3. Declarar o evento usando o delegate
    public event MensagemEnviadaEventHandler MensagemEnviada;
    // Alternativa comum: usar o delegate genérico EventHandler<TEventArgs>
    // public event EventHandler<MensagemEventArgs> MensagemEnviadaAlternativa;

    public void EnviarNovaMensagem(string conteudo)
    {
        Console.WriteLine($"Publicador: Preparando para enviar mensagem '{conteudo}'...");
        // 4. Disparar o evento (apenas a classe publicadora pode fazer isso)
        //    É boa prática verificar se há assinantes antes de invocar.
        OnMensagemEnviada(new MensagemEventArgs(conteudo));
    }

    // Método protegido e virtual para disparar o evento (padrão comum)
    protected virtual void OnMensagemEnviada(MensagemEventArgs e)
    {
        // Cria uma cópia temporária para evitar problemas com concorrência (thread safety)
        // se um assinante se desinscrever durante a invocação.
        MensagemEnviadaEventHandler handler = MensagemEnviada;
        handler?.Invoke(this, e); // O operador '?.' verifica se handler é null antes de chamar Invoke

        // Equivalente a:
        // if (handler != null)
        // {
        //    handler(this, e);
        // }
    }
}

// --- Assinando o Evento (na classe Assinante) ---
public class Assinante
{
    private string _nome;

    public Assinante(string nome)
    {
        _nome = nome;
    }

    // 5. Criar um método manipulador de evento (event handler) com a mesma assinatura do delegate
    public void QuandoMensagemForEnviada(object sender, MensagemEventArgs e)
    {
        Console.WriteLine($"Assinante '{_nome}': Recebeu mensagem às {e.Timestamp} de '{sender?.GetType().Name}': {e.Mensagem}");
    }

    public void Inscrever(Publicador p)
    {
        // 6. Inscrever o método manipulador no evento do publicador
        p.MensagemEnviada += QuandoMensagemForEnviada;
        Console.WriteLine($"Assinante '{_nome}': Inscrito para receber mensagens.");
    }

    public void Desinscrever(Publicador p)
    {
        // 7. (Opcional, mas importante para evitar memory leaks) Desinscrever
        p.MensagemEnviada -= QuandoMensagemForEnviada;
        Console.WriteLine($"Assinante '{_nome}': Desinscrito.");
    }
}

// --- Exemplo de Uso ---
public class DemoEventos
{
    public static void Rodar()
    {
        Publicador publicador = new Publicador();
        Assinante assinante1 = new Assinante("Alice");
        Assinante assinante2 = new Assinante("Bob");

        assinante1.Inscrever(publicador);
        assinante2.Inscrever(publicador);

        publicador.EnviarNovaMensagem("Promoção especial!");
        // Saída (ordem dos assinantes não é garantida):
        // Publicador: Preparando para enviar mensagem 'Promoção especial!'...
        // Assinante 'Alice': Inscrito para receber mensagens.
        // Assinante 'Bob': Inscrito para receber mensagens.
        // Assinante 'Alice': Recebeu mensagem às [timestamp] de 'Publicador': Promoção especial!
        // Assinante 'Bob': Recebeu mensagem às [timestamp] de 'Publicador': Promoção especial!

        assinante2.Desinscrever(publicador);
        publicador.EnviarNovaMensagem("Lembrete importante!");
        // Saída:
        // Publicador: Preparando para enviar mensagem 'Lembrete importante!'...
        // Assinante 'Bob': Desinscrito.
        // Assinante 'Alice': Recebeu mensagem às [timestamp] de 'Publicador': Lembrete importante!
    }
}
    ```