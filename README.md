Заметки по CMake
================

1. [Основы](#основы):
   [сборка существующего проекта](#сборка-существующего-проекта),
   [примеры](#примеры),
   [настройка цели](#настройка-цели)
1. [Опции и переменные](#опции-и-переменные)
1. [pkg_check_modules()](#pkg_check_modules)
1. [Кросс-компиляция](#кросс-компиляция)
1. [Прочее](#прочее)

# Основы

### Сборка существующего проекта

Собрать проект:
1. Создать каталог для сборки:
   ```
   $ mkdir build && cd build
   ```
1. Сконфигурировать:
   ```
   $ cmake ..
   ```
   Если всё прошло успешно, то теперь у нас есть `Makefile`.
1. Скомпилировать:
   ```
   $ make
   ```
1. Установить:
   ```
   # make install
   ```

Примечания:
1. В любой непонятной ситуации ставить пакет `extra-cmake-modules`.
1. При компиляции можно вывести обращения к компилятору со всеми аргументами, поможет переменная окружения `VERBOSE=defined`.
1. При установке можно задать каталог:
   ```
   # make install DESTDIR=⟨path⟩
   ```

Запустить авто-тесты:
```
$ ctest
```
Примечания:
1. Можно добавить переменную окружения `CTEST_OUTPUT_ON_FAILURE=1`.

### Примеры

Корневой `CMakeLists.txt` для простейшего проекта с 1 исполняемым файлом `test`:
```
cmake_minimum_required(VERSION 3.10.0)

project(test-executable)

set(test_SOURCES
  test.cpp
)
add_executable(test ${test_SOURCES})
install(TARGETS test DESTINATION bin)
```

Корневой `CMakeLists.txt` для простейшего проекта с 1 библиотекой `libtest.so`:
```
cmake_minimum_required(VERSION 3.10.0)

project(test-lib)

set(test_SOURCES
  test.cpp
)
add_library(test SHARED ${test_SOURCES})
install(TARGETS test LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

set(test_PUBLIC_HEADERS
  test.hpp
)
install(FILES ${test_PUBLIC_HEADERS} DESTINATION include/test)
```

### Настройка цели

Задать флаги компилиции для цели `test` (исполняемого файла или библиотеки):
```
target_compile_options(test PRIVATE -Werror -I/usr/include/glib-2.0)
```
Примечания:
1. `target_compile_options()`, как и пр. функции ниже, не перезаписвает список флагов, а добавляет в конец новые значения, при этом дубли убираются.

Задать каталоги с заголовками для цели `test`:
```
target_include_directories(test PRIVATE /usr/include/glib-2.0 /usr/include/librsvg-2.0)
```
Примечания:
1. `-I` не нужно.

Задать флаги линковки для цели `test`:
```
target_link_libraries(test ${common_libs})
```

# Опции и переменные

Для управления ходом компиляции у нас есть опции и переменные.
Их можно указать на этапе конфигурации при помощи `-D`.
Например, зададим переменную `MYVAR`:
```
$ cmake -DMYVAR=ololo ..
```

Вывести значение переменной:
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

# pkg_check_modules()

`pkg_check_modules()` подключает к проекту библиотеку (модуль pkg-config).
Если это возможно, следует не дёргать `pkg_check_modules()`, а использовать определения библиотек от CMake (пакет `extra-cmake-modules`).

Чтобы команда `pkg_check_modules()` стала доступной, надо подключить `PkgConfig`:
```
find_package(PkgConfig)
```

Подключить библиотеку:
```
pkg_check_modules(⟨prefix⟩ ⟨pkg-config module⟩)
```
Подключить библиотеку, без которой сборка невозможна:
```
pkg_check_modules(⟨prefix⟩ REQUIRED ⟨pkg-config module⟩)
```
Примечания:
1. Будут созданы переменные `⟨prefix⟩_FOUND`, `⟨prefix⟩_VERSION`, `⟨prefix⟩_LDFLAGS`, `⟨prefix⟩_CFLAGS` и др.

Пример:
```
pkg_check_modules(POPPLER_GLIB poppler-glib)
message("POPPLER_GLIB_FOUND=${POPPLER_GLIB_FOUND}")
message("POPPLER_GLIB_VERSION=${POPPLER_GLIB_VERSION}")
```
Вывод:
```
-- Checking for module 'poppler-glib'
--   Found poppler-glib, version 20.09.0
POPPLER_GLIB_FOUND=1
POPPLER_GLIB_VERSION=20.09.0
```

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
