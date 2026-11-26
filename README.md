# Домашнее задание к занятию 16 «Платформа мониторинга Sentry»

**Способ развертывания:** Self-Hosted Sentry, версия **26.10.1** на виртуальной машине (Proxmox VE)
**Конфигурация VM:** 4 vCPU, 32 ГБ RAM, 132 ГБ SSD
**ОС:** Debian 13 (trixie)

---

## Задание 1. Меню Projects

Sentry развёрнут по официальной инструкции из репозитория [getsentry/self-hosted](https://github.com/getsentry/self-hosted) (тег `26.10.1`). После входа в систему открыто меню **Projects**.

![Меню Projects](screenshots/1-projects.png)

---

## Задание 2. Sample Event, Stack Trace и Resolved

Создан Python-проект `homework-project`. С помощью официального SDK `sentry-sdk` сгенерировано исключение `ZeroDivisionError`. В событии открыт раздел **Stack Trace** с полной трассировкой стека.

**Код скрипта `sample2.py`:**

```
import sentry_sdk

sentry_sdk.init(dsn="http://93812f12add431c71c3501878aea0ade@192.168.0.18:9000/2")

try:
    1 / 0
except ZeroDivisionError:
    sentry_sdk.capture_exception()

print("Событие отправлено. Проверьте Issues в Sentry.")
```

**Stack Trace события:**

![Stack Trace](screenshots/2-stacktrace.png)

После изучения событие переведено в статус **Resolved** через список Issues.

**Список Issues после нажатия Resolved:**

![Список Issues после Resolved](screenshots/3-resolved.png)

---

## Задание 3. Алёртинг и email-уведомление

Создано правило алёртинга:

- **Триггер:** A new issue is created (создание нового issue).
- **Условия:** все фильтры в положении **All** (без дополнительной настройки).
- **Действие:** отправка email на адрес `info@akamsoft.ru`.

Для реальной доставки почты настроен внешний SMTP-сервер **Яндекс.Почта**:

- **SMTP host:** `smtp.yandex.ru`
- **Порт:** `465` (SSL)
- **Аутентификация:** логин — полный адрес ящика, пароль — пароль приложения (создан в Яндекс ID).
- **Параметры в `sentry/config.yml` и `sentry/sentry.conf.py`:** `mail.from` совпадает с `mail.username` (требование Яндекса).

После повторной генерации события через SDK письмо успешно доставлено на `info@akamsoft.ru`.

**Тело полученного письма из оповещения:**

![Тело письма](screenshots/4-email.png)

---

## Задание повышенной сложности

К Python-скрипту подключён Sentry SDK (`sentry-sdk` 2.x). Отправлены три типа событий:

1. **Простое сообщение** через `sentry_sdk.capture_message()`.
2. **Перехваченное исключение** через `sentry_sdk.capture_exception()` (`ZeroDivisionError`).
3. **Исключение с дополнительным контекстом** — теги (`homework: task_4`) и extra-параметр, заданные через `sentry_sdk.new_scope()` (в SDK 2.x `configure_scope()` и `push_scope()` удалены, вместо них используются `new_scope()` / `isolation_scope()`).

**Список Issues с событиями, отправленными из SDK:**

![Issues проекта](screenshots/5-issues-sdk.png)

**Код скрипта `sample3.py`:**

![Код скрипта](screenshots/6-code.png)

---
