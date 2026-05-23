# Домашнее задание к занятию 3 «Использование Ansible»
## Ansible Playbook: ClickHouse + Vector + Lighthouse

## Описание

Данный playbook автоматизирует установку и настройку трёх компонентов для сбора, хранения и визуализации логов на Debian 12:

- **ClickHouse** – колоночная СУБД для хранения больших объёмов данных.
- **Vector** – высокопроизводительный сборщик и маршрутизатор логов.
- **Lighthouse** – лёгкий веб-интерфейс для просмотра данных в ClickHouse.

Playbook предназначен для использования на трёх отдельных хостах (по одному на каждый компонент).

## Требования

- Ansible >= 2.12
- Целевые хосты с ОС Debian 12
- Доступ по SSH с использованием ключа (пользователь `debian`)
- Наличие файла инвентаря с группами `clickhouse`, `vector`, `lighthouse`

## Переменные

Переменные задаются в файле `vars.yml`:

| Имя | Описание | Значение по умолчанию |
| --- | -------- | --------------------- |
| `clickhouse_version` | Версия ClickHouse | `22.8.5.29` |
| `clickhouse_packages` | Список пакетов ClickHouse | `[clickhouse-client, clickhouse-server, clickhouse-common-static]` |
| `vector_version` | Версия Vector | `0.44.0` |
| `vector_config_path` | Путь конфига Vector | `/etc/vector/vector.yaml` |
| `clickhouse_host` | Адрес сервера ClickHouse | (обязательно задать) |
| `lighthouse_version` | Ветка репозитория Lighthouse | `master` |
| `lighthouse_install_dir` | Путь установки Lighthouse | `/var/www/lighthouse` |
| `nginx_user` | Пользователь Nginx | `www-data` |

## Использование

Запуск playbook целиком:

```bash
ansible-playbook -i inventory/prod.yml site.yml
```

## Теги

Playbook содержит набор тегов для выборочного выполнения задач.

| Тег | Play | Описание |
| --- | ---- | -------- |
| `clickhouse` | Install Clickhouse | Все задачи, связанные с ClickHouse |
| `db` | Install Clickhouse | Создание базы данных `logs` |
| `table` | Install Clickhouse | Создание таблицы `vector_logs` |
| `distr` | Install Clickhouse, Install Vector | Скачивание дистрибутивов |
| `start service` | Install Clickhouse | Перезапуск сервиса ClickHouse |
| `wait` | Install Clickhouse | Ожидание готовности ClickHouse |
| `vector` | Install Vector | Все задачи Vector |
| `config` | Install Vector | Деплой конфигурации Vector |
| `restart service` | Install Vector | Перезапуск Vector |
| `Lighthouse` | Install Lighthouse | Все задачи Lighthouse |
| `config nginx` | Install Lighthouse | Деплой конфигурации Nginx |

Примеры использования тегов:

```bash
# Только создание таблицы в ClickHouse
ansible-playbook -i inventory/prod.yml site.yml --tags "table"

# Только обновление конфигурации Vector и перезапуск
ansible-playbook -i inventory/prod.yml site.yml --tags "config,restart service"

# Установка всего, кроме Lighthouse
ansible-playbook -i inventory/prod.yml site.yml --skip-tags "Lighthouse"
```

## Устранение неполадок
- Если Vector не может писать в ClickHouse – проверьте, что в таблице logs.vector_logs есть все поля, которые отправляет Vector (см. задачу Create table в плейбуке).
- Если Lighthouse показывает ошибку подключения – убедитесь, что параметр clickhouse_host в vars.yml указывает на правильный IP-адрес сервера ClickHouse.
- Для повторного запуска playbook без ошибок Git используется параметр update: no в задаче клонирования Lighthouse.
