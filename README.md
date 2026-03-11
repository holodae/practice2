# CI/CD Pipeline для Python-приложения на GitHub Actions

## 📋 Описание проекта
Данный репозиторий содержит конфигурацию автоматического пайплайна непрерывной интеграции и доставки (CI/CD) для Python-приложения с использованием **GitHub Actions**.

**Цель пайплайна:** Автоматизировать процессы линтинга, тестирования и сборки при каждом изменении в коде, чтобы обеспечить стабильность и качество релизов.

## 🚀 Этапы Pipeline (Workflow)

Наш пайплайн состоит из следующих ключевых этапов:

1.  **Триггер:** Запуск происходит автоматически при событиях `push` или `pull request` в ветку `main`.
2.  **Линтинг (Lint):** Проверка кода на соответствие стандартам PEP8 с помощью `flake8`. *Это помогает поддерживать чистоту кода.*
3.  **Тестирование (Test):** Запуск автоматических тестов через `pytest`. **Важно:** Если тесты падают, дальнейшее выполнение пайплайна останавливается.
4.  **Сборка (Build):** Упаковка приложения в Docker-образ и его публикация в GitHub Container Registry.

## ⚙️ Детальное описание шагов

Здесь представлена информация о каждом джобе и условиях их запуска.

| Шаг (Job) | Название | Запускается на | Действие |
| :--- | :--- | :--- | :--- |
| **Lint** | `lint` | `ubuntu-latest` | Проверка кода линтером (`flake8`). |
| **Test** | `test` | `ubuntu-latest` | Запуск юнит-тестов (`pytest`). |
| **Build** | `build-and-push` | `ubuntu-latest` | Сборка Docker-образа и пуш в реестр. |

> **Примечание:** Job `build-and-push` запускается **только** после успешного завершения `lint` и `test`, и только для `push` в `main`.

## 🔧 Конфигурация Workflow (YAML)

Файл рабочего процесса находится по пути `.github/workflows/ci.yml`. Вот его содержимое:

```yaml
name: Python CI/CD Pipeline

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: pip install flake8
      - name: Lint with flake8
        run: flake8 . --count --statistics

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: pip install pytest
      - name: Test with pytest
        run: pytest

  build-and-push:
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t my-app:${{ github.sha }} .
      - name: Log in to GitHub Container Registry
        run: echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Push Docker image
        run: |
          docker tag my-app:${{ github.sha }} ghcr.io/${{ github.repository }}/my-app:latest
          docker push ghcr.io/${{ github.repository }}/my-app:latest'''




### 🗝️ 5. Инструкции по настройке (Нумерованный список + Ссылки)

```markdown
## 🚦 Запуск и активация пайплайна

Чтобы пайплайн заработал в вашем репозитории, выполните следующие шаги:

1.  **Подготовьте репозиторий:**
    Убедитесь, что в корне вашего проекта есть файл `Dockerfile` и тесты (`tests/`).

2.  **Создайте структуру:**
    Вручную создайте в репозитории папку `.github/workflows/`.

3.  **Добавьте конфигурацию:**
    Поместите файл `ci.yml` (с кодом выше) в папку `.github/workflows/`.

4.  **Настройте секреты (если нужно):**
    Для публикации в GitHub Container Registry используется автоматический токен `secrets.GITHUB_TOKEN`. Он создается автоматически. Если вы используете сторонние реестры (Docker Hub), нужно добавить [секреты репозитория](https://docs.github.com/ru/actions/security-guides/using-secrets-in-github-actions).

5.  **Запушьте изменения:**
    ```bash
    git add .github/
    git commit -m "Add CI/CD pipeline"
    git push origin main



### 🏁 6. Заключение (Горизонтальная линия)

```markdown
---
### ✅ Результат
Теперь при каждом изменении в коде процессы линтинга, тестирования и сборки будут проходить автоматически, что значительно ускоряет разработку и снижает риск ошибок.
