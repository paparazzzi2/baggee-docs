# API overview

Snapshot of the REST API: **96 operations** in **12 groups**. The machine-readable contract is in [`api/openapi.yaml`](../api/openapi.yaml) (OpenAPI 3.1, generated from the FastAPI code). Paste it into <https://editor.swagger.io> to browse it interactively.

`Auth` column: *Bearer token* = JWT required; role checks (moderator / admin) happen on the server and are described in [roles](01-roles-and-permissions.md).

## Auth

Регистрация, вход, подтверждение email, восстановление и смена пароля / Registration, login, email confirmation, password reset

| Method | Path | Description | Auth |
|---|---|---|---|
| `POST` | `/auth/register` | Register | — |
| `POST` | `/auth/login` | Login | — |
| `GET` | `/auth/me` | Me | Bearer token |
| `GET` | `/auth/user/{username}` | Public Profile | — |
| `POST` | `/auth/forgot-password` | Всегда отвечает одинаково — нельзя узнать, зарегистрирован ли email. | — |
| `POST` | `/auth/reset-password` | Reset Password | — |
| `POST` | `/auth/send-confirmation` | Повторная отправка письма подтверждения (кнопка в настройках). | Bearer token |
| `GET` | `/auth/confirm-email` | Переход по ссылке из письма. Редиректит на страницу результата. | — |
| `POST` | `/auth/change-password` | Change Password | Bearer token |
| `POST` | `/auth/change-username` | Смена ника. username — ссылка во многих таблицах (лоты, сделки, чаты, | Bearer token |
| `POST` | `/auth/change-email` | Смена почты: подтверждение сбрасывается, на новый адрес уходит письмо | Bearer token |
| `POST` | `/auth/avatar` | Upload Avatar | Bearer token |
| `POST` | `/auth/cover` | Фон (обложка) шапки профиля. Требования: JPG/PNG/WEBP/GIF, до 10 МБ. | Bearer token |
| `DELETE` | `/auth/cover` | Убрать обложку — вернётся стандартный градиент. | Bearer token |
| `PATCH` | `/auth/avatar/position` | Какую часть фото показывать в круге аватара (object-position, % по X/Y). | Bearer token |
| `PATCH` | `/auth/cover/position` | Какую часть картинки показывать в обложке (background-position, % по X/Y). | Bearer token |

## Profile

Профиль пользователя / User profile

| Method | Path | Description | Auth |
|---|---|---|---|
| `GET` | `/profile/me` | Profile Me | Bearer token |
| `PATCH` | `/profile/me` | Update Profile | Bearer token |

## Lots

Лоты путешественников: маршрут, свободный вес, комиссия / Traveler lots

| Method | Path | Description | Auth |
|---|---|---|---|
| `POST` | `/lots` | Create Lot | Bearer token |
| `GET` | `/lots` | List Lots | — |
| `GET` | `/lots/my` | All lots belonging to the current user, any status. | Bearer token |
| `PATCH` | `/lots/{lot_id}` | Update Lot | Bearer token |
| `GET` | `/lots/{lot_id}` | Get Lot | — |
| `DELETE` | `/lots/{lot_id}` | Delete Lot | Bearer token |
| `GET` | `/lots/map-data` | Public endpoint: active + pending lots for 2D map display. | — |
| `POST` | `/lots/{lot_id}/tickets` | Загрузка билетов владельцем лота. До 5 файлов. Каждый — JPEG/PNG/WebP/GIF/PDF до 15 МБ. | Bearer token |
| `DELETE` | `/lots/{lot_id}/tickets/{filename}` | Delete Lot Ticket | Bearer token |

## Lot moderation

Проверка лотов модератором / Moderator review of lots

| Method | Path | Description | Auth |
|---|---|---|---|
| `GET` | `/lots/pending` | List Pending Lots | Bearer token |
| `GET` | `/lots/owner-stats/{username}` | Досье продавца для модерации: ID аккаунта, стаж и история лотов. | Bearer token |
| `PUT` | `/lots/pending/{lot_id}/approve` | Approve Lot | Bearer token |
| `PUT` | `/lots/pending/{lot_id}/reject` | Reject Lot | Bearer token |

## Deals

Сделки с эскроу: оплата, фото товара, подтверждение, споры / Escrow deals

| Method | Path | Description | Auth |
|---|---|---|---|
| `POST` | `/deals` | Create Deal | Bearer token |
| `GET` | `/deals/my` | Get My Deals | Bearer token |
| `GET` | `/deals/pending-payment` | Для модераторов — сделки ожидающие подтверждения оплаты. | Bearer token |
| `GET` | `/deals/support-history` | Все сделки, где когда-либо вызывали поддержку или открывали спор (для админ-вкладки «Споры»). | Bearer token |
| `GET` | `/deals/{deal_id}` | Get Deal | Bearer token |
| `GET` | `/lots/{lot_id}/deals` | Get Lot Deals | — |
| `POST` | `/deals/{deal_id}/pay` | Buyer Pay | Bearer token |
| `POST` | `/deals/{deal_id}/pay-now` | ДЕМО-оплата (без реальных денег). При PAYMENTS_REAL=true отключена — | Bearer token |
| `POST` | `/deals/{deal_id}/confirm-payment` | Confirm Payment | Bearer token |
| `POST` | `/deals/{deal_id}/reject-payment` | Reject Payment | Bearer token |
| `POST` | `/deals/{deal_id}/seller-photos` | Upload Seller Photos | Bearer token |
| `POST` | `/deals/{deal_id}/confirm-photos` | Confirm Photos | Bearer token |
| `POST` | `/deals/{deal_id}/complete` | Complete Deal | Bearer token |
| `POST` | `/deals/{deal_id}/call-support` | Call Support | Bearer token |
| `POST` | `/deals/{deal_id}/dispute` | Open Dispute | Bearer token |
| `POST` | `/deals/{deal_id}/resolve` | Resolve Dispute | Bearer token |
| `GET` | `/deals/{deal_id}/peer-history` | Полная история обычного чата покупатель↔продавец по лоту сделки. Только для модератора. | Bearer token |
| `POST` | `/deals/{deal_id}/arrived` | Seller Arrived | Bearer token |
| `POST` | `/deals/{deal_id}/cancel` | Cancel Deal Endpoint | Bearer token |
| `POST` | `/deals/{deal_id}/support-msg` | Support Message | Bearer token |
| `GET` | `/deals/{deal_id}/thread` | Deal Thread | Bearer token |
| `POST` | `/deals/{deal_id}/close-support` | Закрыть обращение поддержки. Для спора (disputed) деньги распределяются кнопками | Bearer token |
| `GET` | `/deals/{deal_id}/chat` | Возвращает все сообщения в чате сделки. Доступ — участникам и модераторам. | Bearer token |
| `POST` | `/deals/{deal_id}/chat` | Отправить сообщение в чат сделки. Модератор пишет как «Поддержка». | Bearer token |
| `GET` | `/deals/active-support` | Для модератора — сделки, где вызвана поддержка или открыт спор. | Bearer token |
| `POST` | `/deals/{deal_id}/checkout` | Покупатель инициирует оплату: создаём hold у провайдера, возвращаем ссылку. | Bearer token |
| `GET` | `/deals/{deal_id}/payment` | Актуальный платёж сделки — для плашки статуса эскроу на странице сделки. | Bearer token |

## Payments

Эскроу-платёж через провайдера, вебхук, реквизиты выплат / Escrow checkout, webhook, payout details

| Method | Path | Description | Auth |
|---|---|---|---|
| `POST` | `/payments/webhook` | Уведомления ЮKassa. Подлинность: статус перепроверяется запросом к API | — |
| `GET` | `/payments/my` | Финансы пользователя: как продавца (выплаты) и как покупателя (оплаты). | Bearer token |
| `GET` | `/payments/payout-method` | Get Payout Method | Bearer token |
| `POST` | `/payments/payout-method` | Set Payout Method | Bearer token |
| `GET` | `/payments/payout-card` | Get Payout Requisites | Bearer token |
| `POST` | `/payments/payout-card` | Привязать карту для выплат. Приходит «синоним» от виджета ЮKassa — | Bearer token |
| `DELETE` | `/payments/payout-card` | Отвязать реквизиты. Открытые заявки не трогаем — у них свой снапшот. | Bearer token |
| `POST` | `/payments/payout-sbp` | Привязать телефон СБП. Способ по умолчанию: дешевле карт в 5–9 раз. | Bearer token |

## Wallet

Баланс, журнал операций, вывод средств / Balance, ledger, withdrawals

| Method | Path | Description | Auth |
|---|---|---|---|
| `GET` | `/wallet/summary` | Всё, что нужно странице кошелька, одним запросом. | Bearer token |
| `GET` | `/wallet/history` | Журнал операций: где деньги пришли (+), где ушли (−). | Bearer token |
| `GET` | `/wallet/withdrawals` | My Withdrawals | Bearer token |
| `POST` | `/wallet/withdraw` | Заявка на вывод средств. | Bearer token |
| `POST` | `/wallet/withdraw/{withdrawal_id}/cancel` | Отменить свою заявку. Только пока она не ушла в банк (`pending`). | Bearer token |
| `GET` | `/wallet/` | Wallet Page | — |
| `GET` | `/wallet` | Wallet Page | — |

## Messages

Личные чаты и чат сделки / Direct and deal chats

| Method | Path | Description | Auth |
|---|---|---|---|
| `GET` | `/conversations` | Get Conversations | Bearer token |
| `GET` | `/messages/{peer_username}` | Get Messages | Bearer token |
| `POST` | `/messages/{peer_username}` | Send Message | Bearer token |
| `GET` | `/conversations/unread-count` | Возвращает число ЧАТОВ (не сообщений) с хотя бы одним непрочитанным. | Bearer token |

## Reviews

Отзывы о пользователях / Reviews

| Method | Path | Description | Auth |
|---|---|---|---|
| `GET` | `/reviews/{username}` | Все отзывы о пользователе. | — |
| `POST` | `/reviews` | Оставить отзыв. Один отзыв от одного пользователя на один лот. | Bearer token |
| `PATCH` | `/reviews/{review_id}` | Покупатель редактирует свой отзыв. | Bearer token |
| `DELETE` | `/reviews/{review_id}` | Удаление: администратор — любой отзыв; покупатель — только свой. | Bearer token |

## Route alerts

Подписки на маршруты / Route subscriptions

| Method | Path | Description | Auth |
|---|---|---|---|
| `GET` | `/alerts` | My Alerts | Bearer token |
| `POST` | `/alerts` | Create Alert | Bearer token |
| `DELETE` | `/alerts/{alert_id}` | Delete Alert | Bearer token |

## Notifications

Уведомления / Notifications

| Method | Path | Description | Auth |
|---|---|---|---|
| `GET` | `/notifications` | Get Notifications | Bearer token |
| `PATCH` | `/notifications/read-all` | Mark All Read | Bearer token |

## Admin

Администрирование: роли, отзывы / Administration

| Method | Path | Description | Auth |
|---|---|---|---|
| `GET` | `/admin/users` | List Users | Bearer token |
| `PUT` | `/admin/users/{user_id}/role` | Change Role | Bearer token |
| `GET` | `/admin/reviews` | Все отзывы на платформе — для модерации. | Bearer token |
| `DELETE` | `/admin/reviews/{review_id}` | Admin Delete Review | Bearer token |
| `GET` | `/admin/withdrawals` | Admin List Withdrawals | Bearer token |
| `POST` | `/admin/withdrawals/{withdrawal_id}/approve` | Одобрить заявку и отправить её провайдеру. | Bearer token |
| `POST` | `/admin/withdrawals/{withdrawal_id}/reject` | Отклонить заявку. Деньги возвращаются на баланс пользователя. | Bearer token |
| `POST` | `/admin/wallet/adjust` | Ручная корректировка баланса. Всегда попадает в журнал с автором и причиной. | Bearer token |
| `GET` | `/admin/wallet/audit` | Сверка: кеш баланса против журнала. Любое расхождение — баг, не норма. | Bearer token |
| `GET` | `/admin/metrics` | Get Metrics | Bearer token |
