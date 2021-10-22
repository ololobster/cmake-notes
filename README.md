Заметки по CMake
================

1. [Введение](#введение)
1. [Опции и переменные](#опции-и-переменные)
1. [Кросс-компиляция](#кросс-компиляция)
1. [Прочее](#прочее)

# Введение

Собрать проект:
1. Создать каталог для сборки:
   ```
   $ mkdir build && cd build
   ```
1. Сконфигурировать:
   ```
   $ cmake ..
   ```
1. Собрать:
   ```
   $ cmake --build .
   ```
Примечания:
1. В любой непонятной ситуации ставить пакет `extra-cmake-modules`.
1. На этапе конфигурации создаётся `Makefile`, так что можно собирать и с `make`.

Запустить авто-тесты:
```
$ ctest
```
Примечания:
1. Можно добавить переменную окружения `CTEST_OUTPUT_ON_FAILURE=1`.

# Опции и переменные

Для управления ходом компиляции у нас есть опции и переменные.
Их можно указать на этапе конфигурации при помощи `-D`.
Например, зададим переменную `MYVAR`:
```
$  cmake -DMYVAR=ololo ..
```

Язык CMake позволяет невозбранно выводить значения переменных, например:
```
message("MYVAR=${MYVAR}")
```

Опции имеют только 2 возможных значения, `ON` или `OFF`.
Объявить опцию:
```
option(⟨name⟩ "⟨help text⟩" ⟨default value⟩)
```
Примечание: значение по умолчанию можно не указывать — будет `OFF`.

Определить переменную (normal variable):
```
set(⟨variable⟩ ⟨value⟩)
```

Определить Cache Entry:
```
set(⟨variable⟩ ⟨value⟩ CACHE ⟨type⟩ ⟨docstring⟩ [FORCE])
```
Примечания:
1. Если переменной с таким именем нет, то CMake лезет в кеш.
1. По умолчанию существующие значения не перезаписываются.
   `FORCE` в помощь.

Также можно задать переменные в cmake-файле (при помощи `set()`) и указать на него при конфигурации: `-DCMAKE_TOOLCHAIN_FILE=⟨path⟩`.

# Кросс-компиляция

Примерный набор параметров для кросс-компилиции:
- `CMAKE_SYSTEM_NAME`, например, Linux;
- `CMAKE_SYSTEM_PROCESSOR`, например, mipsel;
- `CMAKE_C_COMPILER` и/или `CMAKE_CXX_COMPILER`;
- `CMAKE_INSTALL_PREFIX`.

Если нужно окружение, то задать:
- `CMAKE_SYSROOT`;
- `CMAKE_FIND_ROOT_PATH_MODE_PROGRAM` = `NEVER`;
- `CMAKE_FIND_ROOT_PATH_MODE_LIBRARY` = `ONLY`;
- `CMAKE_FIND_ROOT_PATH_MODE_INCLUDE` = `ONLY`;
- `CMAKE_FIND_ROOT_PATH_MODE_PACKAGE` = `ONLY`.

# Прочее

Флаги компилятора кладутся в `CMAKE_C_FLAGS`, `CMAKE_CXX_FLAGS`, `CMAKE_EXE_LINKER_FLAGS_INIT`, `CMAKE_SHARED_LINKER_FLAGS_INIT`, `CMAKE_MODULE_LINKER_FLAGS_INIT`.

Настройка генерации `config.h`:
```
#cmakedefine ENABLE_CPRO 1
```
Если переменная существует, то результатом будет `#define ENABLE_CPRO 1`.
См. также `#cmakedefine01`.
