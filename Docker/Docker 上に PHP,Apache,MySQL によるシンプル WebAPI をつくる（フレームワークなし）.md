# Docker 上に PHP,Apache,MySQL によるシンプル WebAPI をつくる（フレームワークなし）

## 完成イメージ

## `Docker-Compose.yml`

```yml
version: "3.9"

services:
  app:
    container_name: docker-practice-app
    build:
      dockerfile: app/Dockerfile
      context: .
    ports:
      - "8080:80"
    volumes:
      - type: bind
        source: ./app/src
        target: /var/www/html/src
    # command: php -S 0.0.0.0:8000 -t /src

  db:
    container_name: docker-practice-db
    build:
      dockerfile: db/Dockerfile
      context: .
    volumes:
      - type: volume
        source: mysql-db-volume
        target: /var/lib/mysql
      - type: bind
        source: ./db/world.sql
        target: /docker-entrypoint-initdb.d/world.sql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_USER: hoge
      MYSQL_PASSWORD: hoge
      MYSQL_DATABASE: testdb

volumes:
  mysql-db-volume:
```

## アプリケーション側（素の PHP と Apache）

## DB 側（MySQL）

## フロント 側（素の HTML と JavaScript）
