# Динамическое масштабирование

Deployment запускает одну реплику. HPA увеличивает их количество до 10 при превышении 80% утилизации памяти. Процент считается от `requests.memory: 10Mi`, то есть целевой уровень — 8 MiB на под. Лимит памяти — `30Mi`.

## Запуск

Команды выполняются из корня репозитория.

```bash
minikube start -p insuretech-task2 --driver=docker --cpus=2 --memory=3072
minikube -p insuretech-task2 addons enable metrics-server
kubectl --context=insuretech-task2 apply -f Task2/deployment.yaml -f Task2/service.yaml -f Task2/hpa.yaml
minikube -p insuretech-task2 service scaletestapp --url
```

Дождаться показаний памяти в `kubectl --context=insuretech-task2 get hpa` вместо `<unknown>`.

## Нагрузочный тест

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install locust
locust -f Task2/locustfile.py
```

Открыть http://localhost:8089. В поле Host указать URL сервиса из предыдущего шага. Задать 1000 пользователей и скорость добавления 5 пользователей/с. Продолжительность теста — 5 минут.

Наблюдать за масштабированием:

```bash
kubectl --context=insuretech-task2 get hpa -w
kubectl --context=insuretech-task2 get pods -l app=scaletestapp
```

## Результат

Тест от 28.09.2026. Перед нагрузкой работали две реплики, оставшиеся от предыдущего запуска. HPA увеличил их количество: 2 → 3 → 5 → 7. Все семь подов работали без перезапусков. Locust выполнил 67 397 запросов без ошибок.

- [Масштабирование и события HPA](scaling.txt).
- [Статистика Locust](locust.txt).
