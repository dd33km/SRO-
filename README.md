# Практическая работа: Контейнеризация, облачная инфраструктура и оркестрация

**Студент:** Рахимжанова Зарина  
**Профиль Docker Hub:** [fhujniomc](https://hub.docker.com/u/fhujniomc)

**Репозиторий GitLab:** [devops-task](https://gitlab.com/dd33km/devops-task)

---

## 1. Теоретическая часть

### Terraform, Ansible и Hardware
* **Terraform:** Инструмент концепции Infrastructure as Code (IaC) для декларативного описания, создания и изменения облачной инфраструктуры (ВМ, сети, файрволы). Хранит текущее состояние ресурсов в файле `terraform.tfstate`.
* **Ansible:** Система управления конфигурациями (Configuration Management). Работает без установки агентов по протоколу SSH через декларативные сценарии (Playbooks в формате YAML). Применяется для настройки ОС, развертывания пакетов и сервисов.
* **Bare-metal / Hardware provisioning:** Управление физическим «железом» без прослойки гипервизора (автоматическая установка ОС через PXE, удаленное управление платами серверов через IPMI/iLO/iDRAC).

### OCI, Docker-образы и Docker Engine
* **OCI (Open Container Initiative):** Открытый отраслевой стандарт, задающий спецификации формата образов (Image Spec) и среды их выполнения на уровне ядра ОС (Runtime Spec, эталонная реализация — `runc`).
* **Docker Image (Образ):** Read-only шаблон из набора слоев (UnionFS), содержащий приложение, библиотеки и настройки среды. Формируется по инструкциям из `Dockerfile`.
* **Docker Engine:** Движок контейнеризации, состоящий из:
  * **dockerd:** Фоновый демон управления контейнерами, томами и сетями.
  * **Docker Engine API:** REST API для взаимодействия клиента с демоном.
  * **Container Runtime:** Среда исполнения контейнеров (`containerd` и `runc`).
  * **Docker Proxy:** Сетевой процесс маршрутизации и проброса портов между хостом и контейнером.

---

## 2. Развертывание сервера в AWS (EC2)

В сервисе **AWS EC2** развернут облачный виртуальный сервер:
* **AMI:** `Ubuntu Server 24.04 LTS`[cite: 4]
* **Тип инстанса:** `t2.micro` (Free Tier)[cite: 4]
* **Правила безопасности (Security Group):**
  * `22/TCP` (SSH) — удаленный доступ к серверу[cite: 4].
  * `80/TCP` (HTTP) — обслуживание входящего веб-трафика[cite: 4].
* **Проверка:** Выполнено успешное подключение к консоли через инструмент **EC2 Instance Connect**[cite: 4].
<img width="960" height="1040" alt="image" src="https://github.com/user-attachments/assets/ba82c3db-585d-4075-99eb-8d99c36d5d5f" />
<img width="959" height="1040" alt="image" src="https://github.com/user-attachments/assets/fae23b6d-73f4-430c-83f2-b5f831ae692b" />

---

## 3. Образы в Docker Hub

Собраны и опубликованы 3 образа в публичный реестр[cite: 4]:

| Название | Образ в реестре | Базовый образ | Команда запуска |
| :--- | :--- | :--- | :--- |
| Сайт 1 | `fhujniomc/site1-zarina:v1`[cite: 4] | `nginx:alpine` | `docker run -d -p 8081:80 fhujniomc/site1-zarina:v1`[cite: 4] |
| Сайт 2 | `fhujniomc/site2-zarina:v1`[cite: 4] | `nginx:alpine` | `docker run -d -p 8082:80 fhujniomc/site2-zarina:v1`[cite: 4] |
| Приложение | `fhujniomc/app-zarina:v1`[cite: 4] | `python:alpine` | `docker run --rm fhujniomc/app-zarina:v1`[cite: 4] |

### Команды сборки и отправки:
```bash
docker login -u fhujniomc
docker build -t fhujniomc/site1-zarina:v1 ./site1
docker build -t fhujniomc/site2-zarina:v1 ./site2
docker build -t fhujniomc/app-zarina:v1 ./app

docker push fhujniomc/site1-zarina:v1
docker push fhujniomc/site2-zarina:v1
docker push fhujniomc/app-zarina:v1
```
<img width="961" height="654" alt="image" src="https://github.com/user-attachments/assets/69c845b6-b943-4318-b620-7333792c9cd6" />

4. Оркестрация через Docker Compose
Для совместного запуска двух сервисов настроен файл docker-compose.yaml[cite: 4]:
web-app: Веб-сервер Nginx (внешний порт 8080 проброшен на 80).
database: СУБД PostgreSQL 15 с постоянным хранилищем данных в именованном томе pgdata.
Листинг docker-compose.yaml
```
version: '3.8'

services:
  web-app:
    image: nginx:alpine
    container_name: web_zarina
    ports:
      - "8080:80"
    restart: always

  database:
    image: postgres:15-alpine
    container_name: db_zarina
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password123
      POSTGRES_DB: main_db
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Управление сервисами:
Запуск всех служб в фоне:

Bash
docker compose up -d
Просмотр статуса работы:

Bash
docker compose ps
Остановка и удаление контейнеров:

Bash
docker compose down

<img width="953" height="894" alt="image" src="https://github.com/user-attachments/assets/62272a1c-414e-4b8b-b6dd-0bff2a5f4586" />

