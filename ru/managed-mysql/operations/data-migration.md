# Миграция данных в {{ mmy-name }}

Чтобы перенести вашу базу данных в сервис {{ mmy-name }}, нужно непосредственно перенести данные, закрыть старую базу данных на запись и перенести нагрузку на кластер БД в Яндекс.Облаке.

Перенести данные в кластер {{ mmy-name }} можно с помощью утилит `mysqldump` и `mysql`: создайте дамп рабочей базы и восстановите его в нужном кластере.

Перед тем, как переносить данные, проверьте, совпадают ли версии СУБД у существующей базы данных и вашего кластера в Облаке. Так как дамп логический и представляет собой набор SQL-запросов, то перенос данных между кластерами с различными версиями возможен, но не гарантируется. Более подробно об этом см. в [MySQL 5.7 FAQ Migration](https://dev.mysql.com/doc/refman/5.7/en/faqs-migration.html).

Ниже сервер СУБД, с которого вы переносите данные, называется _сервер-источник_, а кластер {{ mmy-name }}, на который вы мигрируете — _сервер-приемник_.

Этапы миграции:
1. [Создайте дамп](#dump) переносимой базы.
2. При необходимости [создайте виртуальную машину](#create-vm) в Яндекс.Облаке и загрузите данные на нее.
3. [Создайте кластер {{ mmy-name }}](#create-cluster).
4. [Восстановите данные из дампа](#restore).


## Создание дампа {#dump}

Создать дамп базы данных следует с помощью утилиты `mysqldump`, которая подробно описана в [документации {{ MY }}](https://dev.mysql.com/doc/refman/5.7/en/mysqlpump.html).

1. Перед созданием дампа рекомендуется переключить базу в режим <q>только чтение</q>, чтобы не потерять данные, которые бы появились во время создания дампа. Сам дамп базы данных создайте следующей командой:

    ```bash
    $ mysqldump -h <адрес сервера-источника> \
                --user=<имя пользователя> \
                --password \
                --port=<порт> \
                --set-gtid-purged=OFF \
                --quick
                --single-transaction <имя базы данных> \
                > ~/db_dump.sql
    ```

   Если сервер-источник использует таблицы InnoDB, используйте опцию `--single-transaction` для гарантированной консистентности данных. Для таблиц MyISAM эта опция не имеет смысла, так как транзакции не поддерживаются. Также стоит иметь в виду следующие флаги:

   * `--events` — если в вашей базе есть периодические события;
   * `--routines` — если в вашей базе есть функции и хранимые процедуры.

1. Упакуйте дамп в архив:

    ```
    $ tar -cvzf db_dump.tar.gz ~/db_dump.sql
    ```


## (опционально) Создание виртуальной машины в Облаке и загрузка дампа {#create-vm}

Переносить данные на промежуточную виртуальную машину в {{ compute-full-name }} нужно, если:

* К вашему кластеру {{ mmy-name }} нет доступа из интернета.
* Ваше оборудование или соединение с кластером в Облаке недостаточно надежны.

Нужное количество оперативной памяти и ядер процессора зависит от объема переносимых данных и требуемой скорости переноса.

Чтобы подготовить виртуальную машину для восстановления дампа:

1. В консоли управление создайте новую виртуальную машину из образа Ubuntu 18.04. Параметры виртуальной машины должны зависеть от размера базы, которую Вы хотите перенести. Минимальной конфигурации (1 ядро, 2 ГБ RAM, 10 ГБ дискового пространства) должно хватить для переносы базы до 1 ГБ. Чем больше переносимая база, тем больше должно быть дискового пространства (как минимум в два раза больше, чем размер базы), и больше размер оперативной памяти.

    Виртуальная машина должна находиться в той же сети и зоне доступности, что хост-мастер кластера {{ MY }}. Кроме того, виртуальной машине должен быть присвоен внешний IP-адрес, чтобы вы могли загрузить дамп извне Облака.

2. Установите клиент {{ MY }} и дополнительные утилиты для работы с СУБД. Для Debian/Ubuntu утилиты `mysqldump` и `mysql` поставляются в пакете [`mysql-client`](https://packages.ubuntu.com/search?keywords=mysql-client), его установка:

   ```
   $ sudo apt-get install mysql-client
   ```

3. Перенесите дамп базы данных на виртуальную машину, например, используя утилиту `scp`:

    ```
    scp ~/db_dump.tar.gz <имя пользователя ВМ>@<публичный адрес ВМ>:/tmp/db_dump.tar.gz
    ```

4. Распакуйте дамп:

    ```
    tar -xzf /tmp/db_dump.tar.gz
    ```


## Создание кластера {{ mmy-name }} {#create-cluster}

Инструкцию по созданию кластера можно найти в разделе [{#T}](./cluster-create.md).


## Восстановление данных {#restore}

Восстанавливать дамп базы данных следует с помощью утилиты [mysql](https://dev.mysql.com/doc/refman/5.7/en/mysql.html). Чтобы получать больше информации в случае возникновения ошибок, восстановление рекомендуется проводить установив флаг `--line-numbers`.

* Если вы восстанавливаете дамп с виртуальной машины в Яндекс.Облаке:

    ```
    $ mysql -h <адрес сервера СУБД> \
            --user=<имя пользователя>
            --password \
            --port=3306 \
            --line-numbers <имя базы данных> \
            < /tmp/db_dump.sql
    ```

* Если вы восстанавливаете дамп с собственного сервера, необходимо скачать сертификат и задать параметры SSL `--ssl-cal` и `--ssl-mode`. Команды для подключения к кластеру и ссылку на сертификат можно найти в консоли управления, по кнопке **Подключиться**.

  Команда восстановления базы из дампа:

   ```
   $ mysql -h <адрес сервера СУБД> --user=<имя пользователя> --password --port=3306 --ssl-ca=~/.mysql/root.crt --ssl-mode=VERIFY_IDENTITY --line-numbers <имя базы данных>  < ~/db_dump.sql
   ```