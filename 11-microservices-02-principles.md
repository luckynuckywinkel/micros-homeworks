# Домашнее задание: Микросервисы и DevOps

## Задача 1: **API Gateway**

### Сравнительная таблица API Gateway

| **Параметры**          | **NGINX**                 | **Kong**                    | **Traefik**                 | **AWS API Gateway**          |
|------------------------|---------------------------|-----------------------------|-----------------------------|------------------------------|
| **Маршрутизация**       | Статическая/динамическая, поддержка L7 | Статическая/динамическая    | Динамическая на основе сервисов | Статическая/динамическая    |
| **Аутентификация**      | Через внешние модули (например, OpenID Connect) | Встроенная поддержка OAuth2, JWT | Встроенная поддержка OAuth2, JWT | Встроенная поддержка AWS IAM |
| **HTTPS терминация**    | Да                        | Да                          | Да                          | Да                           |
| **Мониторинг**          | Интеграция с Prometheus, Grafana | Встроенный, Prometheus      | Встроенный, Prometheus      | CloudWatch                  |
| **Простота настройки**  | Высокая гибкость, ручная настройка конфигов | UI + REST API               | REST API, интеграция с Docker | AWS Console, Terraform      |

### Выбор: **NGINX**  
**Обоснование:**  
- Хорошо подходит для локального использования и простого развёртывания.  
- Легко расширяется под задачи маршрутизации и проверки токенов через модули.  
- Обладает высокой производительностью и гибкостью для разработки.

---

## Задача 2: **Брокер сообщений**

### Сравнительная таблица брокеров сообщений

| **Параметры**           | **RabbitMQ**              | **Kafka**                   | **NATS**                    | **Redis Streams**            |
|-------------------------|---------------------------|-----------------------------|-----------------------------|------------------------------|
| **Кластеризация**        | Да                        | Да                          | Да                          | Да                           |
| **Хранение сообщений**   | Да                        | Да                          | Нет                         | Да                           |
| **Скорость**             | Высокая                   | Очень высокая               | Очень высокая               | Высокая                      |
| **Поддержка форматов**   | JSON, XML, Protobuf       | Protobuf, Avro, JSON        | JSON, Protobuf              | JSON                        |
| **Разделение прав**      | Да (ACL)                  | Да (ACL)                    | Ограничено                  | Ограничено                   |
| **Простота эксплуатации**| Средняя                   | Высокая сложность           | Простая                     | Простая                      |

### Выбор: **RabbitMQ**  
**Обоснование:**  
- Подходит для большинства задач с микросервисной архитектурой.  
- Хорошо поддерживает разные форматы сообщений и имеет простой интерфейс для управления.  
- Надёжное хранение сообщений на диске.

---

## Задача 3: **API Gateway с конфигурацией NGINX**

### Docker Compose файл

```yaml
version: '3.9'
services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro

  minio:
    image: minio/minio
    environment:
      MINIO_ACCESS_KEY: "minioadmin"
      MINIO_SECRET_KEY: "minioadmin"
    command: server /data
    ports:
      - "9000:9000"

  uploader:
    image: your_uploader_image
    ports:
      - "8081:8081"

  security:
    image: your_security_image
    ports:
      - "8082:8082"
```

### Конфигурация NGINX  

```
events {}

http {
    upstream uploader {
        server uploader:8081;
    }

    upstream security {
        server security:8082;
    }

    upstream minio {
        server minio:9000;
    }

    server {
        listen 80;

        location /v1/register {
            proxy_pass http://security/v1/user;
        }

        location /v1/token {
            proxy_pass http://security/v1/token;
        }

        location /v1/user {
            proxy_set_header Authorization $http_authorization;
            proxy_pass http://security/v1/user;
        }

        location /v1/upload {
            proxy_set_header Authorization $http_authorization;
            proxy_pass http://uploader/v1/upload;
        }

        location /images/ {
            proxy_set_header Authorization $http_authorization;
            proxy_pass http://minio/images/;
        }
    }
}
```

## Команды для проверки  

### Авторизация:  

```
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token
```

### Загрузка файла:  

```
curl -X POST -H 'Authorization: Bearer <your_token>' -H 'Content-Type: octet/stream' --data-binary @yourfile.jpg http://localhost/upload
```

### Получение файла:  

```
curl -X GET http://localhost/images/<image_id>
```

---



