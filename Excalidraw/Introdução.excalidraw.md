---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠== You can decompress Drawing data with the command palette: 'Decompress current Excalidraw file'. For more info check in plugin settings under 'Saving'



# Controller WebApi-Biblioteca

```cs
namespace Biblioteca.Controllers;
[Route("api/[controller]")]
[ApiController]
public class LivroController : ControllerBase
{
    private readonly ILivroInterface _livroInterface;
    public LivroController(ILivroInterface livroInterface)
    {
        _livroInterface = livroInterface;
    }

    [HttpGet("ListarLivros")]
    public async Task<ActionResult<ResponseModel<List<LivroModel>>>> ListarLivros()
    {
        var livros = await _livroInterface.ListarLivros();
        return Ok(livros);
    }

    [HttpGet("BuscarLivroPorIdAutor/{idAutor}")]
    public async Task<ActionResult<ResponseModel<LivroModel>>> BuscarLivroPorIdAutor(int idAutor)
    {
        var livro = await _livroInterface.BuscarLivroPorIdAutor(idAutor);
        return Ok(livro);
    }

    [HttpGet("BuscarLivroPorId/{idLivro}")]
    public async Task<ActionResult<ResponseModel<LivroModel>>> BuscarLivroPorId(int idLivro)
    {
        var livro = await _livroInterface.BuscarLivroPorId(idLivro);
        return Ok(livro);
    }

    [HttpPost("CriarLivro")]
    public async Task<ActionResult<ResponseModel<List<LivroModel>>>> CriarAutor(LivroCriacaoDTO novoLivro)
    {
        var livro = await _livroInterface.CriarLivro(novoLivro);
        return Ok(livro);
    }

    [HttpPut("EditarLivro")]
    public async Task<ActionResult<ResponseModel<List<LivroModel>>>> EditarLivro(LivroEdicaoDTO livroEdit)
    {
        var livro = await _livroInterface.EditarLivro(livroEdit);
        return Ok(livro);
    }

    [HttpDelete("ExcluirLivro")]
    public async Task<ActionResult<ResponseModel<List<LivroModel>>>> ExcluirAutor(int idLivro)
    {
        var livro = await _livroInterface.ExcluirLivro(idLivro);
        return Ok(livro);
    }
}
```

# Excalidraw Data
## Text Elements
Repository ^SQqETNqW

Service ^tai0n7Sa

Controller ^ymW4BiJ2

%%
## Drawing
```compressed-json
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
```
%%