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
