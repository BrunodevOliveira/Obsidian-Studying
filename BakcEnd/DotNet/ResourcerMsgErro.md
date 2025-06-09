> [!NOTE]
    > Forma de padroinizar as mensagens de erro

# Criando arquivos de resource

1. Dentro da camada de exception crio um arquivo `.resx` 
	1. Botão direito no projeto de Exception terá a opção para criação
	2. A estrutura será de chave -> descrição

2. O nome do arquivo criado será o nome do Objeto com os erros adicionados
	![[ResourceErroMessage.png]]
3. Para adicionar msg em linguas diferentes basta criar um novo arquivo e especificar a tag da lingua -> `ResourceErrorMessages.pt-BR.resx`. <font color="#245bdb">O arquivo que não tenha a tag especificando a linguagem será o default para a API</font>
4. Geralmente arquivos de REsource são criados como `private` . Para torna-los publicos basta entrar no `csproj` do projeto e susbtituir a conteudo da tag `Generator` para esse:

    ```xml
<ItemGroup>  
	  <EmbeddedResource Update="ResourceErrorMessages.resx">  
	    <Generator>PublicResXFileCodeGenerator</Generator>  
	    <LastGenOutput>ResourceErrorMessages.Designer.cs</LastGenOutput>  
	  </EmbeddedResource>
</ItemGroup>
    ```


# Middleware de idiomas
> [!NOTE]
> Uma classe que é executada antes da requisição ser processada para obter iformações que geralmente são armazenadas no Headers da requisição.

