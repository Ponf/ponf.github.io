---
layout: post
title: Как склеить несколько Texture2D / Howto merge few Texture2D
date: Fri, 16 Dec 2011 14:19:00 +0000
comments: true
---

Решил попробовать написать этот пост и на русском, и на английском, так как очень долго гуглил о том как склеить несколько текстур формата Texture2D и сохранить результат, но не нашел ничего подходящего: либо метод не поддерживался в Windows Phone, либо меня не устраивала его производительность.

**Проблема**
Есть несколько текстур с разными координатами, находящиеся на разных уровнях и никак друг с другом не связанные. Необходимо экспортировать это в фото галерею в виде изображения.

**Решение**
В общем-то задача действительно не сложная, но сходу не понятно, как её решить. Нужно как-то склеить все текстуры в одну. И тут нам на помощь приходит класс из XNA под названием [RenderTarget2D](http://msdn.microsoft.com/en-us/library/bb198676.aspx).

<!--more--> 

``` csharp
	RenderTarget2D imageResult;
```

Как можно понять из названия, с помощью этого класса, наследующегося от Texture2D, мы создаем двумерную текстуру, которую потом можно использовать в выборе цели для рендеринга.

По-умолчанию целью для рендеринга является дисплей:

``` csharp
	// Set the sharing mode of the graphics device to turn on XNA rendering
	SharedGraphicsDeviceManager.Current.GraphicsDevice.SetSharingMode(true);
	
	// Create a new SpriteBatch, which can be used to draw textures.
	spriteBatch = new SpriteBatch(SharedGraphicsDeviceManager.Current.GraphicsDevice);
```

При разработки игры с использованием XNA+Silverlight установка цели для рендера происходит в методе OnNavigatedTo(), и вся дальнейшая отрисовка будет производиться, соответственно, на дисплей. Так что для того, чтобы отрендерить все на другую текстуру достаточно просто указать её в SpriteBatch:

``` csharp
	imageResult = new RenderTarget2D(SharedGraphicsDeviceManager.Current.GraphicsDevice, photoW, photoH);
	SharedGraphicsDeviceManager.Current.GraphicsDevice.SetRenderTarget(imageResult);
```


Если мы вставим этот код так же в метод OnNavigatedTo(), то весь вывод информации будет производиться не на дисплей, а в нашу текстуру. Нам этого не нужно, так что необходимо менять цель для рендеринга по какому-то определенному событию, например, по нажатию на кнопку. Далее мы отрисовываем все наши текстуры, как мы бы это делали обычно в методе OnDraw():

``` csharp
	spriteBatch.Begin();
	bg.Draw(spriteBatch, new Vector2((int)(photoW / 2), (int)(photoH / 2)));
	foreach (Sprite sprite in sprites)
	{
	    sprite.Draw(spriteBatch, new Vector2((int)(photoW / 2), (int)(photoH / 2)), bg.Scale);
	}
	spriteBatch.End();
```

После того, как мы закончили отрисовывать наши текстуры необходимо освободить наше устройство для рендеринга, чтобы оно вернулось к отрисовке на экране. Для этого достаточно передать в null в метод SetRenderTarget:

``` csharp
	SharedGraphicsDeviceManager.Current.GraphicsDevice.SetRenderTarget(null);
```

Всё! Теперь мы можем сохранить наше изображение в изолированном хранилище, галерее или использовать его где-нибудь ещё.

``` csharp
	IsolatedStorageFileStream stream = new IsolatedStorageFileStream("/image.jpg", System.IO.FileMode.OpenOrCreate, iso);
	imageResult.SaveAsJpeg(stream, imageResult.Width, imageResult.Height);
	stream.Close();
```


О том, как сохранить наше изображение в галерее вы можете прочитать в моей предыдущей статье "[Интеграция приложения с хабом Фото](http://mne.p0x.ru/2011/12/blog-post.html)".

