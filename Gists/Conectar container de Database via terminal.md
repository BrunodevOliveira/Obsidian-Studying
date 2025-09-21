---
tags: docker
---
# Conectar ao seu container Docker do MySQL

  1.  Abra um novo terminal.
  2.  Execute o seguinte comando para entrar no container e abrir o cliente MySQL:

  ```bash
docker exec -it <nome_container> mysql -u <usuario> -p"<senha>"
```

  3.  Uma vez dentro do cliente MySQL, primeiro selecione seu banco de dados:

```sql
USE task_manager_db;
```

  4.  Agora, execute o comando para listar as tabelas:
```sql
	  SHOW TABLES;
```   

5. Para sair do cliente interativo do banco de dadosbasta digitar `exit` 