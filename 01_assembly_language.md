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
mov eax, 0x3A
mov al, 0x8
mov ebx, eax
mov cx, bx
mov ah, cl
```

- `0x` - HEX формат
- `0b` - binary формат

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

- `.text` - инструкции (executable code)
- `.data` - переменные, значения (initialized data)
- `.rdata` - данные только для чтения (read-only initialized data)
- `.bss` - временные данные, исчезают после перезапуска программы (uninitialized data)
- `.idata` - различные типы данных, хранятся либы (import tables)
- `.tls` - thread-local storage
- `.rsrc` - resources

В `.data` находятся данные (инициализированные), которые сохраняются после перезапуска программы.
В `.bss` находятся временные (неинициализированные) данные, которые исчезают после перезапуска программы.

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
- Убеждаемся что стоит флажок `Keep Size`
- `OK`
- Копируем адрес строки в буфер обмена (`Alt+Ins` или Адрес -> ПКМ -> Copy -> Address)

Примечание: строка должна заканчиваться символом `00` (null terminator).

2. Делаем push адреса ячейки, которая указывает на строку:

```text
push 0x00403050
```

## 08.21. Funcions call (`CALL`)

Ссылки на описание библиотек C:

`printf()` function:

- [https://www.tutorialspoint.com/c_standard_library/c_function_printf.htm](https://www.tutorialspoint.com/c_standard_library/c_function_printf.htm)

format specifiers:

- [https://www.tutorialspoint.com/format-specifiers-in-c](https://www.tutorialspoint.com/format-specifiers-in-c)

Вызвать функцию:

>CALL function

### Вызов функции с 1 параметром

На C:

```c
function(parameter1);
```

Assembly:

```asm
push parameter1         // parameter1 address
call function           // function address
```

1. Вначале мы помещаем параметр (его адрес) в стек
2. Потом вызываем (`CALL`) функцию (по ее адресу), которая принимает 1 параметр

Более подробно:

1.1. Сначала строку помещаем в память

- Окно "Memory Map", двойной ЛКМ по секции `.data`
- В окне "Dump" выбираем свободные ячейки в 4-х столбцах (16 значений `00`)
- Открываем редактор `Ctrl+ E` или ПКМ -> Binary -> Edit
- В ASCII поле вставляем текст "hello world"
- Убеждаемся что стоит флажок `Keep Size`
- `OK`
- Копируем адрес строки в буфер обмена (`Alt+Ins` или Адрес -> ПКМ -> Copy -> Address)

Примечание: строка должна заканчиваться символом `00` (null terminator).

1.2. Делаем `push` адреса ячейки, которая указывает на строку:

```asm
push 0x00403050
```

2.1. Ищем адрес вызываемой функции. Двойной клик по вызову любой функции в коде.
Переход на секцию с `jmp`, где заданы адреса функций.

2.2. В примере нужен адрес функции `printf`. Находим строку `jmp dword ptr ds:[<printf>]`
и копируем ее адрес в буфер обмена (`Alt+Ins` или Адрес -> ПКМ -> Copy -> Address)

2.3. Для перехода в начало программы два раза кликнуть на регистре `EIP`.
Регистр `EIP` содержит адрес entry point программы.

2.4. Делаем `call` адреса ячейки, которая указывает на функцию:

```asm
call 0x00402614
```

## 08.22. Funcions call (`CALL`). Вызов функций с 2 параметрами (строка, строка)

На C:

```c
function(parameter1);
```

Assembly:

```asm
push parameter2         // parameter2 address
push parameter1         // parameter1 address
call function           // function address
```

Здесь параметры добавляются в стек в обратном порядке: сначала второй, потом первый.

### Пример вызова printf с двумя параметрами

1. Чтобы вызвать: `printf("hello, %s", paul);`

Надо написать:

```asm
push paul             // address
push hello, %s        // address
call printf           // address
```

## 08.23. Funcions call (`CALL`). Вызов функций с 2 параметрами (строка, число)

Чтобы вызвать: `printf("number: %d", 1337);`

Надо написать:

```asm
push 0x539            // В десятичной это 1337
push number: %d       // address
call printf           // address
```

Или (более сложно):

Если положить число 539 в память по адресу 00403050 так

```text
39 05 00 00           // little-endian
```

То надо написать:

```asm
push dword ptr ds:[403050]          // Поместить в стек числовое значение (539) по адресу
push number: %d                     // Поместить в стек адрес строки
call printf                         // Вызвать адрес функции
```

- `dword` какой размер данных пишем (4 байта)
- `ptr` указатель
- `ds` - data segment
- `[0x....]` - адрес памяти откуда читаем (свободный адрес из шага 2)

## 08.24. Funcions call (`CALL`). Вызов функций с 3 параметрами

На C:

```c
function(parameter1, parameter2, parameter3);
```

Assembly:

```asm
push parameter3         // parameter3 address
push parameter2         // parameter2 address
push parameter1         // parameter1 address
call function           // function address
```

### Упражнение

Надо вызвать:

```c
printf("hello, %s %s", string1, string2);
```

```asm
push string2
push string1
push "hello, %s %s"
call printf
```

output:

```text
hello, paul chin
```

Решение:

```text
push 0x403038           // 403038:"chin"
push 0x403030           // 403030:"paul"
push 0x403040           // 403040:"hello, %s %s"
call <JMP.&printf>
```

## 09.25. Getting Input using `scanf()` function

В языке C, `scanf()` function for input of numbers:

```c
int number;
scanf("%d", &number);
```

- `"%d"` = format specifier
- `&` - address
- `&number` = address-of `number` variable

В языке C, `scanf()` function for input of strings:

```c
char str[32];           // массив, его наименование также является указателем
scanf("%s", str);
```

- "%s" = format specifier
- `str` = address-of `str`

### Ввод числа и его сохранение в памяти при помощи `scanf`

Как это реализуется в assembler:

1. В стек помещается (`PUSH`) адрес ячейки, куда будет помещено вводимое число.
2. В стек помещается (`PUSH`) адрес ячейки формата вводимого значения (`%d`).
3. Вызов (`call`) указателя на функцию `scanf`.
4. Программа из статуса `Paused` переходит в статус `Running`, вводим число в приложении,
программа опять переходит в статус `Paused`.
5. В целевой ячейке памяти появляется введенное число в hex формате.

Псевдо-код asm:

```text
push addr-of-num
push "%d"
call scanf
```

Более подробно:

#### 1. В окне "CPU" выбираем окно "Dump1". Окно "Memory Map", двойной ЛКМ по секции `.bss`

До этого момента использовали только `.data` - в этой секции хранятся данные, которые сохраняются
после перезапуска приложения.
В секции `.bss` хранятся временные данные, которые исчезнуть после перезапуска приложения.

Вводимые данные можно сохранять как в секции `.bss`, так и в секции `.data`.
В данном примере будет сохранение в `.bss`.

- Копируем адрес свободной ячейки в "Dump1" в буфер обмена (`Alt+Ins` или Адрес -> ПКМ -> Copy -> Address).
- По желанию, заголовок "Dump1" можно переименовать.
- В окне CPU пишем код: `push 0x_адрес_ячейки`

#### 2. В окне "CPU" выбираем окно "Dump2". Окно "Memory Map", двойной ЛКМ по секции `.data`

- Открываем редактор `Ctrl+ E` или ПКМ -> Binary -> Edit
- В ASCII поле вставляем текст "%d" (без кавычек)2
- Убеждаемся что стоит флажок `Keep Size`
- `OK`
- Копируем адрес строки в буфер обмена (`Alt+Ins` или Адрес -> ПКМ -> Copy -> Address)
- По желанию, заголовок "Dump2" можно переименовать.
- В окне CPU пишем код: `push 0x_адрес_ячейки`

#### 3. Вызываем `scanf`

- Ищем адрес вызываемой функции. Двойной клик по вызову любой функции в коде.
Переход на секцию с `jmp`, где заданы адреса функций.

- В примере нужен адрес функции `scanf`. Находим строку `jmp dword ptr ds:[<&scanf>]`
и копируем ее адрес в буфер обмена (`Alt+Ins` или Адрес -> ПКМ -> Copy -> Address)

- Для перехода в начало программы два раза кликнуть на регистре `EIP`.
Регистр `EIP` содержит адрес entry point программы.

- Делаем `call` адреса ячейки, которая указывает на функцию:
`call 0x_адрес_функции`

#### 4. Ввод числа в консоль программы

Запускаем, пошагово выполняем, после вызова scanf, в окне программы вводим число.

#### 5. В целевой ячейке памяти появляется введенное число в hex формате

В памяти `.bss` появляется введенное число в hex формате.

## 09.26. Ввод и вывод числа через `scanf` и `printf`

Псевдо-код asm. Ввод (см. выше):

```text
push addr-of-num
push "%d"
call scanf
```

Вывод (см. еще более выше):

```text
push dword ptr ds:[addr-of-num]
push "%d"
call printf
```

- `dword` какой размер данных пишем (4 байта)
- `ptr` указатель
- `ds` - data segment
- `[0x....]` - адрес памяти откуда читаем

## 09.27. Ввод и вывод строки через `scanf` и `printf`

**Важно**: `scanf` читает строку до первого пробела.

Псевдо-код asm. Ввод строки:

```text
push addr-of-str
push "%s"
call scanf
```

Вывод строки:

```text
push addr-of-str        // здесь указывается адрес строки
push "%s"
call printf
```

Итоговый код (как пример):

```asm
push 0x0405000          // адрес строки (куда сохраняется)
push 0x0403030          // "%s"
call <JMP.&scanf>       // адрес scanf
push 0x0405000          // адрес строки (откуда читается)
push 0x0403030          // "%s"
call <JMP.&printf>      // адрес printf
```

## 10.28. Code cave

Что такое code cave:

- Blank memory area in the `.TEXT` segment
- `.TEXT` segment is Executable region
- Therefore any code injected here will execute

Code caves находятся в самом низу:

![Code caves](/28-code-caves/01_code-caves.jpg)

Здесь инструкции `add byte ptr ds:[eax],al` являются `junk instructions`

### Пояснение google ИИ

Code Cave («пещера кода») - это последовательность неиспользуемых ("пустых") байтов в памяти
процесса или внутри скомпилированного исполняемого файла (например, .exe или .dll).

Этот участок обычно заполнен нулевыми байтами (0x00) или инструкциями NOP (0x90)
и используется для внедрения собственного стороннего кода без изменения общего размера файла.

Откуда берутся Code Caves?

Компиляторы создают эти «пещеры» автоматически, по двум основным причинам:

- Выравнивание данных (Alignment). Для оптимизации производительности процессора секции программы
(код, данные, ресурсы) выравниваются по границам страниц памяти (например, кратными 4096 байтам).
Всё оставшееся свободное место в конце секции заполняется нулями.

- Разделение функций. Небольшие отступы из пустых байтов могут добавляться между
отдельными функциями для корректного перехода процессора.

### Использование code cave. Зачем и как

- Run out of space to write code
- So, jump to code cave to inject more code
- Then jump back to after jump point

![Use code caves](/28-code-caves/02_use-code-caves.jpg)

### Ограничение code cave по размеру. Как определить доступный размер

В памяти размер области code cave больше чем размер пустой области, доступной в файле:

```text
virtual memory > raw (file memory)
```

#### Как определить доступный размер code cave

2 программы

1. **`PE-bear`**

File -> Load PEs -> Открыть исследуемый файл

Вкладка "Section Hdrs" -> Посмотреть offset, где заканчивается секция `.text` и начинается секция `.data`

Например, `.data` начинается с offset `1C00`.

2. **`HxD`** (есть 32х и 64х битные версии - в примере исользуется 32х)

- Открывается исследуемый файл
- Ищем offset `1C00`
- Секция кода выше, где расположены `00`, и есть code cave

В примере, начало code cave по смещению `1B00`, конец по смещению `1BFF`, т.е. всего `FF`
(`1BFF - 1B00`) дополнительно байт доступно для code cave в реальности.

## 10.29. Jump (`JMP`) to the code cave

Как перейти из выполняемого кода в code cave - используется инструкция

>JMP 0xАдрес

```asm
// метод main()
00401530    push epb
00401531    mov epb,esp
00401533    jmp 0x00402701      // переход в code cave
00401538    ...                 // возврат из code cave
...
...
004026FF    какой-то код
// code cave
00402701    jmp 0x00401538      // возврат в main()
...
...
```

## 10.30. Упражнение. Использование code cave и `printf`

Вывести на экран первую строку из code cave, вторую строку из main.

Это если кратко:

```text
printf("Hello from code cave\n");

push str
call printf

printf("Back from code cave\n");

push str
call printf
```

1.1. Сначала строки помещаем в память

Нужно поместить 2 строки в память:

- "Hello from code cave\n"
- "Back from code cave\n"

Для каждой из строк

- Окно "Memory Map", двойной ЛКМ по секции `.data`
- В окне "Dump" выбираем свободные ячейки в 4-х столбцах (16 значений `00`)
- Открываем редактор `Ctrl+ E` или ПКМ -> Binary -> Edit
- В ASCII поле вставляем строку
- Убеждаемся что стоит флажок `Keep Size`
- `OK`
- Копируем адрес строки в буфер обмена (`Alt+Ins` или Адрес -> ПКМ -> Copy -> Address)

Примечание:

- строка должна заканчиваться символом `00` (null terminator).
- символ перевода строки "\n" в памяти записывается как ASCII символ `0x0A`.

1.2. В области code cave делаем `push` адреса ячейки, которая указывает на строку:

```asm
push 0x<Адрес_строки_1>
```

2.1. Ищем адрес вызываемой функции. Двойной клик по вызову любой функции в коде.
Переход на секцию с `jmp`, где заданы адреса функций.

2.2. В примере нужен адрес функции `printf`. Находим строку `jmp dword ptr ds:[<printf>]`
и копируем ее адрес в буфер обмена (`Alt+Ins` или Адрес -> ПКМ -> Copy -> Address)

2.3. В области code cave делаем:

```asm
call 0x<Адрес_printf>
```

3.1. Делаем `jmp 0x<Адрес_main>`

Итого в code cave:

```asm
push 0x<Адрес_строки_1>     // Переход из main, помещение в стек первой строки
call 0x<Адрес_printf>       // Вывод на экран первой строки
jmp 0x<Адрес_main>          // Возврат в main
```

4.1. В main все делаем также, как и в code cave, только для второй строки:

```asm
jmp 0x<Адрес_code_cave>     // Переход в code cave
push 0x<Адрес_строки_2>     // Возврат из code cave, помещение в стек второй строки
call 0x<Адрес_printf>       // Вывод на экран второй строки

## 10.31. Упражнение. Получение имени, фамилии и вывод на экран

```text
Write a program that can prompt the user for firstname,
lastname, then print a message:

Hello, firstname lastname

Example output:

Enter firstname: Paul
Enter lastname: Chin
Hello, Paul Chin

Use the code cave once you run out of memory.
```

### Решение

- Адрес "Enter firstname: " `0x00403030`
- Адрес "Enter lastname: " `0x00403050`
- Адрес "Hello, %s %s\n" `0x00403070` (символ `\n` записывается в ASCII как `0A`)
- Адрес "%s" `0x00403044`
- Адрес функции "scanf" `0x00402604`
- Адрес функции "printf" `0x00402614`
- Адрес для имени (в .bss) `0x00405000`
- Адрес для фамилии (в .bss) `0x00405050`

```asm
push ebp            // Main
mov ebp,esp
jmp 0x0402701       // Jump to code cave
...

push 0x00403030      // "Enter firstname: "
push 0x00403044      // "%s"
call 0x00402614      // Call printf("%s", str)
push 0x00405000      // Адрес записи для имени (в .bss)
push 0x00403044      // "%s"
call 0x00402604      // Call scanf("%s", str)
push 0x00403050      // "Enter lastname: "
push 0x00403044      // "%s"
call 0x00402614      // Call printf("%s", str)
push 0x00405050      // Адрес записи для фамилии (в .bss)
push 0x00403044      // "%s"
call 0x00402604      // Call scanf("%s", str)
push 0x00405050      // Адрес чтения фамилии (в .bss)
push 0x00405000      // Адрес чтения имени (в .bss)
push 0x00403070      // "Hello, %s %s\n"
call 0x00402614      // Call printf("Hello, %s %s\n", firstname, lastname)
jmp 0x0401538       // Jump back to main
```

## 11.32-34. Упражнение. Калькулятор (только сложение)

```text
Write a program that can prompt the user for first number,
second number, then add the numbers and print a message:

firstnum + secondnum = sum

Example output:

Enter firstnum: 2
Enter secondnum: 4
2 + 4 = 6


Use the code cave once you run out of memory.
```

### Решение

- Адрес (`.data`) "Enter firstnum: " `0x00403030`
- Адрес (`.data`) "Enter secondnum: " `0x00403050`
- Адрес (`.data`) "%d + %d = %d\n" `0x00403070`
- Адрес (`.data`) "%d" `0x00403080`
- Адрес (`.data`) "%s" `0x00403084`
- Адрес функции `scanf` `0x00402604`
- Адрес функции `printf` `0x00402614`
- Адрес (`.bss`) первого числа `0x00405000`
- Адрес (`.bss`) второго числа `0x00405010`
- Адрес начала code cave `0x00402701`
- Адрес возврата в main `0x00401565`

**Напоминание. Сложение**

```text
Destination <- Destination + Source
add eax, ecx            // eax <- eax + ecx
```

**Напоминание. Чтение значения из памяти в регистр**

```text
MOV ebx, dword ptr ds:[0x00403050]
```

```asm
Main
===
push 0x00403030                     // "Enter firstnum: "
push 0x00403084                     // "%s"
call 0x00402614                     // printf("%s", str)
push 0x00405000                     // Адрес (`.bss`) первого числа
push 0x00403080                     // "%d"
call 0x00402604                     // scanf("%d", num)
push 0x00403050                     // "Enter secondnum: "
push 0x00403084                     // "%s"
call 0x00402614                     // printf("%s", str)
jmp 0x00402701                      // Переход в code cave
...
...
Code Cave
===
push 0x00405010                     // Адрес (`.bss`) второго числа
push 0x00403080                     // "%d"
call 0x00402604                     // scanf("%d", num)
mov ecx, dword ptr ds:[0x00405000]  // 1 число в регистр ecx
mov eax, dword ptr ds:[0x00405000]  // 1 число в регистр eax
mov ebx, dword ptr ds:[0x00405010]  // 2 число в регистр ebx
add eax, ebx                        // Сложение чисел, запись результата в eax
push eax                            // Результат сложения в стек
push ebx                            // 2 число в стек
push ecx                            // 1 число в стек
push 0x00403070                     // "%d + %d = %d\n"
call 0x00402614                     // printf("%d + %d = %d\n", num0, num1, num2)
jmp 0x00401565                      // Возврат в main
```

Вариант решения в видео:

1. Печать текста без "%s":

```c
printf("Enter firstnum: ")
```

2. Сложение и вывод релизованы так:

```asm
mov eax, 0
add eax, dword ptr ds:[0x00405000]      // 1 слагаемое в eax
add eax, dword ptr ds:[0x00405010]      // Сумма
mov dword ptr ds:[0x00405020], eax      // Сохранение суммы
push dword ptr ds:[0x00405020]          // Сумма в стек
push dword ptr ds:[0x00405010]          // 2 слагаемое в стек
push dword ptr ds:[0x00405000]          // 1 слагаемое в стек
push 0x00403070                         // "%d + %d = %d\n"
call 0x00402614                         // printf("%d + %d = %d\n", num0, num1, num2)
```

## 12.35. Функции, которые возвращают значения

Ссылки на описание библиотек C:

`strlen()` function:

- [https://www.tutorialspoint.com/c_standard_library/c_function_strlen.htm](https://www.tutorialspoint.com/c_standard_library/c_function_strlen.htm)

```c
size_t strlen(const char *str)
```
Вызвать функцию с параметром:

>push параметр
>call 0x<адрес_strlen>

Функция возвращает число в hex, которое записывается в регистр `EAX`

### Упражнение

Вызвать `strlen("hello world")`, функция возвращает `B` (11) и записывает это число в `EAX`

```asm
push 0x00403040         // "hello world" из .data
call 0x<addr_strlen>    // вызов strlen("hello world") и сохранение возвращенного числа в `eax`
```

## 12.36. Упражнение. Определение длины введенного слова и вывод результата


Write a program that can prompt the user to enter a word,
then count the number of characters in that word and print a message:

word has numcharacters

Example output:

```text
Enter a string: apple
apple has 5 characters
```

### Решение

- адрес "Enter a string: " - `0x00403040`           // .data
- адрес "%s has %d characters\n" - `0x00403060`     // .data
- адрес "%s" - `0x00403054`                         // .data
- адрес куда будет сохраняться ввод - `0x00405000`  // .bss
- адрес `printf()` - `0x00402614`
- адрес `scanf()` - `0x00402604`
- aдрес `strlen()` - `0x004025F4`

`\n` символ в ASCII как `0A`

```asm
push 0x00403040     // "Enter a string: "
call 0x00402614     // printf("Enter a string: ")
push 0x00405000     // адрес куда сохранять введенное слово
push 0x00403054     // "%s"
call 0x00402604     // scanf(%s, str)
push 0x00405000     // читаем введенное слово
call 0x00402604     // strlen(str)
push eax            // число символов в str
push 0x00405000     // введенное слово
push 0x00403060     // "%s has %d characters\n"
call 0x00402614     // printf("%s has %d characters\n", str, num)
```

## 13.37-40 Flags Register

![Flags register](/37-flags-register/01_flags-register.jpg)

### `ZF` (The Zero Flag)

- `ZF` is set to 1 when the last calculation results in zero
- `ZF` is cleared to 0 when the last calculation results in non-zero

1. Example:

```text
mov eax, 0x8
mov ecx, 0x8
sub eax, ecx
```

After the sub instruction, `ZF` is **set** to **1**

2. Example:

```text
mov eax, 0x6
mov ecx, 0x6
add eax, ecx
```

After the add instruction, `ZF` will be **cleared** to **0**

### `SF` (The Sign Flag)

- `SF` equals the most significant bit of the last calculation

Used in Two’s Complement Number Representation

- `SF` = 0 means positive
- `SF` = 1 means negative

1. Example:

```text
mov edx, 0
dec edx
```

`SF` = 1, because edx = 0xFFFFFFFF = **1**111.........1111

2. Example

```text
mov edx, 0
inc edx
```

`SF` = 0, because edx = 0x00000001 = **0**000.........0001

### `CF` (The Carry Flag)

`CF` = 1 if the addition of two numbers causes a carry out of the most significant bit. A wrap-around has occurred.

1. Example

```text
mov eax, 0xFFFFFFFF
add eax, 0x1            // eax = 0, CF = 1
```

Means the result you get from the addition is wrong

![CF flag on add](/37-flags-register/02_cf-add.jpg)

2. Example

The `CF` will also be set to 1 if a subtraction requires a borrow from the most significant bit. A wrap-around also occurs.

```text
mov ecx, 0x0
mov edx, 0x3
sub ecx, edx        // ecx = 0xFFFFFFFD, CF = 1
```

![CF flag on sub](/37-flags-register/03_cf-sub.jpg)

3. Example

The `CF` will be cleared to 0 if no Carry occurs. No wrap-arounds.

```text
mov eax, 0x2
mov ecx, 0x8
add eax, ecx            // eax = 0xA, CF = 0
```

### `OF` (The Overflow Flag)

If we assume the numbers are two complements representation (signed numbers), then the `OF` is set to 1 if:

- the addition of two positive numbers -> negative result
- the addition of two negative numbers -> positive result
- positive – negative -> negative result
- negative – positive -> positive result

If the `OF` = 1, it means the result you get from the calculation is wrong

1. Example 1

```text
mov eax, 0x7FFFFFFF
mov edx, 0x1
add eax, edx        // eax = 0x80000000, OF = 1
```

![Set OF to 1](/37-flags-register/04_set-of-to-1.jpg)

2. Example 2

```text
mov eax, 0x7FFFFFFF
mov edx, 0x1
sub eax, edx        // eax = 0x7FFFFFFE, OF = 0
```

![Set OF to 0](/37-flags-register/05_set-of-to-0.jpg)

## 13.41. When to look at `CF` or `OF`

- Both `CF` (Carry Flag) and `OF` (Overflow flag) will change in
every arithmetic operation.

- Depending on how the numbers are being interpreted, you will then
look either at the `CF` flag or the `OF` flag.

- If you program is using unsigned numbers, the you will be
concerned (смотрите на) with the `CF` flag.

- But if you program works with signed numbers, then you will care
about the `OF` flag.

### In Summary

- `ZF` flag set to 1 if the last result was zero
- `SF` flag is set 1 if the last result was negative
- `CF` flag is set 1 if result (numbers unsigned) is wrong
- `OF` flag is set 1 if result (numbers signed) is wrong

## 14.42-45. Intro to JUMPS

2 типа JUMPS:

- Un-conditional jumps

```asm
jmp 0x04012428
```

- Conditional jumps

```asm
jz 0x04012428
jnz 0x04012428
```

Conditional JUMPS based on values inside flags register

### `JZ` (Jump Zero) / `JE` (Jump Equal)

jumps only if the `ZF` is set to **1** (i.e. the result of the last calculation is **zero**)

В x32dbg и x64dbg инструкция `JZ` может обозначаться как `JE` (Jump Equal)

```asm
           mov eax, 0x1
           dec eax          // ZF = 1
           jz  0x04012428   // ---
           add eax, 0x5     //    |
0x04012428 add eax, 0x2     // <--
```

Jump is taken, `EAX = 2`

### `JNZ` (Jump Not Zero) / `JNE` (Jump Not Equal)

jumps only if the `ZF` is cleared to **0** (i.e. the result of the last calculation is **non-zero**)

В x32dbg и x64dbg инструкция `JNZ` может обозначаться как `JNE` (Jump Not Equal)

```asm
           mov eax, 0x1
           inc eax          // ZF = 0
           jnz 0x04012428   // ---
           add eax, 0x5     //    |
0x04012428 add eax, 0x2     // <--
```

Jump is taken, `EAX = 4`

### Simple Loop (using `JZ`)

```asm
           mov eax, 0x0
           mov ecx, 0x3
0x04012428 add eax, ecx
           dec ecx
           jz  0x04012464
           jmp 0x04012428
0x04012464
```

Calculates: 3 + 2 + 1 = 6

### Exercize. Еще один loop `JZ`

Calculate: 1 + 2 + 3 ... + 100 = 5050 (0x13BA)

```asm
           mov eax, 0x0
           mov ecx, 0x64        // здесь модификация: 0x64 = 100
0x04012428 add eax, ecx
           dec ecx
           jz  0x04012464
           jmp 0x04012428
0x04012464
```

### Simple Loop (using `JNZ`)

```asm
           mov eax, 0x0
           mov ecx, 0x3
0x04012428 add eax, ecx
           dec ecx
           jnz 0x04012428
```

Calculates: 3 + 2 + 1 = 6

## 14.46. Other jumps

### `JS` and `JNS`

- `JS` = Jump if `SF` (sign flag) is set to **1**
- `JNS` = Jump if `SF` is cleared to **zero**

### `JC` and `JNC`

- `JC` = Jump if `CF` (carry flag) is set to **1**
- `JNC` = Jump if `CF` is cleared to **zero**

### `JO` and `JNO`

- `JO` = Jump if `OF` (overflow flag) is set to **1**
- `JNO` = Jump if `OF` is cleared to **zero**

### Reading Flags Register Indirectly

- Cannot read Flags Register directly
- So, we can use the conditional jumps as an indirect way to read the Flags Register

## 15.47-48. `CMP` instructions. `JC` / `JB` jumps

- `CMP` is used to compare 2 numbers
- `A – B`, then changes flags accordingly but does not change `A` or `B`

>CMP A, B

Где `A` и `B` это любые регистры (`eax`, `ebx`, ...)

- `JC` is also known as `JB` (Jump if Below)

`JC = 1` или `JB = 1` если `A < B`

### Пример

```text
cmp eax, abx

Это:
eax - ebx:
if eax <  ebx, CF = 1
if eax == ebx, CF = 0
if eax >  ebx, CF = 0
```

```asm
           mov ecx, 0x0
           mov eax, 0x1
           mov ebx, 0x2
           cmp eax, ebx
           jc 0x00401547      // eax < ebx --
           inc ecx            //             |
0x00401547 inc ecx            // <-----------
```

Результат при разных значениях `eax` и `ebx`:

```text
if eax <  ebx,   then ecx == 1
if eax == ebx,   then ecx == 2
if eax >  ebx,   then ecx == 2
```

## 15.49. Comparing unsigned and signed numbers

Допустим есть 2 числа: `0xFFFFFFFF` и `0x00000001`

Если они рассматриваются как беззнаковые (unsigned):

```text
0xFFFFFFFF  > 0x00000001
```

Но, если они рассматриваются как числа со знаком (signed):

```text
0xFFFFFFFF  < 0x00000001
```

Есть отдельные инструкции для сравнения беззнаковых и знаковых чисел.

### `CMP` **unsigned** numbers. `JB`, `JBE`, `JA`, `JAE` jumps

- `JB` (Jump Below)
- `JBE` (Jump Below or Equal)
- `JA` (Jump Above)
- `JAE` (Jump Above or Equal)

```text
JB      EAX < EBX
JBE     EAX >= EBX
JA      EAX > EBX
JAE     EAX >= EBX
```

### `CMP` **signed** numbers. `JL`, `JLE`, `JG`, `JGE` jumps

- JL (Jump if Less)
- JLE (Jump if Less or Equal)
- JG (Jump if Greater)
- JGE (Jump if Greater or Equal)

## 16.50-53. Intro to Structured Programming

Описание проблем безконтрольного использования инструкций jump

- Indiscriminate use of jump instructions may lead to spaghetti code
- Spaghetti code is code which is hard to read and understand
- Also difficult to debug

Решение - использование структурного программирования

- Use high-level programming constructs (HLPC) and implement them using assembly jumps
- Example of HLPC:
  - `if–else` blocks
  - `while`, `for` and `do-while` loops

### `IF-ELSE` statement

Описание псевдокодом. `IF-ELSE`

```text
if eax < edx:                   cmp eax, edx
                                jae else
    eax <- eax + 1              inc eax
                                jmp end_if
else:                       else:
    eax <- eax – 1              dec eax
enf if                      end_if:
```

- `JAE` - Jump Above or Equal (EAX >= EBX) unsigned

Описание псевдокодом. `IF`

```text
if eax < edx:                   cmp eax, edx
                                jae end_if
    eax <- eax + 1              inc eax
end if                      end_if:
```

### `FOR` Loops

- For iterating over a range of numbers
- Eg: Sum all numbers from 0 to 9 (inclusive)

```text
                                mov eax, 0x0
                                mov ecx, 0x0
for ecx from 0 to 9 do:
                            for_loop:
    eax <- eax + ecx            add eax, ecx
                                inc ecx
                                cmp ecx, 0xA
                                jne for_loop
end for
```

- `JNE` Jump if Not Equal (ECX != 0xA) unsingned

### `WHILE` Loops

- Keep looping as long as some condition is true
- Eg: Sum all numbers from 0 to 9 (inclusive)

```text
eax <- 0                        mov eax, 0x0
ecx <- 0                        mov ecx, 0x0
while ecx < 10 do:          while_loop:
                                cmp ecx, 0xA
                                jae end_while
    eax <- eax + ecx            add eax, ecx
    ecx <- ecx + 1              inc ecx
                                jmp while_loop
end while                   end_while:
```

- `JAE` - Jump Above or Equal (EAX >= EBX) unsigned

### `BREAK` out of Loops

- `BREAK`, allows you to exit the loop prematurely
- Eg: Sum all numbers from 0 to 9 , but exit if the sum is >= 5

```text
eax <- 0                        mov eax, 0x0
ecx <- 0                        mov ecx, 0x0
for ecx 0 to 9 do:          for_loop:
    eax <- eax + ecx            add eax, ecx
    if eax >= 5:                cmp eax, 0x5
    break                       jae end_for
                                inc ecx
                                cmp ecx, 0xA
                                jb for_loop
end for                     end_for:
```

- `JAE` - Jump Above or Equal (EAX >= EBX) unsigned
- `JB` - Jump Below (EAX < EBX) unsigned

### Principles of JUMPS

- Every jump should part of:
  - an `IF` statement
  - a `FOR` loop
  - a `WHILE` loop
  - a `BREAK` from a loop
- If a jump is none of the above, then you should not use a jump.

### Direction of jumps

- Forward jumps are part of `IF` or `BREAK` statements
- Jump backwards are used in `FOR` or `WHILE` loops

### Nesting `IF`, `FOR` and `WHILE` constructs

- You may also nest the `IF`, `FOR` and `WHILE` inside each other.
- But avoid too deep nesting – as it makes it difficult to read and understand the code

### Avoid jumps that intersect each other

- Above can be avoided if you design in pseudocode in HLPC first
- Then convert it to assembly

## 16.54. Graph view Trace Animate and Principles of Jumps

В x32dbg и x64dbg можно вывести графическое представление блоков выполняемого кода с более явным
показом jumps между блоками. Это облегчает анализ кода.

Переключение между "плоским" и "блочным "отображением кода:

`G` или ПКМ -> Graph

В блочном представлении можно зуммировать область:

Ctrl + Колесо мыши

Если не зуммируется, то включить опцию: `Options -> Preferences -> GUI -> флаг Graph zoom mode`

Чтобы увидеть какой код был выполнен а какой нет, можно включить опцию

`Tracing -> Trace coverage -> Word`

### Пояснение про tracing от ИИ

В контексте низкоуровневой трассировки и анализа покрытия кода (Trace Coverage) понятия
Bit, Byte и Word отражают то, как именно дебаггер хранит информацию о посещении адресов
в памяти компьютера.Выбор между ними - это всегда компромисс между скоростью работы процессора
и расходом оперативной памяти.

#### 1. Bit Coverage (Битовое покрытие)

При таком подходе на каждую инструкцию (или каждый байт исполняемого кода) выделяется ровно
1 бит в памяти дебаггера.

- Как это работает:
Если адрес не посещался - бит равен 0.
Как только процессор выполнил инструкцию по этому адресу - бит переключается в 1.

- Плюсы: Экстремальная экономия памяти.
Для карты покрытия целого мегабайта кода нужно всего около 131 КБ памяти.

- Минусы: Теряется информация о частоте. Вы знаете только факт: «код выполнялся хотя бы один раз».
Вы не узнаете, выполнился он 1 раз или 1 000 000 раз.

#### 2. Byte Coverage (Байтовое покрытие)

Здесь на каждый адрес выделяется 1 байт (8 бит) информации.

- Как это работает:
Вместо простого переключателя 0/1 дебаггер использует счетчик.
При каждом прохождении через инструкцию значение увеличивается на 1.

- Лимит: Максимальное число посещений, которое может записать один байт - 255.
Если код выполнится 256 раз, счетчик либо «застынет» на 255 (насыщение),
либо сбросится в 0 (переполнение), в зависимости от реализации дебаггера.

- Плюсы: Позволяет видеть базовую интенсивность кода (например, сразу заметны простые циклы).

- Минусы: Памяти требуется в 8 раз больше, чем при битовом подходе.

#### 3. Word Coverage (Вордовое покрытие / 2 байта)

На каждый адрес выделяется 1 машинное слово (Word), что в архитектуре x86/x64 составляет
2 байта (16 бит).

- Как это работает: Полноценный счетчик посещений с максимальным значением 65 535.

- Плюсы: Отлично подходит для анализа тяжелых циклов и алгоритмов фильтрации данных,
где код «крутится» тысячи раз.
Помогает при поиске криптографии (там счетчики на определенных участках резко возрастают).

- Минусы: Расход памяти увеличивается в 16 раз по сравнению с Bit Coverage.
На больших бинарных файлах (сотни мегабайт) это может вызвать ощутимую нагрузку.

(Примечание: В самых мощных профайлерах иногда используется Dword (4 байта) с лимитом
до ~4 миллиардов, но для обычной трассировки в дебаггерах вроде x64dbg это избыточно).

#### Применение в x64dbg / win32dbg

1. Для быстрого поиска «мертвого кода» (который никогда не вызывается) - используют Bit.

2. Для поиска уязвимостей (Fuzzing) и замера производительности - используют Byte или Word,
чтобы видеть, какие ветви кода выполняются чаще всего (горячие точки / Hot Spots).

### Animate over (Анимированный обход)

В контексте отладки в x64dbg / x32dbg (win32dbg), опция Animate over (Анимированный
обход) - это встроенный режим автоматической пошаговой отладки.

Она заставляет дебаггер самостоятельно нажимать клавишу F8 (Step Over) через
фиксированные микроинтервалы времени, визуально прокручивая исполняемый код на экране.

#### Как это работает и в чем разница с другими режимами

В меню `Tracing` обычно доступны две похожие опции анимации:

- Animate over (Ctrl + F8): Дебаггер шагает по коду автоматически.
Если он встречает вызов функции (CALL), он не заходит внутрь, а выполняет её целиком
за один шаг (как при Step Over) и переходит к следующей строке текущей функции.

- Animate into (Ctrl + F7): Дебаггер также шагает сам, но при встрече CALL он
проваливается внутрь вызываемой функции (как при Step Into).

#### Главные задачи Animate over

- Визуальный поиск «подвисаний»: Если программа зациклилась, вы можете включить
анимацию и своими глазами увидеть, между какими ассемблерными инструкциями
(например, `CMP` и `JNE`) безостановочно прыгает выполнение.

- Анализ без рук: Вам не нужно тысячу раз нажимать F8 вручную. Вы просто смотрите на
экран, пока код выполняется в замедленном темпе.

- Совместная работа с Trace Coverage: Когда включена анимация, x64dbg успевает на лету
закрашивать посещенные адреса в зеленый цвет. Это превращает процесс в наглядную карту
покрытия, заполняемую в реальном времени.

#### Полезный лайфхак: Настройка скорости

По умолчанию анимация может идти слишком быстро или слишком медленно. Её скорость регулируется в настройках:

`Options -> Preferences -> вкладка Engine -> параметр Animation delay`

Например, значение 100 сделает 10 шагов в секунду, что идеально
для комфортного чтения кода глазами.

Остановить процесс автоматического шага можно в любой момент,
просто нажав `F12 (Pause)` или кнопку паузы на панели инструментов.

### Почистить кэш xdbg

`Директория, куда уставновлен xdbg/release/x32/db`

Там хранятся временные файлы.
