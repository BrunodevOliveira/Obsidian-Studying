Quando um número de solicitações simultâneas para o aplicativo é maior do que o número  disponível de threads de trabalho, as solicitações vão para o estado pendente até que qualquer umas das solicitações ativas seja concluída.
> [!NOTE]
> Thread(linha de execução) -> É uma sequencia de instruções que podem ser executadas simultaneamente com outras sequências de instruções

A fila de espera tem um limite (de requisições aguardando e de tempo de espera) e se esse limite for ultrapassado, os usuários com novas requisições receberão o famoso HTTP Code 500 internal server erro.


