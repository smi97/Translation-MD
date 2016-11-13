#Глава 34. Библиотека Boost.IOStreams.

###Содержание:

<hr>

- [Устройства](#devices)
- [Фильтры](#filtres)

<hr>

Эта глава описывает библиотеку [Boost.IOStreams](http://www.boost.org/doc/libs/1_62_0/libs/iostreams/doc/index.html). 
Boost.IOStreams разбивает хорошо известные потоки стандартной библиотеки на более мелкие составляющие. Данная библиотека определяет два понятия: *устройство*, которое описывает источники данных и *поток*, который описывает интерфейс для форматированного ввода/вывода, основанный на интерфейсе стандартной библиотеки. Поток, определемый библиотекой Boost.IOStreams, автоматически не подключен к источнику или приемнику данных.

Boost.IOStreams обеспечивает множество реализаций двух понятий. Например, с помощью устройства `boost::iostreams::mapped_file`, которое загружает файл в память частично или полностью. Поток `boost::iostreams::stream` может быть подключен к устройству как `boost::iostreams::mapped_file`, чтобы использовать знакомые операторы `operator<<` и `operator>>` для считывания и записи данных.

В дополнение к `boost::iostreams::stream`, Boost.IOStreams предоставляет поток `boost::iostreams::filtering_stream`, который позволяет вам добавлять фильтры данных. Например, вы можете использовать `boost::iostreams::gzip_compressor`, чтобы записать сжатые данные в формате GZIP.

Boost.IOStreams может быть использована для подключения к платформенно-зависимым объектам. Библиотека дает возможность подключения к хэндлам Windows или дескриптору файлов. Таким образом, объекты API-интерфейсов низкого уровня могут стать доступными на независимом от платформы коде C++. 

Классы и функции, предоставляемые Boost.IOStreams, определены в пространстве имен `boost::iostreams`. Здесь нет основного заголовочного файла. Так как Boost.IOStreams содержит не только заголовочные файлы, она должна быть прекомпилированна. Это важно, потому что в зависимости от того, как Boost.IOStreams будет скомпилирована, поддержка некоторых функций может отсутствовать.

<a name="devices"></a>
##Устройства

Устройства это классы, которые обеспечивают доступ для чтения и записи к объектам, которые обычно не участвуют в процессе, например файлы. Однако вы можете получить доступ к внутренним объектам, например к массивам, как к устройствам.

Устройства – это все лишь класс с функцией `read()` или `write()`. Устройство может быть связано с потоком, и вы можете прочитать или записать форматированные данные, быстрее, чем при обращении напрямую. 

<a name="Ex341"></a>
####Пример 34.1 Использование массива в качестве устройства с помощью `boost::iostreams::array_sink`

```c++
#include <boost/iostreams/device/array.hpp>
#include <boost/iostreams/stream.hpp>
#include <iostream>

using namespace boost::iostreams;
int main()
{
  char buffer[16];
  array_sink sink{buffer};
  stream<array_sink> os{sink};
  os << "Boost" << std::flush;
  std::cout.write(buffer, 5);
}
```

[Пример 34.1](#Ex341) использует устройство `boost::iostreams::array_sink` для записи данных в массив. 
Массив передается в качестве параметра конструктора. После этого устройство соединяется с потоком типа `boost::iostreams::stream`. Ссылка на устройство передается конструктору `boost::iostreams::stream` и тип устройства передается в качестве параметра шаблона для `boost::iostreams::stream`.

В примере используется оператор `operator<<`, чтобы записать в поток строку "Boost". Поток передает данные устройству. Поскольку устройство соединено с массивом, "Boost" сохраняется в первые пять элементов этого массива. Так как содержимое массива записывают в стандартный вывод, "Boost" будет отображаться при запуске примера.

<a name="Ex342"></a>
####Пример 34.2 Использование массива в качестве устройства с помощью `boost::iostreams::array_source`

```c++	
#include <boost/iostreams/device/array.hpp>
#include <boost/iostreams/stream.hpp>
#include <string>
#include <iostream>

using namespace boost::iostreams;

int main()
{
  char buffer[16];
  array_sink sink{buffer};
  stream<array_sink> os{sink};
  os << "Boost" << std::endl;

  array_source source{buffer};
  stream<array_source> is{source};
  std::string s;
  is >> s;
  std::cout << s << '\n';
}
```

[Пример 34.2](#Ex342) основан на предыдущем. 
Строка, записанная с помощью `boost::iostreams::array_sink` в массив, читается с помощью `boost::iostreams::array_source`.

`boost::iostreams::array_source` используется как `boost::iostreams::array_sink`. В то время как `boost::iostreams::array_sink`  поддерживает только операцию записи, `boost::iostreams::array_source` поддерживает только чтение. `boost::iostreams::array` поддерживает и чтение, и запись. 

Стоит заметить, что в `boost::iostreams::array_source` и `boost::iostreams::array_sink` передается ссылка на массив. Массив не должен быть разрушен, когда устройство еще используется.

<a name="Ex343"></a>
####Пример 34.3 Использование вектора в качестве устройства с помощью `boost::iostreams::back_insert_device`

```c++
#include <boost/iostreams/device/array.hpp>
#include <boost/iostreams/device/back_inserter.hpp>
#include <boost/iostreams/stream.hpp>
#include <vector>
#include <string>
#include <iostream>
using namespace boost::iostreams;
int main()
{
  std::vector<char> v;
  back_insert_device<std::vector<char>> sink{v};
  stream<back_insert_device<std::vector<char>>> os{sink};
  os << "Boost" << std::endl;

  array_source source{v.data(), v.size()};
  stream<array_source> is{source};
  std::string s;
  is >> s;
  std::cout << s << '\n';
}
```

В [примере 34.3](#Ex343) применяется устройство типа `boost::iostreams::back_insert_device` вместо `boost::iostreams::array_sink`. Оно может быть использовано при записи данных в любой контейнер, который поддерживает метод `insert()`. Устройство использует этот метод, чтобы записать данные в контейнер.

В примере используется `boost::iostreams::back_insert_device` для записи в вектор строки "Boost". Затем эта строка читается из `boost::iostreams::array_source`. Ссылка на начало вектора и его размер передаются в качестве параметров конструктора `boost::iostreams::array_source`.

[Пример 34.3](#Ex343) выводит `Boost`. 

<a name="Ex344"></a>
####Пример 34.4 Использование файла в качестве устройства с помощью `boost::iostreams::file_source`

```c++
#include <boost/iostreams/device/file.hpp>
#include <boost/iostreams/stream.hpp>
#include <iostream>

using namespace boost::iostreams;

int main()
{
  file_source f{"main.cpp"};
  if (f.is_open())
  {
    stream<file_source> is{f};
    std::cout << is.rdbuf() << '\n';
    f.close();
  }
}
```

[Пример 34.4](#Ex344) использует метод `boost::iostreams::file_source` для чтения файлов. В то время как ранее примененные методы не предоставляют методы, `boost::iostreams::file_source` обеспечивает `is_open()`, чтобы проверить, был ли успешно открыт файл. Он также предоставляет функцию-член `close()` для явного закрытия файла. Вам нет необходимости применять `close()`, потому что деструктор `boost::iostreams::file_source` закроет файл автоматически.

Кроме `boost::iostreams::file_source`, Boost.IOStreams также предоставляет устройство `boost::iostreams::mapped_file_source` для загрузки файла в память частично или полностью. `boost::iostreams::mapped_file_source` определяет метод `data()` для получения указателя на соответствующую область памяти. Таким образом данные могут быть доступны в случайном порядке, без необходимости чтения файла последовательно. 

Для получения доступа к записи в файлы Boost.IOStreams представляет метод `boost::iostreams::file_sink` и `boost::iostreams::mapped_file_sink`.

<a name="Ex345"></a>
####Пример 34.5 Применение `file_descriptor_source` и `file_descriptor_sink` 
```c++
#include <boost/iostreams/device/file_descriptor.hpp>
#include <boost/iostreams/stream.hpp>
#include <iostream>
#include <Windows.h>

using namespace boost::iostreams;

int main()
{
  HANDLE handles[2];
  if (CreatePipe(&handles[0], &handles[1], nullptr, 0))
  {
    file_descriptor_source src{handles[0], close_handle};
    stream<file_descriptor_source> is{src};

    file_descriptor_sink snk{handles[1], close_handle};
    stream<file_descriptor_sink> os{snk};

    os << "Boost" << std::endl;
    std::string s;
    std::getline(is, s);
    std::cout << s;
  }
}
```

В [примере 34.5](#Ex345) используются `boost::iostreams::file_descriptor_source` и `boost::iostreams::file_descriptor_sink`. Это устройства, поддерживающие операции чтения и записи на платформенно-зависимых объектах. На ОС Windows такими объектами являются хэндлы, а на операционных системах POSIX - дескрипторы файлов.

В [примере 34.5](#Ex345) вызывается функция `CreatePipe()` чтобы создать канал. Концы чтения и записи канала передаются в массив **handles**. Конец чтения канала передается в устройство `boost::iostreams::file_descriptor_source`, и конец записи в `boost::iostreams::file_descriptor_sink`. Все, что записывается в поток **os**, который подключен к концу записи, может быть прочитано из потока **is**, который подключен к концу чтения. [Пример 34.5](#Ex345) отправляет строку "Boost" через поток и в стандартный вывод. 

Конструкторы `boost::iostreams::file_descriptor_source` и `boost::iostreams::file_descriptor_sink` принимают два параметра. Первый – хэндл Windows или, если программа запущена на системе POSIX, дескриптор файла. 
Второй параметр должен быть или `boost::iostreams::close_handle`, или `boost::iostreams::never_close_handle`. Этот параметр определяет, закрывает ли деструктор хэндл Windows или дескриптор файла.

<a name="filtres"></a>
##Фильтры

Кроме устройств Boost.IOStreams также предоставляет фильтры, которые работают с устройствами для фильтрации данных, прочитанных или записанных в устройства. Следующие примеры используют `boost::iostreams::filtering_istream` и `boost::iostreams::filtering_ostream`. Они заменяют `boost::iostreams::stream`, который не поддерживает фильтры.

<a name="Ex346"></a>
####Пример 34.6 Использование `boost::iostreams::regex_filter`

```c++
#include <boost/iostreams/device/array.hpp>
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/regex.hpp>
#include <boost/regex.hpp>
#include <iostream>

using namespace boost::iostreams;

int main()
{
  char buffer[16];
  array_sink sink{buffer};
  filtering_ostream os;
  os.push(regex_filter{boost::regex{"Bo+st"}, "C++"});
  os.push(sink);
  os << "Boost" << std::flush;
  os.pop();
  std::cout.write(buffer, 3);
}
```

[Пример 34.6](#Ex346) использует `boost::iostreams::array_sink`, чтобы записать данные в массив. Данные проходят через фильтр `boost::iostreams::regex_filter`, который заменяет символы. Фильтр принимает регулярное выражение и форматированную строку. Регулярное выражение описывает то, что надо заменить. Форматированная строка определяет, чем должны быть заменены символы. Пример заменяет "Boost" на "C++". Фильтр будет соответствовать одному или нескольким последовательным экземплярам буквы "о" в "Boost", но не нулям. 

Фильтры и устройства соединены потоком `boost::iostreams::filtering_ostream`.
Этот класс предоставляет функцию `push()`, в которую передаются фильтры и устройства.

Фильтр(-ы) должны быть переданы перед устройствами, порядок важен. Вы можете передавать один или несколько фильтров, но в тот момент, когда устройство передано, поток будет завершен, и вы не должны больше вызывать `push()`.

Фильтр `boost::iostreams::regex_filter` не может обрабатывать данные по символам, потому что регулярные выражения должны просматриваться группой символов. Поэтому `boost::iostreams::regex_filter` начинает фильтровать только после того, как завершится операция записи и все данные станут доступными. Это происходит, когда устройство удалено из потока при помощи `pop()`. [Пример 34.6](#Ex346) вызывает `pop()` после того, как "Boost" был записан в поток. Без вызова `pop()`, `boost::iostreams::regex_filter` данные не будут обработаны, и не будут переданы в устройство. 

Обратите внимание на то, что вы не должны использовать поток, который не связан с устройством. Тем не менее, вы можете завершить поток, если вы добавите устройство с помощью `push()` после вызова `pop()`.

[Пример 34.6](#Ex346) выводит `С++`.

<a name="Ex347"></a>
####Пример 34.7 Доступ к фильтрам в `boost::iostreams::filtering_ostream`

```c++
#include <boost/iostreams/device/array.hpp>
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/counter.hpp>
#include <iostream>

using namespace boost::iostreams;

int main()
{
  char buffer[16];
  array_sink sink{buffer};
  filtering_ostream os;
  os.push(counter{});
  os.push(sink);
  os << "Boost" << std::flush;
  os.pop();
  counter *c = os.component<counter>(0);
  std::cout << c->characters() << '\n';
  std::cout << c->lines() << '\n';
}
```

[Пример 34.7](#Ex347) использует фильтр `boost::iostreams::counter`, который считает символы и строки. Класс предоставляет методы `characters()` и `lines()`.

`boost::iostreams::filtering_stream` обеспечивает метод `component()` для доступа к фильтрам. 
Индекс соответствующего фильтра должен быть передан в качестве параметра. Поскольку `component()` – это шаблон, тип фильтра должен передан в качестве шаблонного параметра. `component()` возвращает указатель на фильтр (0 – если неверный тип фильтра передается в качестве шаблонного параметра).
[Пример 34.7](#Ex347) Записывает пять символов в поток. Он не записывает переход на новую строку (“\n”). Таким образом выводится `5` и `0`.

<a name="Ex348"></a>
####Пример 34.8 Запись и чтение данных, сжатых с помощью ZLIB

```c++
#include <boost/iostreams/device/array.hpp>
#include <boost/iostreams/device/back_inserter.hpp>
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/zlib.hpp>
#include <vector>
#include <string>
#include <iostream>

using namespace boost::iostreams;

int main()
{
  std::vector<char> v;
  back_insert_device<std::vector<char>> snk{v};
  filtering_ostream os;
  os.push(zlib_compressor{});
  os.push(snk);
  os << "Boost" << std::flush;
  os.pop();

  array_source src{v.data(), v.size()};
  filtering_istream is;
  is.push(zlib_decompressor{});
  is.push(src);
  std::string s;
  is >> s;
  std::cout << s << '\n';
}
```

[Пример 34.8](#Ex348) использует поток `boost::iostreams::filtering_istream` в дополнение к `boost::iostreams::filtering_ostream`. Этот поток используется, когда вам необходимо считать данные при помощи фильтра. В примере сжатые данные записываются и читаются снова.

Boost.IOStreams поддерживает несколько видов фильтров, касающихся сжатых файлов. `boost::iostreams::zlib_compressor` сжимает данные в формате ZLIB. Чтобы разархивировать данные из формата ZLIB, используется `boost::iostreams::zlib_decompressor`. Фильтры добавляются в поток при помощи `push()`. 

[Пример 34.8](#Ex348) записывает "Boost" в вектор **v** в сжатом виде и строку **s** в несжатом виде. 
Пример отображает `Boost`.

<blockquote>Обратите внимание на то, что в ОС Windows скомпилированная библиотека Boost.IOStreams не поддерживает сжатые данные, поскольку по умолчанию, библиотека скомпилирована с макросом `NO_ZLIB` на ОС Windows. Вам необходимо убрать определение этого макроса, определить ZLIB_LIBPATH и ZLIB_SOURCE, и перекомпилировать для получения поддержки ZLIB на ОС Windows. </blockquote>


#Глава 35. Boost.Filesystem

###Содержание:

<hr>

- [Пути](#Paths)
- [Файлы и Каталоги](#)
- [Итераторы Каталогов](#)
- [Файловые потоки](#)

<hr>

Библиотека [Boost.Filesystem](http://www.boost.org/doc/libs/1_62_0/libs/filesystem/doc/index.htm) позволяет легко работать с файлами и каталогами. Она предоставляет класс, которые называется **`boost::filesystem::path`**, который обрабатывает пути. Кроме того, многие автономные функции способны выполнять такие задачи как создание директорий или проверки, существует ли файл. 

Boost.Filesystem была переработана несколько раз. В этой главе представлена текущая версия Boost.Filesystem 3. Она предоставлялась по умолчанию начиная с Boost C++ Libraries 1.46.0. Boost.Filesystem 2 была в последний раз использована в версии 1.49.0. 

<a name="Paths"></a>
##Пути 

`boost::filesystem::path` является основным классом в Boost.Filesystem, отвечающим за отображение и обработку путей. Определения можно найти в пространстве имен `boost::filesystem` и в заголовочном файле *boost/filesystem.hpp*. Пути могут быть созданы посредством передачи строку в конструктор `boost::filesystem::path` (см. [Пример 35.1](#Ex351)).
<a name="Ex351"></a>
####Пример 35.1. Использование `boost::filesystem::path`

```c++
#include <boost/filesystem.hpp>

using namespace boost::filesystem;

int main()
{
  path p1{"C:\\"};
  path p2{"C:\\Windows"};
  path p3{L"C:\\Boost C++ \u5E93"};
}
```

`boost::filesystem::path` может быть инициализирован с помощью wide-строк. Wide-строки интерпретируются как строки Unicode, что позволяет создавать пути, используя символы почти любого языка. Это очень важное отличие от Boost.Filesystem 2, которая предоставляла различные классы, такие как `boost::filesystem::path` и `boost::filesystem::wpath`, для различных типов строк. 

Обратите внимание, что Boost.Filesystem не поддерживает `std::u16string` или `std::u32string`. Ваш компилятор выдаст ошибку при попытке инициализации `boost::filesystem::path` строкой одного из этих типов.

Ни один из конструкторов `boost::filesystem::path` не проверяет пути и наличие предоставленных файлов или каталогов. Таким образом, `boost::filesystem::path` может быть проинициализирован даже сторокой, содержащей бессмысленный путь.

<a name="Ex352"></a>
####Пример 35.2. Бессмысленные пути с `boost::filesystem::path`

```c++
#include <boost/filesystem.hpp>

using namespace boost::filesystem;

int main()
{
  path p1{"..."};
  path p2{"\\"};
  path p3{"@:"};
}
```
[Пример 35.2](#Ex352) запускается без каких-либо проблем, потому что пути являются просто строками. 
`boost::filesystem::path` только обрабатывает строки, не обращаясь к файловой системе.

Поскольку `boost::filesystem::path` обрабатывает строку, данный класс предоставляет несколько методово для того, чтобы вернуть путь в виде строки. 

В целом, Boost.Filesystem различает нативные и универсальные пути. Нативные пути зависят от спецификации системы и должны быть использованы при вызове функций операционной системы. Универсальные пути являются портативными и не зависят от операцонной системы.

<a name="Ex353"></a>
####Пример 35.3. Получение пути из `boost::filesystem::path` в виде строк

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"C:\\Windows\\System"};

#ifdef BOOST_WINDOWS_API
  std::wcout << p.native() << '\n';
#else
  std::cout << p.native() << '\n';
#endif

  std::cout << p.string() << '\n';
  std::wcout << p.wstring() << '\n';

  std::cout << p.generic_string() << '\n';
  std::wcout << p.generic_wstring() << '\n';
}
```

Методы `native()`, `string()` и `wstring()` возвращают пути в нативном формате. Например, на ОС Windows, [Пример 35.3](#Ex353) выведет **`C:\Windows\System`** в стандартный поток вывода три раза.

Методы `generic_string()` и `generic_wstring()` возвращают пути в универсальном формате. Это портируемые пути; строка нормируется на основе правил стандарта POSIX. Универсальные пути идентичны тем, что используются в Linux. Например, слэш используется в качестве разделителя для каталогов. Если [Пример 35.3](#Ex353) запустить на ОС Windows, оба метода `generic_string()` и `generic_wstring()` выведут **`C:/Windows/System`** в стандартный поток вывода.
