# RemindersPlus - Kubernetes Конфигурација (GitOps)

Овој репозиториум служи како единствен извор на вистина (*single source of truth*) за сите Kubernetes манифести потребни за распоредување на **RemindersPlus** апликацијата.

Репозиториумот е подготвен за следење од **Argo CD** или друга GitOps алатка, овозможувајќи целосно автоматизиран и декларативен процес на испорака. Дополнително, користи **Kustomize** за управување со конфигурациите и лесно ажурирање на верзиите на Docker сликите.

| Главно Репо (Апликација) | Конфигурациско Репо (Kubernetes) | Docker Hub |
| :--- | :--- | :--- |
| [`AndrejT03/KIII_proekt`](https://github.com/AndrejT03/KIII_proekt) | [`AndrejT03/KIII_proekt-config`](https://github.com/AndrejT03/KIII_proekt-config) | [`andrejt12`](https://hub.docker.com/u/andrejt12) |

***

## Преглед

Овој репозиториум ги содржи сите YAML манифести кои ја опишуваат саканата состојба на апликацијата во Kubernetes кластер. Сите ресурси се распоредуваат во `reminders` именскиот простор (namespace).

## Структура на Манифестите

-   **`namespace.yaml`**: Го креира `reminders` именскиот простор, изолирајќи ги сите ресурси на апликацијата.

-   **`config-secrets.yaml`**:
    -   **`ConfigMap` (`reminders-config`)**: Ги дефинира не-чувствителните конфигурациски податоци (име на база, хост на базата: `reminders-db-service`).
    -   **`Secret` (`reminders-secrets`)**: Ги чува чувствителните податоци (корисничко име и лозинка за базата) во base64 енкодиран формат.

-   **`postgres-deployment.yaml`**:
    -   **`StatefulSet` (`reminders-db`)**: Го дефинира `StatefulSet`-от за PostgreSQL базата, што е најдобра пракса за stateful апликации. Користи `PersistentVolumeClaim` за да обезбеди трајно складирање на податоците од 1Gi.
    -   **`Service` (`reminders-db-service`)**: Создава "headless" сервис (`clusterIP: None`) кој овозможува стабилен и предвидлив DNS запис за секој под од `StatefulSet`-от.

-   **`backend-deployment.yaml`**:
    -   **`Deployment` (`reminders-backend`)**: Го дефинира `Deployment`-от за Spring Boot апликацијата.
    -   **`Service` (`reminders-backend`)**: Создава `ClusterIP` сервис кој овозможува пристап до API-то внатре во кластерот.

-   **`frontend-deployment.yaml`**:
    -   **`Deployment` (`reminders-frontend`)**: Го дефинира `Deployment`-от за React/Nginx апликацијата.
    -   **`Service` (`reminders-frontend-service`)**: Создава сервис од тип `NodePort`, кој ја изложува апликацијата надвор од кластерот за лесен пристап.

-   **`ingress.yaml`**:
    -   **`Ingress` (`reminders-ingress`)**: Го дефинира начинот на рутирање на сообраќајот. Сите барања кои почнуваат со `/api` се пренасочуваат кон `reminders-backend` сервисот, додека сите останати барања (`/`) одат кон `reminders-frontend-service`.

-   **`kustomization.yml`**:
    -   Ова е централниот фајл за Kustomize. Тој ги наведува сите ресурси што треба да се распоредат и овозможува динамично ажурирање на ознаките (tags) на Docker сликите. CI/CD процесот го користи овој фајл за автоматски да ги ажурира верзиите на сликите.
