# Домашнее заание №4. Apache Druid.
## История развития СУБД.
Apache Druid – это колоночная открытая база данных, разработка которой была начата в 2013 году и продолжается до сих пор. Druid спроектирован с целью быстрой обработки больших, редко изменяющихся массивов данных и немедленного предоставления доступа к ним.Он был разработан с целью обслуживания и поддержания 100% времени безотказной работы перед лицом развертывания кода, сбоев машин и других возможностей производственной системы.

## Инструменты для взаимодействия с СУБД.
Apache Druid имеет много интеграций с различными технологиями, такими как Kafka, Cloud Storage, S3, Hive, HDFS, DataSketches, Redis и т. Д.

## Какой database engine используется в вашей СУБД?
Druid хранит данные в источниках данных, которые аналогичны таблицам в традиционной системе управления реляционными базами данных (СУБД). Модель данных Druid имеет общие черты как с реляционными моделями данных, так и с моделями данных временных рядов.

## Как устроен язык запросов в вашей СУБД? Разверните БД с данными и выполните ряд запросов. 
Для установки Druid нужно скачать [последний релиз](https://www.apache.org/dyn/closer.cgi?path=/druid/0.22.1/apache-druid-0.22.1-bin.tar.gz)
и разархивировать его командами 
``tar -xzf apache-druid-0.22.1-bin.tar.gz``
``cd apache-druid-0.22.1``
Для запуска используем инструмент от разработчиков micro-quickstart
``./bin/start-micro-quickstart``
После чего мы можем открыть Druid console по адресу http://localhost:8888
Выглидит это так:
![1](https://github.com/PavelSubbotin/Apache_Druid_Overview/blob/main/pictures/1.png)
После добавления рекомендованного датасета нажимаем на значок настроек и выбираем Query with SQL:
![4](https://github.com/PavelSubbotin/Apache_Druid_Overview/blob/main/pictures/4.png)
Примеры запросов: 
![2](https://github.com/PavelSubbotin/Apache_Druid_Overview/blob/main/pictures/2.png)
![3](https://github.com/PavelSubbotin/Apache_Druid_Overview/blob/main/pictures/3.png)
Очень подробно весь процесс настройки и запуска запросов описан в [официальной докусентации](https://druid.apache.org/docs/latest/design/index.html)
Для написания запросов используются ["native queries"](https://druid.apache.org/docs/latest/querying/querying.html) на основе JSON и [Druid SQL](https://druid.apache.org/docs/latest/querying/sql.html), преобназующий SQL запросы с небольшими ограничениями в "native queries"
Схема SQL подобного запроса:
![7](https://github.com/PavelSubbotin/Apache_Druid_Overview/blob/main/pictures/7.png)
Демобаза с запросами хранится на гитхабе [вот здесь](https://github.com/apache/druid/tree/master/examples) и помимо запросов содержит множество примеров различных настроек и тренировочных датасетов


## Распределение файлов БД по разным носителям?
Схема потока данных в Druid имеет следующий вид:
![6](https://github.com/PavelSubbotin/Apache_Druid_Overview/blob/main/pictures/6.png)
Причем в качестве storage может выступать HDFS или Amazon S3, так что воозможность распределенного хранения обеспечивается этими сервисами

## На каком языке/ах программирования написана СУБД?
Проект целиком написан на Java, небольшая часть кода, которая, как я понял, связана с визуальным отображением написана на TypeScript

## Какие типы индексов поддерживаются в БД? Приведите пример создания индексов.
В Druid поддерживаются 2 типа задач для индексирования: 
* `index_parallel:` может выполнять несколько задач индексирования задач параллельно, хорошо работает в продакшене.
Пример:
``` json
{
  "type": "index_parallel",
  "spec": {
    "dataSchema": {
      "dataSource": "wikipedia_parallel_index_test",
      "timestampSpec": {
        "column": "timestamp"
      },
      "dimensionsSpec": {
        "dimensions": [
          "country",
          "page",
          "language",
          "user",
          "unpatrolled",
          "newPage",
          "robot",
          "anonymous",
          "namespace",
          "continent",
          "region",
          "city"
        ]
      },
      "metricsSpec": [
        {
          "type": "count",
          "name": "count"
        },
        {
          "type": "doubleSum",
          "name": "added",
          "fieldName": "added"
        },
        {
          "type": "doubleSum",
          "name": "deleted",
          "fieldName": "deleted"
        },
        {
          "type": "doubleSum",
          "name": "delta",
          "fieldName": "delta"
        }
      ],
      "granularitySpec": {
        "segmentGranularity": "DAY",
        "queryGranularity": "second",
        "intervals": [
          "2013-08-31/2013-09-02"
        ]
      }
    },
    "ioConfig": {
      "type": "index_parallel",
      "inputSource": {
        "type": "local",
        "baseDir": "examples/indexing/",
        "filter": "wikipedia_index_data*"
      },
      "inputFormat": {
        "type": "json"
      }
    },
    "tuningConfig": {
      "type": "index_parallel",
      "partitionsSpec": {
        "type": "single_dim",
        "partitionDimension": "country",
        "targetRowsPerSegment": 5000000
      },
      "maxNumConcurrentSubTasks": 2
    }
  }
}
```
* `index:` единовременно выполняет одну задачу индексации. Испоользуется в средах разработки и тестирования.
Пример:
``` json
{
  "type" : "index",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "wikipedia",
      "timestampSpec" : {
        "column" : "timestamp",
        "format" : "auto"
      },
      "dimensionsSpec" : {
        "dimensions": ["country", "page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","region","city"],
        "dimensionExclusions" : []
      },
      "metricsSpec" : [
        {
          "type" : "count",
          "name" : "count"
        },
        {
          "type" : "doubleSum",
          "name" : "added",
          "fieldName" : "added"
        },
        {
          "type" : "doubleSum",
          "name" : "deleted",
          "fieldName" : "deleted"
        },
        {
          "type" : "doubleSum",
          "name" : "delta",
          "fieldName" : "delta"
        }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "DAY",
        "queryGranularity" : "NONE",
        "intervals" : [ "2013-08-31/2013-09-01" ]
      }
    },
    "ioConfig" : {
      "type" : "index",
      "inputSource" : {
        "type" : "local",
        "baseDir" : "examples/indexing/",
        "filter" : "wikipedia_data.json"
       },
       "inputFormat": {
         "type": "json"
       }
    },
    "tuningConfig" : {
      "type" : "index",
      "partitionsSpec": {
        "type": "single_dim",
        "partitionDimension": "country",
        "targetRowsPerSegment": 5000000
      }
    }
  }
}
```
## Как строится процесс выполнения запросов в вашей СУБД?
Схема выполнения запроса:
![8](https://github.com/PavelSubbotin/Apache_Druid_Overview/blob/main/pictures/8.png)
В обработке запросов участвуют ноды трёх видов: broker, realtime и historical. Запрос приходит в broker, который знает, на каких нодах какие сегменты находятся. Он распределяет запрос по historical (и realtime) нодам, хранящим нужные сегменты. Historical ноды также распараллеливают вычисления насколько это возможно, отправляют результаты брокеру, а тот отдает их клиенту. Благодаря сочетанию этой схемы с колоночным хранением данных Druid может очень быстро обрабатывать большие объемы информации.
## Есть ли для вашей СУБД понятие «план запросов»? Если да, объясните, как работает данный этап.
Для планирования запроса в Druid используется [apache calcite](https://calcite.apache.org/), в котором за создания плана зпроса отвечают классы VolcanoPlanner, использующий подход динамического программирования для построения плана, и класс class HepPlanner, запускающий последовательность правил в более фиксированном порядке. Так же есть возможность испоользования кастомного класса. К сожалению подробной информации о работе данных классов найти не удалось, а по исходному коду [volcano](https://github.com/apache/calcite/blob/b9c2099ea92a575084b55a206efc5dd341c0df62/core/src/main/java/org/apache/calcite/plan/volcano/VolcanoRuleCall.java) понять механизм действия очень сложно и трудозатратно.
Для того, чтобы узнать как именно будет выполняться запрос, можно написать ``EXPLAIN PLAN FOR`` перед ``SELECT``, в таком случае запрос фактически не будет выполнен, но будет получен план его выполнения, для интерпретации которого можно использовать [документацию](https://druid.apache.org/docs/latest/querying/sql.html#query-translation) 

## Поддерживаются ли транзакции в вашей СУБД? Если да, то расскажите о нем. Если нет, то существует ли альтернатива?
В Druid ingection осуществляется только для добавления (для потоков) и для массовоой перезаписи/добавления (для бачей), что не вписывается в концепцию транзакций ACID.

## Какие методы восстановления поддерживаются в вашей СУБД. Расскажите о них.
Как я понял в Druid нет специальных методов для восстановления данных, и так как в качестве хранилища данных могут использоваться различные решение такие как HDFS и Amazon S3 резервные копии данных создаются и хранятся на их уровне, HDFS, например, поддерживает технологию "снэпшотинга", так же в Apache Hadoop есть утилита [DistCp](https://hadoop.apache.org/docs/stable1/distcp2.html) (distributed copy). Amazon так же имеет свои внутренние возможности для резервного копирования.

## Расскажите про шардинг в вашей конкретной СУБД. Какие типы используются? Принцип работы.
Druid поддерживает двухуровневый механизм распределения данных. Так как данный проект заточен под быструю онлайн обработку, в первую очередь данные делятся по времени на time chunks, внутри чанков так же используется вторичное распределение, для увеличения перфоманса авторы так же советуют разделить данные по "натуральному" измерению, которое часто используется в запросах. 
![5](https://github.com/PavelSubbotin/Apache_Druid_Overview/blob/main/pictures/5.png)
[Официальная документация про это](https://druid.apache.org/docs/latest/ingestion/partitioning.html)

## Возможно ли применить термины Data Mining, Data Warehousing и OLAP в вашей СУБД?
Для Data Mining нет механизмов. 

Термины Data Warehousing и OLAP отличноо применимы к Druid, благодаря многочисленным оптимизициям и заточенности на непрерывную онлайн обработку событий, о чем говорят и сами создатели [здесь](https://druid.apache.org/faq) и [здесь](https://druid.apache.org/druid).

## Какие методы защиты поддерживаются вашей СУБД? Шифрование трафика, модели авторизации и т.п.
Apache Druid поддерживает аутенфикацию пользователей и выдачу им ролей (ограничение прав на все действия, кроме необходимых)
Из возможных прав пользователей есть права на чтения и записать, на практике обычно используются 2 класса пользователей - администраторы имеющие право на запись и обычные пользователи имеющие права только для чтения.
Из возможных методов аутенфикации поддерживается традиционный метод с использованием пары логин, пароль, а так же аутенфикация через LDAP.
Так же в базе данных существует возможность включения transport layer security для шифрования данных.
Из дополнительных средств защиты есть воозможность установки паролей для защиты хранилищ ключей и метаданных.
Стоит отметить, что изначально все настройки безопасности выключены для упрощения развертывания базы, так что настройками безопасности нельзя пренебрегать.
Так же создатели дают много советов по обеспечению безопасности [вот здесь](https://druid.apache.org/docs/latest/operations/security-overview.html).

## Какие сообщества развивают данную СУБД? Кто в проекте имеет права на коммит и создание дистрибутива версий? Расскажите об этих людей и/или компаниях.
Druid создан известной компанией [apache](https://www.apache.org/), имеющей огромное количество [продуктов](https://www.apache.org/index.html#projects-list), в том числе известных и широкоиспользуемых, например, Hadoop.
Данная организация была основана в 1999 году как некоммерческая организация основанная на членстве и существующая на пожертвования. Занимается написание множества open source продуктов, многие из которых становятся популярными и высоко ценятся коммьюнити.
Членами правления компании являются
* Rich Bowen	
* Bertrand Delacretaz
* Christofer Dutz
* Roy T. Fielding	
* Sharan Foga
* Willem Ning Jiang
* Sam Ruby	
* Roman Shaposhnik
* Sander Striker

## Как продолжить самостоятельное изучение языка запросов с помощью демобазы. Если демобазы нет, то создайте ее.
Вот [здесь](https://github.com/apache/druid/tree/master/examples) есть примеры примерно на все сценарии работы с Druid, так что в случае необходимости, скорее всего, там можно найти необходимый код. Так как в Druid используется практически SQL, то для изучения языка запросов можно воспользоваться https://www.sql-ex.ru, официальной документацией и другими ресурсами, которых по данному языку полно в свободном доступе. В общем для того, чтобы полноценно начать пользоваться Druid следует развернуть свой сервер, при помощи [гайда](https://druid.apache.org/docs/latest/tutorials/index.html) и начать писать запросы, на уже знакомом SQL, в случае чего обязательно поможет демобаза или [официальная документация](https://druid.apache.org/docs/latest/design/index.html), которая очень неплохо написана. В случае возникновения вопросов авторы проекта призывают писать вопросы на druid-user@googlegroups.com 

## Где найти документацию и пройти обучение.
[официальная документация](https://druid.apache.org/docs/latest/design/index.html)
[хороший ресурс для обучения](https://www.sql-ex.ru)
Так как в запросах используется SQl с небольшими ограничениями, то довольно легко самому нагуглить интересующие вещи.

## Как быть в курсе происходящего?
Авторы предлагают следить за релизами [вот здесь](https://github.com/apache/druid/releases)
