Kafka on windows.

Run this command in terminal:
$ docker-compose -f docker-compose.yml up -d

stop containers:
$ docker-compose down 

Run Kafka command lina:
$ docker exec -it kafka /bin/sh

$ cd opt/kafka_2.13-2.7.1 

./bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 100 --topic selftutorial

./bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 100 --topic test
./bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 100 --topic demo

./bin/kafka-topics.sh --list --zookeeper zookeeper:2181
Output:
demo
selftutorial
test

./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic channel --from-beginning














Запись и чтение сообщений
Создаем топик с регистрациями

./bin/kafka-topics.sh --create --topic registrations --bootstrap-server localhost:9092
Посмотрим на его конфигурацию

./bin/kafka-topics.sh --describe --topic registrations --bootstrap-server localhost:9092
Давайте запишем первое сообщение

./bin/kafka-console-producer.sh --topic registrations --bootstrap-server localhost:9092
>Hello world!
>Hello Slurm!
Попробуем его прочитать

./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092
И… ничего не происходит!

В эту ситуацию попадают многие люди, впервые использующие кафку. Все дело в том, что консьюмер Kafka по умолчанию начинает читать данные с конца топика (см. настройку auto.offset.reset). Для того чтобы прочитать данные с начала, мы должны переопределить эту конфигу.

./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest
Или можно сказать “--from-beginning”. Ура, мы произвели запись и чтение! Запишем еще одно сообщение — увидим, что оно появляется в консюмере. Обратим внимание на лог и увидим, что каждый запуск консольного консюмера создает новую группу.

Давайте вместо этого зададим свою:

./bin/kafka-console-consumer.sh --topic registrations --group slurm --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest
Видим сообщения, все ок, а теперь давайте перезапустим... Снова ничего! Вспомним из прошлой лекции, что консюмер группы в Кафке могут коммиттить свои оффсеты для топик-партиции, чтобы при перезапуске продолжать чтение с последней запомненной позиции. Именно это поведение мы и видим. Давайте проверим

./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurm --describe
А теперь сбросим позицию обратно на начало

./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurm --to-earliest --reset-offsets --execute --topic registrations
При новом запуске консюмера снова давайте отключим автоматический коммит оффсетов

./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092 --group slurm --consumer-property auto.offset.reset=earliest --consumer-property enable.auto.commit=false
Если запустим консюмера снова, то увидим, что сообщения читаются. Этим мы можем подтвердить, что оффсеты не коммитятся (+ оставим запущенным консюмера, чтобы увидеть идентификатор и адрес).
