# Домашнее задание к занятию «Очереди RabbitMQ
### Задание 1. Установка RabbitMQ

Используя Vagrant или VirtualBox, создайте виртуальную машину и установите RabbitMQ.
Добавьте management plug-in и зайдите в веб-интерфейс.

*Итогом выполнения домашнего задания будет приложенный скриншот веб-интерфейса RabbitMQ.*

![Interface](https://github.com/OhotinDY/sdb-04/blob/main/interface.jpg)

---

### Задание 2. Отправка и получение сообщений

Используя приложенные скрипты, проведите тестовую отправку и получение сообщения.
Для отправки сообщений необходимо запустить скрипт producer.py.

Для работы скриптов вам необходимо установить Python версии 3 и библиотеку Pika.
Также в скриптах нужно указать IP-адрес машины, на которой запущен RabbitMQ, заменив localhost на нужный IP.

```shell script
$ pip install pika
```

Зайдите в веб-интерфейс, найдите очередь под названием hello и сделайте скриншот.
После чего запустите второй скрипт consumer.py и сделайте скриншот результата выполнения скрипта

*В качестве решения домашнего задания приложите оба скриншота, сделанных на этапе выполнения.*

Для закрепления материала можете попробовать модифицировать скрипты, чтобы поменять название очереди и отправляемое сообщение.

Очередь в веб-интерфейсе после выполнения скрипта producer.py:

![producer](https://github.com/OhotinDY/sdb-04/blob/main/queue1.jpg)

Очередь в веб-интерфейсе после выполнения скрипта consumer.py:

![consumer](https://github.com/OhotinDY/sdb-04/blob/main/consumer.jpg)
![consumer](https://github.com/OhotinDY/sdb-04/blob/main/consumer1.jpg)

Консоль:

![consol](https://github.com/OhotinDY/sdb-04/blob/main/cmdreceive.jpg)

---

### Задание 3. Подготовка HA кластера

Используя Vagrant или VirtualBox, создайте вторую виртуальную машину и установите RabbitMQ.
Добавьте в файл hosts название и IP-адрес каждой машины, чтобы машины могли видеть друг друга по имени.

Пример содержимого hosts файла:
```shell script
$ cat /etc/hosts
192.168.0.10 rmq01
192.168.0.11 rmq02
```
После этого ваши машины могут пинговаться по имени.

Затем объедините две машины в кластер и создайте политику ha-all на все очереди.

*В качестве решения домашнего задания приложите скриншоты из веб-интерфейса с информацией о доступных нодах в кластере и включённой политикой.*

Также приложите вывод команды с двух нод:

```shell script
$ rabbitmqctl cluster_status
```

Для закрепления материала снова запустите скрипт producer.py и приложите скриншот выполнения команды на каждой из нод:

```shell script
$ rabbitmqadmin get queue='hello'
```

После чего попробуйте отключить одну из нод, желательно ту, к которой подключались из скрипта, затем поправьте параметры подключения в скрипте consumer.py на вторую ноду и запустите его.

*Приложите скриншот результата работы второго скрипта.*

---

Прописываем соответствия ip-адреса доменному имени:

```shell script
$ cat /etc/hosts
echo "192.168.1.105 rabbit1" >> /etc/hosts
echo "192.168.1.88 rabbit2" >> /etc/hosts
```

Для работы кластера RabbitMQ все узлы, участвующие в кластере, должны иметь одинаковые файлы cookie.
Скопируем содержимое файла Cookie с первой (main) машины (ноды) на вторую, которую будем добавлять в кластер. Файл Cookie находится здесь: /var/lib/rabbitmq/.erlang.cookie.

На второй ноде (rabbit2) перезапустим службу:

```shell script
systemctl restart rabbitmq-server
```

Остановим и сбросим приложение:

```shell script
rabbitmqctl stop_app
rabbitmqctl reset
```

Подключим к кластеру и запустим:

```shell script
rabbitmqctl join_cluster rabbit@rabbit1
rabbitmqctl start_app
```

На ноде1 (rabbit1) cоздаем политику, которая позволяет зеркалить очереди для всех узлов в кластере:

```shell script
rabbitmqctl set_policy ha-all ".*" '{"ha-mode":"all"}'  
```

![rabbit1](https://github.com/OhotinDY/sdb-04/blob/main/cluster.jpg)
![rabbit1](https://github.com/OhotinDY/sdb-04/blob/main/policies.jpg)

![status1](https://github.com/OhotinDY/sdb-04/blob/main/cl_status1.jpg)
![status2](https://github.com/OhotinDY/sdb-04/blob/main/cl_status2.jpg)

Устанавливаем и запускаем утилиту rabbitmqadmin:

![rabbitadmin](https://github.com/OhotinDY/sdb-04/blob/main/rma2.jpg)
![rabbitadmin](https://github.com/OhotinDY/sdb-04/blob/main/rma1.jpg)

![notrun](https://github.com/OhotinDY/sdb-04/blob/main/not_run.jpg)

Выполнение скрипта consumer.py на погашенной ноде1 - rabbit1:

![notrun](https://github.com/OhotinDY/sdb-04/blob/main/last_cinsumer.jpg)

