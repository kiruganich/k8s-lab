# Лабораторная работа №5: Основы Kubernetes (k8s)

## Структура проекта

```
k8s-lab/
├── examples/
│   ├── frontend/           # Frontend приложение (Node.js)
│   │   ├── app.js
│   │   ├── package.json
│   │   ├── Dockerfile
│   │   └── public/
│   │       └── index.html
│   └── backend/            # Backend приложение (Go)
│       ├── main.go
│       ├── go.mod
│       └── Dockerfile
├── k8s-manifests/          # Kubernetes манифесты
│   ├── namespace.yaml
│   ├── frontend-deployment.yaml
│   ├── backend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backend-service.yaml
│   ├── configmap.yaml          # Задание 1: ConfigMap
│   ├── secret.yaml             # Задание 1: Secret
│   ├── ingress.yaml            # Задание 2: Ingress
│   └── hpa.yaml                # Задание 4: Horizontal Pod Autoscaler
├── theory.md               # Теория и задания
└── README.md               # Этот файл
```

## Инструкция по выполнению

### Шаг 1: Подготовка Docker-образов

```bash
# Сборка образа frontend
cd examples/frontend
docker build -t k8s-frontend:1.0 .

# Сборка образа backend
cd ../backend
docker build -t k8s-backend:1.0 .
```

### Шаг 2: Проверка образов

```bash
docker images | findstr k8s
```

Должны появиться:
```
k8s-frontend   1.0    <image-id>    <time>    <size>
k8s-backend    1.0    <image-id>    <time>    <size>
```

### Шаг 3: Применение Kubernetes манифестов

```bash
# Создание namespace и всех ресурсов
kubectl apply -f k8s-manifests/

# Или по отдельности:
kubectl apply -f k8s-manifests/namespace.yaml
kubectl apply -f k8s-manifests/backend-deployment.yaml
kubectl apply -f k8s-manifests/backend-service.yaml
kubectl apply -f k8s-manifests/frontend-deployment.yaml
kubectl apply -f k8s-manifests/frontend-service.yaml
```

### Шаг 4: Проверка развертывания

```bash
# Проверка namespace
kubectl get namespace lab5

# Проверка deployments
kubectl get deployments -n lab5

# Проверка pods
kubectl get pods -n lab5

# Проверка services
kubectl get services -n lab5

# Подробная информация о pod
kubectl describe pod <pod-name> -n lab5

# Просмотр логов
kubectl logs <pod-name> -n lab5
```

### Шаг 5: Доступ к приложению

#### Через NodePort:
Откройте браузер и перейдите по адресу: **http://localhost:30080**

#### Через port-forwarding:
```bash
kubectl port-forward service/frontend-service 8080:80 -n lab5
```
Затем откройте: **http://localhost:8080**

### Шаг 6: Тестирование

1. Откройте приложение в браузере
2. Нажмите кнопку "Fetch Data from Backend"
3. Убедитесь, что данные успешно получены
4. Проверьте, что имя Pod отображается в ответе

### Шаг 7: Масштабирование

```bash
# Масштабирование frontend до 3 реплик
kubectl scale deployment frontend-deployment --replicas=3 -n lab5

# Масштабирование backend до 5 реплик
kubectl scale deployment backend-deployment --replicas=5 -n lab5

# Проверка количества pods
kubectl get pods -n lab5
```

### Шаг 8: Обновление приложения

```bash
# Сборка новой версии backend
cd examples/backend
docker build -t k8s-backend:2.0 .

# Обновление deployment
kubectl set image deployment/backend-deployment backend=k8s-backend:2.0 -n lab5

# Проверка статуса обновления
kubectl rollout status deployment/backend-deployment -n lab5

# Просмотр истории обновлений
kubectl rollout history deployment/backend-deployment -n lab5

# Откат к предыдущей версии (если нужно)
kubectl rollout undo deployment/backend-deployment -n lab5
```

### Шаг 9: Применение дополнительных манифестов (самостоятельная работа)

```bash
# Применение ConfigMap и Secret
kubectl apply -f k8s-manifests/configmap.yaml
kubectl apply -f k8s-manifests/secret.yaml

# Применение HPA
kubectl apply -f k8s-manifests/hpa.yaml
```

### Шаг 10: Очистка ресурсов

```bash
# Удаление всех ресурсов в namespace
kubectl delete namespace lab5

# Или удаление по манифестам
kubectl delete -f k8s-manifests/
```

## Полезные команды

| Команда | Описание |
|---------|----------|
| `kubectl get pods -n lab5` | Показать все pods в namespace lab5 |
| `kubectl get services -n lab5` | Показать все services |
| `kubectl get deployments -n lab5` | Показать все deployments |
| `kubectl describe pod <name> -n lab5` | Подробная информация о pod |
| `kubectl logs <pod-name> -n lab5` | Просмотр логов pod |
| `kubectl exec -it <pod-name> -n lab5 -- bash` | Подключиться к pod |
| `kubectl apply -f k8s-manifests/` | Применить все манифесты |
| `kubectl delete -f k8s-manifests/` | Удалить все ресурсы |
| `kubectl scale deployment <name> --replicas=N -n lab5` | Масштабировать deployment |
| `kubectl rollout status deployment <name> -n lab5` | Статус обновления |
| `kubectl rollout undo deployment <name> -n lab5` | Откатить обновление |

## Ответы на контрольные вопросы

См. теоретический материал в файле `theory.md`.
