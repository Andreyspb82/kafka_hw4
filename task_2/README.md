# Kafka Самостоятельная работа №4
## Задание 2. Настройка защищенного соединения и управление доступом

### 1) Запуск проекта и управление доступом:
####  1.1) В дирректории `/kafka_hw4/task_2` запускаем проект командой:
```
docker compose up -d
```
####  1.2) Заходим в терминал брокера `kafka-1` с помощью команды:
```
docker exec -it kafka-1 bash
```
####  1.3) Пробуем создать `topic-1` с правами `consumer`,  с помощью команды:
```
kafka-topics --bootstrap-server kafka-1:9093 --topic topic-1 --create --partitions 3 --replication-factor 3 --command-config /tmp/configs/consumer-configs.conf
```
#####  Возвращается ответ:
```
Error while executing topic command : Authorization failed.
[2025-04-13 10:10:12,973] ERROR org.apache.kafka.common.errors.TopicAuthorizationException: Authorization failed.
 (kafka.admin.TopicCommand$)
```
####  1.4) Пробуем создать `topic-1` с правами `producer`,  с помощью команды:
```
kafka-topics --bootstrap-server kafka-1:9093 --topic topic-1 --create --partitions 3 --replication-factor 3 --command-config /tmp/configs/producer-configs.conf
```
#####  Возвращается ответ:
```
Error while executing topic command : Authorization failed.
[2025-04-13 10:11:16,505] ERROR org.apache.kafka.common.errors.TopicAuthorizationException: Authorization failed.
 (kafka.admin.TopicCommand$)
```
####  1.5) Пробуем создать `topic-1` с правами `admin`, с помощью команды:
```
kafka-topics --bootstrap-server kafka-1:9093 --topic topic-1 --create --partitions 3 --replication-factor 3 --command-config /tmp/configs/admin-configs.conf
```
#####  Возвращается ответ:
```
Created topic topic-1.
```
####  1.6) Создаем `topic-2` с правами `admin`,  с помощью команды:
```
kafka-topics --bootstrap-server kafka-1:9093 --topic topic-2 --create --partitions 3 --replication-factor 3 --command-config /tmp/configs/admin-configs.conf
```
#####  Возвращается ответ:
```
Created topic topic-2.
```
####  1.7) Открываем новое окно консоли и создаем продюсера для `topic-1`, с помощью команды:
```
kafka-console-producer --bootstrap-server kafka-1:9093 --topic topic-1 --producer.config /tmp/configs/producer-configs.conf
```
#####  Отправляем сообщение, в консоли получаем сообщение вида:
```
[2025-04-13 10:24:38,958] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 4 : {topic-1=TOPIC_AUTHORIZATION_FAILED} (org.apache.kafka.clients.NetworkClient)
[2025-04-13 10:24:38,962] ERROR [Producer clientId=console-producer] Topic authorization failed for topics [topic-1] (org.apache.kafka.clients.Metadata)
[2025-04-13 10:24:38,964] ERROR Error when sending message to topic topic-1 with key: null, value: 4 bytes with error: (org.apache.kafka.clients.producer.internals.ErrorLoggingCallback)
org.apache.kafka.common.errors.TopicAuthorizationException: Not authorized to access topics: [topic-1]
```
####  1.8) Открываем новое окно консоли и создаем консьюмера для `topic-1`, с помощью команды:
```
kafka-console-consumer --bootstrap-server kafka-1:9093 --topic topic-1 --consumer.config /tmp/configs/consumer-configs.conf --from-beginning
```
#####  Возвращается ответ:
```
[2025-04-13 10:30:34,525] WARN [Consumer clientId=console-consumer, groupId=group-1] Error while fetching metadata with correlation id 2 : {topic-1=TOPIC_AUTHORIZATION_FAILED} (org.apache.kafka.clients.NetworkClient)
[2025-04-13 10:30:34,526] ERROR [Consumer clientId=console-consumer, groupId=group-1] Topic authorization failed for topics [topic-1] (org.apache.kafka.clients.Metadata)
[2025-04-13 10:30:34,528] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TopicAuthorizationException: Not authorized to access topics: [topic-1]
```
####  1.9) Разрешаем продюссеру записывать сообщения в `topic-1` и в `topic-2` с помощью команд:
```
kafka-acls --authorizer-properties zookeeper.connect=zookeeper:2181 --add --allow-principal User:producer --operation Write --topic topic-1
```
```
kafka-acls --authorizer-properties zookeeper.connect=zookeeper:2181 --add --allow-principal User:producer --operation Write --topic topic-2
```
####  1.10) Открываем новое окно консоли и создаем продюссера для `topic-1`, с помощью команды:
```
kafka-console-producer --bootstrap-server kafka-1:9093 --topic topic-1 --producer.config /tmp/configs/producer-configs.conf
```
#####  Отправляем сообщение, сообщение уходит.
####  1.11) Открываем новое окно консоли и создаем продюссера для `topic-2`, с помощью команды:
```
kafka-console-producer --bootstrap-server kafka-1:9093 --topic topic-2 --producer.config /tmp/configs/producer-configs.conf
```
#####  Отправляем сообщение, сообщение уходит.
####  1.12) Разрешаем консьюмеру читать сообщения из `topic-1`, с помощью команды:
```
kafka-acls --authorizer-properties zookeeper.connect=zookeeper:2181 --add --allow-principal User:consumer --group group-1 --operation Read --topic topic-1
```
####  1.13) Открываем новое окно консоли и создаем консюмера для `topic-1`, с помощью команды:
```
kafka-console-consumer --bootstrap-server kafka-1:9093 --topic topic-1 --consumer.config /tmp/configs/consumer-configs.conf --from-beginning
```
#####  Получаем сообщение отправленное в `topic-1`.
####  1.14) Открываем новое окно консоли и создаем консюмера для `topic-2`, с помощью команды:
```
kafka-console-consumer --bootstrap-server kafka-1:9093 --topic topic-2 --consumer.config /tmp/configs/consumer-configs.conf --from-beginning
```
#####  Возвращается ответ:
```
[2025-04-13 10:48:05,118] WARN [Consumer clientId=console-consumer, groupId=group-1] Error while fetching metadata with correlation id 2 : {topic-2=TOPIC_AUTHORIZATION_FAILED} (org.apache.kafka.clients.NetworkClient)
[2025-04-13 10:48:05,119] ERROR [Consumer clientId=console-consumer, groupId=group-1] Topic authorization failed for topics [topic-2] (org.apache.kafka.clients.Metadata)
[2025-04-13 10:48:05,121] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TopicAuthorizationException: Not authorized to access topics: [topic-2]
Processed a total of 0 messages
```
