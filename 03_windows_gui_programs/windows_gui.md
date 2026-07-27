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
