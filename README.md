AITrain: детекция препятствий и элементов дорожной инфраструктуры
=================================

Соревнование алгоритмов определения препятствий и элементов ЖД-инфраструктуры.  

Одним из способов повышения безопасности ЖД перевозок является создание интеллектуальных систем, предупреждающих машиниста о возможном столкновении с потенциально опасными объектами. В рамках соревнования требуется создать алгоритм, детектирующий такие объекты, а также определяющий элементы дорожной инфраструктуры, определяющие движение поезда - рельсы, стрелочные переводы и светофоры.  
В качестве входных данных используются RGB изображения, полученные с различных камер, установленных на электропоезде и аннотации к ним.

## Постановка задачи

На основе размеченных фотографий необходимо создать алгоритм детекции следующих объектов:
- "Car" (автомобиль),
- "Human" (человек),
- "Wagon" (вагон)*,
- "FacingSwitchL" (стрелочный перевод по ходу движения, влево),
- "FacingSwitchR" (стрелочный перевод по ходу движения, вправо),
- "FacingSwitchNV" (стрелочный перевод по ходу движения, вне видимости),
- "TrailingSwitchL" (стрелочный перевод против хода движения, влево),
- "TrailingSwitchR" (стрелочный перевод против хода движения, вправо),
- "TrailingSwitchNV" (стрелочный перевод против хода движения, вне видимости),
- "SignalE" (разрешающий сигнал светофора),
- "SignalF" (запрещающий сигнал светофора).

Одновременно требуется разметить сегментационные маски для следующих элементов:
 - 6 - "MainRailPolygon" (главный ЖД-полигон);
 - 7 - "AlternativeRailPolygon" (вспомогательный ЖД-полигон);
 - 10 - "Train" (поезд)*.

Примечание: требуется детектировать первый и последний вагоны поезда, для остальных вагонов требуются сегментационные маски.


## Формат решения

В проверяющую систему необходимо отправить код алгоритма, запакованный в ZIP-архив. Решения запускаются в изолированном окружении при помощи Docker. Время и ресурсы во время тестирования ограничены. Участнику нет необходимости разбираться с технологией Docker.

### Содержимое контейнера

В корне архива обязательно должен быть файл metadata.json следующего содержания:
```json
{
    "image": "sberbank/rgd-python",
    "entry_point": "python detect_railway_objects.py $PATH_INPUT $PATH_OUTPUT/output.csv"
}
```

Здесь `image` — поле с названием docker-образа, в котором будет запускаться решение, `entry_point` — команда, при помощи которой запускается решение. Для решения текущей директорией будет являться корень архива. 

Во время запуска, в переменной окружения `DATASETS_PATH` расположен путь к актуальным открытым наборам данных, которые доступны из контейнера с решением.

Для запуска решений можно использовать существующие окружения:

- `sberbank/rgd-python` — Python3 с установленным большим набором библиотек
- `gcc` - для запуска компилируемых C/C++ решений
- `node` — для запуска JavaScript
- `openjdk` — для Java
- `mono` — для C#

Подойдет любой другой образ, доступный для загрузки из DockerHub. При необходимости, Вы можете подготовить свой образ, добавить в него необходимое ПО и библиотеки (см. инструкцию по созданию Docker-образов); для использования его необходимо будет опубликовать на DockerHub.

### Ограничения

Контейнер с решением запускается в следующих условиях:

- решению доступны ресурсы
  - 16 Гб оперативной памяти;
  - 4 vCPU;
  - 1 GPU Tesla V100.
- время на выполнение решения: 10 минут
- решение не имеет доступа к ресурсам интернета
- максимальный размер упакованного и распакованного архива с решением: 3 Гб
- максимальный размер используемого Docker-образа: 10 Гб

## Проверка качества

В качестве метрики качества Третьей Задачи Конкурса используется метрика panoptic quality (PQ).

PQ – это метрика, используемая для оценки эффективности моделей сегментации (Panoptic Segmentation). В числителе дроби суммируются коэффициенты Intersection over Union (IoU) для всех True Positive решений модели. В знаменателе суммируются по модулю число всех True Positive результатов модели, половина всех False Positive и False Negative результатов.

Пояснения:
Intersection over Union (IoU) – коэффициент, который показывает, насколько точно модель определила местоположение объекта на конкретном изображении (принимает значения от 0 до 1).
Решение считается True Positive, если IoU>0,5.
Решение считается False Positive, если 0<IoU<0,5.
Решение считается False Negative, если 0=IoU

Качество решения оценивается на отложенной выборке из 1 050 RGB изображений такого же формата как у тренировочных данных. По значениям метрики panoptic quality формируется лидерборд.

В случае одинаковых значений метрики panoptic quality несколькими Участниками их решения оцениваются по метрике processing time (время, затраченное на обработку задач).  

Для финальной оценки можно выбрать 3 решения. По умолчанию это решения с наилучшей метрикой на лидерборде.
