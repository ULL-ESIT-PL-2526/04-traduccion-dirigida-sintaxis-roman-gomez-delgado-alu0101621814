# Syntax Directed Translation with Jison

Jison is a tool that receives as input a Syntax Directed Translation and produces as output a JavaScript parser  that executes
the semantic actions in a bottom up ortraversing of the parse tree.
 

## Compile the grammar to a parser

See file [grammar.jison](./src/grammar.jison) for the grammar specification. To compile it to a parser, run the following command in the terminal:
``` 
➜  jison git:(main) ✗ npx jison grammar.jison -o parser.js
```

## Use the parser

After compiling the grammar to a parser, you can use it in your JavaScript code. For example, you can run the following code in a Node.js environment:

```
➜  jison git:(main) ✗ node                                
Welcome to Node.js v25.6.0.
Type ".help" for more information.
> p = require("./parser.js")
{
  parser: { yy: {} },
  Parser: [Function: Parser],
  parse: [Function (anonymous)],
  main: [Function: commonjsMain]
}
> p.parse("2*3")
6
```



## 3. Análisis del Analizador Léxico (Lexer)

### 3.1. Diferencia entre `/* skip whitespace */` y devolver un token
Cuando el lexer encuentra un espacio en blanco (`\s+`), ejecuta una acción que **no retorna nada**. Esto hace que el analizador léxico simplemente ignore esos caracteres y pase al siguiente sin enviar ninguna información al analizador sintáctico. 

Por el contrario, **devolver un token** (como `return 'NUMBER'`) interrumpe el proceso del lexer para entregar un objeto con el tipo y valor del símbolo encontrado al **parser**, permitiendo que este avance en la validación de la estructura de la gramática.

---

### 3.2. Secuencia exacta de tokens para la entrada `123**45+@`
La secuencia de tokens generada por el analizador léxico para esta entrada es la siguiente:

1. **Token:** `NUMBER`, **Valor:** `"123"`
2. **Token:** `OP`, **Valor:** `"**"`
3. **Token:** `NUMBER`, **Valor:** `"45"`
4. **Token:** `OP`, **Valor:** `"+"`
5. **Token:** `INVALID`, **Valor:** `"@"`

> **Nota:** El proceso se detiene tras el último token debido a un error sintáctico, ya que el parser espera un operando después del símbolo `+` y recibe un token de tipo `INVALID`.

---

### 3.3. Prioridad del operador `**` sobre `[-+*/]`
Al colocar `**` antes en la definición del lexer, obligamos al analizador a intentar reconocer primero la secuencia de dos caracteres (coincidencia más larga). 

Si el operador de multiplicación simple (`*`) estuviera antes, el lexer identificaría el primer carácter de `**` como una multiplicación y dejaría el segundo suelto, rompiendo la lógica de la potencia. Colocarlo primero garantiza que el operador de potencia se identifique correctamente como una **única unidad léxica**.

---

### 3.4. Cuándo se devuelve `EOF`
El token `EOF` (*End Of File*) se devuelve única y exclusivamente cuando el lexer alcanza el **final físico de la entrada de texto**. 

Este token sirve como señal de parada para el analizador sintáctico, indicándole que el flujo de datos ha terminado y que puede proceder a cerrar la regla raíz de la gramática y entregar el resultado final del cálculo.

---

### 3.5. Utilidad de la regla `.` (INVALID)
La regla comodín `.` garantiza que el lexer siempre tenga una respuesta para cualquier entrada, evitando bloqueos o comportamientos indefinidos. 

Su función principal es el **manejo de errores**: identifica caracteres "extraños" o no permitidos (como `@` o `$`) para que el sistema pueda informar de un error de sintaxis de manera estructurada y precisa, en lugar de fallar de forma inesperada durante la fase de lectura.