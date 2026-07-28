# Windows GUI Programs

## 2-4. Взлом `CrackMe1.exe`

Файл, который будет ломаться, здесь - [crackMe1.zip](../challenges/crackMe1.zip)

[Описание](../challenges/description.md)

Задание:

- Найти валидный серийный номер внутри файла
- Сделать так, чтобы валидным был любой вводимый текст

### Анализ PE файлов при помощи утилиты "Detect It Easy"

Для первичного анализа файла используется утилита `DIE` (Detect It Easy)

Подробнее см. в [02_x64dbg_debugger/4.3. Утилита `DIE` (Detect It Easy)](../02_x64dbg_debugger/x64dbg_debugger.md)

```text
Адрес EntryPoint программы = EntryPoint + ImageBase = 0x000013cf + 0x00400000 = 0x004013cf
```

Именно адрес `0x004013cf` является входной точкой программы `CrackMe1.exe`.
На этом адресе происходит останов при запуске программы в xdbg по F9 (run).

### Первичная настройка x32dbg/x64dbg

- Выключить TLS Callbacks
- В Ignored Exceptions добавить значение `0x00000000 - 0xFFFFFFFF`

Подробнее см. в [02_x64dbg_debugger/5. Debugger Stepping Basics/Setting Preferences. Предварительная настройка x32dbg/x64dbg)](../02_x64dbg_debugger/x64dbg_debugger.md)

## 5. Setting breakpoints on strings

На примере оконного приложения `CrackMe1.exe`

Последовательность шагов:

- Открываем файл в x32dbg/x64dbg
- Search for strings for bad message
- Put BP (breakpoint) on bad message
- Search for where serial key is being compared
- Put BP on the comparison instruction
- Extract the Serial Key

Под "bad message" понимается сообщение об ошибке, когда введено неправильное значение и т.п.

### Search for strings for bad message

Узнаем из неуспешного запуска сообщение об ошибке.

Запускаем программу в x32dbg/x64dbg -> `Run` (F9)

ПКМ -> Search For -> Current Module -> String references

Можем искать по подстроке в `Search:`

Подробнее см. в [02_x64dbg_debugger/7.2. Setting Breakpoints on Strings/Setting Preferences/2 Способ поиска строк, для больших программ)](../02_x64dbg_debugger/x64dbg_debugger.md)

Находим строку вида: "Wrong serial key. Try again", переходим по адресу этой строки.

### Search for where serial key is being compared

Пропускаем установку breakpoint на bad message. Видим чуть выше success message.

А еще выше видим инструкцию условного перехода `JNE`, которая выбирает какое сообщение будет показано.

 `JNE` - это Jump Not Equal, см. в [01_assembly_language/14.42-45. Intro to JUMPS/`JNZ` (Jump Not Zero) / `JNE` (Jump Not Equal)](../01_assembly_language/assembly_language.md)

Ставим breakpoint на `JNE`.

Выше идет сравнение значений в регистре, а еще выше чтение сообщения из поля ввода текста:

```asm
call dword ptr ds:[<&GetDlgItemTextA>]          // чтение ввода, ставим breakpoint сюда
...
lea eax, dword ptr ss:[ebp-30]                  // в eax появляется введенное значение
...
инструкции mov, cmp, jne
```

В ecx видна строка со значением, с которым производится сравнение eax.
В ecx содержится валидное слово.

Если это валидное слово ввести в окно `CrackMe1.exe`, то будет получено сообщение "Well done!"

## 6. Windows API functions

Windows API Functions = win32 API

Описания win32 функций смотреть на MSDN. Например для `MessageBox`:

[MSDN - MessageBox](https://learn.microsoft.com/ru-ru/windows/win32/api/winuser/nf-winuser-messagebox)

Или для функции `GetDlgItemTextA`:

[MSDN - GetDlgItemTextA](https://learn.microsoft.com/ru-ru/windows/win32/api/winuser/nf-winuser-getdlgitemtexta)

(Извлекает заголовок или текст, связанный с элементом управления в диалоговом окне.)

### 6.1. `MessageBox`

```asm
push 0                                  // [ Button Type ]
push "Sorry!"                           // [ Caption ]
push "Wrong serial key. Try again!"     // [ Text ]
push 0                                  // [ Parent Window ]
call MessageBox
```

Функция `MessageBox` - отображает модальное диалоговое окно, содержащее
системный значок, набор кнопок и краткое сообщение для конкретного приложения,
например сведения о состоянии или ошибке. Окно сообщения возвращает
целочисленное значение, указывающее, какую кнопку нажал пользователь.

Перед вызовом функции с параметрами все параметры
сначала помещаются в стек (инструкция 'PUSH'), а потом вызывается сама функция.

`MessageBox` (в C++) вызывается с 4 параметрами:

```cpp
int MessageBox(
  [in, optional] HWND    hWnd,
  [in, optional] LPCTSTR lpText,
  [in, optional] LPCTSTR lpCaption,
  [in]           UINT    uType
);
```

## 7. Pushing parameters to the stack

The stack is reverse of the push, the stack grows from bottom up.

Все параметры передаются в стек в обратном поряжке. В случае с `MessageBox`,
начиная с 4 по 1, а потом вызывается сама функция.

```text
[--- Stack ---]
0
Wrong serial key. Try again!
Sorry
0
```

Более подробно про стек см. в [01_assembly_language](../01_assembly_language/assembly_language.md):

- 07.18. The stack. Операция `PUSH`
- 07.20 Pushing constants and strings to the stack
- 08.21. Funcions call (`CALL`)
- 08.22. Funcions call (`CALL`). Вызов функций с 2 параметрами (строка, строка)
- 08.23. Funcions call (`CALL`). Вызов функций с 2 параметрами (строка, число)
- 08.24. Funcions call (`CALL`). Вызов функций с 3 параметрами

## 8. Bypassing messages. Показ другого (нужного) сообщения

Для `CrackMe1.exe` помимо секретного слова надо еще модифицировать (patch) файл так, чтобы он
всегда показывал Congrats сообщение при вводе любого текста (см. [description.md](../challenges/description.md))

### 8.1. Поиск мест для patching файла

1. Ищем строку сообщения об ошибке через

ПКМ на окне -> Search for -> Current Module -> String references

Подробнее см. в [02_x64dbg_debugger/2 Способ поиска строк, для больших программ](../02_x64dbg_debugger/x64dbg_debugger.md)

2. Видим `JNE`, который указывает параметры и на вызов MessageBox с сообщением об ошибке.

Ставим на `JNE` BreakPoint.

3. В режиме отладки, находясь на `JNE` можно переключить jump на другую ветку, исправив флаг `ZF`.

Подробнее см. в [02_x64dbg_debugger/8. Reversing Jumps](../02_x64dbg_debugger/x64dbg_debugger.md)

К сожалению, в этом случае Congrats сообщение не будет показано (впрочем как и сообщение об ошибке)
из-за некорректной передачи параметров в стек перед вызовом MessageBox с поздравлениями:

```text
[--- Wrong parameter 1 --]
push 0                // Button type     Верно
push "Congrats!"      // Caption         Верно
push "Well done!"     // Text            Верно
push eax              // Parent Window   Неверно, в eax лежит FFFFFFFF (-1), и это передается в стек
call MessageBox       // Не вызывается из-за Parent Window
```

Правильные параметры, которые должны быть:

```text
push 0                // Button type
push "Congrats!"      // Caption
push "Well done!"     // Text
push 0                // Parent Window   В eax надо положить 0
call MessageBox       // Должно быть OK
```

### 8.2. Правки файла

1. Удаляем инструкцию `JNE`. Вместо нее добавляются 2 инструкции `NOP` (No Operation).

Как вставить `NOP` см. в [02_x64dbg_debugger/9.1. How to patch a program](../02_x64dbg_debugger/x64dbg_debugger.md)

2. В `eax` надо записать `0`, чтобы MessageBox можно было вызвать.

У нас 2 байта для новой инструкции.

НО, если в `EAX` поместить 0:

```asm
MOV eax, 0
```

Такая инструкция занимает 5 байт - нам не хватает места.

Решение, надо поместить 0 в младший регистр `AL`:

```asm
MOV al, 0
```

Такая инструкция уже занимает 2 байта - нам хватает места.

```text
EAX занимает 4 байта
AX  занимает 2 байта
AL  занимает 1 байт
FFFFFFFF     4 байта
```

![Register size](08-bypassing-messages/01_register_size.jpg)

3. Patch файл

Подробнее см. в [02_x64dbg_debugger/9.1. How to patch a program/Запомнить изменения в файле (Patching)](../02_x64dbg_debugger/x64dbg_debugger.md)

Будет всего 2 изменения.

Почему-то такой "финт" с AL срабатывает - цель достигнута.

## 8.3. Как я модифицировал

Считаю, что препод не правильно сделал изменения в файле.

Я переставил инструкции, чтобы они все уместились. И нормальный push добавил.

Было:

![Initial file](08-bypassing-messages/02_initial_file.jpg)

Снес 4 инструкции. 2 инструкции перенес просто выше и добавил `push 0`. Еще и свободное место осталось.

Стало:

![Cracked file](08-bypassing-messages/03_cracked_file.jpg)

Крякнутое приложение (моя версия) [CrackMe1-edit.zip](src/CrackMe1-edit.zip)

Или можно использовать серийник/пароль, выдернутый из файла [CrackMe1-serial.txt](src/CrackMe1-serial.txt)
