### Orientações gerais
- Criar testes para cenários positivos e negativos de uma funcionalidade
- AAA -> padrão utilizado em testes de unidade para garantir que eles funcionem da maneira correta, independentes e isolados uns dos outros:
	- Arrange -> Configuração do que iremos utilizar para o teste
	- Act -> O teste que será executado
	- Assert -> O que esperamos de resultado do teste
		- Nessa etapa utilizamos uma Lib como `FluentAssertions` ou `Shoudly`

- `[fact]` -> Decorator que transforma a função em um teste de unidade
- `[Theory]` -> Habilita a passagem de parâmetros para a função de teste
	- `[InlineData(<valor>)]` -> Valores que serão testados como argumento da função

	```csharp
public class RegisterExpenseValidatorTests {
	[Fact] //Decorator que transforma a função em um teste de unidade  
	public void Success()  
	{  
		//Arrange => Etapa onde configuramos as instâncias de tudo que iremos utilizar no teste  
		var validator = new RegisterExpenseValidator();  
		var request = RequestRegisterExpenseJsonBuilder.Build();  
	  
		//Act => O teste que devemos executar   
		var result = validator.Validate(request);  
	  
		//Assert => O que esperamos de resultado do teste  
		//Assert.True(result.IsValid);    
		result.IsValid.Should().BeTrue();  
	}
	
	[Theory]  
	[InlineData("")]  
	[InlineData("         ")]  
	[InlineData(null)]  
	public void Error_Title_Empty(string title)  
	{  
	    //Arrange => Etapa onde configuramos as instâncias de tudo que iremos utilizar no teste  
	    var validator = new RegisterExpenseValidator();  
	    var request = RequestRegisterExpenseJsonBuilder.Build();  
	    request.Title = title; //Força o erro  
	  
	    //Act => O teste que devemos executarvar result = validator.Validate(request);  
	  
	    //Assert => O que esperamos de resultado do teste  
	    result.IsValid.Should().BeFalse();  
	    result.Errors.Should().ContainSingle().And.Contain(err => err.ErrorMessage.Equals(ResourceErrorMessages.TITLE_REQUIRED));  
	}
}
```

### Projetos (Class Library)
- Validators -> Projeto responsável por testar as regras de validações utilizadas no UseCAse
	- Criar um teste para cada regra de negócio
 

- CommonTestutilities -> Métodos úteis que poderão ser utilizados em outros projetos de teste


### Nuget packet
- Bogus -> criação de dados Fake dinâmicos
-  FluentAssertions ->  para ter mais possibilidades de teste
	- Virou pago agora 💀
	- Shouldly -> Outra biblioteca de testes que é gratuita


