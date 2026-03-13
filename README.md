# Zork — Умное видеонаблюдение с AI‑уведомлениями

**Zork** — это интеллектуальная система видеонаблюдения, объединяющая [Frigate](https://frigate.video/) (детекция объектов на основе AI) и сервис уведомлений на [NestJS](https://nestjs.com/). 

Для связи компонентов используется MQTT. Система мгновенно отправляет оповещения в Telegram при обнаружении людей или автомобильных номеров, а в будущем может легко интегрироваться с умным домом.

---

## ✨ Возможности

* **Детекция на основе AI** — Frigate распознаёт людей, автомобили и номерные знаки (LPR).
* **Мгновенные уведомления** — получайте сообщения в Telegram при обнаружении человека или конкретного авто.
* **MQTT как основа** — все события публикуются в MQTT, что упрощает интеграцию с Home Assistant и другими системами.
* **Модульность и расширяемость** — NestJS‑сервис легко доработать для новых каналов уведомлений или бизнес‑логики.
* **Оптимизированная трансляция видео** — используется go2rtc (встроенный в Frigate) для низкой задержки и высокого качества.
* **Готовность к Docker** — простой запуск всего стека через Docker Compose.

---

## 🏗️ Архитектура

Проект состоит из трёх основных компонентов:

1.  **Frigate**
    * Выполняет детекцию объектов и распознавание номеров (LPR) с помощью моделей глубокого обучения.
    * Публикует события (человек, автомобиль, номер) в MQTT‑брокер.
    * Опционально отдаёт RTSP‑потоки через встроенный go2rtc.
2.  **MQTT‑брокер** (например, Mosquitto)
    * Служит шиной сообщений между Frigate и сервисом уведомлений.
    * Топики: `frigate/events` (события) и `frigate/+/+/snapshot` (снимки).
3.  **Сервис уведомлений (NestJS)**
    * Подписывается на MQTT‑топики, фильтрует нужные события и отправляет сообщения в Telegram.
    * Может быть расширен для отправки email или команд умному дому.

### Поток данных:
`[Камеры] → [Frigate] → [MQTT] → [NestJS Service] → [Telegram]`  
`                          ↓`  
`            [Home Assistant / Умный дом]`

---

## 🛠️ Технологический стек

* **Frigate:** Python, OpenCV, TensorFlow Lite, Go2RTC.
* **NestJS:** Node.js, TypeScript, MQTT.js.
* **MQTT‑брокер:** Mosquitto (или совместимый).
* **База данных:** SQLite/PostgreSQL (Frigate); сервис уведомлений опционально поддерживает любую БД.
* **Контейнеризация:** Docker, Docker Compose.

---

## 📋 Предварительные требования

* Установленные Docker и Docker Compose.
* Работающий MQTT‑брокер (можно запустить в контейнере).
* IP‑камеры или USB‑камеры, совместимые с Frigate.
* Токен Telegram‑бота (получить у [@BotFather](https://t.me/botfather)).

---

## 🚀 Быстрый старт

### 1. Клонируйте репозиторий
```bash
git clone --recursive [https://github.com/yourusername/zork.git](https://github.com/yourusername/zork.git)
cd zork
```

### 2. Настройте переменные окружения
Создайте файл .env в корне проекта:
```.env
# Telegram
TELEGRAM_BOT_TOKEN=токен_вашего_бота
TELEGRAM_CHAT_ID=id_чата_для_уведомлений

# MQTT
MQTT_BROKER_URL=mqtt://mosquitto:1883
MQTT_USERNAME=ваш_пользователь
MQTT_PASSWORD=ваш_пароль

# Frigate
FRIGATE_MQTT_TOPIC_PREFIX=frigate
```
### 3. Запустите стек
```bash
docker-compose up -d
```

###4. Настройте Frigate
Отредактируйте config/frigate/config.yml. Пример минимальной конфигурации с LPR:

```yaml
mqtt:
  enabled: true
  host: mosquitto
  port: 1883
  user: ваш_пользователь
  password: ваш_пароль

cameras:
  driveway:
    ffmpeg:
      inputs:
        - path: rtsp://user:pass@ip:554/stream1
          roles:
            - detect
            - record
    detect:
      enabled: true
      width: 1280
      height: 720
      fps: 5
    objects:
      track:
        - person
        - car
        - license_plate
    lpr:
      enabled: true
      enhancement: 3
    record:
      enabled: true
      retain:
        days: 3
    mqtt:
      enabled: true
      timestamp: true
      bounding_box: true
      crop: true
      quality: 80
```

## ⚙️ Параметры конфигурации

### Сервис уведомлений (NestJS)

| Переменная | Описание | По умолчанию |
| :--- | :--- | :--- |
| `TELEGRAM_BOT_TOKEN` | Токен бота | — |
| `TELEGRAM_CHAT_ID` | ID чата для оповещений | — |
| `MQTT_BROKER_URL` | URL MQTT‑брокера | `mqtt://localhost:1883` |
| `MQTT_TOPIC_EVENTS` | Топик событий Frigate | `frigate/events` |
| `NOTIFICATION_FILTER` | Список объектов для уведомлений | `person,car,license_plate` |

## 🔮 Планы на будущее

- [ ] Интеграция с Home Assistant через MQTT auto‑discovery.
- [ ] Поддержка нескольких каналов уведомлений (email, push).
- [ ] Веб‑панель для просмотра последних событий.
- [ ] Распознавание лиц.

## 📄 Лицензия

MIT License.

---
*Zork — наблюдает за вашим миром, кадр за кадром.*
