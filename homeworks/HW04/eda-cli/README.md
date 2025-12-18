# EDA CLI + FastAPI

Проект предоставляет:

* **CLI‑утилиту для EDA CSV‑файлов** (Seminar S03);
* **FastAPI для эвристической оценки качества датасета** (Seminar S04), использующую то же EDA‑ядро.

---

## Требования

* Python **3.11+**
* Установленный менеджер окружений **uv**
  ([https://docs.astral.sh/uv/](https://docs.astral.sh/uv/))

---

## Установка и инициализация

В корне проекта выполните:

```bash
uv sync
```

Команда:

* создаёт виртуальное окружение `.venv`;
* устанавливает зависимости из `pyproject.toml`;
* устанавливает проект в editable‑режиме.

---

## CLI: `eda-cli`

CLI предназначен для быстрого **exploratory data analysis (EDA)** CSV‑файлов.

Запуск осуществляется через `uv run`:

```bash
uv run eda-cli --help
```

### Команда `overview`

Краткий текстовый обзор датасета:

* число строк и столбцов;
* типы данных;
* количество и доля пропусков;
* число уникальных значений;
* базовые числовые статистики.

Пример:

```bash
uv run eda-cli overview data/example.csv
```

Параметры:

* `--sep` — разделитель CSV (по умолчанию `,`)
* `--encoding` — кодировка файла (по умолчанию `utf-8`)

---

### Команда `report`

Генерирует **полный EDA‑отчёт** в указанной директории.

Создаваемые артефакты:

* `report.md` — Markdown‑отчёт;
* `summary.csv` — статистика по колонкам;
* `missing.csv` — таблица пропусков;
* `correlation.csv` — матрица корреляций числовых признаков;
* `top_categories/*.csv` — top‑k категорий для категориальных признаков;
* изображения:

  * `hist_*.png` — гистограммы числовых колонок;
  * `missing_matrix.png` — карта пропусков;
  * `correlation_heatmap.png` — тепловая карта корреляций.

Базовый вызов:

```bash
uv run eda-cli report data/example.csv --out-dir reports
```

#### Параметры команды `report`

* `--out-dir <path>` — каталог для отчёта (по умолчанию `reports`)
* `--sep <str>` — разделитель CSV
* `--encoding <str>` — кодировка файла
* `--max-hist-columns <int>` — максимум числовых колонок для гистограмм (по умолчанию `6`)
* `--top-k-categories <int>` — количество выводимых категорий (по умолчанию `5`)
* `--title <str>` — заголовок в `report.md` (по умолчанию `report`)

Пример расширенного вызова:

```bash
uv run eda-cli report data/example.csv \
    --out-dir reports_example \
    --sep ";" \
    --encoding "utf-8" \
    --max-hist-columns 10 \
    --top-k-categories 8 \
    --title "Полный анализ Example.csv"
```

---

## Эвристики качества данных

И CLI‑отчёт, и API используют единое EDA‑ядро и вычисляют набор флагов качества:

* `too_few_rows` — слишком мало строк;
* `too_many_columns` — слишком много колонок;
* `too_many_missing` — слишком высокая доля пропусков;
* `has_constant_columns` — есть константные колонки;
* `has_suspicious_id_duplicates` — возможные дубликаты идентификаторов;
* `has_high_cardinality_categoricals` — категориальные признаки с высокой кардинальностью;
* `max_missing_share` — максимальная доля пропусков по колонке;
* `quality_score` — интегральная оценка качества (0..1).

---

## FastAPI

FastAPI‑сервис предоставляет REST‑интерфейс для оценки качества датасетов.

### Запуск сервиса

Пример локального запуска:

```bash
uv run uvicorn eda_cli.api:app --reload
```

Swagger‑документация будет доступна по адресу:

```
http://localhost:8000/docs
```

---

### Эндпоинты

#### `GET /health`

Health‑check сервиса.

Ответ:

```json
{
  "status": "ok",
  "service": "dataset-quality",
  "version": "0.2.0"
}
```

---

#### `POST /quality`

Эвристическая оценка качества по **агрегированным признакам датасета**.

Используется как заглушка ML‑модели.

Вход:

* `n_rows`
* `n_cols`
* `max_missing_share`
* `numeric_cols`
* `categorical_cols`

Выход:

* `ok_for_model` — можно ли обучать модель;
* `quality_score` — скор качества (0..1);
* `flags` — диагностические флаги;
* `latency_ms` — время обработки;
* `dataset_shape` — размеры датасета.

---

#### `POST /quality-from-csv`

Принимает **CSV‑файл**, выполняет EDA‑анализ и возвращает агрегированную оценку качества.

Вход:

* CSV загружается как `multipart/form-data`;

Выход:

* `ok_for_model` — можно ли обучать модель;
* `quality_score` — скор качества (0..1);
* `flags` — только булевые флаги;
* `latency_ms` — время обработки;
* `dataset_shape` — размеры датасета.
---

#### `POST /quality-flags-from-csv`

Принимает CSV‑файл и возвращает **полный набор флагов качества**
(включая числовые значения и итоговый скор).

Удобно для отладки и аналитики.

---
