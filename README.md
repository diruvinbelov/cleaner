# Cleaner

Веб-приложение для анализа и очистки диска C: на Windows-компьютерах сети.
Список компьютеров берётся из GLPI, очистка выполняется по SSH набором PowerShell-скриптов.

## Этапы

- [x] **Этап 1** — вход через AD, загрузка компьютеров `WS*` «В работе» из GLPI, таблица, поиск/фильтры/сортировка, проверка доступности (порт SSH), выбор компьютеров
- [ ] **Этап 2** — обновление данных о диске напрямую по SSH
- [ ] **Этап 3** — скрипты очистки и очистка одного компьютера с живым логом
- [ ] **Этап 4** — массовая очистка и история
- [ ] **Этап 5** — эксплуатация: сервисная учётка, ключи SSH, группа AD

## Структура

```
app/
  main.py      — веб-сервер (FastAPI), API
  config.py    — настройки из переменных окружения
  auth.py      — вход через Active Directory
  glpi.py      — работа с GLPI REST API
  netcheck.py  — проверка доступности компьютеров
  db.py        — локальная база SQLite (/data/cleaner.db)
  static/      — веб-страница (HTML, CSS, JS)
scripts/       — PowerShell-скрипты очистки (этап 3)
```

## Развёртывание в Coolify

1. Залить код в приватный репозиторий GitHub.
2. Coolify → **Projects → + New → Resource → Private Repository (with GitHub App)** → выбрать репозиторий.
3. **Build Pack: Docker Compose** (файл `docker-compose.yml`).
4. **Environment Variables**: заполнить по образцу `.env.example` (минимум `GLPI_URL`, `GLPI_APP_TOKEN`, `GLPI_USER_TOKEN`, `SECRET_KEY`).
5. В настройках сервиса `cleaner` указать домен, например `http://cleaner.wp1.local` или `https://cleaner.narvahaigla.ee` (порт контейнера — 8000).
6. **Deploy**.

## Локальный запуск (для отладки)

```bash
cp .env.example .env   # заполнить
# раскомментировать ports в docker-compose.yml
docker compose up --build
# открыть http://localhost:8080
```

## Безопасность

- Токены и пароли — только в переменных окружения Coolify, не в git.
- Вход разрешён только пользователям из `LDAP_ALLOWED_USERS` или группы `LDAP_ALLOWED_GROUP`.
- 5 неудачных попыток входа с одного IP — блокировка на 5 минут.
