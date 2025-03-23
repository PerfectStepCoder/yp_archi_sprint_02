.PHONY: help
COMPOSE_FILE = compose.yaml

## start -> Запуск всех сервисов
start:
	docker-compose -f $(COMPOSE_FILE) up -d

## stop -> Остановка всех сервисов
stop:
	docker-compose -f $(COMPOSE_FILE) down

## status -> Статус всех сервисов
status:
	docker-compose -f $(COMPOSE_FILE) ps

## init-mongodb-configs -> Инициализация сервера конфигурации
init-mongodb-configs:
	@echo "Инициализация Replica Set для mongodb-configs..."
	@docker exec -i mongodb-configs mongosh --port 27017 --eval 'rs.initiate({ _id: "config_server", configsvr: true, members: [{ _id: 0, host: "mongodb-configs:27017" }] });' || echo "Ошибка инициализации или Replica Set уже инициализирован"
	@echo "Инициализация завершена."

## init-mongodb-shards -> Инициализация шардов
init-mongodb-shards:
	@echo "Инициализация shard1..."
	@docker exec -i shard1 mongosh --port 27018 --eval 'rs.initiate({_id : "shard1", members: [{ _id : 0, host : "shard1:27018" }]})' || echo "Ошибка инициализации или shard1 уже инициализирован"
	@echo "Инициализация завершена."
	@echo "Инициализация shard2..."
	@docker exec -i shard2 mongosh --port 27019 --eval 'rs.initiate({_id : "shard2", members: [{ _id : 1, host : "shard2:27019" }]})' || echo "Ошибка инициализации или shard2 уже инициализирован"
	@echo "Инициализация завершена."

## init-mongodb-router -> Инициализация роутера
init-mongodb-router:
	@echo "Настройка шардирования через mongos-router..."
	@docker exec -i mongos-router mongosh --port 27020 --eval 'sh.addShard("shard1/shard1:27018"); sh.addShard("shard2/shard2:27019"); sh.enableSharding("somedb"); sh.shardCollection("somedb.helloDoc", { "name": "hashed" });' || echo "Ошибка настройки или шарды уже добавлены"
	@echo "Настройка роутера завершена."

## add-mongodb -> Добавление новых данных
add-mongodb:
	@echo "Добавление новых данных через mongos-router..."
	@docker exec -i mongos-router mongosh --port 27020 --eval 'db = db.getSiblingDB("somedb"); for (var i = 0; i < 1000; i++) db.helloDoc.insert({ age: i, name: "ly" + i }); print("Количество документов: " + db.helloDoc.countDocuments());' || echo "Ошибка при добавлении данных"
	@echo "Данные добавлены."

## stats-shards -> Статистика распределения документов по шардам
stats-shards:
	@echo "Статистика распределения документов по шардам..."
	@docker exec -i shard1 mongosh --port 27018 --eval 'db = db.getSiblingDB("somedb"); print("Количество документов на shard1: " + db.helloDoc.countDocuments());' || echo "Ошибка при проверке shard1"
	@docker exec -i shard2 mongosh --port 27019 --eval 'db = db.getSiblingDB("somedb"); print("Количество документов на shard2: " + db.helloDoc.countDocuments());' || echo "Ошибка при проверке shard2"

# Выводит список доступных команд и их описания
help:
	@echo "Доступные команды:"
	@sed -n 's/^##//p' $(MAKEFILE_LIST) | column -t -s ':' |  sed -e 's/^/ /'
