
<h1 align="center">DevOps Engineer</h1>

<p align="center">
  Linux · Docker · Kubernetes · CI/CD · Infrastructure as Code · Networking
</p>

<p align="center">
  <a href="https://t.me/knshnxxx"><img alt="Telegram" src="https://img.shields.io/badge/telegram-@knshnxxx-2AABEE?style=flat-square&logo=telegram&logoColor=white"></a>
  <a href="https://habr.com/ru/sandbox/297958/"><img alt="Habr" src="https://img.shields.io/badge/Habr-статья-77A2B6?style=flat-square&logo=habr&logoColor=white"></a>
  <img alt="Status" src="https://img.shields.io/badge/open_to-small_projects_&_infra_tasks-2ea44f?style=flat-square">
</p>

---

## Hi there👋, I'm Vyacheslav

Практикую DevOps — строю реальную инфраструктуру, а не туториальные todo-list'ы.
Сюда пришел из другой плоскости (backend/desktop-эксперименты), и последние месяцы плотно копаю Kubernetes и мониторинг. 

---

## Стек

**Containers & Orchestration**
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white)
![K3s](https://img.shields.io/badge/K3s-FFC61C?style=flat-square&logo=k3s&logoColor=black)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=flat-square&logo=helm&logoColor=white)

**CI/CD**
![GitLab CI](https://img.shields.io/badge/GitLab_CI-FC6D26?style=flat-square&logo=gitlab&logoColor=white)
![Kaniko](https://img.shields.io/badge/Kaniko-1E88E5?style=flat-square&logo=google&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)

**Infra & Cloud**
![Ubuntu](https://img.shields.io/badge/Ubuntu_24.04-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![Sber Cloud](https://img.shields.io/badge/Cloud.ru-21A038?style=flat-square)
![Yandex Cloud](https://img.shields.io/badge/Yandex_Cloud-5282FF?style=flat-square&logo=yandexcloud&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat-square&logo=terraform&logoColor=white)

**Networking & Reverse Proxy**
![Traefik](https://img.shields.io/badge/Traefik-24A1C1?style=flat-square&logo=traefikproxy&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat-square&logo=nginx&logoColor=white)
![cert-manager](https://img.shields.io/badge/cert--manager-326CE5?style=flat-square&logo=letsencrypt&logoColor=white)
![Xray](https://img.shields.io/badge/Xray_core-000?style=flat-square)

**Databases & Runtime**
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white)

---

## Что делаю руками

### CI/CD и сборка артефактов
- Пишу `.gitlab-ci.yml` со стадиями **lint → test → build → deploy**, с кэшем pip/npm и параллельными job'ами по фронту и бэку.
- Собираю Docker-образы через **Kaniko** — daemonless, в user-space, без `docker.sock` и `privileged: true`. Пайплайн стал чисто push-based, без единого SSH-ключа.
- Публикую в приватный GitLab Container Registry, стягиваю в кластер через `imagePullSecrets` (`docker-registry` секрет).
- **GitLab Runner** живёт **внутри самого кластера** — в отдельном namespace. 
- Деплой в prod. Никаких сюрпризов из feature-веток.

### Kubernetes (K3s, K8s, RKE2, kind)
- CoreDNS, Traefik и Local Path Provisioner из коробки, минимум оверхеда для стартового проекта. 
- **StatefulSet + PVC + Headless Service** для PostgreSQL — данные переживают смерть пода (проверено `kubectl delete pod postgres-0` под нагрузкой, база возвращается со всеми записями).
- **Traefik Ingress + Middleware `stripPrefix`** для роутинга `/api → backend`, `/ → frontend`. Отдельно наступил на `@cuberdns` вместо `@kubernetescrd` — теперь помню.
- **cert-manager + Let's Encrypt** (`ClusterIssuer letsencrypt-prod`). 
- **Zero-Downtime**: `RollingUpdate`, `readinessProbe` + `livenessProbe` на `/health`. 
- **Resource requests/limits** на всех подах (троттлинг подов)
- Metrics Server для базовой видимости в `k9s`/`kubectl top`.

### Базы и интеграция
- PostgreSQL как `StatefulSet` с `PVC`
- Креды инжектю через `Secret` + `secretKeyRef`, секрет создаётся идемпотентно из CI:
  ```bash
  kubectl create secret generic postgres-secrets \
    --from-literal=DB_USER="${DB_USER}" ... \
    --dry-run=client -o yaml | kubectl apply -f -
  ```

### Сети и VPN
Self-hosted VPN на **Xray core**, полностью боевой сетап:
- протоколы: **VLESS**, **Reality**, **XTLS-Vision**;
- транспорты: **gRPC**, **TCP**, **xhttp**;
- панель **3x-ui** для управления и учёта клиентов;

### Мониторинг и Observability (Prometheus + Grafana)
- Базовый стек в единой bridge-сети docker-compose/кубера: **Prometheus**, **Grafana**, **Node Exporter** (хост/железо), **cAdvisor** (контейнеры).
- **pull-модель**: `prometheus.yml`, `scrape_interval`, таргеты.
- **TSDB**: горячие сэмплы в RAM, **WAL** для защиты от потери при рестарте, упаковка в chunks.
- Пишу **PromQL**: агрегации `sum without(...)`, матчеры с regex и `!=`
- **архитектурные лимиты**: почему **high cardinality** (`user_id` в лейблах) экспоненциально раздувает индекс и роняет Prometheus по OOM; где заканчивается один инстанс и начинается **remote write** (напр. в VictoriaMetrics).

### Linux и облака
- Ubuntu на **Cloud.ru** и **Yandex Cloud**.
- Base hygiene: non-root deploy user, ключевой SSH, Security Groups на уровне облака, `systemd`-юниты, ротация логов, backups как отдельная задача (не «оно как-нибудь».).

---

## Мои проектики

| Проект | Что внутри |
|---|---|
| **[Hybrid Lab](https://github.com/vyacheslavwv/HybridLab/blob/main/README.md)** — full-cycle DevOps demo | GitLab CI (Kaniko) → private registry → K3s (Traefik + cert-manager + PVC + probes + limits) → HTTPS-домен. Frontend (React/TS) + Backend (FastAPI) + PostgreSQL. Прошёл 4 краш-теста: ZDD, kill app-pod, kill DB-pod, `wrk -c400` стресс. |
| **Self-hosted VPN** *(private)* | Xray core: VLESS + Reality + XTLS-Vision, gRPC/TCP, 3x-ui, remote admin с телефона. |
| **Monitoring Stack** *(work in progress)* | Prometheus + Grafana + Alertmanager поверх K3s, node/app exporters, алерты по CPU/памяти/пробам. |
| **Terraform Lab** *(planned)* | Yandex Cloud + Terraform: VPC, VM, security groups, ClusterIssuer через IaC. |
И другое
---

## Публикации

- 📝 **Habr** — «[Из песочницы Compose в боевой Kubernetes: как я построил отказоустойчивую архитектуру за 5 месяцев, изучая всё с нуля](https://habr.com/ru/sandbox/297958/)».
  Разбор перехода от `docker-compose` к single-node K3s: почему StatefulSet+PVC для БД, как Kaniko выкинул DinD, что ломается в связке Traefik+cert-manager, и четыре краш-теста поверх всего.

---

## Контакт

Открыт к небольшим проектам и техническим задачам по DevOps / инфраструктуре.
Telegram → **[@knshnxxx](https://t.me/knshnxxx)**

Почта → vyacheslav.hmm@gmail.com
