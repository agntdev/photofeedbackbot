# PhotoFeedbackBot — Bot specification

**Archetype:** custom

**Voice:** профессиональный и дружелюбный — write every user-facing message, button label, error, and empty state in this voice.

Приватный Telegram-бот, который анализирует загруженные фотографии и предоставляет пользователю оценку от 0 до 100, категоризированные оценки и практические советы по улучшению фото. Результаты видны только пользователю и владельцу бота.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- люди, желающие получить обратную связь по своим фотографиям
- пользователи, ожидающие быстрых и полезных рекомендаций

## Success criteria

- пользователь получает полезный и понятный фидбек по фото
- бот обрабатывает фото и возвращает результаты в течение 10 секунд
- бот обеспечивает приватность данных и результатов

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Открывает главное меню с инструкцией и кнопками
- **Отправить фото** (button, actor: user, callback: upload:start) — Запускает процесс загрузки и анализа фото
  - inputs: фото
  - outputs: подтверждение получения фото, результаты анализа
- **Правила и помощь** (button, actor: user, callback: help:start) — Открывает раздел с правилами использования и помощью
  - outputs: информация о правилах и помощи
- **/history** (command, actor: user, command: /history) — Открывает историю прошлых оценок

## Flows

### main_menu
_Trigger:_ /start

1. отправить приветственное сообщение
2. показать кнопки: 'Отправить фото' и 'Правила и помощь'

_Data touched:_ User

### photo_upload
_Trigger:_ upload:start

1. ожидать загрузку фото
2. подтвердить получение фото
3. начать анализ фото
4. отправить результаты анализа

_Data touched:_ Photo, Feedback

### history
_Trigger:_ /history

1. показать краткие карточки прошлых оценок

_Data touched:_ User, Photo, Feedback

### error_reporting
_Trigger:_ critical error

1. отправить уведомление в админ-чат
2. сохранить информацию об ошибке

_Data touched:_ User, Photo, Feedback

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram-идентификатор пользователя и его настройки
  - fields: telegram_id, privacy_settings, free_usage_count, premium_status
- **Photo** _(retention: persistent)_ — Загруженное изображение и его метаданные
  - fields: image_id, upload_time, user_id, saved_to_history
- **Feedback** _(retention: persistent)_ — Результаты анализа фото
  - fields: photo_id, overall_score, category_scores, advice, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging
- **Payment Gateway** (optional) — Обработка платных премиум-запросов
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- установка лимита бесплатных оценок в месяц
- настройка цен и текста продажи для премиум-пакета
- настройка админ-чата для уведомлений
- просмотр статистики использования бота

## Notifications

- уведомления пользователей о результатах анализа
- уведомления администратора о критических ошибках
- уведомления администратора о возможных злоупотреблениях

## Permissions & privacy

- результаты видны только пользователю и владельцу бота
- фото сохраняются только если пользователь явно выбрал 'Сохранить в историю'
- данные пользователей хранятся минимально и конфиденциально

## Edge cases

- пользователь загружает фото с низким разрешением
- пользователь отправляет несколько фото одновременно
- бот не может обработать фото из-за технических проблем
- пользователь превысил лимит бесплатных оценок

## Required tests

- тестирование обработки фото и возврата результатов
- тестирование приватности данных
- тестирование лимитов бесплатных оценок
- тестирование платной функции (если включена)

## Assumptions

- пользователь доверяет боту в выборе параметров оценки
- одно фото за один запрос упрощает UX
- общий формат оценки интуитивно понятен пользователям
- владелец бота настроит админ-чат для уведомлений
