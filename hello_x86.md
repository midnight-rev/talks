---
author: "Midnight Reverser"
date: 2024-12-16
paging: "%d / %d"
---

```
~~~figlet
Hello, Assembly!
~~~
```

# Hello World em Assembly x86

Nessa curta talk, irei falar um pouco sobre o básico necessário para montar e linkar um arquivo executável a partir de um source Assembly.
<br>

O objetivo é prover um pouco de entendimento sobre o funcionamento de um computador e a construção/execução de um programa. Aliás, podemos dizer que é um pequeno passo para *shellcoding*! 
<br>

Para a parte prática, caso deseje acompanhar escrevendo o código, execute uma nova instância do container docker, em um outro terminal, com a seguinte linha de comando:

```sh
docker run -it hello_world_x86 bash
```

> **NOTA**: caso algum slide esteja malformatado, tente alterar o tamanho da tela ou reexecutar o container.


---

```
~~~figlet
???
~~~
```

# O que é um executável?

Para comerçarmos, é importante saber o que é um executável. Executável, durante essa talk, será tratado como um arquivo "binário", que pode ser carregado por um software *loader* e execute *instruções*.
<br>

Calma! Iremos passar por todas essas terminologias logo logo, mas por agora é necessário se atentar ao formato de executável em Linux: o **Executable and Linking Format**, ou **ELF**.
<br>

O assunto sobre formato ELF exigiria mais que uma talk, provavelmente um minicurso! Mas vejamos alguns pontos importantes sobre ele...


---

# Formato ELF

Esse formato de arquivos é utilizado para identificar binários executáveis, relocáveis e bibliotecas (shared objects) no Linux. Ele é composto, principalmente, de *cabeçalhos*, *seções* e *segmentos*:
<br>

- Cabeçalhos (Headers): trazem informações relevantes sobre o binário, as seções e segmentos. Exemplos são: endereço base de carregamento do programa, offsets de estruturas, tipo de arquivo ELF (executável, biblioteca, etc).

> Os principais cabeçalhos são o *ELF Header*, *Program Header* e *Section Header*.

---

# Formato ELF

- Seções (Sections): delimitam áreas com código ou dados e são usadas durante o processo de *linking*, não sendo obrigatórias em runtime - *embora haja exceções!*.
<br>

Elas são nomeadas, geralmente, com um nome curto e começando por ponto (.text, .data, .bss, etc) e contém informações que, após o linking, são usadas para definir os segmentos.

> Devido a sua importância para o linking, seções são obrigatórias em relocáveis.

---

# Formato ELF

- Segmentos: são definidos pelo *Program Header* por meio das informações obtidas das seções durante o linking. Eles são carregados em memória durante a execução do programa, em um endereço especificado pelo *Program Header*. 
<br>

Além disso, cada segmento tem suas páginas de memória configuradas de acordo com as permissões (RWX) definidas durante o linking.

> Devido a sua importância para a execução, eles são obrigatórios em executáveis e bibliotecas.

> **NOTA**: para maiores informações:

```bash
man 5 elf
```

---

# Formato ELF

```
+ ================ +  
|                  |
|    ELF Header    |
|                  |
+ ================ +
|                  |
|  Program Header  |
|                  |
+ ================ +
|  Sections        |
|      .text       |
|      .data       |
|      .bss        |
|      .symtab     |
+ ================ +
|                  |
|  Section Header  |
|                  |
+ ================ +
```

---

# Instruções e arquitetura de computadores

- A arquitetura de computadores (x86, ARM, amd64) define e disponibiliza o conjunto de operações disponíveis através da *Instruction Set Architecture* (ISA). Cada arquitetura tem suas particularidades e suas instruções diferem totalmente uma da outra - *e essa é um dos motivos de não compatibilidade!*.
<br>

- Instruções são "interfaces" para executar certa operação, codificadas em hardware (ou software, em caso de máquinas virtuais). As operações podem ser simples, como um flip de bits, ou complexas quanto operações multimídia ou de ponto flutuante.
<br>

- Elas são compostas por um *opcode*, que identifica a operação, e opcionalmente um ou mais *operandos*.
<br>

- Note que, como estamos operando em baixo nível, todas as coisas são números! (ex: instrução **0x90**). Esse é o famoso *código de máquina*, pouco legível por seres humanos mas totalmente interpretável pelo processador.
<br>

- Assembly é uma representação 1 para 1 (função bijetora, para os mais matemáticos!) do código de máquina para *mnemônicos*, legíveis por seres humanos. Por exemplo, o código de máquina **0x90**, em x86, é representado pelo mnemônico **NOP**.

---

# Registradores, imediatos & memória, hardware!

```
+ ================= +  
|        CPU        | 
| ================= |
| R1 | R2 | R2 | R3 |
+ ================= +

          |
          v

+ ================= +  
|        RAM        |
+ ================= +
```

---

# Registradores, imediatos & memória, hardware!

- Registradores são a memória de mais rápido acesso por estarem fisicamente dentro do processador. Entretanto, eles têm apenas alguns bytes de capacidade.
<br>

- Geralmente são utilizados como memória intermediária para ler dados/escrever na RAM e para operações lógico e aritméticas.
<br>

## Registradores de propósito geral

Abaixo estão alguns registradores da arquitetura x86.

```
+ ----------------- +                    
|        EAX        | -> 32 bits         
+ ----------------- +                    
          + ------- +
          |   AX    | -> 16 bits        
          + ------- +                   
          + ------- +                    
          | AH | AL | -> 8 bits cada       
          + ------- +   

```

---

```
+ ----------------- +
|        EBX        | -> 32 bits
+ ----------------- +
          + ------- +
          |   BX    | -> 16 bits
          + ------- +
          + ------- +
          | BH | BL | -> 8 bits cada
          + ------- +


+ ----------------- +
|        ECX        | -> 32 bits
+ ----------------- +
          + ------- +
          |   CX    | -> 16 bits
          + ------- +
          + ------- +
          | CH | CL | -> 8 bits cada
          + ------- +

```

---

```

+ ----------------- +
|        EDX        | -> 32 bits
+ ----------------- +
          + ------- +
          |   DX    | -> 16 bits
          + ------- +
          + ------- +
          | DH | DL | -> 8 bits cada
          + ------- +
          

+ ----------------- +                   + ----------------- +           
|        ESI        | -> 32 bits        |        EDI        | -> 32 bits
+ ----------------- +                   + ----------------- +           
          + ------- +                             + ------- +           
          |   SI    | -> 16 bits                  |   DI    | -> 16 bits
          + ------- +                             + ------- +           

```

---
# Exemplos de instruções x86

Abaixo está um ínfimo subset de instruções x86:

```
MOV eax, 1
LEA edx, [eax + 2]

ADD ecx, edx
SUB eax, eax

NOT eax
OR eax, 1
AND edx, 8
XOR eax, eax
```

---

# Sistema operacional e syscalls

- O sistema operacional é uma camada de abstração acima da ISA e que gerencia o hardware - como endereçamento e alocações de memória.
<br>
- Para evitar que qualquer programa quebre ou mexa indevidamente no sistema, o acesso a funcionalidades é restrito: apenas o kernel do sistema (ring 0) tem acesso direto ao hardware. Abrir arquivos, escrever na tela e outras interações só acontecem via kernel.
<br>
- Para que programas de usuário (ring 3) consigam acessar essas funções, no kernel existe uma tabela que mapeia um inteiro não negativo para uma funcionalidade, chamada de syscall. Um programa de usuário, então, configura *registradores* e chama uma instrução TRAP ou interrupção para que o kernel *execute aquela funcionalidade por ele*.
<br>
- Para uma lista mais completa de syscalls x86/Linux, visite [](https://x86.syscall.sh/).
<br>

---

# Montagem

Um arquivo .asm ou .s, contendo mnemonicos assembly, pode ser montado utilizando um software Assembler (note a diferença na grafia). Nessa talk, vamos utilizar o NASM devido a sua facilidade de criação de código.

A montagem é o processo de transformar os mnemonicos em código de máquina. Muitas vezes, a montagem também envolve a interpretação de certas estruturas específicas do software Assembler e criação de um formato de arquivos específico - *no nosso caso, ELF*.

> Montagem não significa linking! Ela apenas irá criar um relocável (ou arquivo-objeto). Falaremos mais sobre isso nessa talk.

---

# Linking

O linking é o processo de unir os relocáveis em um executável final, resolvendo endereços de funções e criando os segmentos e Program Header.

Para essa tarefa, utilizaremos o software `ld`.

---

# Programa mais simples que um Hello World

```asm
section .text
    global _start

_start:
    mov ebx, 0
    mov eax, 1
    int 80h

```

Para montar, execute:

```bash
nasm -f elf32 main.asm
```

Para linkar:

```bash
ld -m elf_i386 -o main main.o
```

---

# Hello World, finalmente

```asm
section .data
    msg: db "Hello, World!", 0xA
    len_msg: equ $ - msg

section .text
    global _start

_start:
    mov edx, len_msg
    mov ecx, msg
    mov ebx, 1
    mov eax, 4
    int 80h

    mov ebx, 0
    mov eax, 1
    int 80h
```

---

# Desafio

- Com base no que foi aprendido pelos slides e a talk, crie, monte e faça a linkedição de um programa que:
    1. Cria e abre um arquivo meuarquivo.txt;
    2. Escreve algo nesse arquivo;
    3. Fecha o arquivo;
    4. Termina o programa.
