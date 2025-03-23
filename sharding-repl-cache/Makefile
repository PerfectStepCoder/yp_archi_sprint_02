.PHONY: help
COMPOSE_FILE = compose.yaml
REPLICA_FILE = compose.replicas.yaml
CASH_FILE = compose.cash.yaml

## start -> Запуск всех сервисов
start:
	docker network create mongo-cluster || true
	docker-compose -f $(COMPOSE_FILE) up -d
	docker-compose -f $(REPLICA_FILE) up -d
	docker-compose -f $(CASH_FILE) up -d

## stop -> Остановка всех сервисов
stop:
	docker-compose -f $(COMPOSE_FILE) down
	docker-compose -f $(REPLICA_FILE) down
	docker-compose -f $(CASH_FILE) down

## status -> Статус всех сервисов
status:
	docker-compose -f $(COMPOSE_FILE) -f $(REPLICA_FILE) -f $(CASH_FILE) ps

## init -> Инициализация
init:
	@echo "Инициализация mongodb-configs..."
	@docker exec -i mongodb-configs mongosh --port 27017 --eval 'rs.initiate({ _id: "config_server", configsvr: true, members: [{ _id: 0, host: "mongodb-configs:27017" }] });' || echo "Ошибка инициализации уже инициализирован"
	@sleep 10
	@echo "Инициализации репликации по шардам..."
	@docker-compose -f $(COMPOSE_FILE) -f $(REPLICA_FILE) exec -i shard1 mongosh --port 27018 --eval 'rs.initiate({ _id: "shard1_rs", members: [{ _id: 0, host: "shard1:27018", priority: 2 }, { _id: 1, host: "shard1-replica1:27031", priority: 1 }, { _id: 2, host: "shard1-replica2:27032", priority: 1 }]});' || echo "Ошибка при инициализации реплицировании"
	@sleep 10
	@docker-compose -f $(COMPOSE_FILE) -f $(REPLICA_FILE) exec -i shard2 mongosh --port 27019 --eval 'rs.initiate({ _id: "shard2_rs", members: [{ _id: 0, host: "shard2:27019", priority: 2 }, { _id: 1, host: "shard2-replica1:27041", priority: 1 }, { _id: 2, host: "shard2-replica2:27042", priority: 1 }]});' || echo "Ошибка при инициализации реплицировании"
	@sleep 10
	@echo "Настройка шардирования через mongos-router..."
	@docker-compose -f $(COMPOSE_FILE) exec -i mongos-router mongosh --port 27020 --eval 'sh.addShard("shard1_rs/shard1:27018"); sh.addShard("shard2_rs/shard2:27019");' || echo "Ошибка настройки или шарды уже добавлены"
	@sleep 10
	@echo "Создаем базу данных через mongos-router..."
	@docker-compose -f $(COMPOSE_FILE) exec -i mongos-router mongosh --port 27020 --eval 'sh.enableSharding("somedb"); sh.shardCollection("somedb.helloDoc", { "name": "hashed" });' || echo "Ошибка создания базы данных"
	@echo "Настройка роутера завершена."
	@echo "Инициализация завершена."

init_mongodb-configs:
	@echo "Инициализация mongodb-configs..."
	@docker exec -i mongodb-configs mongosh --port 27017 --eval 'rs.initiate({ _id: "config_server", configsvr: true, members: [{ _id: 0, host: "mongodb-configs:27017" }] });' || echo "Ошибка инициализации уже инициализирован"

init_replica_shard1:
	@echo "Инициализации репликации по шардам... shard1"
	@docker-compose -f $(COMPOSE_FILE) -f $(REPLICA_FILE) exec -i shard1 mongosh --port 27018 --eval 'rs.initiate({ _id: "shard1_rs", members: [{ _id: 0, host: "shard1:27018", priority: 2 }, { _id: 1, host: "shard1-replica1:27031", priority: 1 }, { _id: 2, host: "shard1-replica2:27032", priority: 1 }]});' || echo "Ошибка при инициализации реплицировании"
	@echo "Настройка shard1 завершена."

init_replica_shard2:
	@echo "Инициализации репликации по шардам... shard2"
	@docker-compose -f $(COMPOSE_FILE) -f $(REPLICA_FILE) exec -i shard2 mongosh --port 27019 --eval 'rs.initiate({ _id: "shard2_rs", members: [{ _id: 0, host: "shard2:27019", priority: 2 }, { _id: 1, host: "shard2-replica1:27041", priority: 1 }, { _id: 2, host: "shard2-replica2:27042", priority: 1 }]});' || echo "Ошибка при инициализации реплицировании"
	@echo "Настройка shard2 завершена."

init_mongos-router:
	@echo "Настройка шардирования через mongos-router..."
	@docker-compose -f $(COMPOSE_FILE) exec -i mongos-router mongosh --port 27020 --eval 'sh.addShard("shard1_rs/shard1:27018"); sh.addShard("shard2_rs/shard2:27019");' || echo "Ошибка настройки или шарды уже добавлены"
	@echo "Настройка роутера завершена."

init_db:
	@echo "Создаем базу данных через mongos-router..."
	@docker-compose -f $(COMPOSE_FILE) exec -i mongos-router mongosh --port 27020 --eval 'sh.enableSharding("somedb"); sh.shardCollection("somedb.helloDoc", { "name": "hashed" });' || echo "Ошибка создания базы данных"
	@echo "Инициализация БД."

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
	@echo "Статистика распределения документов по репликам..."
	@docker exec -i shard1-replica1 mongosh --port 27031 --eval 'db = db.getSiblingDB("somedb"); print("Количество документов на shard1-replica1: " + db.helloDoc.countDocuments());' || echo "Ошибка при проверке shard1-replica1"
	@docker exec -i shard1-replica2 mongosh --port 27032 --eval 'db = db.getSiblingDB("somedb"); print("Количество документов на shard1-replica2: " + db.helloDoc.countDocuments());' || echo "Ошибка при проверке shard1-replica2"
	@docker exec -i shard2-replica1 mongosh --port 27041 --eval 'db = db.getSiblingDB("somedb"); print("Количество документов на shard2-replica1: " + db.helloDoc.countDocuments());' || echo "Ошибка при проверке shard2-replica1"
	@docker exec -i shard2-replica2 mongosh --port 27042 --eval 'db = db.getSiblingDB("somedb"); print("Количество документов на shard2-replica2: " + db.helloDoc.countDocuments());' || echo "Ошибка при проверке shard2-replica2"

# Выводит список доступных команд и их описания
help:
	@echo "Доступные команды:"
	@sed -n 's/^##//p' $(MAKEFILE_LIST) | column -t -s ':' |  sed -e 's/^/ /'
