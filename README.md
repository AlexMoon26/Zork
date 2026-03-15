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
* **NestJS:** Node.js, TypeScript.
* **MQTT‑брокер:** Mosquitto.
* **ONVIF:** Библиотека onvif для управления PTZ-камерами.
* **База данных:** SQLite/PostgreSQL (Frigate).
* **Контейнеризация:** Docker, Docker Compose.

---

## 📋 Предварительные требования

* Установленные Docker и Docker Compose.
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
  host: mqtt
  port: 1883
  user: ''
  password: ''
  topic_prefix: frigate

lpr:
  enabled: True

objects:
  track:
    - person

go2rtc:
  streams:
    cam1:
      - rtsp://USERNAME:PASSWORD@IP_ADDRESS:554/stream1
    cam1_sub:
      - rtsp://USERNAME:PASSWORD@IP_ADDRESS:554/stream2
    cam360:
      - rtsp://USERNAME:PASSWORD@IP_ADDRESS:554/stream1
    cam360_sub:
      - rtsp://USERNAME:PASSWORD@IP_ADDRESS:554/stream2

cameras:
  cam1:
    ffmpeg:
      inputs:
        - path: rtsp://127.0.0.1:8554/cam1
          roles:
            - record
        - path: rtsp://127.0.0.1:8554/cam1_sub
          roles:
            - detect
    detect:
      enabled: true
      width: 1280
      height: 720
      fps: 5
    objects:
      track:
        - person
    record:
      enabled: true
      continuous:
        days: 3
    mqtt:
      enabled: true
      timestamp: true
      bounding_box: true
      crop: true
      height: 270
      quality: 80

  cam360:
    ffmpeg:
      inputs:
        - path: rtsp://127.0.0.1:8554/cam360
          roles:
            - record
        - path: rtsp://127.0.0.1:8554/cam360_sub
          roles:
            - detect
    detect:
      enabled: true
      width: 1280
      height: 720
      fps: 5
    objects:
      track:
        - person
    record:
      enabled: true
      continuous:
        days: 3
    onvif:
      host: IP_ADDRESS
      port: 2020
      user: cam360
      password: PASSWORD
    mqtt:
      enabled: true
      timestamp: true
      bounding_box: true
      crop: true
      height: 270
      quality: 80

version: 0.17-0
```

## ⚙️ Параметры конфигурации

### Сервис уведомлений (NestJS)

| Переменная | Описание | По умолчанию |
| :--- | :--- | :--- |
| `TELEGRAM_BOT_TOKEN` | Токен бота | — |
| `TELEGRAM_CHAT_ID` | ID чата для оповещений | — |
| `FRIGATE_URL` | Путь сервера frigate:5000 | — |
| `MQTT_BROKER_URL` | Путь сервера mqtt:1883 | — |

## 🔮 Планы на будущее

- [ ] Интеграция с Home Assistant через MQTT auto‑discovery.

## 📄 Лицензия

MIT License.

---
*Zork — наблюдает за вас, кадр за кадром.*
