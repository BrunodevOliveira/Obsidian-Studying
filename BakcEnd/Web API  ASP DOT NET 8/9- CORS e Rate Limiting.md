# Overview CORS

# Habilitando CORS
> [!NOTE]
> Existem 3 maneiras para habilitar o CORS 


## 1- No Middleware usando uma política nomeada ou uma política padrão

configuro o Middleware na classe `Program.cs` com as origens permitidas
```C#
var OrigensComAcessoPermitido = "_origensComAcessoPermitido";  
builder.Services.AddCors(options =>  
{  
    options.AddPolicy(name: OrigensComAcessoPermitido,  
        policy =>  
        {  
            policy.WithOrigins("https://apirequest.io");  
        });});
```

Habilito o Middleware  na classe `Program.cs` sempre <span style="color:rgb(254, 0, 65)">após</span> o `app.UseRouting` e <span style="color:rgb(107, 255, 174)">antes</span> do `app.UseAuthorization`:
```C#
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseCors(OrigensComAcessoPermitido);

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```


## 2- Usando roteamento de endpoint

## 3- Com o atributo `[EnablreCors]`
