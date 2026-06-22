# x64dg debugger for reverse engeneering beginners

## 1-2. Installing tools

- Virtual Box c Windows 7/10
- xdbg (x32dbg и x64dbg)
- die (Detect It Easy)

## 3. Где скачать файлы CrackMe

Для этого курса:

[https://crackinglessons.com/category/challenges/](https://crackinglessons.com/category/challenges/)

Ссылки с сайта ведут в репозиторий:
[https://github.com/cspinstructor/github-crackmes](https://github.com/cspinstructor/github-crackmes)

Passwords to unzip crackme's:

```text
crackinglessons.com
```

Описание файлов crackme вместе с описанием локально доступны [тут](src/description.md)

## 4. Workspace for reverse engineering. Типичный маршрут reverse engineering

Скачиваем и распаковываем файл [01-mexican.zip](src/01-mexican.zip)

Информация по нему [здесь](src/description.md)

Типичный маршрут reverse engineering

### 4.1. Опционально. Добавить файл в исключение антивируса

Может ОС или антивирус автоматом удалить даже безобидный `crackme`.

Лучше всего добавить рабочую директорию, где находятся `crackme` файлы в исключения.

### 4.2. Желательно. Сделать снимок на виртуальной машине

В любой современной программе для виртуализации есть функция снимков состояния
(Snapshots). Это встроенный механизм, который работает как "точка восстановления"
в компьютерной игре. Если вы сделаете снимок, запустите вирус, и он полностью уничтожит
систему, вы сможете вернуть виртуалку к исходному состоянию за один клик.

Снимки в разных программах

- VirtualBox: Виртуалка в списке -> Кнопка Вкладки  -> Snapshots -> Take
- VMware Workstation: VM (верхний угол) -> Snapshot -> Take Snapshot
- Hyper-V: ПКМ по виртуал6ке -> Checkpoint

Порядок работы со снимками

1. Настроить чистую систему
2. Установить все инструменты (Detect It Easy, x64dbg)
3. Добавить папки в исключения
4. Сделать снимок

Как восстановить

1. Выключить виртуалку
2. Меню снимков -> Восстановить (Restore / Revert) к чистому снимку.

**Важное правило безопасности** перед запуском файла **отключить в виртуалке**:

1. Общие папки (Shared Folders)
2. Общий буфер обмена (Shared Clipboard)
3. Drag and Drop (перетаскивание файлов мышкой)
4. Сетевой адаптер (переведите в режим «Host-only» или отключите совсем)

### 4.3. Утилита `DIE` (Detect It Easy)

Узнать информацию о ломаемом файле. Пример, что можно узнать

- Type: `PE` - Portable Executable - тип файла, запускаемый файл
- EntryPoint - Адрес точки входа
- Size
- Compiler - компилятор
- Linker - линкер
- Может быть еще указан packer - упаковщик, может затруднить reverse

Кнопка `PE` и прочие кнопки раскрывает более подробную информацию по файлу.

От версии Detect It Easy зависит объем потенциально доступной информации по файлу.

Главное тут выяснить какой битности файл 32 или 64.
В зависимости от битности запускается x32dbg или x64dbg.

### 4.4. Disassembler/debugger `x32dbg`, `x64dbg`

Вводный рассказ по x32dbg. Рассказано про функции/кнопки open, restart.
Обозначены окна:

- dissasembler,
- memory, RAM (dump'ы)
- register panel,
- 2 окна со стеками
