#Глава 34. Библиотека Boost.IOStreams.

###Содержание:

<hr>

- [Устройства](#devices)
- [Фильтры](#filtres)

<hr>

Эта глава посвящена библиотеке [Boost.IOStreams](http://www.boost.org/doc/libs/1_62_0/libs/iostreams/doc/index.html). 
Boost.IOStreams разбивает хорошо известные потоки стандартной библиотеки на меньшие составляющие. Данная библиотека определяет два понятия: *устройство*, которое описывает источники данных и *поток*, который описывает интерфейс для форматированного ввода/вывода, основанный на интерфейсе стандартной библиотеки. Поток, определеямый библиотекой Boost.IOStreams, автоматически не подключен к источнику или приемнику данных.

С помощью Boost.IOStreams по-разному можно реализовать эти два понятия. Например, это возможно с помощью `boost::iostreams::mapped_file`, которое загружает файл в память частично или полностью. Поток `boost::iostreams::stream` может быть подключен к устройству как `boost::iostreams::mapped_file`, чтобы использовать знакомые операторы `operator<<` и `operator>>` для считывания и записи данных.

В дополнение к `boost::iostreams::stream`, Boost.IOStreams предоставляет поток `boost::iostreams::filtering_stream`, который позволяет вам добавлять фильтры данных. Например, вы можете использовать `boost::iostreams::gzip_compressor`, чтобы записать сжатые данные в формате GZIP.

Boost.IOStreams может быть использована для подключения к платформенно-зависимым объектам. Библиотека дает возможность подключения к хэндлам Windows или дескриптору файлов. Таким образом, объекты API-интерфейсов низкого уровня становятся доступными на независимом от платформы коде C++. 

Классы и функции, предоставляемые Boost.IOStreams, определены в пространстве имен `boost::iostreams`. Здесь нет основного заголовочного файла, а так как Boost.IOStreams содержит не только заголовочные файлы, она должна быть прекомпилированна. Это важно, потому что поддержка некоторых функций может отсутствовать в зависимости от того, как Boost.IOStreams будет скомпилирована.

<a name="devices"></a>
##Устройства

Устройства - это классы, которые обеспечивают доступ для чтения и записи в объекты, которые обычно не участвуют в процессе, например файлы. Однако вы можете получить доступ к внутренним объектам, например массивам, как к устройствам.

Устройства – это все лишь класс с функцией `read()` или `write()`. Если устройство связано с потоком, то прочитать или записать форматированные данные, получится быстрее, чем при обращении напрямую. 

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

В примере используется оператор `operator<<`, чтобы записать в поток строку "Boost". Поток передает данные устройству. Поскольку устройство соединено с массивом, "Boost" сохраняется в первые пять элементов этого массива. Так как содержимое массива записывают в стандартный вывод, "Boost" отобразится при запуске примера.

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

`boost::iostreams::array_source` используется как `boost::iostreams::array_sink`. В то время как `boost::iostreams::array_sink`  поддерживает только операцию записи, `boost::iostreams::array_source` - только чтение, а `boost::iostreams::array` - и чтение, и запись. 

Стоит заметить, что в `boost::iostreams::array_source` и `boost::iostreams::array_sink` передается ссылка на массив. Массив не должен быть разрушен, когда устройство еще используется.

<a name="Ex343"></a>
####Пример 34.3 Использование вектора в качестве устройства при помощи `boost::iostreams::back_insert_device`

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

В [примере 34.3](#Ex343) применяется устройство типа `boost::iostreams::back_insert_device` вместо `boost::iostreams::array_sink`. Оно может применяться при записи данных в любой контейнер, поддерживающий метод `insert()`. Этот метод здесь необходим для записи данных в контейнер.

В примере используется `boost::iostreams::back_insert_device`, чтобы записать в вектор строку "Boost". Затем эта строка читается из `boost::iostreams::array_source`. Ссылка на начало вектора и его размер передаются в качестве параметров конструктора `boost::iostreams::array_source`.

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

[Пример 34.4](#Ex344) использует метод `boost::iostreams::file_source` для чтения файлов. В то время как ранее примененные устройства не предоставляют методы, `boost::iostreams::file_source` обеспечивает `is_open()`, чтобы проверить, был ли успешно открыт файл. Вам не нужно применять `close()`, потому что деструктор `boost::iostreams::file_source` закроет файл автоматически.

Кроме `boost::iostreams::file_source`, Boost.IOStreams также предоставляет устройство `boost::iostreams::mapped_file_source` для того, чтобы загрузить файл в память частично или полностью. `boost::iostreams::mapped_file_source` определяет метод `data()` для получения указателя на соответствующую область памяти. Таким образом данные будут доступны в случайном порядке, следовательно чтение файла последовательно не обязательно.

Для записи данных в файлы Boost.IOStreams предоставляет метод `boost::iostreams::file_sink` и `boost::iostreams::mapped_file_sink`.

<a name="Ex345"></a>
####Пример 34.5 Использование `file_descriptor_source` и `file_descriptor_sink` 
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

В [примере 34.5](#Ex345) представлены `boost::iostreams::file_descriptor_source` и `boost::iostreams::file_descriptor_sink`. Это устройства, поддерживающие операции чтения и записи на платформенно-зависимых объектах. На ОС Windows - это хэндлы, а на операционных системах POSIX - дескрипторы файлов.

В [примере 34.5](#Ex345) вызывается функция `CreatePipe()` чтобы создать канал. Концы чтения и записи канала передаются в массив **handles**. Конец чтения канала передается в устройство `boost::iostreams::file_descriptor_source`, и конец записи в `boost::iostreams::file_descriptor_sink`. Все, что записывается в поток **os**, который подключен к концу записи, может быть прочитано из потока **is**, подключенного к концу чтения. [Пример 34.5](#Ex345) отправляет строку "Boost" через поток и в стандартный вывод. 

Конструкторы `boost::iostreams::file_descriptor_source` и `boost::iostreams::file_descriptor_sink` принимают два параметра. Первый – хэндл Windows или, если программа запущена на системе POSIX, дескриптор файла. 
Второй параметр - это или `boost::iostreams::close_handle`, или `boost::iostreams::never_close_handle`. Он определяет, закрывает ли деструктор хэндл Windows или дескриптор файла.

<a name="filtres"></a>
##Фильтры

Кроме устройств в Boost.IOStreams также представлены фильтры, которые работают с устройствами для фильтрации данных, прочитанных или записанных в устройства. Следующие примеры используют `boost::iostreams::filtering_istream` и `boost::iostreams::filtering_ostream`. Они заменяют `boost::iostreams::stream`, который фильтры не поддерживает.

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

[Пример 34.6](#Ex346) применяет `boost::iostreams::array_sink`, чтобы записать данные в массив. Данные проходят через фильтр `boost::iostreams::regex_filter`, заменяющий символы. Фильтр принимает регулярное выражение и форматированную строку. Регулярное выражение описывает то, что надо заменить. Форматированная строка определяет, чем должны быть заменены символы. Пример заменяет "Boost" на "C++". Фильтр будет соответствовать одному или нескольким последовательным экземплярам буквы "о" в "Boost", но не нулям. 

Фильтры и устройства соединены потоком `boost::iostreams::filtering_ostream`.
Этот класс имеет функцию `push()`, в которую передаются фильтры и устройства.

Фильтры должны быть переданы перед устройствами, их порядок важен. Вы можете передавать один или несколько фильтров, но в тот момент, когда устройство передано, поток будет завершен, и вы не должны больше вызывать `push()`.

Фильтр `boost::iostreams::regex_filter` не может обрабатывать данные по символам, потому что регулярные выражения должны просматриваться группой символов. Поэтому `boost::iostreams::regex_filter` начинает фильтровать только после того, как завершится операция записи и все данные станут доступными. Это происходит, когда устройство удалено из потока с помощью `pop()`. [Пример 34.6](#Ex346) вызывает `pop()` после того, как в "Boost" был записан в поток. Без вызова `pop()`, `boost::iostreams::regex_filter` данные не будут обработаны и не будут переданы в устройство. 

Обратите внимание на то, что вы не должны использовать поток, который не связан с устройством. Тем не менее, вы можете завершить его, если используете `push()` после вызова `pop()`.

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

[Пример 34.7](#Ex347) использует фильтр `boost::iostreams::counter`, который считает символы и строки. Класс содержит методы `characters()` и `lines()`.

`boost::iostreams::filtering_stream` обеспечивает метод `component()` для доступа к фильтрам. 
Индекс соответствующего фильтра передается в качестве параметра. Поскольку `component()` – это шаблон, тип фильтра должен передан в качестве шаблонного параметра. `component()` возвращает указатель на фильтр (вернет 0 – если неверный тип фильтра передается в качестве шаблонного параметра).
[Пример 34.7](Ex347) Записывает пять символов в поток без перехода на новую строку (“\n”). Таким образом выводится `5` и `0`.

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

[Пример 34.8](#Ex348) использует поток `boost::iostreams::filtering_istream` в дополнение к `boost::iostreams::filtering_ostream`. Этот поток используется, когда вам нужно считать данные при помощи фильтра. В примере сжатые данные записываются и читаются снова.

Boost.IOStreams поддерживает несколько видов фильтров, касающихся сжатых файлов. `boost::iostreams::zlib_compressor` сжимает данные в формате ZLIB. Чтобы разархивировать данные из этого формата, используют `boost::iostreams::zlib_decompressor`. Фильтры добавляются в поток при помощи `push()`. 

[Пример 34.8](#Ex348) записывает "Boost" в вектор **v** в сжатом виде и строку **s** в несжатом виде. 
Пример отображает `Boost`.

<blockquote>Обратите внимание на то, что в ОС Windows скомпилированная библиотека Boost.IOStreams не поддерживает сжатые данные, поскольку, по умолчанию, библиотека скомпилирована с макросом `NO_ZLIB` на ОС Windows. Вам необходимо убрать определение этого макроса, после чего определить ZLIB_LIBPATH и ZLIB_SOURCE, и перекомпилировать для получения поддержки ZLIB на ОС Windows. </blockquote>


#Глава 35. Boost.Filesystem

###Содержание:

<hr>

- [Пути](#Paths)
- [Файлы и Каталоги](#F&D)
- [Итераторы Каталогов](#DI)
- [Файловые потоки](#FS)

<hr>

Библиотека [Boost.Filesystem](http://www.boost.org/doc/libs/1_62_0/libs/filesystem/doc/index.htm) позволяет легко работать с файлами и каталогами. Она предоставляет класс, который называется **`boost::filesystem::path`**, он обрабатывет пути. Кроме того, многие автономные функции способны создавать директории или проверять существование файла. 

Boost.Filesystem была переработана несколько раз. В этой главе представлена текущая версия Boost.Filesystem 3. Она предоставлялась по умолчанию начиная с Boost C++ Libraries 1.46.0. Boost.Filesystem 2 была в последний раз использована в версии 1.49.0. 

<a name="Paths"></a>
##Пути 

`boost::filesystem::path` является основным классом в Boost.Filesystem, отвечающим за отображение и обработку путей. Определения можно найти в пространстве имен `boost::filesystem` и в заголовочном файле *boost/filesystem.hpp*. Пути могут быть созданы посредством передачи строки в конструктор `boost::filesystem::path` (см. [Пример 35.1](#Ex351)).
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

Обратите внимание, что Boost.Filesystem не поддерживает `std::u16string` или `std::u32string`. Компилятор выдаст ошибку при попытке инициализации `boost::filesystem::path` строкой одного из этих типов.

Ни один из конструкторов `boost::filesystem::path` не проверяет ни пути, ни наличие используемых файлов или каталогов. Таким образом, `boost::filesystem::path` может быть проинициализирован даже сторокой, содержащей бессмысленный путь.

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
[Пример 35.2](#Ex352) запускается без каких-либо проблем, потому что пути - просто строки. 
`boost::filesystem::path` обрабатывает строки, не обращаясь к файловой системе.

Поскольку `boost::filesystem::path` обрабатывает строку, данный класс предоставляет несколько методов для того, чтобы вернуть путь в виде строки. 

В целом, Boost.Filesystem различает нативные и универсальные пути. Нативные пути зависят от спецификации системы, они должны быть использованы при вызове функций операционной системы. Универсальные пути являются портативными и не зависят от операцонной системы.

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

Если метод возвращает нативный путь, то этот путь зависит от операцонной системы. В случае методов, возвращающих универсальные пути, такой зависимости нет. Универсальные пути уникальным образом идентифицируют файлы и каталоги, независимо от операцонной системы и, таким образом, позволяют легко писать платформенно-независимый код.

Так как `boost::filesystem::path` может быть инициализирован строками различных типов, некоторые методы, направлены на то, чтобы получить пути из этих строк. В то время как `string()` и `generic_string()` возвращают строку типа `std::string`, `wstring()` и `generic_wstring()` возвращают строку типа `std::wstring`.

Возвращаемое значение `native()` зависит от операцонной системы, под которую была скомпилированна программа. На ОС Windows это строка типа `std::wstring`. 

Конструктор `boost::filesystem::path`  поддерживает как универсальные, так и платформенно-зависимые пути. В [Примере 35.3](#Ex353) путь "C:\\Windows\\System" является определенным на ОС Windows и не может быть перенесен на другие ОС. Потому как обратный слэш в C++ - это escape-последовательность, стоит заметить, что перед ним нужно поставить еще один обратный слэш. Этот путь будет корректно распознан Boost.Filesystem, только если программа запущена на ОС Windows. При ее выполнении на POSIX системе, например Linux, этот пример вернет **`C:\Windows\System`** для всех методов. Так как обратный слэш не используется в качестве разделителя на Linux как в портируемом, так и в собственном формате, Boost.Filesystem не распознает этот символ как разделитель для файлов и каталогов.

Макрос BOOST_WINDOWS_API, предоставленный Boost.System, определен, если пример скомпилирован на Windows. Соответствующим макросом для операционных систем POSIX является BOOST_POSIX_API.

[Пример 35.4](#Ex354) использует переносимый путь, чтобы инициализировать `boost::filesystem::path`.

<a name="Ex354"></a>
####Пример 35.4. Использование переносимого пути для инициализации `boost::filesystem::path`

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"/"};
  std::cout << p.string() << '\n';
  std::cout << p.generic_string() << '\n';
}
```

Так как `generic_string()` возвращает портируемый путь, его значение будет символом слэш("/"), таким же, как было использовано для инициализации `boost::filesystem::path`. Тем не менее, метод `string()` возвращает разные значения, в зависимости от платформы. На ОС Windows и Linux он возвращает "/". Вывод одинаков, потому что ОС Windows разрешает использование наклонной черты, в качестве разделителя каталогов несмотря на то, что обратный слэш более предпочитителен. 

`boost::filesystem::path` обеспечивает несколько методов для получения доступа к определенным компонентам пути.

<a name="Ex355"></a>
####Пример 35.5. Получение доступа к компонентам пути

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"C:\\Windows\\System"};
  std::cout << p.root_name() << '\n';
  std::cout << p.root_directory() << '\n';
  std::cout << p.root_path() << '\n';
  std::cout << p.relative_path() << '\n';
  std::cout << p.parent_path() << '\n';
  std::cout << p.filename() << '\n';
}
```

Если [Пример 35.5](#Ex355) выполняется на ОС Windows, строка "C:\\Windows\\System" интерпретируется как платформенно-зависимый путь. Следовательно, `root_name()` возвращает **`"C:"`**, `root_directory()` возвращает **`"\"`**, `root_path()` возвращает **`"C:\`**, `relative_path()` возвращает **`"Windows\System"`**, `parent_path()` возвращает **`"C:\Windows"`**, а `filename()` возвращает **`"System"`**. 

Все методы возвращают платформенно-зависимые пути, потому что `boost::filesystem::path` хранит пути в зависимом от платформы формате. Чтобы получить пути в переносимом формате, методы, такие как generic_string() нужно вызвать явно.

Если [пример 35.5](#Ex355) запущен в ОС Linux, возвращаемые значения будут отличаться. Большинство методов вернут пустую строку, за исключением `relative_path()` и `filename()`, которые вернут **`"C:\Windows\System"`**. Это означает, что строка "C:\\Windows\\System" интерпретируется как имя файла в ОС Linux, что очевидно, так как она не является ни портируемой, ни платформенно-зависимой кодировкой пути в ОС Linux. Поэтому у Boost.Filesystem нет выбора, кроме как интерпретировать это строку как имя файла.

Boost.Filesystem обеспечивает дополнительные методы, чтобы проверить, содержит ли путь определенную подстроку. Это такие методы как: `has_root_name()`, `has_root_directory()`, `has_root_path()`, `has_relative_path()`, `has_parent_path()` и `has_filename()`. Каждый из них возвращает значение типа bool.

Существует еще два метода, которые могут разделить имя файла на его компоненты. Их можно вызвать, только если has_filename() возвращает true. Иначе они возвращают пустые строки, так как при отсутствии имени файла нечего разделять. 

<a name="Ex356"></a>
####Пример 35.6. Получение имени и расширения файла

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"photo.jpg"};
  std::cout << p.stem() << '\n';
  std::cout << p.extension() << '\n';
}
```

[Пример 35.6](#Ex356) возвращает **`"photo"`** для метода `stem()` и **`".jpg"`** для метода `extension()`.

Вместо того, чтобы получать доступ к компонентам пути через вызов методов, Вы можете также проходить по компонентам итерационно.

<a name="Ex357"></a>
####Пример 35.7. Итерационный проход по компонентам пути

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"C:\\Windows\\System"};
  for (const path &pp : p)
    std::cout << pp << '\n';
}
```

При выполнении на ОС Windows [пример 35.7](#Ex357) успешно выведет **`"C:"`**, **`"/"`**, **`"Windows"`** и **`"System"`**. На ОС Linux - **`"C:\Windows\System"`**. 

В то время как предыдущие примеры представляли различные методы, чтобы получить доступ к компонентам пути, [пример 35.8](#Ex358) использует метод для изменения пути.

<a name="Ex358"></a>
####Пример 35.8. Конкатенация путей с помощью оператора `operator/=`

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"C:\\"};
  p /= "Windows\\System";
  std::cout << p.string() << '\n';
}
```

Используя оператор `operator/=`, [Пример 35.8](#Ex358) добавляет один путь к другому. На ОС Windows программа выводит **`C:\Windows\System`**. На ОС Linux - **`C:\/Windows\System`**, так как слэш используется в качестве разделителя для файлов и каталогов. Слэш - также причина, по которой оператор `operator/=` был перегружен; в конце концов, слэш - часть оператора. 

Помимо оператора `operator/=`, Boost.Filesystem обеспечивает методы `remove_filename()`, `replace_extension()`, `make_absolute()`, и `make_preferred()` для изменения путей. Последний упомянутый метод разработан для использования на ОС Windows.

<a name="Ex359"></a>
####Пример 35.9. Предпочтительная нотация с `make_preferred()`

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"C:/Windows/System"};
  std::cout << p.make_preferred() << '\n';
}
```

Даже при том, что обратный слэш используется в качестве разделителя для файлов и каталогов по умолчанию, ОС Windows все еще принимает слэш. Поэтому "C:/Windows/System" - допустимый нативный путь. При помощи метода `make_preferred()` такой путь может быть преобразован в предпочтительную нотацию для ОС Windows. 
[Пример 35.9](#Ex359) выводит **`"C:\Windows\System"`**.

Метод `make_preferred()` никак не действует на POSIX-совместимых операционных системах, таких как ОС Linux.

Обратите внимание на то, что метод `make_preferred()` не только возвращает переделанный путь, но и изменяет объект, у которого он был вызван. **p** содержит "C:\Windows\System" после вызова.

<a name="F&D"></a>
##Файлы и Каталоги

Методы, предоставленные `boost::filesystem::path` просто обрабатывают строки. Они получают доступ к отдельным компонентам пути, добавляют пути друг к другу и т.д.
Чтобы работать с физическими файлами и каталогами на жестком диске, существует несколько автономных функций. Они принимают один или несколько параметров типа boost::filesystem::path вызывают непосредственно функции операционной системы. 

До представления различных функций важно понять то, что происходит в случае ошибки. Все методы вызывают функции операционной системы, которые могут перестать работать. Поэтому в Boost.Filesystem есть два варианта функций, которые ведут себя по-разному в случае ошибки:

- Первый вариант бросает исключение типа `boost::filesystem::filesystem_error`. Этот класс является производным от `boost::system::system_error` и вписывается в платформу Boost.System. 
- Второй вариант принимает объект типа `boost::system::error_code` как дополнительный параметр. Этот объект передан ссылкой и может быть использован после вызова функции. В случае отказа, объект сохраняет соответствующий код ошибки.

`boost::system::system_error` и `boost::system::error_code` описаны в [Главе 55](https://theboostcpplibraries.com/boost.system). В дополнение к наследуемому интерфейсу от `boost::system::system_error`, `boost::filesystem::filesystem_error` предоставляет двa метода path1() и path2(), каждый из которых возвращает объект типа `boost::filesystem::path`. Поскольку существуют функции, которые принимают два параметра типа `boost::filesystem::path`, эти два метода обеспечивают простой способ получения соответствующих путей в случае отказа.

<a name="Ex3510"></a>
####Пример 35.10. Использование `boost::filesystem::status()`

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"C:\\"};
  try
  {
    file_status s = status(p);
    std::cout << std::boolalpha << is_directory(s) << '\n';
  }
  catch (filesystem_error &e)
  {
    std::cerr << e.what() << '\n';
  }
}
```

[Пример 35.10](#Ex3510) представляет `boost::filesystem::status()`, который запрашивает состояние файла или каталога. Эта функция возвращает объект типа `boost::filesystem::file_status`, который может быть передан в дополнительные вспомогательные функции для оценки. Например, `boost::filesystem::is_directory()` возвращает `true` если было запрошено состояние для каталога. Кроме `boost::filesystem::is_directory()` доступны такие функции как `boost::filesystem::is_regular_file()`, `boost::filesystem::is_symlink()` и `boost::filesystem::exists()`, каждая из которых возвращает значение типа `bool`.

Функция `boost::filesystem::symlink_status()` запрашивает состояние символьной ссылки. `boost::filesystem::status()` позволяет запрашивать статус файла, на который ссылается символьная ссылка. В операционной системе Windows, символьные ссылки обозначаются расширением файла ***lnk***. 

<a name="Ex3511"></a>
####Пример 35.11. Использование `boost::filesystem::file_size()`

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"C:\\Windows\\win.ini"};
  boost::system::error_code ec;
  boost::uintmax_t filesize = file_size(p, ec);
  if (!ec)
    std::cout << filesize << '\n';
  else
    std::cout << ec << '\n';
}
```

Другая категория функций позволяет запрашивать атрибуты. Функция `boost::filesystem::file_size()` возвращает размер файла в байтах. Возвращаемое значение имеет тип `boost::uintmax_t`, который является определением типа unsigned long long. Тип предоставляется Boost.Integer.

[Пример 35.11](#Ex3511) использует объект типа `boost::system::error_code`, который нужно оценить в явном виде, чтобы определить, был ли успешным вызов boost::filesystem::file_size().

<a name="Ex3512"></a>
####Пример 35.12. Использование `boost::filesystem::last_write_time()`

```c++
#include <boost/filesystem.hpp>
#include <iostream>
#include <ctime>

using namespace boost::filesystem;

int main()
{
  path p{"C:\\Windows\\win.ini"};
  try
  {
    std::time_t t = last_write_time(p);
    std::cout << std::ctime(&t) << '\n';
  }
  catch (filesystem_error &e)
  {
    std::cerr << e.what() << '\n';
  }
}
```

Чтобы определить время, когда файл был изменен в последний раз, используют `boost::filesystem::last_write_time()` (см. [Пример 35.12](#Ex3512)).


<a name="Ex3513"></a>
####Пример 35.13. Использование `boost::filesystem::space()`

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"C:\\"};
  try
  {
    space_info s = space(p);
    std::cout << s.capacity << '\n';
    std::cout << s.free << '\n';
    std::cout << s.available << '\n';
  }
  catch (filesystem_error &e)
  {
    std::cerr << e.what() << '\n';
  }
}
```

`boost::filesystem::space()` получает общее и свободное дисковое пространство (см. [Пример 35.13](#Ex3513)). Он возвращает объект типа `boost::filesystem::space_info`, который предоставляет три открытых поля: **capacity**, **free**, и **available** типа `boost::uintmax_t`. Дисковое пространство - в байтах.

Функции, представленные выше не затрагивают файлы и каталоги, но существует несколько функций, которые могут быть использованы для создания, переименования или удаления файлов и каталогов.

<a name="Ex3514"></a>
####Пример 35.14. Создание, переименование и удаление каталогов

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"C:\\Test"};
  try
  {
    if (create_directory(p))
    {
      rename(p, "C:\\Test2");
      boost::filesystem::remove("C:\\Test2");
    }
  }
  catch (filesystem_error &e)
  {
    std::cerr << e.what() << '\n';
  }
}
```

[Пример 35.14](#Ex3514) должен быть очевиден.  При ближайшем рассмотрении можно увидеть, что в функции не всегда передается объект типа `boost::filesystem::path`, так же это может быть простая строка. Это возможно, так как `boost::filesystem::path` предоставляет неявный конструктор, который преобразует строки в объекты типа `boost::filesystem::path`. Это позволяет легко использовать Boost.Filesystem, так как не требуется создавать пути в явном виде.

<blockquote>В [примере 35.14](#Ex3514) `boost::filesystem::remove()` вызван явно, используя пространство имен. Иначе Visual C++ 2013 перепутал бы функцию с `remove()` из заголовочного файла ***stdio.h***.</blockquote>

Также доступны дополнительные функции, такие как `create_symlink()`, чтобы создать символьные ссылки, или `copy_file()` и `copy_directory()`, чтобы копировать файлы и каталоги.

<a name="Ex3515"></a>
####Пример 35.15. Использование `boost::filesystem::absolute()`

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  try
  {
    std::cout << absolute("photo.jpg") << '\n';
  }
  catch (filesystem_error &e)
  {
    std::cerr << e.what() << '\n';
  }
}
```

В [примере 35.15](#Ex3515) представлена функция, которая создает абсолютный путь на основе имени файла или части пути. Выведенный на экран путь зависит оттого, из какого каталога запущена программа. Например, если бы программа была запущена в ***C:\***, вывод был бы **`"C:\photo.jpg"`**.

Чтобы получить абсолютный путь относительно различных каталогов, в `boost::filesystem::absolute()` может быть передан второй параметр.

<a name="Ex3516"></a>
####Пример 35.16. Создание абсолютного пути относительно другого каталога

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  try
  {
    std::cout << absolute("photo.jpg", "D:\\") << '\n';
  }
  catch (filesystem_error &e)
  {
    std::cerr << e.what() << '\n';
  }
}
```

[Пример 35.16](#Ex3516) выводит **`"D:\photo.jpg"`**.

Последний пример в этом разделе, [Пример 35.17](#Ex3517), вводит полезную вспомогательную функцию для получения текущего рабочего каталога.

<a name="Ex3517"></a>
####Пример 35.17. Использование `boost::filesystem::current_path()`

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  try
  {
    std::cout << current_path() << '\n';
    current_path("C:\\");
    std::cout << current_path() << '\n';
  }
  catch (filesystem_error &e)
  {
    std::cerr << e.what() << '\n';
  }
}
```

[Пример 35.17](#Ex3517) вызывает `boost::filesystem::current_path()` несколько раз. Если функция вызывается без параметров, текущий рабочий каталог возвращается. Если же передается объект типа `boost::filesystem::path`, то текущий рабочий каталог устанавливается.

<a name="DI"></a>
##Итераторы Каталогов 

Boost.Filesystem предоставляет итератор `boost::filesystem::directory_iterator`, чтобы перебирать файлы в каталоге (см. [Пример 35&18](#Ex3518)).

<a name="Ex3518"></a>
####Пример 35.18. Перебор файлов в каталоге

```c++
#include <boost/filesystem.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p = current_path();
  directory_iterator it{p};
  while (it != directory_iterator{})
    std::cout << *it++ << '\n';
}
```

`boost::filesystem::directory_iterator` инициализируется с помощью пути, для получения итератора, указывающего на начало каталога. Чтобы извлечь конец директории, класс должен быть создан с помощью конструктора по умолчанию. Записи могут быть созданы или удалены во время итерации без ее нарушения. Тем не менее, не определено, будут ли изменения видимыми во время итерации или нет. Например, итератор не может указывать на только что созданные файлы. Для того, чтобы убедиться, что все текущие записи доступны, нужно перезапустить итерацию. 

Чтобы рекурсивно перебирать каталоги и подкаталоги, Boost.Filesystem использует итератор `boost::filesystem::recursive_directory_iterator`. 

<a name="FS"></a>
##Файловые потоки

Стандарт определяет различные файловые потоки в заголовочном файле fstream. Эти потоки не принимают параметры типа `boost::filesystem::path`. Если Вы хотите открыть файловые потоки с объектами типа `boost::filesystem::path` включите заголовочный файл ***boost/filesystem/fstream.hpp***.

<a name="Ex3519"></a>
####Пример 35.19. Использование `boost::filesystem::ofstream`

```c++
#include <boost/filesystem/fstream.hpp>
#include <iostream>

using namespace boost::filesystem;

int main()
{
  path p{"test.txt"};
  ofstream ofs{p};
  ofs << "Hello, world!\n";
}
```

[Пример 35.19](#Ex3519) открывает файл при помощи класса `boost::filesystem::ofstream`. Объект типа `boost::filesystem::path` передается в конструтор `boost::filesystem::ofstream`. Метод `open()` также принимает параметр типа `boost::filesystem::path`.
