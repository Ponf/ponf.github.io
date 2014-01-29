---
layout: post
title: Работа с базой данных в WP7 для ленивых :)
date: Tue, 11 Oct 2011 10:14:00 +0000
comments: true
---

Так уж получилось, что никогда раньше у меня не было необходимости использовать в моих проектах базы данных для хранения информации, так что все мои знания ограничивались парой посещений лекций по СУБД на втором или третьем курсе :-[.Но во время написания приложения HabraReader под Windows Phone 7 мной и моим коллегой [@kin9pin](http://twitter.com/#!/kin9pin) было принято самое логичное решение - хранить посты в БД. Погуглив, мы нашли довольно много [статей](http://msdn.microsoft.com/ru-ru/windowsphone/hh425066#mark_15) и [пошаговых инструкций](http://msdn.microsoft.com/en-us/library/hh202876(v=VS.92).aspx) по созданию и работе с  БД в Windows Phone, но всё это нас по тем или иным причинам не устраивало (например, из-за привязки к MVVM, до которого всё никак руки не доходят).

Но выход нашелся! Причем оказался настолько простым и элегантным, что с помощью этого решения любой сможет создать и подключить БД к своему проекту за считанные минуты.

<!--more--> 


## 1. Создаём структуру базы данных

Для начала нам понадобится база данных в формате .sdf. Как её создать? Очень просто!
Идём в Visual Studio, создаём новый проект (например, Windows Forms Application)

{% img http://3.bp.blogspot.com/-gdQoljIac8Y/TpPquXelmsI/AAAAAAAAABQ/qXO0lz1uOIM/s640/1.png %}

Добавляем новый элемент в наш проект типа "Local Database"

{% img http://3.bp.blogspot.com/-CswLeqPrTBo/TpPrhQqMxUI/AAAAAAAAABY/rNCPNEzb9CU/s640/2.png %}

Жмём Next и Finish. Далее идём в View и выбираем Server Explorer (Ctrl +W, L).

В разделе Data Connections мы видим нашу только что добавленную базу данных. Теперь мы можем создать структуру нашей будущей базы данных. Начнем с того, что добавим одну таблицу.

{% img http://4.bp.blogspot.com/-9tkuprH_aYA/TpPtmHVTfCI/AAAAAAAAABg/vEcuvGpC3Aw/s640/3.png %}


Для простейшего примера нам будет достаточно таблицы с 5 столбцами: ID (идентификатор постов, всегда уникальный, автоматически увеличивается на 1 и служит первичным ключом), Title (название поста, текст, не больше 150 символов), Description (описание поста, текст, не больше 1000 символов), Date (дата и время создания поста, тип datetime) и Link (ссылка на полную версию поста, текст, не длиннее 100 символов).

В итоге выглядит это примерно так:

{% img http://4.bp.blogspot.com/-t5x81PCYiYE/TpPwJ_ig_SI/AAAAAAAAABo/M79pk4ckRHs/s640/4.png %}

Теперь надо перестроить проект и на этом Visual Studio можно на некоторое время закрыть и расслабиться: самая сложная часть осталась позади :). В папке с вашим проектом появился файл MyDB.sdf. Дальнейшая работа будет проходить именно с ним, так что весь солюшн, кроме MyDB.sdf можно удалить.

Файл MyDB.sdf я положил в D:\Tests для большего удобства в выполнении следующего шага.

## 2. Создаём DataContext для работы с БД
Что такое DataContext? Это некоторый класс, который служит нам для работы с БД и выполнения основных CRUD операций (create, read, update, delete). Для того, чтобы его создать нужно открыть командную строку Visual Studio:

Пуск -> Все программы -> Microsoft Visual Studio 2010 -> Visual Studio Tools -> Visual Studio Command Prompt

В командной строке с помощью утилиты SQLmetal мы сгенерируем DataContext для нашей базы данных MyDB.sdf

```
sqlmetal D:\Tests\MyDB.sdf /code:D:\Tests\MyDB.cs /language:csharp /namespace:DBDemo /context:MyDBContext /pluralize
```

В этой команде мы указали путь к нашей БД и путь, куда поместить наш DataContext, выбрали язык C#, указали название пространства имён и имя DataContext.

{% img http://1.bp.blogspot.com/-LBGvx5TfMNo/TpP3Z8qUfkI/AAAAAAAAABw/unmlzuVwu-4/s640/5.png %}

Теперь в нашей папке Tests лежит файл MyDB.cs, который и является DataContext нашей БД.

## 3. Пример использования

Давайте создадим новый проект под WP7 (обязательно укажите Windows Phone 7.1, т.к. поддержка баз данных появилась только в WP7 Mango).

В проект добавляем наш MyDB.cs, в референсы добавляем System.Data.Linq.

Единственное, что придется изменить в MyDB.cs - это namespace на пространство имён нашего проекта и удалить два следующих метода:


``` csharp
public MyDBContext(System.Data.IDbConnection connection) :
        base(connection, mappingSource)
{
    OnCreated();
}
```

и


``` csharp
public MyDBContext(System.Data.IDbConnection connection, System.Data.Linq.Mapping.MappingSource mappingSource) :
        base(connection, mappingSource)
{
    OnCreated();
}
```


так как интерфейс IDbConnection не поддерживается телефоном.

Следующим шагом мы создадим промежуточный Helper - класс для большего удобства работы с базой данных. При таком подходе в MyDB.cs вносить изменения больше не придётся.

``` csharp
public class DBHelper
{
    private const string ConnectionString = @"isostore:/ReaderDB.sdf";

    ///

    /// Создание БД в изолированном хранилище.
    ///
    public static void CreateDatabase()
    {
        using (var context = new MyDBContext(ConnectionString))
        {
            if (!context.DatabaseExists())
            {
                // create database if it does not exist
                context.CreateDatabase();
            }
        }
    }

    ///

    /// Удаление БД из изолированного хранилища.
    ///
    public static void DeleteDatabase()
    {
        using (var context = new MyDBContext(ConnectionString))
        {
            if (context.DatabaseExists())
            {
                // delete database if it exist
                context.DeleteDatabase();
            }
        }
    }

    ///

    /// Добавляет статью в БД.
    ///
    ///
    public static void AddPost(Post post)
    {
        using (var context = new MyDBContext(ConnectionString))
        {
            if (context.DatabaseExists())
            {
                context.Posts.InsertOnSubmit(post);
                context.SubmitChanges();
            }
        }
    }

    ///

    /// Получает список статей из БД.
    ///
    ///Список статей.
    public static IList&lt;Post> GetPosts()
    {
        IList&lt;Post> posts = new List&lt;Post>();

        using (var context = new MyDBContext(ConnectionString))
        {
            posts = (from emp in context.Posts select emp).ToList();
        }
        return posts;
    }
}
```


Теперь в коде можно просто вызывать статичные методы DBHelper для различного взаимодействия с БД! Например, будем создавать БД (если она отсутствует) при загрузке страницы.


``` csharp
private void LayoutRoot_Loaded(object sender, RoutedEventArgs e)
{
    DBHelper.CreateDatabase();
}
```


## 4. Немного о ConnectionString
В общем случае ConnectionString - некоторая строка, в которой могут быть указаны различные параметры для подключения к базе данных.

### Источник данных

Источник данных - параметр по-умолчанию, то есть вы можете не указывать свойство "Data Source". База данных может располагаться как в изолированном хранилище, так и в директории с данными приложения.


``` csharp
string ConnectionString = @"isostore:/ReaderDB.sdf";
string ConnectionString = @"appdata:/ReaderDB.sdf";

string ConnectionString = @"DataSource = 'isostore:/ReaderDB.sdf';";
string ConnectionString = @"DataSource = 'appdata:/ReaderDB.sdf';";
```


### Пароль

Пароль используется чтобы зашифровать базу данных при создании и так же его необходимо указывать при подключении к ней. Максимальная длина пароля - 40 символов.

``` csharp
string ConnectionString = @"Data Source = 'isostore:/ReaderDB.sdf'; Password = 'mypassword';";
```


### Режим

Существуют 4 режима открытия БД ("Read Write", "Read Only", "Exculsive" и "Shared Read"), которые обозначают вид доступа.

``` csharp
// Позволяет нескольким процессам читать и писать в БД одновременно. Это режим по-умолчанию.
string ConnectionString = @"Data Store = 'isostore:/ReaderDB.sdf'; File Mode = 'Read Write';";

// Разрешает процессам производить только чтение из БД.
string ConnectionString = @"Data Store = 'isostore:/ReaderDB.sdf'; File Mode = 'Read Only';";

// Разрешает только одному процессу читать и писать в БД.
string ConnectionString = @"Data Store = 'isostore:/ReaderDB.sdf'; File Mode = 'Exclusive';";

// Разрешает всем процессам читать из БД, но только один может писать.
string ConnectionString = @"Data Store = 'isostore:/ReaderDB.sdf'; File Mode = 'Shared Read';";
```


### Максимальный размер БД

Размер указывается в мегабайтах, в пределах от 32 до 512.

``` csharp
string ConnectionString = @"Data Store = 'isostore:/ReaderDB.sdf'; Max Database Size = 128;";
```


### Максимальный размер буфера

Указывается в килобайтах, в пределах от 384 до 5120.


``` csharp
	string ConnectionString = @"Data Store = 'isostore:/ReaderDB.sdf'; Max Buffer Size = 2048;";
```

### Культура

Параметр культуры указывается в виде "язык-Страна", например, "ru-RU" для русского языка в России

``` csharp
	string ConnectionString = @"Data Store = 'isostore:/ReaderDB.sdf'; Culture Identifier = ru-RU;";
```


### Чувствительность к регистру

В параметрах можно так же указать, будет ли база данных чувствительна к регистру.


``` csharp
	string ConnectionString = @"Data Store = 'isostore:/ReaderDB.sdf'; Case Sensitive = true;";
```


## Заключение
Надеюсь, вам удалось разобраться с помощью моей статьи в работе с базами данных в Windows Phone 7. Как вы видите, создание и подключение базы - данных достаточно простые операции, и, используя сгенерированный с помощью sqlmetal класс, можно разобраться в структуре и основных принципах работы. После этого уже ничто не помешает вам редактировать структуру базы данных в коде и создавать новые проекты, использующие БД, с нуля.

Не забывайте подписываться на мой твиттер: [@ponfius](http://twitter.com/#!/ponfius). И, как обычно, stay tuned :)


<i>В статье использовалась информация из блога [http://www.kunal-chowdhury.com](http://www.kunal-chowdhury.com/)</i>

