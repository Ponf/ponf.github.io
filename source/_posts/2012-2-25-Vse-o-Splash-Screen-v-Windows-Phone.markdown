---
layout: post
title: Все о Splash Screen в Windows Phone
date: Sat, 25 Feb 2012 09:17:00 +0000
comments: true
---



Какое бы вы приложение не писали, даже самое простое, его запуск на телефоне будет занимать некоторое время, поэтому хорошей идеей будет показывать какой-нибудь Splash Screen во время загрузки. Существует несколько вариантов использования Splash Screen:


* Использовать изображение
* Использовать анимированный Splash Screen
* Не использовать Splash Screen


Давайте поподробнее остановимся на каждом из этих пунктов.

<!--more--> 


### Использование изображения 

При создании проекта в Visual Studio у вашего приложения уже есть Splash Screen по-умолчанию:

{% img http://1.bp.blogspot.com/-NPNjM2xI4Iw/T0nuhjRsEoI/AAAAAAAAAKM/Y6Eys9BCasA/s200/SplashScreenImage.jpg %}

Для замены его на свой Splash Screen необходимо просто добавить в проект файл с именем SplashScreenImage.jpg (**внимание**: имя должно быть именно таким). Также у вашего Splash Screen  разрешение должно быть 480х800px (480 пикселей в ширину и 800, соответственно, в высоту), и не забудьте указать в свойствах изображения Build Action: Content.

### Использование анимированного Splash Screen

Если загрузка первой страницы вашего приложения занимает некоторое время (например, используются какие-то данные из Интернета), то создание анимированного экрана загрузки – так же хорошая идея. Итак, алгоритм работы  будет следующий:


* При загрузке изображения показывается SplashScreenImage.jpg 
* После загрузки открывается **MainPage** с открытым **Popup **окном, развернутым на весь экран. Этот **Popup** может быть основан на вашем **SplashScreenImage** для создания наилучшего эффекта
* Как только загрузка данных закончится – **Popup** закроется, и пользователь увидит содержимое **MainPage**.


Как вы можете заметить, термин «анимированный Splash Screen» на самом деле не совсем верен: мы используем обычный статический экран заставки, а потом просто показываем пользователю страницу, похожую на экран заставки, тем временем выполняя в фоне какие-нибудь действия.

Для того чтобы наша загрузка выполнялась в отдельном потоке мы будем использовать класс [**BackgroundWorker**](http://msdn.microsoft.com/en-us/library/system.componentmodel.backgroundworker.aspx). Этот класс позволяет делать какую-то работу в фоне, оставляя UI отзывчивым для пользователя. Также у нас будет возможность отслеживать состояние фонового процесса и убрать **Popup **при наступлении какого-нибудь события.

Для начала давайте создадим свой UserControl, для примера назовем его **AnimatedSplashScreen**.

Далее нам нужно добавить обработку **Popup** в MainPage.xaml.cs. Для этого добавим следующие пространства имен:

``` csharp
using System.Threading;
using System.Windows.Controls.Primitives;
using System.ComponentModel;
```

Теперь необходимо создать BackgroundWorker и наш Popup:

``` csharp
BackgroundWorker backroungWorker;
Popup myPopup;
// Constructor
public MainPage()
{
    InitializeComponent();
    myPopup = new Popup() { IsOpen = true, Child = new AnimatedSplashScreen() };
    backroungWorker = new BackgroundWorker();
    RunBackgroundWorker();
}

private void RunBackgroundWorker()
{
    backroungWorker.DoWork += ((s, args) =>
    {
        Thread.Sleep(5000);
    });

    backroungWorker.RunWorkerCompleted += ((s, args) =&gt;
    {
        this.Dispatcher.BeginInvoke(() =&gt;
        {
            this.myPopup.IsOpen = false;
        }
    );
    });
    backroungWorker.RunWorkerAsync();
}
```

Что мы здесь делаем:

* Создаем **Popup**, по-умолчанию открытый, который содержит наш UserContol – **AnimatedSplashScreen**;
* Создаем **BackgroundWorker**, который в фоне замораживает поток на 5000 мс.
* Как только 5 секунд проходит, **BackgroundWorker **переходит в состояние **RunWorkerCompleted **и наш **Popup **закрывается.

Основная логика есть, теперь давайте вернемся к содержимому нашего **Popup**. У себя в примере я использую изображение **SplashScreenImage**, **TextBlock** и **PerformanceProgressBar** из **[WP7 Toolkit](http://silverlight.codeplex.com/releases/view/75888)**.



Для того чтобы немного оживить загрузку, я в **Expression Blend**создал анимацию мигания для текста:



Теперь осталось только добавить и запустить эту анимацию в конструкторе класса AnimatedSplashScreen:

``` csharp
public AnimatedSplashScreen()
{
    InitializeComponent();
    Storyboard Blinking = this.Resources["Blinking"] as Storyboard;
    Blinking.Begin();
}
```
Всё, наш анимированный **Splash Screen** готов! Запустив проект, мы увидим следующую картину:


{% img http://1.bp.blogspot.com/-2gjXoRFZl_E/T0n2-jZ0b5I/AAAAAAAAAKU/q0ggrmwHG4E/s320/screen.png %}

Пример моего анимированного** Splash Screen** вы можете скачать по ссылке: [SkyDrive](https://skydrive.live.com/redir.aspx?cid=a2567e038f37be43&amp;resid=A2567E038F37BE43!512&amp;parid=A2567E038F37BE43!463).



В качестве альтернативы **Popup** можно использовать отдельную страницу, но тогда нужно предусмотреть чтобы при нажатии на кнопку **Назад **пользователь выходил из приложения, а не попадал снова на страницу загрузки. Это можно сделать, например, удалив страницу из стека навигации:

``` csharp
if (NavigationService.CanGoBack == true)
{
    NavigationService.RemoveBackEntry();
}
```

### Не использовать Splash Screen 

Для того чтобы убрать у вашего приложения экран заставки, достаточно просто удалить файл SplashScreenImage.jpg из вашего решения.

Итак, мы рассмотрели различные варианты создания Splash Screen в Windows Phone. Надеюсь, что эта статья оказалась для вас полезной :).


