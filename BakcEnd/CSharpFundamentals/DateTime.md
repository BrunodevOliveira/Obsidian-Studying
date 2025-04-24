# Trabalhando com Datas em C\#

No desenvolvimento de aplicações, a manipulação de datas é uma tarefa comum e essencial. O C\# oferece diversas classes e métodos para facilitar esse trabalho, permitindo obter, formatar, comparar e manipular datas de maneira eficiente. Vamos explorar os principais aspectos da manipulação de datas no C\#.

## Obtendo Valores da Data

O C\# utiliza principalmente a estrutura `DateTime` para representar datas e horas. Esta estrutura oferece várias propriedades e métodos para extrair informações específicas.

### Propriedades Básicas

```csharp
// Criando uma instância de DateTime
DateTime dataAtual = DateTime.Now;

// Obtendo componentes individuais
int ano = dataAtual.Year;
int mes = dataAtual.Month;
int dia = dataAtual.Day;
int hora = dataAtual.Hour;
int minuto = dataAtual.Minute;
int segundo = dataAtual.Second;
int milissegundo = dataAtual.Millisecond;

// Obtendo o dia da semana
DayOfWeek diaDaSemana = dataAtual.DayOfWeek;
int numeroDiaDaSemana = (int)dataAtual.DayOfWeek; // 0 (Domingo) a 6 (Sábado)

// Obtendo apenas a parte da data (sem a hora)
DateTime apenasData = dataAtual.Date;
```


### Extraindo o Dia da Semana

Para obter o nome do dia da semana, podemos usar o método `ToString()` com formatação específica:

```csharp
DateTime data = new DateTime(2025, 4, 23);

// Nome completo do dia da semana na cultura atual
string nomeDiaSemana = data.ToString("dddd"); // "quarta-feira" (em pt-BR)

// Nome abreviado do dia da semana
string nomeDiaSemanaAbreviado = data.ToString("ddd"); // "qua" (em pt-BR)

// Nome do dia da semana em outra cultura
string nomeDiaSemanaEmEspanhol = data.ToString("dddd", new CultureInfo("es-ES")); // "miércoles"
```


## Formatando Datas

O C\# oferece várias maneiras de formatar datas para exibição, usando especificadores de formato padrão ou personalizados.

### Métodos de Formatação Básicos

```csharp
DateTime data = new DateTime(2025, 4, 23, 14, 30, 45);

// Formatos pré-definidos
string dataAbreviada = data.ToShortDateString(); // "23/04/2025" (depende da cultura)
string dataCompleta = data.ToLongDateString(); // "quarta-feira, 23 de abril de 2025" (depende da cultura)
string horaAbreviada = data.ToShortTimeString(); // "14:30" (depende da cultura)
string horaCompleta = data.ToLongTimeString(); // "14:30:45" (depende da cultura)
```


### Usando String.Format e ToString()

```csharp
DateTime data = new DateTime(2025, 4, 23, 14, 30, 45);

// Usando ToString com especificadores de formato
string formatoPersonalizado1 = data.ToString("dd/MM/yyyy HH:mm:ss"); // "23/04/2025 14:30:45"
string formatoPersonalizado2 = data.ToString("yyyy-MM-dd"); // "2025-04-23"
string formatoPersonalizado3 = data.ToString("dddd, dd 'de' MMMM 'de' yyyy"); // "quarta-feira, 23 de abril de 2025"

// Usando String.Format
string formatoComString = String.Format("{0:dd/MM/yyyy}", data); // "23/04/2025"
```


## Padrões de Formatação

O C\# oferece especificadores de formato padrão e personalizados para datas.

### Especificadores de Formato Padrão

```csharp
DateTime data = new DateTime(2025, 4, 23, 14, 30, 45);

// Especificadores de formato padrão
string d = data.ToString("d"); // Formato de data curta: "23/04/2025"
string D = data.ToString("D"); // Formato de data longa: "quarta-feira, 23 de abril de 2025"
string t = data.ToString("t"); // Formato de hora curta: "14:30"
string T = data.ToString("T"); // Formato de hora longa: "14:30:45"
string f = data.ToString("f"); // Data longa e hora curta: "quarta-feira, 23 de abril de 2025 14:30"
string F = data.ToString("F"); // Data longa e hora longa: "quarta-feira, 23 de abril de 2025 14:30:45"
string g = data.ToString("g"); // Data curta e hora curta: "23/04/2025 14:30"
string G = data.ToString("G"); // Data curta e hora longa: "23/04/2025 14:30:45"
string s = data.ToString("s"); // Formato ISO 8601: "2025-04-23T14:30:45"
string u = data.ToString("u"); // Formato universal: "2025-04-23 14:30:45Z"
```


### Especificadores de Formato Personalizados

```csharp
DateTime dt = new DateTime(2025, 4, 23, 14, 30, 45, 123);

// Especificadores para ano
string ano = dt.ToString("y"); // "abril de 2025"
string anosVariados = dt.ToString("y yy yyy yyyy"); // "5 25 025 2025"

// Especificadores para mês
string mes = dt.ToString("M"); // "23 de abril"
string mesesVariados = dt.ToString("M MM MMM MMMM"); // "4 04 abr abril"

// Especificadores para dia
string diasVariados = dt.ToString("d dd ddd dddd"); // "23 23 qua quarta-feira"

// Especificadores para hora
string horasVariadas = dt.ToString("h hh H HH"); // "2 02 14 14" (h: formato 12h, H: formato 24h)

// Especificadores para minuto e segundo
string minutosESegundos = dt.ToString("m mm s ss"); // "30 30 45 45"

// Especificadores para frações de segundo
string fracoes = dt.ToString("f ff fff"); // "1 12 123" (frações de segundo)

// Formato personalizado completo
string personalizado = dt.ToString("dd 'de' MMMM 'de' yyyy 'às' HH:mm:ss"); // "23 de abril de 2025 às 14:30:45"
```


## Adicionando Valores em Data

O C\# permite adicionar ou subtrair intervalos de tempo a datas usando métodos específicos.

### Métodos Add

```csharp
DateTime dataInicial = new DateTime(2025, 4, 23);

// Adicionando dias
DateTime dataMaisDias = dataInicial.AddDays(5); // 28/04/2025

// Adicionando meses
DateTime dataMaisMeses = dataInicial.AddMonths(2); // 23/06/2025

// Adicionando anos
DateTime dataMaisAnos = dataInicial.AddYears(1); // 23/04/2026

// Adicionando horas, minutos, segundos
DateTime dataMaisHoras = dataInicial.AddHours(3); // 23/04/2025 03:00:00
DateTime dataMaisMinutos = dataInicial.AddMinutes(45); // 23/04/2025 00:45:00
DateTime dataMaisSegundos = dataInicial.AddSeconds(30); // 23/04/2025 00:00:30

// Combinando adições
DateTime dataCombinada = dataInicial.AddDays(1).AddHours(12).AddMinutes(30); // 24/04/2025 12:30:00
```


### Usando TimeSpan

```csharp
DateTime dataInicial = new DateTime(2025, 4, 23);

// Criando um TimeSpan
TimeSpan intervalo = new TimeSpan(1, 12, 30, 0); // 1 dia, 12 horas e 30 minutos

// Adicionando o TimeSpan à data
DateTime dataResultante = dataInicial.Add(intervalo); // 24/04/2025 12:30:00

// Alternativa usando operador +
DateTime dataResultante2 = dataInicial + intervalo; // 24/04/2025 12:30:00
```


## Comparação de Datas

O C\# oferece várias maneiras de comparar datas.

### Usando Operadores de Comparação

```csharp
DateTime data1 = new DateTime(2025, 4, 23);
DateTime data2 = new DateTime(2025, 5, 1);

bool anterior = data1 &lt; data2; // true
bool posterior = data1 &gt; data2; // false
bool igual = data1 == data2; // false
bool diferente = data1 != data2; // true
```


### Usando o Método Compare

```csharp
DateTime data1 = new DateTime(2025, 4, 23);
DateTime data2 = new DateTime(2025, 5, 1);

int resultado = DateTime.Compare(data1, data2);

string relacao;
if (resultado &lt; 0)
    relacao = "é anterior a";
else if (resultado == 0)
    relacao = "é igual a";
else
    relacao = "é posterior a";

Console.WriteLine($"{data1} {relacao} {data2}"); // "23/04/2025 00:00:00 é anterior a 01/05/2025 00:00:00"
```


### Usando o Método Equals

```csharp
DateTime data1 = new DateTime(2025, 4, 23);
DateTime data2 = new DateTime(2025, 4, 23);
DateTime data3 = new DateTime(2025, 4, 23, 12, 0, 0); // Com hora

bool saoIguais1 = data1.Equals(data2); // true
bool saoIguais2 = DateTime.Equals(data1, data2); // true
bool saoIguais3 = data1.Equals(data3); // false (a hora é diferente)

// Para comparar apenas a data, ignorando a hora
bool mesmaData = data1.Date.Equals(data3.Date); // true
```


## CultureInfo

A classe `CultureInfo` permite formatar datas de acordo com diferentes culturas e idiomas.

```csharp
DateTime data = new DateTime(2025, 4, 23);

// Cultura brasileira
CultureInfo ptBR = new CultureInfo("pt-BR");
string dataBrasileira = data.ToString("d", ptBR); // "23/04/2025"
string diaSemanaPortugues = data.ToString("dddd", ptBR); // "quarta-feira"

// Cultura americana
CultureInfo enUS = new CultureInfo("en-US");
string dataAmericana = data.ToString("d", enUS); // "4/23/2025"
string diaSemanaIngles = data.ToString("dddd", enUS); // "Wednesday"

// Cultura alemã
CultureInfo deDE = new CultureInfo("de-DE");
string dataAlema = data.ToString("d", deDE); // "23.04.2025"
string diaSemanaAlemao = data.ToString("dddd", deDE); // "Mittwoch"

// Cultura francesa
CultureInfo frFR = new CultureInfo("fr-FR");
string dataFrancesa = data.ToString("d", frFR); // "23/04/2025"
string diaSemanaFrances = data.ToString("dddd", frFR); // "mercredi"
```


## TimeZone e Fusos Horários

O C\# oferece classes para trabalhar com fusos horários.

### TimeZoneInfo

```csharp
// Obtendo o fuso horário local
TimeZoneInfo fusoLocal = TimeZoneInfo.Local;
Console.WriteLine($"Fuso horário local: {fusoLocal.DisplayName}");
Console.WriteLine($"Nome padrão: {fusoLocal.StandardName}");
Console.WriteLine($"Nome de horário de verão: {fusoLocal.DaylightName}");

// Obtendo um fuso horário específico
TimeZoneInfo fusoPacific = TimeZoneInfo.FindSystemTimeZoneById("Pacific Standard Time");

// Convertendo hora entre fusos horários
DateTime dataLocal = DateTime.Now;
DateTime dataUTC = TimeZoneInfo.ConvertTimeToUtc(dataLocal);
DateTime dataPacific = TimeZoneInfo.ConvertTime(dataLocal, fusoLocal, fusoPacific);

// Verificando se está em horário de verão
bool ehHorarioVerao = fusoLocal.IsDaylightSavingTime(dataLocal);
```


### Trabalhando com UTC

```csharp
// Obtendo a hora UTC atual
DateTime horaUTC = DateTime.UtcNow;

// Convertendo de local para UTC
DateTime horaLocal = DateTime.Now;
DateTime horaEmUTC = horaLocal.ToUniversalTime();

// Convertendo de UTC para local
DateTime horaLocalDeUTC = horaUTC.ToLocalTime();
```


## TimeSpan

A estrutura `TimeSpan` representa um intervalo de tempo e é muito útil para cálculos de duração.

### Criando TimeSpan

```csharp
// Diferentes construtores
TimeSpan intervalo1 = new TimeSpan(2, 30, 0); // 2 horas e 30 minutos
TimeSpan intervalo2 = new TimeSpan(1, 12, 45, 30); // 1 dia, 12 horas, 45 minutos e 30 segundos
TimeSpan intervalo3 = new TimeSpan(10000000); // 1 segundo (em ticks)

// Métodos estáticos
TimeSpan umDia = TimeSpan.FromDays(1);
TimeSpan duasHoras = TimeSpan.FromHours(2);
TimeSpan trintaMinutos = TimeSpan.FromMinutes(30);
TimeSpan dezSegundos = TimeSpan.FromSeconds(10);
```


### Propriedades do TimeSpan

```csharp
TimeSpan intervalo = new TimeSpan(1, 12, 45, 30);

// Propriedades de componentes
int dias = intervalo.Days; // 1
int horas = intervalo.Hours; // 12
int minutos = intervalo.Minutes; // 45
int segundos = intervalo.Seconds; // 30
int milissegundos = intervalo.Milliseconds; // 0

// Propriedades de totais
double totalDias = intervalo.TotalDays; // 1.53...
double totalHoras = intervalo.TotalHours; // 36.75...
double totalMinutos = intervalo.TotalMinutes; // 2205.5
double totalSegundos = intervalo.TotalSeconds; // 132330
double totalMilissegundos = intervalo.TotalMilliseconds; // 132330000
```


### Operações com TimeSpan

```csharp
TimeSpan t1 = new TimeSpan(5, 10, 50); // 5h, 10min, 50s
TimeSpan t2 = new TimeSpan(7, 11, 12); // 7h, 11min, 12s

// Operações aritméticas
TimeSpan soma = t1.Add(t2); // ou t1 + t2
TimeSpan diferenca = t1.Subtract(t2); // ou t1 - t2
TimeSpan multiplicacao = t1.Multiply(3.0); // multiplica t1 por 3
TimeSpan divisao = t1.Divide(4.0); // divide t1 por 4

// Comparações
bool t1MaiorQueT2 = t1 &gt; t2; // false
bool t1MenorQueT2 = t1 &lt; t2; // true
bool t1IgualT2 = t1 == t2; // false

// Formatação
string formatoPadrao = t1.ToString(); // "05:10:50"
string formatoCompacto = t1.ToString("c"); // "05:10:50"
string formatoPersonalizado = t1.ToString(@"hh\:mm\:ss"); // "05:10:50"
```


### Calculando Diferenças entre Datas

```csharp
DateTime dataInicio = new DateTime(2025, 1, 1);
DateTime dataFim = new DateTime(2025, 8, 18, 13, 30, 30);

// Calculando o intervalo entre as duas datas
TimeSpan intervalo = dataFim - dataInicio;

Console.WriteLine($"Dias: {intervalo.Days}"); // Componente de dias
Console.WriteLine($"Total de dias: {intervalo.TotalDays}"); // Total em dias
Console.WriteLine($"Total de horas: {intervalo.TotalHours}"); // Total em horas
Console.WriteLine($"Total de minutos: {intervalo.TotalMinutes}"); // Total em minutos
```
