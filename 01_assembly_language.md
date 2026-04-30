# Assembly Language Programming for Reverse Engineering

## 01.01. Tools

- Virtual Box c Windows 10
- xdbg (x32dbg и x64dbg)
- die (Detect It Easy)

## 01.02. Двоичная и шестнадцатеричная системы. Ссылки

- Decimal-hex-binary conversion table:
[https://kb.iu.edu/d/afdl](https://kb.iu.edu/d/afdl)

- Khan Academy's Free Lessons on Binary and hexadecimal number systems:
[https://www.khanacademy.org/math/algebra-home/alg-intro-to-algebra/algebra-alternate-number-bases/v/number-systems-introduction](https://www.khanacademy.org/math/algebra-home/alg-intro-to-algebra/algebra-alternate-number-bases/v/number-systems-introduction)

## 02.03-04. Intro to xdbg debugger

- Написать комментарий - `;` или ПКМ -> Comment
- Поставить Breakpoint - `F2` или ПКМ -> Breakpoint -> Toggle
- Step over - `F8`
- Запуск - `F9`
- Поиск по строке - `Shift+D` или ПКМ -> Search for -> Current Module -> String References

## 02.05. Как hollow out exe file

Т.е. как удалить тело функции в файле .exe не сломав его.

У функции есть **Function prologue** (инструкции начала функции):

```text
push ebp
mov ebp,esp
...
```

и ***Function epilogue** (инструкции конца функции):

```
...
leave
ret
```

Все остальное (тело функции) между этими инструкциями можно удалить.

### Удаление или заполнение NOP

1) Выделить удаляемое
2) ПКМ -> Binary -> Fill with NOPs или `Ctrl+9`

Выделенное заполниться флагом `nop` - no operation.

### Запомнить изменения в файле (Patch)

File -> Patch file... или `Ctrl+P`

## 03.06. Intro to Registers

![Registers](/01-03-registers/01_registers.jpg)

### Basic general purpose registers

Регистры общего назначения

- **EAX**
- **EBX**
- **ECX**
- **EDX**

```text
|                    eax (32 bit)                   |
|                         |       ax (16 bit)       |
|                         | ah (8 bit) | al (8 bit) |
```

**EAX** is dword which is 4 bytes (32 bits)
**AX** is a word which is 2 bytes (16 bits)
**AH** is a byte (8 bits)
**AL** is also a byte (8 bits)

```text
If EAX = 0x12345678
AX = 5678
AH = 56
AL = 78
```

Аналогично для остальных регистров: EBX, ECX, EDX

```text
|                    eax (32 bit)                   |
|                         |       ax (16 bit)       |
|                         | ah (8 bit) | al (8 bit) |

|                    ebx (32 bit)                   |
|                         |       bx (16 bit)       |
|                         | bh (8 bit) | bl (8 bit) |

|                    ecx (32 bit)                   |
|                         |       cx (16 bit)       |
|                         | ch (8 bit) | cl (8 bit) |

|                    edx (32 bit)                   |
|                         |       dx (16 bit)       |
|                         | dh (8 bit) | dl (8 bit) |
```

#### 64-bit registers

```text
|                            rax (64 bit)                                     |
|                                 |              eax (32 bit)                 |
|                                                   |       ax (16 bit)       |
|                                                   | ah (8 bit) | al (8 bit) |
```

**RAX** is qword which is 8 bytes (64 bits)
**EAX** is a dword which is 4 bytes (32 bits)
**AX** is word which is 2 bytes (16 bits)
**AH** is a byte (8 bits)
**AL** is also a byte (8 bits)

### ESI, EDI, EIP, ESP, EBP registers

#### String index registers ESI EDI

- **ESI** = Source Index
- **EDI** = Destination Index

Используются при копировании строк.

```text
|       esi (32 bit)       |
|             | si (8 bit) |
```

#### Instruction pointer EIP

Указатель на текущую инструкцию

**EIP** indicates the memory current address of current instruction

```text
|       eip (32 bit)       |
|             | ip (8 bit) |
```

#### Stack Frame Pointers ESP EBP

Указатели на стек

- **ESP** = address of top of stack
- **EBP** = address of bottom of stack

Адрес ESP вверху, адрес EBP внизу.
Адрес ESP < адреса EBP

![Stack](/01-03-registers/02_stack.jpg)

## 03.07. MOV и JMP Instructions. Вставка команд в xdbg

Вставка асемблера в xdbg:

`Space` или ПКМ -> Assemble

Сохранение изменений:

`Ctrl+P` или File -> Patch file...

### MOV Instruction

Copies data from Source to Destination

Синтакс:

>MOV Destination, Source

MOV - также называется mnemonic

Пример:

```text
mov eax, 0x3A           0x - HEX формат
mov al, 0x8
mov ebx, eax
mov cx, bx
mov ah, cl
```

### JMP Instruction

Также называется "unconditional jump"

Синтакс:

>JMP адрес_куда_прыгать

Пример:

```text
jmp 0x401533
```

## 03.08 - Addition using full registers

Синтакс:

>ADD destination, source/value

```text
Destination <- Destination + Source
```

Пример:

```text
add eax, ecx            // eax <- eax + ecx
add edi, 0x2A           // edi <- edi + 0x2A
add cx, si              // cx, si - partial registers
```

### Практический пример. mov, add для полных регистров

```text
mov esi, 0x1        // инициализация
mov eax, 0x2        // инициализация
mov ebx, 0x3        // инициализация
add eax, ebx
add eax, eax
mov esi, 0xFFFFFFFF
add ebx, esi            // переполнение, появляется carry-flag
add esi, eax            // переполнение, появляется carry-flag
```
