Quando um número de solicitações simultâneas para o aplicativo é maior do que o número  disponível de threads de trabalho, as solicitações vão para o estado pendente até que qualquer umas das solicitações ativas seja concluída.



```C#
public interface IRepository<T>
{
    IEnumerable<T> GetAll();
    T? Get(Expression<Func<T, bool>> predicate);
    
    T Create(T entity);
    T Update(T entity);
    T Delete(T entity);
}
```
- GetAll e Get precisam acessar o banco de dados
- Os contratos Create, Update e Delete não acessa o banco de dados e realizam as operações na memória
![[Assincronismo]]

