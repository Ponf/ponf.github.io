---
layout: post
title: Создание игр на Silverlight + XNA. Часть 1. Отличия от XNA.
date: Sun, 18 Sep 2011 17:25:00 +0000
comments: true
---

С выходом обновления Mango для Windows Phone 7 и обновлением SDK, появилась возможность создавать приложения использующие обе технологии вместе. В этой статье я расскажу вам об этих возможностях и опишу процесс миграции с XNA на Silverlight+XNA.

## Что это даёт разработчику?

В первую очередь, у разработчиков игр появились следующие возможности:

* Быстрое создание UI с использованием контролов из Silverlight
* Удобная навигация между страницами, используя Navigation Service
* Использование WebClient для интеграции различных социальных сервисов
* Создание интерфейса в Expression Blend


Причем стоит заметить, что не все страницы должны совмещать в себе и Silverlight и XNA.
Итак, что же нужно сделать с готовым приложением, чтобы оно заработало в связке Silverlight + XNA?

<!--more--> 


## 1. Новый проект

При создании нового проекта необходимо выбрать **"Windows Phone Silverlight and XNA Application"**. Создастся новый проект с 2мя страницами **MainPage** и **GamePage**.



* **MainPage** - обычная страница Silverlight-приложения, на которой располагается кнопка "Change to game page", при нажатии на которую, соответственно, открывается страница **GamePage**.
* **GamePage** - как раз наша страница с XNA. Если посмотреть на содержание её xaml файла, то вместо разметки страницы там будет всего одна строчка:

``` xml
<!--No XAML content is required as the page is rendered entirely with the XNA Framework-->
```

То есть "XAML код не требуется, т.к. страница отображается полностью с помощью XNA Framework". Таким образом, когда мы захотим добавить контролы на эту страницу нам нужно будет заменить этот комментарий на наш код.
Теперь давайте рассмотрим файл GamePage.xaml.cs, в котором и находится вся логика нашего XNA приложения.

## Где размещаются конструкторы?

Теперь давайте по порядку разберемся где какие конструкторы находятся.
* Конструктор **SharedGraphicsDeviceManager **(который в Silverlight/XNA приложении заменяет **GraphicsDeviceManager**) уже находится в файле **App.xaml**:

``` xml
<!--The SharedGraphicsDeviceManager is used to render with the XNA Graphics APIs-->
<xna:SharedGraphicsDeviceManager />
```

* Конструктор **ContentManager **находится в файле **App.xaml.cs**, но ничто не мешает вам создавать новые экземпляры **ContentManager**:

``` csharp
// Create the ContentManager so the application can load precompiled assets
Content = new ContentManager(Services, "Content");
```



* **GameTimer** следит за прошедшим временем для каждой страницы в отдельности, он используется в методах рисования и обновления логики, но об этом немного позже.
* Отключение фиксированных временных интервалов, за которое раньше отвечало свойство **IsFixedTimestep**, теперь можно организовать установив **UpdateInterval **в **TimeSpan.Zero**.
* По умолчанию разрешение экрана составляет 480x800 пикселей в портретной ориентации и 800х480, если вы выставите ландшафтную ориентацию. Вы можете выставить разрешение для всего приложения в файле **App.xaml**, либо выставлять его для каждой страницы, вызывая **SetSharingMode(true)**.

## 3. Где теперь загружать контент?
При совместном использовании XNA и Silverlight нет необходимости использовать отдельного метода для загрузки контента, как это было в чистом XNA приложении (**LoadContent**). В общем, вы можете загружать игровой контент когда хотите, однако вы должны убедиться, что во время загрузки **графического **контента приложение находится в режиме XNA рендеринга (вызывая метод **SetSharingMode**). Если вы попытаетесь загрузить графический контент во время Silverlight-рендеринга, то это вызовет исключение. Не графический контент вы можете загружать в любое время, когда у вас создан **ContentManager**.

В примерах контент обычно загружается в методе  OnNavigatedTo, после включения XNA рендеринга. Рекомендуется всегда загружать контент в данном методе, так как это безопасно, но это не обязательное требование.

## 4. Циклы обновления и отрисовки
Давайте теперь рассмотрим поподробнее циклы обработки в нашем проекте и разберемся, как происходит обновление данных в логике приложения и его отрисовка.
В первую очередь мы создаём объект типа **GameTimer **и указываем частоту его увеличения. Величина 333333 - стандартная для WP7 и соответствует 30 кадрам в секунду.

```csharp
public GamePage()
{
    InitializeComponent();

    // Get the content manager from the application
    contentManager = (Application.Current as App).Content;

    // Create a timer for this page
    timer = new GameTimer();
    timer.UpdateInterval = TimeSpan.FromTicks(333333);
    timer.Update += OnUpdate;
    timer.Draw += OnDraw;
}
```

Затем в методе **OnNavigatedTo **мы указываем что делать при переходе на заданную страницу. В этом методе стоит загружать контент для приложения. Так как сейчас мы рассматриваем стандартный шаблон, то и загружать ничего не будем.


``` csharp
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    // Set the sharing mode of the graphics device to turn on XNA rendering
    SharedGraphicsDeviceManager.Current.GraphicsDevice.SetSharingMode(true);

    // Create a new SpriteBatch, which can be used to draw textures.
    spriteBatch = new SpriteBatch(SharedGraphicsDeviceManager.Current.GraphicsDevice);

    // TODO: use this.content to load your game content here

    // Start the timer
    timer.Start();

    base.OnNavigatedTo(e);
}
```


Далее идут самые интересные для нас методы - **OnUpdate **и **OnDraw**. В них мы задаём логику приложения (аналог **Update **в чистом XNA) и выводим изображение на экран (аналог, соответственно, метода **Draw**).

``` csharp
/// Allows the page to run logic such as updating the world,
/// checking for collisions, gathering input, and playing audio.
///
private void OnUpdate(object sender, GameTimerEventArgs e)
{
    // TODO: Add your update logic here
}

/// Allows the page to draw itself.
///
private void OnDraw(object sender, GameTimerEventArgs e)
{
    SharedGraphicsDeviceManager.Current.GraphicsDevice.Clear(Color.CornflowerBlue);

    // TODO: Add your drawing code here
}
```


Основное преимущество использования GameTimer таким способом является то, что можно создавать различные таймеры для главного игрового цикла и для каких-то игровых процессов. Кроме того, поскольку GameTimer использует события, можно использовать несколько различных методов для рисования и обновления логики для различных частей игры.

## 5. Логика, использующая GameTime
Если вы перестраиваете проект с чистого XNA на Silverlight+XNA, то при портировании логики, использующей **GameTime **могут возникнуть проблемы. Поскольку **GameTime **был тесно связан с **Game Class**, а в приложениях Silverlight+XNA используется интегрированная модель, то вам придется использовать **GameTimerEventArgs**, в которой хранится информация о времени, которая хранилась в **GameTime**.

Если ваше приложение использует **GameTime **для игровой логики, вы можете легко его портировать используя вместо **Game Class** (предоставляющего информацию о времени ранее) **GameTime**.

Было:

``` csharp
	public void Update(GameTime gameTime) {}
```

Стало:



``` csharp
	public void Update(TimeSpan elapsedTime, TimeSpan totalTime) {}
```


Учтите, что в данном случае под игровой логикой подразумевается подразумевается какая-то ваша логика, из дополнительных классов вашего приложения. Основной же цикл обработки (**Update **или **OnUpdate**) всё равно должны принимать либо **GameTime **(для приложений на XNA, использующих **Game Class**), либо **GameTimeEventArgs **(для приложений на Silverlight+XNA).

## 6. Игровые компоненты
Что делать с игровыми компонентами, если приложение, основанное на **Game Class** использовало **GameComponents **для реализации логики? К сожалению, GameComponents так же как и **GameTime **относятся к **Game Class**, поэтому их невозможно использовать в приложении Silverlight+XNA.

Существует несколько способов решения этой проблемы, самый простой из которых - сделать все методы публичными. В таком случае ваше приложение может просто создать список объектов и использовать их по мере необходимости, обращаясь к ним непосредственно по мере необходимости. Плюс данного способа - в простоте, так как вам не придется переписывать много кода. Минус же в том, что вы усложните логику приложения, возложив на страницы лишнюю работу.

Другой способ - модифицировать ваши методы таким образом, чтобы они подписывались на события **Update** и **Draw**. Это позволит вам создавать объекты, которые смогут подключаться к существующим циклам обновления и рисования, но это тесно свяжет ваш код с XNA+Silverlight фрэймворком и могут возникнуть проблемы, если вы захотите использовать этот код для Windows/XBox 360.

Последним, самым лучшим решением, будет переопределение базовых типов и интерфейсов для **GameComponent**. Это потребует большего понимания принципов работы **GameComponent**, но позволит вам переносить код с Windows Phone на Windows и XBox.

**В следующей статье мы создадим первое приложение, использующее компоненты Silverlight и графику XNA на одной странице.**

Материал по миграции приложений - вольный перевод с дополнениями инструкции с сайта [create.msdn.com](http://create.msdn.com/en-US/education/catalog/article/migration_guide_moving_to_silverlight_xna).


