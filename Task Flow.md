# Entidades centrais

   1. `User` (Usuário):
       * Propósito: Representa quem usa o sistema. Um usuário pode criar projetos, ser membro de projetos, criar tarefas e ser
         responsável por elas.
       * Propriedades principais: Id, Name, Email, Password.

   2. `Project` (Projeto):
       * Propósito: É o contêiner principal que agrupa um conjunto de tarefas relacionadas.
       * Propriedades principais: Id, Name, Description.

   3. `Task` (Tarefa):
       * Propósito: A unidade de trabalho a ser feita. Uma tarefa sempre pertence a um projeto e pode ser atribuída a um usuário.
       * Propriedades principais: Id, Title, Description, DueDate (Prazo), Priority (Prioridade), Status.

   4. `Comment` (Comentário):
       * Propósito: Permite a comunicação dentro de uma tarefa específica.
       * Propriedades principais: Id, Text, CreatedAt (Data de criação).
       * 

## Como elas se conectam

   * Um Project pode ter várias Tasks (Relacionamento 1-para-N).
   * Uma Task pode ter vários Comments (Relacionamento 1-para-N).
   * Um User pode ser o autor de vários Comments e responsável por várias Tasks.
   * E o mais interessante: um User pode participar de vários Projects, e um Project pode ter vários Users (Relacionamento N-para-N ou muitos-para-muitos).

  Esse relacionamento N-para-N entre User e Project é um ponto chave que torna essa API um ótimo aprendizado. No banco de dados, isso
  exigirá uma "tabela de ligação" para conectar os dois.

