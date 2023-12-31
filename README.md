# Формирование образа и предпочтений инопланетян
![alien.jpeg](https://github.com/underham2ter/sirius-ml-2023/blob/main/alien.jpeg?raw=true)

Земля стала популярной туристической точкой для жителей планеты Kepler-442 b. Оказывается, что земной кинематограф очень интересен пришельцам, способным проводить целые дни за просмотром обычного **телевизора**. Поэтому присутствие данного аппарата - ключевое условие при выборе отеля. 

Однако не все так просто, ведь вся жизнь на планете Kepler-442 b является растительной, а наши друзья - разумные растения. С их формой связаны еще несколько критериев: земное солнце проигрывает в мощности родной звезде пришельцев, поэтому в отеле обязательно должна быть **кухня**, приборы которой оказались довольно удобными для изготовления жизненно важных, питательных удобрений.

Также прищельцам нравится, когда в отеле есть хоть какие-то **растения**, ведь во время долгого путешествия они напоминают о родных краях и производят успокаивающий эффект.
Таким образом, всем фотографиям будем присваивать тэги, соответствующие следующим факторам: 
1. На фотографии есть телевизор
1. На фотографии изображена кухня
1. На фотографии есть растительность

# Тема и проблематика
### Что такое image captioning?
Как можно понять из названия image captioning - задача автоматизации создания текстового описания к изображению.
### Почему над этим работают?
Два применения, что сразу приходят на ум: автоматическое создание тэгов в соц. сетях для улучшения рекомендаций и помощь слабовидящим людям.
### Как формулируется задача?

# Обзор датасета
В работе я использовал датасет по ссылке [Hotels-50K](https://github.com/GWUvision/Hotels-50K) -> input -> dataset.tar.gz
### Что представлено на изображениях?
На изображениях представлен интерьер различных помещений отеля: от вестибюля до туалетных комнат в самих апартаментах.
### Сколько объектов в датасете? Сколько уникальных классов? Сбалансирован ли датасет?
В датасете представлено 1124215 уникальных объектов, разбитых по уникальным отелям на 50000 классов, которые сами являются сущностями 93 гостиничных сетей. Датасет не сбалансирован ни по уникальным отелям, ни по гостиничным сетям.
Hotel name | Number of photos
------------- |-------------
Quality Inn                 |3228
Quality Inn & Suites        |2348
Comfort Inn                 |2157
Econo Lodge                 |1846
Comfort Inn & Suites        |1673
...                         | ... 
Edgewater Inn Shady Cove     |  4
Super 8 Franklin Pa          |  4
Hotel Wagner                 |  4
Hotel Baseraa                 | 4

Chain name | Number of photos
------------- |-------------
unknown            |657626
Holiday Inn         |52995
Hampton             |28541
Best Western        |22517
Comfort Inn         |18773
...                    | ...  
Gaylord               |152
Curio                 |124
Hawthorn Suites        |41
Wingate                |30
Four Points            |15


Как видно из таблицы количество фотографий у одного класса может отличаться от другого на несколько порядков.
### Какие параметры у изображений? Размер фотографий?
Фото цветные, размер варьируется в большом диапазоне: может встретится 350x215 и 5312x2988.

# Обогащение датасета описаниями
Работа выполнялась в прикрепленном jupiter блокноте.

Есть два варианта запуска:

а) Скачать архив Hotels-50K.rar из репозитория, распаковать его в то же место что и solution_full.ipybn, после чего запустить скрипт с самого начала. Изображения отелей скачаются автоматически.

б) Скачать архив [Hotels-50K-images.zip](https://drive.google.com/file/d/1p4XIQRByxm0ct4672lC6g6yw6WQaYPC-/view?usp=sharing) с гугл диска с уже готовыми изображениями, распаковать его в то же место что и solution_full.ipybn. Тогда скрипт нужно запускать с части "Создание тэгов".

Для демонстрации работы алгоритма я предлагаю выбрать 50 случайных отелей, а количество фотографий ограничить одним десятком, чтобы не затрачивать дополнительные вычислительные ресурсы. Предположим, что такого количества фотографий хватит для описания индивидуального отеля.

### Выполнение работы. Трудности.
Основная трудность - выбор порога срабатывания для скора(logits), возвращаемого нейросетью. Задача нетривиальная: на изображение с кухней при текстовом запросе *a room with a kitchen* нейросеть может вернуть скор как 25, так и 20, в зависимости от случая. При этом если мы хотим поймать все фотографии с кухней, устанавливая низкий порог, мы получим кучу ложных положительных срабатываний.

Для решения этой проблемы я решил скармливать в модель два противоположных высказывания и считать истиной случай с наибольшим скором. Например: *a room with a kitchen*,  *a room without a kitchen*. Таким образом меняя порог от случая к случаю.

Однако в то время как с кухнями такой трюк сработал на ура, с детекцией телевизоров все оказалось гораздо сложнее. Нейросеть часто путает их с обычными картинами, уверенно заявляя, что видит перед собой телевизор.

В таких случаях всё-таки пришлось ввести твердый порог прохода для минимизации ложных положительных, чтобы случайно не обмануть пришельцев.

### Результат

![tagged_images.jpg](https://github.com/underham2ter/sirius-ml-2023/blob/main/tagged_images.jpg)

В результате из 500 картинок 5% получилось отсортировать по тэгам, но не идеально: телевизоры часто путаются с картинами, а кухни с туалетными комнатами. Большинство тэгов было присвоено в разделе телевизоров:

![barplot.png](tags_barplot.png?raw=true)

# Модификация изображений

![im3.jpg](https://github.com/underham2ter/sirius-ml-2023/blob/main/im3.jpg?raw=true)

![im3_drawing.jpeg](https://github.com/underham2ter/sirius-ml-2023/blob/main/im3_drawing.jpeg?raw=true)

![im3_inpainted.jpeg](https://github.com/underham2ter/sirius-ml-2023/blob/main/im3_inpainted.jpeg?raw=true)

Я думаю, добавление растительности в декор - самый интересный и логичный выбор. Телевизор добавить очень просто: нужно лишь нарисовать черный прямоугольник на стене комнаты, выделить его и сделать inpainting с нужным промтом. Кухни же физически невозможно добавить в интерьер комнаты, поэтому гостей обманывать мы не будем. 

Чтобы изменить только нужную нам часть изображения, необходимо наложить на эту область маску. Также для более успешного выполнения запроса я делал наброски зелени прямо на фотографии.

Про пририсовку растительности я нашел довольно полезный [тред на реддите](https://www.reddit.com/r/StableDiffusion/comments/10qq4fm/i_cant_make_sd_to_fill_this_room_with_way_more/). Промпты типа:

(very dense) (green ivy_1.5) on a (white wall),shadows

(wall covered with ivy)

(dense dark green grass_1.5)  on a brown hardwood floor

оказались очень полезными.

Про лучшие настройки я читал в этой [статье](https://stable-diffusion-art.com/inpainting_basics/)
