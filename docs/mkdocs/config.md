sdf
```yaml
service:
  name: "anomaly_detection_lstm_victoriametrics"
  version: "1.0"
  description: "Сервис для обнаружения аномалий в одномерных и многомерных временных рядах из VictoriaMetrics с использованием LSTM"
```
```yaml
victoriametrics:
  url: "http://victoriametrics:8428"  # URL VictoriaMetrics
  promql_queries:  # Список PROMQL-запросов для получения данных
    - name: "cpu_usage"
      query: "rate(cpu_usage_total[1m])"
    - name: "memory_usage"
      query: "memory_usage_bytes"
  lookback_period: "1h"  # Период для запроса исторических данных
  step: "1m"  # Шаг между точками данных
```
```yaml
data:
  output_path: "/data/output/anomalies.csv"  # Путь для сохранения результатов
  time_column: "timestamp"  # Название колонки с временными метками
  training_window: "7d"  # Временное окно для переобучения модели
  window_size: 30  # Размер окна анализа для предсказания
```
```yaml
models:
  - name: "cpu_usage_model"
    metric: "cpu_usage"  # Название метрики
    type: "LSTM"
    layers:
      - type: "LSTM"
        units: 64
        return_sequences: True
      - type: "LSTM"
        units: 32
        return_sequences: False
      - type: "Dense"
        units: 1
    loss: "mean_squared_error"
    optimizer: "adam"
    epochs: 50
    batch_size: 32
    validation_split: 0.2
    dropout: 0.2  # Dropout для предотвращения переобучения
    learning_rate: 0.001
    retrain:
      schedule: "24h"  # Переобучение каждые 24 часа
      drift_detection: True  # Переобучение при дрейфе данных
      validation_threshold: 0.1  # Порог ошибки для переобучения

  - name: "memory_usage_model"
    metric: "memory_usage"  # Название метрики
    type: "LSTM"
    layers:
      - type: "LSTM"
        units: 128
        return_sequences: True
      - type: "LSTM"
        units: 64
        return_sequences: False
      - type: "Dense"
        units: 1
    loss: "mean_absolute_error"
    optimizer: "rmsprop"
    epochs: 30
    batch_size: 64
    validation_split: 0.2
    dropout: 0.3
    learning_rate: 0.0005
    retrain:
      schedule: "12h"  # Переобучение каждые 12 часов
      drift_detection: True
      validation_threshold: 0.15
```
```yaml
anomaly_detection:
  threshold: 3.0  # Порог для определения аномалий
  smoothing_window: 5  # Окно для сглаживания предсказаний
  group__window: 20 # Окно для определения групповой аномалии
```
```yaml
telegram:
  enabled: True
  bot_token: "YOUR_TELEGRAM_BOT_TOKEN"
  chat_id: "YOUR_TELEGRAM_CHAT_ID"
  message_template: "Обнаружена аномалия! Метрика: {metric}, Время: {timestamp}, Значение: {value}"
```
```yaml
infrastructure:
  gpu_enabled: True
  workers: 4
  memory_limit: "8G"
  model_storage: "/models"  # Путь для сохранения моделей
```
```yaml
monitoring:
  log_path: "/logs/anomaly_detection.log"
  metrics:
    - "mse"
    - "mae"
    - "anomaly_count"
  alerting:
    enabled: True
    threshold: 100
```
```yaml
api:
  enabled: True
  host: "0.0.0.0"
  port: 8080
  endpoints:
    - "/predict"
    - "/health"
```