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
|        esi (32 bit)       |
|             | si (16 bit) |
```

#### Instruction pointer EIP

Указатель на текущую инструкцию

**EIP** indicates the memory current address of current instruction

```text
|        eip (32 bit)       |
|             | ip (16 bit) |
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

MOV - также называется key или mnemonic

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

## 04.08-09. Addition using full and partial registers

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
mov esi, 0x1            // инициализация
mov eax, 0x2            // инициализация
mov ebx, 0x3            // инициализация
add eax, ebx
add eax, eax
mov esi, 0xFFFFFFFF
add ebx, esi            // переполнение, появляется carry-flag
add esi, eax            // переполнение, появляется carry-flag
```

### Практический пример. mov, add для неполных регистров

Здесь тоже есть переполнение. Также, если есть переполнение, то старший разряд отбрасывается.

```text
mov edi, 0xAB29FFFF     // инициализация
mov ecx, 0x00000703     // инициализация
mov eax, 0x000000FF     // инициализация
add al, ch
add di, cx
mov edi, 0xAB29FFFF
add edi, ecx
```

## 04.10. Basic Arithmetic SUB Instruction

Синтакс:

>SUB destination, source/value

```text
Destination <- Destination - Source
```

Пример:

```text
sub eax, edx            // eax <- eax - edx
sub edi, 0x2A           // edi <- edi - 0x2A
sub cl, dl              // partial registers
```

### Практический пример. Операции sub

```text
mov eax, 0x1A       // инициализация
mov ebx, 0x3        // инициализация
mov ecx, 0x2        // инициализация
sub eax, ebx
add eax, ebx
sub ecx, ebx        // пример отрицательного значения в виде результата = FFFFFFFF
add ecx, eax
sub cl, al          // пример отрицательного значения в виде результата = FF
```

## 05.11. INC and DEC Instructions

Синтакс:

>INC register
>DEC register

```text
Register <- Register + 1
Register <- Register - 1
```

Пример:

```text
inc eax             // eax <- eax + 1
dec si              // si <- si - 1
```

### Практический пример. Операции inc и dec

```text
mov eax, FFFFFFFE   // инициализация
inc eax
inc al
dec al
inc ax
dec ax
inc eax
inc eax
```

## 05.12. MUL Instructions

MUL - Multiply

>MUL register

Примеры:

```text
mul register1b      // ax <- al * register
mul register2b      // dx:ax <- ax * register    (сохр. часть в dx, часть в ax)
mul register4b      // edx:eax <- eax * register (сохр. часть в edx, часть в eax)
```

```text
mul ecx         // edx:eax <- eax * ecx     (сохр. часть в edx, часть в eax)
mul si          // dx:ax <- ax * si
mul al          // ax <- al * al
mul 0x6B        // INVALID!, нельзя передавать константы
```

- Результат умножения по размеру больше исходного регистра в 2 раза
- Результат умножения распределяется в разные регистры

### Практический пример. Операции mul

```text
mov edx, 0xAB1E2FFF     // инициализация
mov eax, 0x3            // инициализация
mov ecx, 0x2            // инициализация
mul ecx
mul ecx
mov ax, 0xEEEE
mul ax
mul cl
```

## 05.13. DIV Instructions and excercises

DIV - Divide

>DIV register

```text
byte register
1-byte  <-  2-bytes  /  1-byte

al <- ax / register         // quotient     частное
ah <- ax % register         // remainder    остаток
```

```text
word register (2-byte)
2-byte  <-  2-byte:2-byte  /  2-byte

ax <- dx:ax / register      // quotient     частное
dx <- dx:ax % register      // remainder    остаток
```

```text
dword register (4-byte)
4-byte  <-  4-byte:4-byte  /  4-byte

eax <- edx:eax / register   // quotient     частное
edx <- edx:eax % register   // remainder    остаток
```

Примеры:

```text
div ch                  // 1 byte
al <- ax / ch           // quotient     частное
ah <- ax % ch           // remainder    остаток

div di                  // 2 byte
ax <- dx:ax / di        // quotient     частное
dx <- dx:ax % di        // remainder    остаток

div esi                 // 4 byte
eax <- edx:eax / esi    // quotient     частное
edx <- edx:eax % esi    // remainder    остаток

div 0x8C        // INVALID!, нельзя передавать константы
```

### Исключения (ошибки)

- **division by 0** (деление на 0)

```text
mov eax, 0x3
mov edx, 0x0
mov ecx, 0x0

div ecx
// eax <- edx:eax / ecx,    т.е. eax <- 0x0:0x3 / 0x0
// edx <- edx:eax % ecx,    т.е. edx <- 0x0:0x3 % 0x0
```

- **quotient overflow** (переполнение частного - число слишком большое)

```text
mov eax, 0x00005678
mov edx, 0xFFFFFFFF
mov ecx, 0x2

div ecx
// eax <- edx:eax / ecx,    т.е. eax <- 0xFFFFFFFF:0x00005678 / 0x2
```

Исключения в xdbg показываются после выполнения инструкции внизу окна.

- При *division by 0* код ошибки "EXCEPTION_INT_DIVIDE_BY_ZERO"
- При *quotient overflow* код ошибки "EXCEPTION_INT_OVERFLOW"

## 05.14. DIV excercises

1) Посмотреть исключения, из предыдущего соседнего раздела

2) Упражение 1

```text
mov ecx, 0x2
mov edx, 0x0
mov eax, 0x8
div ecx
inc ecx
div ecx
```

3) Упражение 2

```text
mov ebx, 0x3A
mov edx, 0x20
mov eax, 0x0
div ebx
div bx
mov bl, 0xFE
div bl
```

## 06.15. How to read and write to memory

Внизу окна xdbg дамп памяти.

Отдельная вкладка "Memory Map".

Окно "Memory Map", рассматриваются секции на примере процесса `template1.exe`:

- `.text` - инструкции
- `.data` - переменные, значения
- `.rdata`, `.bss`, `.idata` - различные типы данных
- `.idata` - хранятся либы

Важна для нас секция `.data`:

ПКМ на секции -> Follow in Dump или просто двойной ЛКМ по секции `.data`.

Пустые сегменты (одни `0`) можно использовать для своих нужд (нужд программирования).

### Вставка значения в память через регистр

1. Помещаем значение в регистр

```text
mov eax,2
```

2. Копируем свободный адрес из окна dump

`Alt+Ins` или Адрес -> ПКМ -> Copy -> Address

3. Помещаем значение в сегмент памяти

>MOV dword ptr ds:[0x00403050], eax

- `dword` какой размер данных пишем (4 байта)
- `ptr` указатель
- `ds` - data segment
- `[0x....]` - адрес памяти куда пишем (свободный адрес из шага 2)
- `eax` - регистр - источник данных для записи

### Отображение в памяти

```text
00403050 | 02 00 00 00 | 00 00 00 00 | ...
```

Данные в памяти сохраняются в обратном порядке младший байт записывается слева.
Это известно как **Little endian** convention.

### Чтение значения из памяти в регистр

>MOV ebx, dword ptr ds:[0x00403050]

- `ebx` регистр, куда сохраняются данные
- `dword ptr ds:[0x00403050]` - источник данных - память

Чтение из памяти, также как и запись туда, выполняется в обратном порядке.

## 06.16. MOV to memory and direct memory patching

### MOV to memory

- Окно "Memory Map", двойной ЛКМ по секции `.data`
- Копируем свободный адрес в буфер обмена.

>MOV dword ptr ds:[0x00403050], 0x12

### Direct memory patching

- В окне "Dump" выбираем 4 значения -> `Ctrl+ E` или ПКМ -> Binary -> Edit
- В разделе Hex: редактируем значения
- Убеждаемся что стоит флажок `Keep Size`
- `OK`

Patch файл сохраняет изменения в memory, сделанные через прямое редактирование.

## 06.17. Memory Exersize (Практическое самостоятельное упражнение)

Write a program that can do the following. Put the hex values 13374 and 2C3 into
memory. Then, add the two values and put the result in register ECX. Compare the value
in ECX with your programmer's calculator to see if it is correct.

## 07.18. The stack

ОС выделяет память для каждого процесса. Такая выделенная память
называется **стеком**. Каждый процесс запускается в своем стеке.

Регистр `ESP` - указатель, всегда указывает на top стека.

Плюс в отдельном окне есть указатели (смещения) вида:

- `[esp+4]`
- `[esp+8]`
- `[esp+C]`
- `[esp+10]`
- `[esp+14]`

## 07.18. Операция PUSH

Если мы в стек делаем `PUSH`, то адрес в `ESP` смещается на `-4` байта
и в этот адрес записывается значение из регистра.

```text
esp <- esp - 4
dword[esp] <- value
```

![Stack push](/18-stack/01_stack_push.jpg)

```text
mov eax, 0x35A626
mov ebx, 0x100484
mov ecx, 0x000047
push eax
push ebx
push ecx
```

## 07.19 Операция POP

И наоборот в случае операции `POP`. Структура `LIFO` (last in first out).

Если мы из стека делаем `POP`, то из текущего `ESP` адреса стека извлекается значение
и записывается в регистр, адрес в `ESP` смещается на `+4` байта.

```text
register <- dword[esp]
esp <- esp + 4
```

Демонстрация одного из назначений стека - временное хранилище данных регистров.

В примере мы меняем значения стеков (операции `INC`), а потом восстанасливаем их значения
из стека.

Из `PUSH`:

```text
mov eax, 0x35A626
mov ebx, 0x100484
mov ecx, 0x000047
push eax
push ebx
push ecx
```

Теперь `POP`:

```text
inc eax     // изменяем eax
inc ebx     // изменяем ebx
inc ecx     // изменяем ecx
pop ecx
pop ebx
pop eax
```

## 07.20 Pushing constants and strings to the stack

### Pushing constants

Просто делаем

```text
push 0x53
```

### Pushing strings

1. Сначала строку помещаем в память

- Окно "Memory Map", двойной ЛКМ по секции `.data`
- В окне "Dump" выбираем свободные ячейки в 4-х столбцах (16 значений `00`)
- Открываем редактор `Ctrl+ E` или ПКМ -> Binary -> Edit
- В ASCII поле вставляем текст "hello world"
- Копируем адрес строки в буфер обмена (`Alt+Ins` или Адрес -> ПКМ -> Copy -> Address)

Примечание: строка должна заканчиваться символом `00` (null terminator).

2. Делаем push адреса ячейки, которая указывает на строку:

```text
push 0x00403050
```
