## Выполнение домашнего задания

#### Запуск Mongo в Docker

docker
```
docker pull mongo
docker run -it --rm --name mongodb -d mongo
docker exec -it mongodb mongo
```

docker-compose
```
cd mongo
docker-compose up
```

#### Загрузка данных в Mongo из csv

Разные датасеты данных
https://habr.com/ru/company/edison/blog/480408/

Попробуем загрузить в Mongo данные Mall Customers Dataset через csv
https://www.kaggle.com/datasets/shwetabh123/mall-customers

Перекладываем файл в папку data и монтируем внутрь контейнера с Mongo (добавлено в docker-compose)

Затем нужно войти в контейнер и использовать `mongoimport`

Создание базы и коллекции

```
show dbs

use test
db.createCollection("customers")

show dbs
db.getCollectionNames()
```

Загрузка данных

```
docker exec -it mongodb bash

mongoimport -u "root" -p "example" --authenticationDatabase "admin" --db test --collection customers --type csv --headerline --ignoreBlanks --file /home/root/mall_customers.csv

2022-10-30T16:17:13.069+0000	connected to: mongodb://localhost/
2022-10-30T16:17:13.101+0000	200 document(s) imported successfully. 0 document(s) failed to import.

```

Проверка данных

```
db.getSiblingDB("test").getCollection("customers").countDocuments()
200
```


#### Запросы на выборку и обновление данных

Посмотр данных

```
-- 5 записей
db.getSiblingDB("test").getCollection("customers").find({}).limit(5)

-- 5 записей с сортировкой по полю Genre
db.getSiblingDB("test").getCollection("customers").find({}).limit(5).sort({Genre: 1})

-- Все записи с возрастом от 20 и сортировкой по возрасту
db.getSiblingDB("test").getCollection("customers").find({Age: {$gt: 20}}).sort({Age: 1})

-- И количество таких записей
db.getSiblingDB("test").getCollection("customers").find({Age: {$gt: 20}}).count()
```

Обновление данных

```
-- Обновление возраста у конкретной записи
db.getSiblingDB("test").getCollection("customers").updateOne({_id: new ObjectId("635ea5e1e7fed62beb092775")}, {"$set": {Age: new NumberInt("40")}})

-- Добавить всем 10 лет
db.getSiblingDB("test").getCollection("customers").updateMany({}, {$inc: {Age:10}})

-- Русификация мужского пола
db.getSiblingDB("test").getCollection("customers").updateMany({Genre : "Male"}, {$set: {Genre : "Мужской"}})
```

