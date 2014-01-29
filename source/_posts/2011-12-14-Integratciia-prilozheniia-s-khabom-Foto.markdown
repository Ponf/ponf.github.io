---
layout: post
title: Интеграция приложения с хабом "Фото"
date: Wed, 14 Dec 2011 10:08:00 +0000
comments: true
---



Однажды мне понадобилось интегрировать приложение в хаб "Фото" в Windows Phone 7. Что я понимаю под интеграцией:

* Добавление приложения на главную страницу хаба;
* Добавление приложения в меню "Программы" (программы, которыми можно отредактировать изображение);
* Выбор в программе изображения из галереи и с камеры;
* Сохранение отредактированного изображения в галерее.


Вроде бы все понятно, в интернете много информации и никаких проблем возникнуть не должно, но на практике, к сожалению, не всё так просто.

<!--more--> 


## 1. Добавление приложения на главную страницу хаба Фото

Беглый поиск [показал](http://programming4.us/mobile/3758.aspx), что для интеграции приложения в хаб Pictures нам необходимо создать файл Extra.xml со следующим содержимым:



``` xml
<Extras>
  <PhotosExtrasApplication>
    <Enabled>true</Enabled> 
  </PhotosExtrasApplication>
</Extras>
```


И создать некоторый обработчик в методе OnNavigatedTo().

Сделал я по этой инструкции и ничего у меня не заработало. Оказалось, что с приходом WP7 Mango все немного изменилось. Таким образом, метод, приведенный выше, не работает!



С приходом WP7 Mango нам теперь не нужно создавать отдельный файл Extra.xml, информация о всех расширениях должна находиться в WMAppManifest.xml.

Для отображения нашего приложения в хабе Фото необходимо в файл WMAppManifest.xml добавить следующие строчки:

``` xml
<Extensions>
      <Extension ExtensionName="Photos_Extra_Hub" ConsumerID="{5B04B775-356B-4AA0-AAF8-6491FFEA5632}" TaskID="_default" /> 
</Extensions>
```

## 2. Добавление приложения в контекстное меню хаба Фото

Здесь механизм добавления приложения не отличается от добавления приложения на главную страницу хаба, только ExtensionName будет "Photos_Extra_Viewer", а ConsumerID станет "{5B04B775-356B-4AA0-AAF8-6491FFEA5632}".


В итоге WMAppManifest.xml примет следующий вид:

``` xml
        </Tokens>
    	<Extensions>
      		<Extension ExtensionName="Photos_Extra_Hub" ConsumerID="{5B04B775-356B-4AA0-AAF8-6491FFEA5632}" TaskID="_default" />
      		<Extension ExtensionName="Photos_Extra_Viewer" ConsumerID="{5B04B775-356B-4AA0-AAF8-6491FFEA5632}" TaskID="_default" />
   		</Extensions>
  	</App>
</Deployment>
``` 


## 3. Создание обработчика

Итак, теперь мы умеем не только вызывать нашу программу из хаба, а ещё и передавать ей любое изображение. Теперь надо его обрабатывать. Для этого необходимо перегрузить метод OnNavigatedTo:

``` csharp    
    BitmapImage bitmap;

    protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        base.OnNavigatedTo(e);

        //Get a dictionary of query string keys and values
        IDictionary<string, string> queryStrings = this.NavigationContext.QueryString;

        //WaitingPic.Visibility = Visibility.Collapsed;
        //Ensure at least one querysting
        if (queryStrings.ContainsKey("token"))
        {
            MediaLibrary library = new MediaLibrary();
            Picture picture = library.GetPictureFromToken(queryStrings["token"]);

            //Create bitmap
            bitmap = new BitmapImage();
            bitmap.SetSource(picture.GetImage());
            this.NavigationContext.QueryString.Clear();
        }
    }
```

Теперь мы можем делать с нашим изображением всё, что захотим :).

## 4. Получение изображений из камеры и галереи
Также мы хотим осуществить в нашем приложении доступ к изображениям в галерее и, если нужное изображение отсутствует, предоставить возможность сделать фото.

Для этого нам необходимо использовать PhotoChooserTaskиCameraCaptureTask, которые находятся в пространстве имён Microsoft.Phone.Tasks.

В общем случае код выглядит примерно так:
``` csharp
    private void ChoosePhoto_Click(object sender, RoutedEventArgs e)
    {
        try
        {
            selectphoto = new PhotoChooserTask();
            selectphoto.Completed += new EventHandler<PhotoResult>(photo_Completed);
            selectphoto.Show();
        }
        catch
        {
            MessageBox.Show("Some problem with loading your page");
            return;
        }
    }

    private void LaunchCamera_Click(object sender, RoutedEventArgs e)
    {
        CameraCaptureTask cameraCapture = new CameraCaptureTask();
        cameraCapture.Completed += new EventHandler<PhotoResult>(photo_Completed);
        cameraCapture.Show();
    }

    void photo_Completed(object sender, PhotoResult e)
    {
        if (e.TaskResult == TaskResult.OK)
        {
            try
            {
                bitmap = new BitmapImage();
                bitmap.UriSource = new Uri(e.OriginalFileName);
            }
            catch
            {
                MessageBox.Show("Some problem with loading your page");
            }
        }
    }
```


Как вы можете видеть, ничего сложно в этом нет, данный код вернет нам изображение в приложение. Но мне такая реализация не очень нравится. Я предпочитаю делать следующее:

``` csharp
    void photo_Completed(object sender, PhotoResult e)
    {
        if (e.TaskResult == TaskResult.OK)
        {
            try
            {
                bitmap = new BitmapImage();
                bitmap.CreateOptions = BitmapCreateOptions.None;
                bitmap.ImageOpened += new EventHandler<RoutedEventArgs>(bitmap_ImageOpened);
                bitmap.ImageFailed += new EventHandler<ExceptionRoutedEventArgs>(bitmap_ImageFailed);
                bitmap.UriSource = new Uri(e.OriginalFileName);
            }
            catch 
            {
                MessageBox.Show("Some problem with loading your page");
            }
        }
    }

    public void bitmap_ImageOpened(object sender, RoutedEventArgs e)
    {
        try
        {
            //Делаем что-нибудь с нашим изображением
        }
        catch 
        {
             //Обрабатываем ошибку 
        }
    }

    public void bitmap_ImageFailed(object sender, RoutedEventArgs e)
    {
        MessageBox.Show("Problem with loading your image. Please try again.");
    }
```

Теперь наше приложение умеет ещё выбирать изображения из галереи и делать фотографии.

Осталось только сохранять результат работы в галерее.

## 5. Сохранение изображения в галерее

Для сохранения изображения в галерее нам во-первых необходимо подключить следующие пространства имен:

``` csharp
    using System.IO.IsolatedStorage;
    using Microsoft.Xna.Framework.Media;
```



Теперь мы можем сохранить фотографию:

``` csharp
    try
    {
        IsolatedStorageFileStream stream = new IsolatedStorageFileStream("/image.jpg", System.IO.FileMode.OpenOrCreate, iso);
        imageResult.SaveAsJpeg(stream, imageResult.Width, imageResult.Height);
        stream.Close();

        stream = iso.OpenFile("/image.jpg", System.IO.FileMode.Open, System.IO.FileAccess.Read);
        MediaLibrary library = new MediaLibrary();
        Picture pic = library.SavePicture("SavedPicture.jpg", stream);
        MessageBox.Show("Image saved to saved pictures album");
        stream.Close();
    }
    catch
    {
        MessageBox.Show("Some problem with saving image :(");
    }
```

Сначала мы сохраняем изображение в изолированном хранилище. В данном примере - imageResult - экземпляр класса RenderTarget2D, который наследуется от Texture2D, у которого, в свою очередь, есть методы для сохранения в png и jpg. Мы записываем jpg файл в изолированные хранилище, затем создаем поток на чтение этого файла. Наконец, нам нужно передать поток чтения в метод SavePicture экземпляра класса MediaLibrary для сохранения изображения в галерее. Все, изображение сохранять мы тоже научились.

## 6. Заключение

Мне кажется, в этой статье я рассмотрел все кейсы, которые могут пригодиться при разработке приложения для работы с фотографиями и изображениями. Возможно, я недостаточно внимания уделил задачам выбора изображения и работе с камерой, но вся эта информация есть в учебном курсе по [Windows Phone](http://msdn.microsoft.com/ru-ru/windowsphone/hh420944#mark_7). Если у вас остались какие-либо вопросы, не стесняйтесь задавать их в комментариях.

Также хочется напомнить, что сейчас идет новый цикл вечерней школы со Стасом Павловым

[http://msdn.microsoft.com/ru-ru/windowsphone/hh544009](http://msdn.microsoft.com/ru-ru/windowsphone/hh544009), на которой вы можете научиться разрабатывать под Windows Phone, а так же задать все интересующие вопросы. Ближайшее занятие состоится 22 декабря, не пропустите!