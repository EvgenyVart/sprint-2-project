# Итоговое приложение сприт 2

## Запуск прилоежения

Запускаем mongodb в режиме шардирования 2 шарда и приложение в одном инстансе

```shell
docker compose up -d
```

### Далее необходимо произвести инициализацию:

- Сервер конфигурации MongoDB:

Подключение к контейнеру
```shell
docker exec -it configSrv mongosh --port 27017
```
Выполнение команды инициализации
```
rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);
```
Выход из консоли контейнера
```
exit();
```

- Инициализируйте шарды:

Подключение к контейнеру шард-1
```shell
docker exec -it shard1 mongosh --port 27018
```
Выполнение команды инициализации
```
rs.initiate(
    {
      _id : "shard1",
      members: [
        { _id : 0, host : "shard1:27018" }
      ]
    }
);
```
Выход из консоли контейнера
```
exit();
```

Подключение к контейнеру шард-2
```shell
docker exec -it shard2 mongosh --port 27019
```
Выполнение команды инициализации
```
rs.initiate(
    {
      _id : "shard2",
      members: [
        { _id : 1, host : "shard2:27019" }
      ]
    }
);
```
Выход из консоли контейнера
```
exit();
```

- Инцициализируем роутер:

Подключение к контейнеру
```shell
docker exec -it mongos_router mongosh --port 27020
```
Выполнение команды инициализации
```
sh.addShard( "shard1/shard1:27018");
```
```
sh.addShard( "shard2/shard2:27019");
```
```
sh.enableSharding("somedb");
```
```
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )
```

Выход из консоли контейнера
```
exit();
```

Заполняем mongodb данными для linux систем

```shell
./scripts/mongo-init.sh
```

Заполняем mongodb данными для windows систем

Подключение к контейнеру
```shell
docker exec -it mongos_router mongosh --port 27020
```

Переключаем текущую схему:

```
use somedb
```

Выполняем скрип генерации данных, за один раз добавляется 1000 записей:

```
for(var i = 0; i < 1000; i++) db.helloDoc.insertOne({age:i, name:"ly"+i})
```


## Проверка

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080

### Если вы запускаете проект на предоставленной виртуальной машине

Узнать белый ip виртуальной машины

```shell
curl --silent http://ifconfig.me
```

Откройте в браузере http://<ip виртуальной машины>:8080

## Доступные эндпоинты

Список доступных эндпоинтов, swagger http://<ip виртуальной машины>:8080/docs