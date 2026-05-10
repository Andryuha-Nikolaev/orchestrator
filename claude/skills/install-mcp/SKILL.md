---
name: install-mcp
description: Используй когда нужно установить, удалить, настроить или отладить MCP сервер для Claude Code на Windows. Содержит правильный формат команды claude mcp add, специфику Windows/PowerShell, типичные ошибки и решения.
---

# Установка MCP серверов для Claude Code (Windows)

## Требования

- **OS:** Windows 10/11
- **Shell:** PowerShell 5.1+ или cmd
- **Node:** v20+ с npx (`node --version`, `npx --version`)
- **Claude CLI:** установлен и доступен в `PATH` (`where.exe claude`)
- **Конфиг MCP:** `%USERPROFILE%\.claude.json` → секция `mcpServers`
  (обычно `C:\Users\<username>\.claude.json`)

---

## Правильный формат команды

```powershell
claude mcp add --transport stdio --env KEY=value <имя_сервера> -- npx -y @namespace/package
```

### Критические правила:

1. **Все флаги ДО имени сервера:** `--transport`, `--env`, `--scope`
2. **`--` отделяет команду запуска** от аргументов Claude CLI
3. **Всегда `-y` у npx** — иначе npx ждёт подтверждения и stdio зависает
4. **Scope по умолчанию = local** (в `%USERPROFILE%\.claude.json`)
5. **На Windows npx — это `npx.cmd`.** Если запуск падает с `spawn npx ENOENT`,
   оборачивай команду через `cmd /c`:
   ```powershell
   claude mcp add --transport stdio <имя> -- cmd /c npx -y @namespace/package
   ```

---

## Примеры установки

### Playwright (браузер)

```powershell
claude mcp add --transport stdio playwright -- cmd /c npx -y @playwright/mcp@latest --output-dir tmp/.playwright-mcp
```

### GitHub

```powershell
claude mcp add --transport stdio --env GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxx github -- cmd /c npx -y @modelcontextprotocol/server-github
```

### Context7 (документация)

```powershell
claude mcp add --transport stdio --env CONTEXT7_API_KEY=ctx7sk-xxx context7 -- cmd /c npx -y @upstash/context7-mcp@latest
```

### n8n

```powershell
claude mcp add --transport stdio --env N8N_API_URL=https://xxx --env N8N_API_KEY=xxx --env MCP_MODE=stdio --env LOG_LEVEL=error n8n -- cmd /c npx -y n8n-mcp
```

### HTTP-сервер (без npx)

```powershell
claude mcp add --transport http <имя> <url>
```

---

## Scope: куда сохраняется

| Флаг                             | Файл                                    | Когда использовать             |
| -------------------------------- | --------------------------------------- | ------------------------------ |
| (по умолчанию / `--scope local`) | `%USERPROFILE%\.claude.json`            | Личный сервер                  |
| `--scope project`                | `.mcp.json` в корне проекта             | Для команды (коммитится в git) |
| `--scope user`                   | `%USERPROFILE%\.claude.json`            | Кросс-проектный личный         |

---

## Управление и отладка

```powershell
claude mcp list                 # все серверы и статус
claude mcp get <имя>            # детали конкретного
claude mcp remove <имя>         # удалить
```

Внутри Claude Code: `/mcp` — статус всех серверов + авторизация OAuth.

### Переменные окружения в PowerShell

В PowerShell **не работает** bash-синтаксис `VAR=value claude`. Используй:

```powershell
$env:MCP_TIMEOUT = "10000"
$env:MAX_MCP_OUTPUT_TOKENS = "50000"
claude
```

В cmd:

```cmd
set MCP_TIMEOUT=10000 && claude
```

### Если сервер не работает:

1. **Проверь, что пакет запускается отдельно:**
   ```powershell
   npx -y @namespace/package --help
   ```
2. **`spawn npx ENOENT`** → оберни в `cmd /c npx ...` (см. правило 5 выше)
3. **Увеличь таймаут:** `$env:MCP_TIMEOUT = "10000"; claude`
4. **Увеличь лимит вывода:** `$env:MAX_MCP_OUTPUT_TOKENS = "50000"; claude`
5. **Логи:** `%USERPROFILE%\.claude\logs\`
6. **Конфиг руками:** открой `%USERPROFILE%\.claude.json` и проверь секцию `mcpServers`

---

## Частые ошибки

| Ошибка                       | Причина                                                                | Решение                               |
| ---------------------------- | ---------------------------------------------------------------------- | ------------------------------------- |
| Флаги игнорируются           | Написаны после имени сервера                                           | Перенести ДО имени                    |
| `Connection closed`          | npx ждёт подтверждения                                                 | Добавить `-y`                         |
| `spawn npx ENOENT`           | Windows не находит `npx` без `.cmd`                                    | Запускать через `cmd /c npx -y ...`   |
| Таймаут при старте           | Медленная загрузка пакета                                              | `$env:MCP_TIMEOUT = "10000"`          |
| `VAR=value` не работает      | PowerShell не поддерживает bash-синтаксис inline-env                   | `$env:VAR = "value"; claude`          |
| Дубли конфигов               | Есть и `%USERPROFILE%\.claude.json` и `%USERPROFILE%\.claude\mcp_servers.json` | Удалить дубль                  |
| Пути с пробелами падают      | Пробел в `C:\Users\...` или `C:\dev home\...`                          | Кавычить аргумент целиком: `"C:\..."` |
