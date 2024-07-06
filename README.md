# Projecto 2 - (Compiladores CS3025)

## Resumen
En este proyecto se implementaron mejoras al compilador del lenguaje IMP-FUN que se desarrolló en el curso de Compiladores CS3025. Las mejoras implementadas fueron las siguientes:

1. **Typechecker y Codegen**: Se implementó un `typechecker` y un `codegen` para la gramática `IMP-FUN`.



## Tabla de Contenidos

- [Projecto 1 - (Compiladores CS3025)](#projecto-1---compiladores-cs3025)
  - [Resumen](#resumen)
  - [Tabla de Contenidos](#tabla-de-contenidos)
  - [Gramática del lenguaje IMP-DEC](#gramática-del-lenguaje-imp-dec)
  - [1. Typechecker y Codegen](#1-typechecker-y-codegen)
    - [1.1. Typechecker](#11-typechecker)
		- [Implementación del Typechecker](#implementación-del-typechecker)
    - [1.2. Codegen](#12-codegen)
		- [Modificaciones en el Typechecker](#modificaciones-en-el-typechecker)
		- [Implementación del Codegen](#implementación-del-codegen)
    - [Ejemplo de uso](#ejemplo-de-uso)
	- [Consideraciones finales](#consideraciones-finales)
  - [2. Generación de Código I](#2-generación-de-código-i)
    - [Cambios en el Scanner](#cambios-en-el-scanner)
    - [Cambios en el Parser](#cambios-en-el-parser)
      - [Comentarios al final de una declaración de variable o statement](#comentarios-al-final-de-una-declaración-de-variable-o-statement)
      - [Comentarios aislados en cualquier parte del código](#comentarios-aislados-en-cualquier-parte-del-código)
    - [Cambios en los visitors](#cambios-en-los-visitors)
      - [ImpVisitor](#impvisitor)
      - [ImpValueVisitor e ImpTypeVisitor](#impvaluevisitor-e-imptypevisitor)
    - [Ejemplo de funcionamiento](#ejemplo-de-funcionamiento)
    - [Consideraciones finales](#consideraciones-finales)
  - [3. Sentencia Do While](#3-sentencia-do-while)
    - [Ejemplo de funcionamiento](#ejemplo-de-funcionamiento-1)
    - [Consideraciones finales](#consideraciones-finales-1)
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
ParamDecList 		::= Type id ("," Type id)*
VarDec          ::= "var" Type VarList ";"
VarList         ::= id ("," id)*
Type            ::= id
StatementList   ::= Stm (";" Stm)*
Stm             ::= id "=" Exp                                            |
                    "print" "(" CExp ")"                                  |
                    "if" CExp "then" Body ["else" Body] "endif"           |
                    "while" CExp "do" Body "endwhile"                     |
                    "return" "(" [Exp] ")"                                |
                    "for" id "in" "(" Exp "," Exp ")" "do" Body "endfor"  |
Exp             ::= BExp
BExp            ::= CExp (("and" | "or") BExp)?
CExp            ::= AExp (("==" | "<" | "<=" ) AExp)?
AExp            ::= Term (("+" | "-") Term)*
Term            ::= FExp (("*" | "/") FExp)*
FExp            ::= Unary ("**" FExp)?
Unary           ::= "-" Factor | "!" Factor | Factor
Factor          ::= id                                       	| 
                    num                                      	| 
                    true                                     	|
                    false                                    	|
                    "(" Exp ")"                              	| 
                    "ifexp" "(" Exp "," Exp "," Exp ")"      	|
										id "(" [ArgList] ")"
ArgList				  ::= Exp ("," Exp)*
```

## 1. Typechecker y Codegen

En esta sección se describe la implementación del Typechecker y Codegen para el lenguaje IMP-FUN.

### 1.1. Typechecker

#### Implementación del Typechecker

### 1.2. Codegen

#### Implementación del Codegen

### 1.3. Ejemplo de uso

### 1.4. Consideraciones finales

> **⚠️ Reporte: ¿Cómo se calculó el espacio requerido para las variables locales y la altura máxima de pila para las funciones? ¿Cómo se calculó las direcciones de las variables globales y locales, así como la posición en la pila del valor de retorno?** 

## 2. Implementación de FCallStm

### 1.1. Cambios en el Scanner

### 1.2. Cambios en el Parser

### 1.3. Printer

### 1.4. Interpreter

### 1.5. Typechecker

### 1.6. Codegen

### 1.7. Ejemplo de funcionamiento

### 1.8. Consideraciones finales

> **⚠️ Reporte: Indicar los cambios al programa (parser, typechecker, codegen, etc) y las definiciones de typecheck y codegen.**

## 3. Implementación de ForDoStm

### 1.1. Cambios en el Scanner

### 1.2. Cambios en el Parser

### 1.3. Printer

### 1.4. Interpreter

### 1.5. Typechecker

### 1.6. Codegen

### 1.7. Ejemplo de funcionamiento

### 1.8. Consideraciones finales

> **⚠️ Reporte: Indicar los cambios al programa (parser, typechecker, codegen, etc) y las definiciones de typecheck y codegen.**


## Autores

| **Benjamin Soto** | **Fabrizzio Vilchez** | **Jeffrey Monja** |
|:------------:|:------------:|:------------:|
| ![Benjamin Soto](https://avatars.githubusercontent.com/u/104233590?v=4) | ![Fabrizzio Vilchez](https://avatars.githubusercontent.com/u/115495332?v=4) | ![Jeffrey Monja](https://avatars.githubusercontent.com/u/104637993?v=4) |
| [https://github.com/SotoBenjamin](https://github.com/SotoBenjamin) | [https://github.com/Fabrizzio20k](https://github.com/Fabrizzio20k) | [https://github.com/jeffreymonjacastro](https://github.com/jeffreymonjacastro) |


## Referencias 

- W. Appel. (2002) Modern compiler implementation in Java. 2.a edición. CambridgeUniversity Press.
- Kenneth C. Louden. (2004) Compiler Construction: Principles and Practice.Thomson.
