# Projecto 2 - (Compiladores CS3025)

## Resumen
En este proyecto se implementaron mejoras al compilador del lenguaje IMP-FUN que se desarrolló en el curso de Compiladores CS3025. Las mejoras implementadas fueron las siguientes:

1. **Typechecker y Codegen**: Se implementó un `typechecker` y un `codegen` para la gramática `IMP-FUN`.
2. **FCallStm**: Se implementó la funcionalidad de llamadas a funciones sin necesidad de asignarse a una variable en el lenguaje `IMP-FUN`.
3. **ForDoStatement**: Se implementó la funcionalidad de un bucle `for` en el lenguaje `IMP-FUN`.


## Tabla de Contenidos

- [Projecto 2 - (Compiladores CS3025)](#projecto-2---compiladores-cs3025)
  - [Resumen](#resumen)
  - [Tabla de Contenidos](#tabla-de-contenidos)
  - [Gramática del lenguaje IMP-FUN](#gramática-del-lenguaje-imp-fun)
  - [1. Typechecker y Codegen](#1-typechecker-y-codegen)
    - [1.1. Typechecker para funciones](#11-typechecker-para-funciones)
    - [1.2. Codegen para funciones](#12-codegen-para-funciones)
    - [1.3. Ejemplo de uso](#13-ejemplo-de-uso)
  - [2. Implementación de FCallStm](#2-implementación-de-fcallstm)
    - [1.1. Cambios en el Parser](#11-cambios-en-el-parser-1)
    - [1.2. Printer](#12-printer)
    - [1.3. Interpreter](#13-interpreter)
    - [1.4. Typechecker](#14-typechecker)
    - [1.5. Codegen](#15-codegen)
    - [1.6. Ejemplo de funcionamiento](#16-ejemplo-de-funcionamiento)
  - [3. Implementación de ForDoStm](#3-implementación-de-fordostm)
    - [1.1. Cambios en el Scanner](#11-cambios-en-el-scanner)
    - [1.2. Cambios en el Parser](#12-cambios-en-el-parser)
    - [1.3. Printer](#13-printer-1)
    - [1.4. Interpreter](#14-interpreter-1)
    - [1.5. Typechecker](#15-typechecker-1)
    - [1.6. Codegen](#16-codegen-1)
    - [1.7. Ejemplo de funcionamiento](#17-ejemplo-de-funcionamiento-1)
  - [Autores](#autores)
  - [Referencias](#referencias)
  


## Gramática del lenguaje IMP-FUN
La gramática que se utilizó en el proyecto es la siguiente:

```js
Program         ::= VarDecList FunDecList
VarDecList      ::= (VarDec)* 
FunDecList      ::= (FunDec)+
FunDec          ::= "fun" Type id "(" [ParamDecList] ")" Body "endfun"
Body            ::= VarDecList StatementList  
ParamDecList    ::= Type id ("," Type id)*
VarDec          ::= "var" Type VarList ";"
VarList         ::= id ("," id)*
Type            ::= id
StatementList   ::= Stm (";" Stm)*
Stm             ::= id "=" Exp                                            |
                    "print" "(" CExp ")"                                  |
                    "if" CExp "then" Body ["else" Body] "endif"           |   
                    "while" CExp "do" Body "endwhile"                     |
                    "return" "(" [Exp] ")"                                |
                    id "(" [ArgList] ")"                                  |
                    "for" id "in" "(" Exp "," Exp ")" "do" Body "endfor"  |
Exp             ::= BExp
BExp            ::= CExp (("and" | "or") BExp)?
CExp            ::= AExp (("==" | "<" | "<=" ) AExp)?
AExp            ::= Term (("+" | "-") Term)*
Term            ::= FExp (("*" | "/") FExp)*
FExp            ::= Factor ("**" FExp)? | Factor
Factor          ::= id                                       	| 
                    num                                      	| 
                    true                                     	|
                    false                                    	|
                    "(" Exp ")"                              	| 
                    "ifexp" "(" Exp "," Exp "," Exp ")"      	|
                    id "(" [ArgList] ")"
ArgList         ::= Exp ("," Exp)*
```

## 1. Typechecker y Codegen

En esta sección se describe la implementación del Typechecker y Codegen para el lenguaje IMP-FUN.

### 1.1. Typechecker para funciones

En el typechecker para funciones, cada función posee una variable max_stack y mem_locals, de modo que se pueda calcular el espacio requerido para las variables locales y la altura máxima de pila para las funciones. 

El ReturnStatement disminuye la altura de la pila en 1, ya que se debe eliminar el valor de retorno de la pila.
El FCallExp aumenta la altura de la pila en 1, ya que se debe agregar espacio en el valor de retorno a la pila.


### 1.2. Codegen para funciones

En el codegen para funciones, se le pasa como argumento al constructor el typechecker, de modo que se pueda acceder a todas las funciones así como sus variables locales de cada una de ellas.

Además, cada variable tiene un booleano que diferencia si es una variable local o global, con el fin de diferenciar las instrucciones a usar en el codegen.

Codegen para `AssignStatement`:
```cpp
void ImpCodeGen::visit(AssignStatement *s) {
  s->rhs->accept(this);
  VarEntry ventry = direcciones.lookup(s->id);
  if (ventry.is_global)
    codegen(nolabel, "store", ventry.dir);
  else
    codegen(nolabel, "storer", ventry.dir);
  return;
}
```

Codegen para `IdExp`:
```cpp
int ImpCodeGen::visit(IdExp *e) {
  VarEntry ventry = direcciones.lookup(e->id);
  if (ventry.is_global)
    codegen(nolabel, "load", ventry.dir);
  else
    codegen(nolabel, "loadr", ventry.dir);
  return 0;
}
```

Para la declaración de Funciones, se hizo el siguiente código:
```cpp
void ImpCodeGen::visit(FunDec *fd) {
  FEntry fentry = analysis->ftable.lookup(fd->fname);
  current_dir = 0;
  int m = fd->types.size();
  VarEntry ventry;

  list<string>::iterator it;
  for (it = fd->vars.begin(); it != fd->vars.end(); ++it) {
    current_dir++; // OJO
    ventry.dir = current_dir - (m + 3);
    ventry.is_global = process_global;
    direcciones.add_var(*it, ventry);
  }

  ventry.dir = -(m + 3);
  direcciones.add_var("return", ventry);
  codegen(get_flabel(fd->fname), "skip");
  codegen(nolabel, "enter", fentry.mem_locals + fentry.max_stack);
  codegen(nolabel, "alloc", fentry.mem_locals);
  num_params = m;
  fd->body->accept(this);
  return;
}
```

Para el `ReturnStatement`:
```cpp
void ImpCodeGen::visit(ReturnStatement *s) {
  VarEntry ventry = direcciones.lookup("return");

  if (s->e != NULL) {
    s->e->accept(this);
    codegen(nolabel, "storer", ventry.dir);
  }

  codegen(nolabel, "return", -ventry.dir);
  return;
}
```

Para el `FCallExp`:
```cpp
int ImpCodeGen::visit(FCallExp *e) {

  FEntry fentry = analysis->ftable.lookup(e->fname);
  ImpType ftype = fentry.ftype;

  codegen(nolabel, "alloc", 1);

  list<Exp *>::iterator it;
  for (it = e->args.begin(); it != e->args.end(); ++it) {
    (*it)->accept(this);
  }

  codegen(nolabel, "mark");
  codegen(nolabel, "pusha", get_flabel(e->fname));
  codegen(nolabel, "call");
  return 0;
}
```

Para el `Program`:
```cpp
void ImpCodeGen::visit(Program *p) {
  current_dir = 0; 
  direcciones.add_level();
  process_global = true;
  p->var_decs->accept(this);
  process_global = false;

  mem_globals = current_dir;

  codegen("start", "skip");
  codegen(nolabel, "enter", mem_globals);
  codegen(nolabel, "alloc", mem_globals);
  codegen(nolabel, "mark");
  codegen(nolabel, "pusha", get_flabel("main"));
  codegen(nolabel, "call");
  codegen(nolabel, "halt");

  p->fun_decs->accept(this);
  direcciones.remove_level();
  return;
}
```

### 1.3. Ejemplo de uso

Para probar el código, se ejecutó el programa `ejemplo_suma.imp`:

```txt
var int a, b;

fun int suma(int x)
  var int accum;
  accum = 0;
  print(15);
  while 0 < x do
    var int zz;
    accum = accum + x;
    x = x - 1
  endwhile;
  print(16);
  return (accum)
endfun


fun void main()
  var int y;
  a = 10;
  b = 11;
  print(14);
  y = suma(4);
  print(y);
  return ()
endfun
```


El código máquina de la ejecución del programa se almacenó en el archivo `ejemplo_suma.imp.sm`:

```txt
start: skip
enter 2
alloc 2
mark
pusha Lmain
call
halt
Lsuma: skip
enter 4
alloc 2
push  0
storer 1
push  15
print
L0: skip
push  0
loadr -3
lt
jmpz L1
loadr 1
loadr -3
add
storer 1
loadr -3
push  1
sub
storer -3
goto L0
L1: skip
push  16
print
loadr 1
storer -4
return 4
Lmain: skip
enter 3
alloc 1
push  10
store 1
push  11
store 2
push  14
print
alloc 1
push  4
mark
pusha Lsuma
call
storer 1
loadr 1
print
return 3
```

> **⚠️ Reporte: ¿Cómo se calculó el espacio requerido para las variables locales y la altura máxima de pila para las funciones? ¿Cómo se calculó las direcciones de las variables globales y locales, así como la posición en la pila del valor de retorno?** 

En el typechecker, se calculó el espacio requerido para las variables locales y la altura máxima de pila para las funciones, mediante las funciones `sp_incr()` y `sp_decr()`. Además, se calculó las direcciones de las variables globales y locales en base a la cantidad de parámetros en cada función, así como tomando en consideración los espacios reservados en la pila para las variables de control.

## 2. Implementación de FCallStm

En esta sección se describe la implementación de la funcionalidad de llamadas a funciones sin necesidad de asignarse a una variable en el lenguaje IMP-FUN. Esta funcionalidad se asemeja mucho al FCallExp, con la diferencia que no se retorna una expresión.


### 1.1. Cambios en el Parser

Se agregó la clase `FCallStm` y se definieron sus visitors en el archivo `imp.hh`:

```cpp
class FCallStm : public Stm {
public:
  string fname;
  list<Exp *> args;
  FCallStm(string fname, list<Exp *> args);
  void accept(ImpVisitor *v);
  void accept(ImpValueVisitor *v);
  void accept(TypeVisitor *v);
  ~FCallStm();
};
```

Además, se modificó el parser para que soporte la regla `id "(" [ArgList] ")"`:

```cpp
Stm *Parser::parseStatement() {
  ...
  if (match(Token::ID)) {
    string lex = previous->lexema;
    // function stm
    if (match(Token::LPAREN)) {
      list<Exp *> args;
      if (!check(Token::RPAREN)) {
        args.push_back(parseCExp());
        while (match(Token::COMMA)) {
          args.push_back(parseCExp());
        }
      }
      if (!match(Token::RPAREN))
        parserError("Expecting rparen");
      return new FCallStm(lex, args);
    } else {
      if (!match(Token::ASSIGN)) {
        cout << "Error: esperaba =" << endl;
        exit(0);
      }
      s = new AssignStatement(lex, parseCExp());
    }
  }
  ...
}
```

### 1.2. Printer

```cpp
void ImpPrinter::visit(FCallStm *s) {
  cout << s->fname << "(";
  list<Exp *>::iterator it;
  bool first = true;
  for (it = s->args.begin(); it != s->args.end(); ++it) {
    if (!first)
      cout << ",";
    first = false;
    (*it)->accept(this);
  }
  cout << ")";
  return;
}
```

### 1.3. Interpreter

```cpp
void ImpInterpreter::visit(FCallStm *s) {
  FunDec *fdec = fdecs.lookup(s->fname);
  env.add_level();
  list<Exp *>::iterator it;
  list<string>::iterator varit;
  list<string>::iterator vartype;
  ImpVType tt;

  for (it = s->args.begin(), varit = fdec->vars.begin(),
      vartype = fdec->types.begin();
       it != s->args.end(); ++it, ++varit, ++vartype) {
    tt = ImpValue::get_basic_type(*vartype);
    ImpValue v = (*it)->accept(this);
    env.add_var(*varit, v);
  }
  retcall = false;
  fdec->body->accept(this);

  if (!retcall) {
    cout << "Error: Funcion " << s->fname << " no ejecuto RETURN" << endl;
    exit(0);
  }

  retcall = false;
  env.remove_level();
  return;
}
```

### 1.4. Typechecker

```cpp
void ImpTypeChecker::visit(FCallStm *s) {
  if (!env.check(s->fname)) {
    cout << "(Function call): " << s->fname << " no existe" << endl;
    exit(0);
  }
  ImpType funtype = env.lookup(s->fname);
  if (funtype.ttype != ImpType::FUN) {
    cout << "(Function call): " << s->fname << " no es una funcion" << endl;
    exit(0);
  }

  sp_incr();

  ImpType tipo_funcion = env.lookup(s->fname);
  if (tipo_funcion.types.size() - 1 != s->args.size()) {
    cout << "El numero de argumentos no coincide con el numero de parametros" << endl;
    exit(0);
  }

  int count = 0;
  list<Exp *>::iterator it;
  for (it = s->args.begin(); it != s->args.end(); ++it) {
    ImpType argtype = (*it)->accept(this);
    ImpType funcparam;
    funcparam.set_basic_type(tipo_funcion.types[count]);

    if (!argtype.match(funcparam)) {
      cout << "El tipo de los argumentos no coincide con los parametros" << endl;
      exit(0);
    }
    count++;
  }
  return;
}
```

### 1.5. Codegen

```cpp
void ImpCodeGen::visit(FCallStm *s) {
  FEntry fentry = analysis->ftable.lookup(s->fname);
  ImpType ftype = fentry.ftype;

  codegen(nolabel, "alloc", 1);

  list<Exp *>::iterator it;
  for (it = s->args.begin(); it != s->args.end(); ++it) {
    (*it)->accept(this);
  }

  codegen(nolabel, "mark");
  codegen(nolabel, "pusha", get_flabel(s->fname));
  codegen(nolabel, "call");
  return;
}
```

### 1.6. Ejemplo de funcionamiento

Para probar el código, se ejecutó el programa `ejemplo_fcallstm.imp`:

```txt
var int x;

fun void foo(int x)
  print(x+1);
  return ()
endfun

fun void main()
  x = 20;
  print(x);
  foo(x);
  print(x);
  return ()
endfun
```	

El código máquina de la ejecución del programa se almacenó en el archivo `ejemplo_fcallstm.imp.sm`:

```txt
start: skip
enter 1
alloc 1
mark
pusha Lmain
call
halt
Lfoo: skip
enter 2
alloc 0
loadr -3
push  1
add
print
return 4
Lmain: skip
enter 3
alloc 0
push  20
storer -3
loadr -3
print
alloc 1
loadr -3
mark
pusha Lfoo
call
loadr -3
print
return 3
```

## 3. Implementación de ForDoStm

En esta sección se describe la implementación de la funcionalidad de un bucle `for` en el lenguaje IMP-FUN.

### 1.1. Cambios en el Scanner

Se agregaron las palabras reservadas para que el scanner pueda reconocer la palabra `for`, `in` y `endfor`:

```cpp
Scanner::Scanner(string s) : input(s), first(0), current(0) {
  ...
  reserved["for"] = Token::FOR;
  reserved["in"] = Token::IN;
  reserved["endfor"] = Token::ENDFOR;
  ...
}
```

### 1.2. Cambios en el Parser

Se agregó la clase `ForDoStatement` y se definieron sus visitors en el archivo `imp.hh`:

```cpp
class ForDoStatement : public Stm {
public:
  string id;
  Exp *e1, *e2;
  Body *body;
  ForDoStatement(string id, Exp *e1, Exp *e2, Body *body);
  void accept(ImpVisitor *v);
  void accept(ImpValueVisitor *v);
  void accept(TypeVisitor *v);
  ~ForDoStatement();
};
```

Además, se modificó el parser para que soporte la regla `for id in "(" Exp "," Exp ")" do Body endfor`:

```cpp
Stm *Parser::parseStatement() {
  ...
  } else if (match(Token::FOR)) {
    // for id in (exp1, exp2) do body endfor
    if (!match(Token::ID))
      parserError("Esperaba id en for");
    string id = previous->lexema;
    if (!match(Token::IN))
      parserError("Esperaba in en for");
    if (!match(Token::LPAREN))
      parserError("Esperaba lparen en for");
    e1 = parseCExp();
    if (!match(Token::COMMA))
      parserError("Esperaba comma en for");
    e2 = parseCExp();
    if (!match(Token::RPAREN))
      parserError("Esperaba rparen en for");
    if (!match(Token::DO))
      parserError("Esperaba do en for");
    tb = parseBody();
    if (!match(Token::ENDFOR))
      parserError("Esperaba endfor en for");
    s = new ForDoStatement(id, e1, e2, tb);
  }
  ...
}
```

### 1.3. Printer

```cpp
void ImpPrinter::visit(ForDoStatement *s) {
  cout << "for " << s->id << " in (";
  s->e1->accept(this);
  cout << ", ";
  s->e2->accept(this);
  cout << ") do" << endl;
  indent++;
  s->body->accept(this);
  indent--;
  cout << setw(indent * 2) << "" << flush;
  cout << "endfor";
  return;
}
```

### 1.4. Interpreter

En el interpreter se siguió una nueva lógica para el bucle for. La variable asociada al for puede o no existir. Si no existe, se crea una nueva variable en el entorno. Si existe, se guarda el valor anterior para luego ser restaurado al final del bucle.

```cpp
void ImpInterpreter::visit(ForDoStatement *e) {
  ImpValue v1 = e->e1->accept(this);
  ImpValue v2 = e->e2->accept(this);
  ImpValue vi;
  ImpValue vold;

  if (v1.type != TINT || v2.type != TINT) {
    cout << "Error de tipos: tienen que ser enteros" << endl;
    exit(0);
  }

  int iv1 = v1.int_value;
  int iv2 = v2.int_value;

  bool flag = false;

  vi.set_default_value(TINT);

  if (!env.check(e->id)) {
    env.add_var(e->id, vi);
  } else {
    flag = true;
    vold = env.lookup(e->id);
  }

  for (int i = iv1; i < iv2; i++) {
    vi.int_value = i;
    env.update(e->id, vi);
    e->body->accept(this);
  }

  if (flag) {
    env.update(e->id, vold);
  }
}
```

### 1.5. Typechecker

Al igual que en el interpreter, si la variable del for no está en el environment, se agrega una dir más.

```cpp
void ImpTypeChecker::visit(ForDoStatement *s) {
  if (!s->e1->accept(this).match(inttype) ||
      !s->e2->accept(this).match(inttype)) {
    cout << "Expresiones en for deben de ser int" << endl;
    exit(0);
  }

  // Si la variable del for no está en el environment, se agrega una dir más
  if (!env.check(s->id)) {
    env.add_var(s->id, inttype);
    dir++;
    max_dir++;
  }

  sp_decr();
}
```

### 1.6. Codegen

La lógica de codegen del for es la siguiente: se crea una nueva variable local si no existe, y se guarda el valor anterior para luego ser restaurado al final del bucle. Luego, se evalúa si la variable local almacenada es menor al segundo número del for. Si es así, se ejecuta el cuerpo del bucle y se incrementa la variable local para luego ser almacenada. Sino, se salta al final del bucle.

```cpp
void ImpCodeGen::visit(ForDoStatement *s) {
  string l1 = next_label();
  string l2 = next_label();

  VarEntry ventry = direcciones.lookup(s->id);

  s->e1->accept(this);
  codegen(nolabel, "storer", ventry.dir);
  codegen(l1, "skip");
  codegen(nolabel, "loadr", ventry.dir);
  s->e2->accept(this);
  codegen(nolabel, "lt");
  codegen(nolabel, "jmpz", l2);
  s->body->accept(this);
  codegen(nolabel, "loadr", ventry.dir);
  codegen(nolabel, "push", 1);
  codegen(nolabel, "add");
  codegen(nolabel, "storer", ventry.dir);
  codegen(nolabel, "goto", l1);
  codegen(l2, "skip");

  return;
}
```

### 1.7. Ejemplo de funcionamiento

Para probar el código, se ejecutó el programa `ejemplo_fordo.imp`:

```txt
var int x;

fun void main()
  x = 20;
  print(x);
  for x in (1+3, 4+4) do
    print(x)
  endfor;

  print(x);

  return ()
endfun
```

El código máquina de la ejecución del programa se almacenó en el archivo `ejemplo_fordo.imp.sm`:

```txt
start: skip
enter 1
alloc 1
mark
pusha Lmain
call
halt
Lmain: skip
enter 3
alloc 0
push  20
store 1
load 1
print
push  1
push  3
add
storer 1
L0: skip
loadr 1
push  4
push  4
add
lt
jmpz L1
load 1
print
loadr 1
push 1
add
storer 1
goto L0
L1: skip
load 1
print
return 3
```

## Autores

| **Benjamin Soto** | **Fabrizzio Vilchez** | **Jeffrey Monja** |
|:------------:|:------------:|:------------:|
| ![Benjamin Soto](https://avatars.githubusercontent.com/u/104233590?v=4) | ![Fabrizzio Vilchez](https://avatars.githubusercontent.com/u/115495332?v=4) | ![Jeffrey Monja](https://avatars.githubusercontent.com/u/104637993?v=4) |
| [https://github.com/SotoBenjamin](https://github.com/SotoBenjamin) | [https://github.com/Fabrizzio20k](https://github.com/Fabrizzio20k) | [https://github.com/jeffreymonjacastro](https://github.com/jeffreymonjacastro) |


## Referencias 

- W. Appel. (2002) Modern compiler implementation in Java. 2.a edición. CambridgeUniversity Press.
- Kenneth C. Louden. (2004) Compiler Construction: Principles and Practice.Thomson.
