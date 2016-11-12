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

`boost::filesystem::path` является основным классом в Boost.Filesystem, отвечающим за отображение и обработку путей. Определения можно найти в пространстве имен `boost::filesystem` и в заголовочном файле *boost/filesystem.hpp*. Пути могут быть созданы посредством передачи строку в конструктор `boost::filesystem::path` (см. [Пример 35.1](#Ex35.1)).
<a name="Ex35.1"></a>
####Пример 35.1. Использование `boost::filesystem::path`

`
#include <boost/filesystem.hpp>

using namespace boost::filesystem;

int main()
{
  path p1{"C:\\"};
  path p2{"C:\\Windows"};
  path p3{L"C:\\Boost C++ \u5E93"};
}
`
