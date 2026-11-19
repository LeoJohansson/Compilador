# 🧠 Compilador Completo em Python (End-to-End)

Este projeto implementa um **compilador didático completo** desenvolvido em Python como parte da disciplina de **Compiladores**. 

O sistema cobre desde a leitura dos caracteres do código-fonte até a geração final de código de baixo nível, operando de forma resiliente mesmo diante de erros estruturais na entrada.

O compilador realiza quatro etapas principais:
- 🔎 **Análise Léxica** → Transforma o código-fonte em uma sequência de tokens.
- 🌳 **Análise Sintática** → Organiza os tokens em uma árvore através de um *Recursive Descent Parser* que gera uma AST (*Abstract Syntax Tree*).
- 🧬 **Análise Semântica** → Gerencia o escopo através de uma Tabela de Símbolos, realiza a inferência dinâmica de tipos e gera diagnósticos (avisos/erros).
- ⚙️ **Geração de Código** → Emite **código intermediário de três endereços** (estilo Assembly, com registradores temporários) durante a própria análise sintática.

---

# 🚀 Funcionalidades Reconhecidas

O sistema reconhece e processa os seguintes elementos da linguagem (baseada na sintaxe do Python):
- 🔑 **Palavras Reservadas** (`if`, `while`, `return`, `class`, `def`, `import`, etc.)
- 🏷️ **Identificadores** (nomes de variáveis, funções e classes)
- 🔢 **Literais Numéricos e Strings** (suporte nativo a inteiros e ponto flutuante)
- ➕ **Operadores Matemáticos** (`+`, `-`, `*`, `/`) com precedência rigorosa.
- 📌 **Delimitadores** (`()`, `{}`, `[]`, `:`, `,`, `.`)
- 📄 **Recuperação de Falhas (Panic Mode)** que permite continuar a compilação mesmo sob erros sintáticos graves.

---

# 🛠️ Como Funciona cada Fase

### 1. Analisador Léxico
Utiliza a técnica de **dois ponteiros** (`inicio` e `atual`) para percorrer o arquivo caractere por caractere. Conta com um mecanismo de **Lookahead** (espiar próximo caractere) para capturar perfeitamente tokens compostos (como `==`, `!=`) e validar constantes de ponto flutuante, acusando erros específicos como números mal formados (`0.0.0`).

### 2. Analisador Sintático (Parser)
Implementado por meio da técnica de **Parser Descendente Recursivo**. Cada regra gramatical é mapeada em um método específico. O parser respeita a precedência matemática (multiplicação e divisão aninhadas abaixo da soma e subtração) e isola o corpo de funções e classes. Ele captura erros de sintaxe (como a falta de termos à direita de uma atribuição) e avança para a próxima linha de forma segura sem abortar o processo.

### 3. Analisador Semântico (Single-Pass com Tipagem)
Operando de forma integrada à sintaxe (Tradução Dirigida pela Sintaxe), ele gerencia uma **Tabela de Símbolos** dinâmica e um mapa de uso. Suas funções incluem:
* **Verificação de Declaração Prévia:** Bloqueia a leitura de variáveis que nunca foram criadas no escopo.
* **Inferência Dinâmica de Tipos:** Promove tipos de dados em expressões binárias automaticamente (ex: $int + float = float$).
* **Gerador de Warnings:** Identifica e avisa no terminal sobre reatribuições de variáveis ou identificadores que foram declarados mas estão ociosos consumindo memória.
* **Controle de Escopo Local:** Copia, isola e destrói o contexto de variáveis locais ao encerrar o processamento de funções.

### 4. Geração de Código (Tradução Dirigida pela Sintaxe)
O código é emitido **durante o parsing**: assim que cada regra da gramática é reconhecida, o `Parser` gera as instruções correspondentes. As expressões usam **registradores temporários** (`T1`, `T2`, ...) no formato de três endereços. Instruções geradas:
* `MOV` → carrega constantes e variáveis em temporários.
* `ADD`, `SUB`, `MUL`, `DIV` → operações aritméticas (`MUL T7, T5, T6`).
* `STORE` → grava o resultado de uma expressão em uma variável.
* `FUNC` / `PARAM` / `RETURN` / `ENDFUNC` → definição de funções.

Exemplo para `x = 10 + 5 * 2`:
```asm
MOV T4, 10
MOV T5, 5
MOV T6, 2
MUL T7, T5, T6
ADD T8, T4, T7
STORE x, T8
```

---

# 📂 Estrutura das Classes do Projeto

- `TipoToken` → Enum com as categorias lexicais.
- `Token` → Estrutura do objeto que armazena tipo, lexema, linha e coluna.
- `AnalisadorLexico` → Responsável pelo fatiamento do texto em tokens.
- `TabelaSimbolos` → Dicionário interno encarregado do armazenamento de tipos e escopo do programa.
- `Parser` → Motor sintático e semântico: monta a AST, gera os alertas e emite o código intermediário.
- `processar()` → Orquestrador central que gerencia o fluxo de arquivos de entrada e saída.

---

# ▶️ Como Executar

### 1. Prepare o arquivo de entrada
Crie o arquivo de testes contendo seu código-fonte na mesma pasta do script (ex: `teste.py`).

### 2. Execute o Compilador
```bash
python compilador.py
```

### 3. Confira os resultados
O compilador mostra no terminal os **avisos** e **erros** encontrados e gera quatro arquivos:

| Arquivo | Conteúdo |
|---|---|
| `saida_tokens.txt` | Lista de tokens com linha, coluna e categoria |
| `saida_ast.txt` | Árvore sintática abstrata (AST) |
| `saida_tabela.txt` | Tabela de símbolos com o tipo inferido de cada identificador |
| `saida_codigo.asm` | Código intermediário gerado (três endereços) |

### Exemplo de saída no terminal
O arquivo `teste.py` do repositório contém erros de propósito, para demonstrar a recuperação de falhas:

```text
⚠️  AVISOS SEMÂNTICOS (WARNINGS): 6
 - [Linha 7] Aviso Semântico: Reatribuição/Redeclaração da variável 'x'.
 - A variável 'resultado' foi declarada, mas nunca foi utilizada.
 ...

❌ ERROS ENCONTRADOS (Sintáticos e Semânticos): 5
 - [Linha 8] Erro Semântico: A variável 'z' não foi declarada.
 - [Linha 26] Token inesperado: =
 ...
```

---

# 🧰 Tecnologias
- Python 3 (somente biblioteca padrão, sem dependências externas)
