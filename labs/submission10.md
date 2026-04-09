## Task 1 — Исследование artifact registries

### 1.1 Краткое введение

Artifact registry в облачной инфраструктуре нужен для хранения, публикации и распространения артефактов сборки: контейнерных образов, OCI-артефактов и пакетов для разных языков программирования. Если сравнивать AWS, GCP и Azure, то видно, что у Google Cloud эта зона выглядит более цельной: Artifact Registry задуман как единая точка хранения для разных типов артефактов. В AWS и Azure контейнерные артефакты и обычные пакеты чаще разведены по разным сервисам. ([Google Cloud Documentation][1])

### 1.2 Основные сервисы у разных облачных провайдеров

#### AWS

В AWS нельзя выделить один полностью универсальный artifact registry для всех сценариев. На практике здесь используются два основных сервиса.

**Amazon ECR** — это managed registry для хранения контейнерных образов и связанных артефактов. По официальной документации AWS, ECR предназначен для хранения, публикации и развертывания container software, поддерживает secure sharing, интеграцию с AWS-средой и модель оплаты без предварительных обязательств, где основные расходы связаны с хранением данных и сетевой передачей. Для production-сценариев ECR удобен, когда команда разворачивает образы через ECS, EKS или другие AWS-сервисы. ([Amazon Web Services, Inc.][2])

**AWS CodeArtifact** — это отдельный сервис для пакетов приложений. Он описывается AWS как managed artifact repository для хранения и совместного использования software packages. CodeArtifact работает с популярными менеджерами пакетов и поддерживает форматы npm, PyPI, Maven, NuGet, Swift, Ruby, Cargo и generic packages. На мой взгляд, его главный плюс в том, что он закрывает задачу внутренних зависимостей и централизованного контроля библиотек внутри команды или компании. ([AWS Documentation][3])

#### GCP

В Google Cloud основным сервисом является **Artifact Registry**. По официальной документации Google, это централизованное хранилище для artifacts и build dependencies внутри экосистемы Google Cloud. Сервис поддерживает Docker и OCI-артефакты, Helm charts в OCI-формате, а также Maven, npm, Python, Go и другие поддерживаемые форматы. Кроме обычных репозиториев, у него есть remote repositories и virtual repositories, что удобно для кэширования внешних источников и объединения нескольких источников в одну точку доступа. Также Google отдельно документирует vulnerability scanning для Artifact Registry и отдельную тарификацию на хранение, трафик и сканирование. ([Google Cloud Documentation][1])

#### Azure

В Azure, как и в AWS, логика тоже разделяется на два сервиса.

**Azure Container Registry (ACR)** используется для хранения container images и связанных artifacts. Microsoft описывает ACR как managed registry для хранения и управления container images и related artifacts. Для ACR важны такие возможности, как интеграция с Azure-сервисами, работа с приватными registry, а также разные уровни сервиса — Basic, Standard и Premium, где Premium дает расширенные возможности, например geo-replication. Поэтому ACR хорошо подходит для production-контейнеров в Azure-native окружении. ([Microsoft Learn][4])

**Azure Artifacts** — это сервис для пакетных зависимостей в Azure DevOps. Официальная документация указывает, что один feed может содержать несколько типов пакетов: npm, NuGet, Maven, Python, Cargo и Universal Packages. Кроме того, Azure Artifacts поддерживает upstream sources, то есть умеет сохранять и проксировать пакеты из публичных registry внутри собственного feed. Это удобно, когда нужно контролировать зависимости и уменьшить зависимость от внешних репозиториев. По цене Azure DevOps дает бесплатный объем хранения до 2 GiB, после чего включается помегабайтная тарификация. ([Microsoft Learn][5])

### 1.3 Сравнительная таблица

Ниже я свела основные различия в одну таблицу. Она основана на официальной документации AWS, Google Cloud и Microsoft. ([Amazon Web Services, Inc.][2])

| Облако | Основной сервис / сервисы                  | Что поддерживается                                                                                                          | Основные сильные стороны                                                                            | Интеграции                                     | Базовая модель цены                                                           | Когда особенно подходит                                |
| ------ | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------ |
| AWS    | Amazon ECR + AWS CodeArtifact              | ECR: Docker/OCI-образы и совместимые артефакты; CodeArtifact: npm, PyPI, Maven, NuGet, Swift, Ruby, Cargo, generic packages | Хорошая интеграция с AWS, IAM, хранение контейнеров отдельно от пакетов, managed package repository | ECS, EKS, EC2, IAM и другие AWS-сервисы        | ECR: хранение + трафик; CodeArtifact: хранение, запросы и исходящий трафик    | Когда инфраструктура уже глубоко завязана на AWS       |
| GCP    | Artifact Registry                          | Docker, OCI, Helm OCI, Maven, npm, Python, Go, remote и virtual repositories                                                | Единая модель сервиса, централизованное хранение, vulnerability scanning, remote/virtual repos      | Cloud Build, GKE, Cloud Run, IAM               | Хранение + трафик; scanning оплачивается отдельно                             | Когда нужен единый слой хранения контейнеров и пакетов |
| Azure  | Azure Container Registry + Azure Artifacts | ACR: Docker/OCI и related artifacts; Azure Artifacts: npm, NuGet, Maven, Python, Cargo, Universal Packages                  | Geo-replication, Azure DevOps feeds, upstream sources, strong Azure integration                     | AKS, App Service, Azure ML, Azure DevOps и др. | ACR: тарифные SKU; Azure Artifacts: 2 GiB бесплатно, затем оплата за хранение | Когда команда работает внутри Azure и Azure DevOps     |

### 1.4 Какой registry я бы выбрала для multi-cloud стратегии

Если рассматривать именно **multi-cloud стратегию**, я бы выбрала **Google Artifact Registry**.

Мой главный аргумент в том, что у GCP эта часть выглядит наиболее цельной. Один сервис закрывает и контейнерные артефакты, и несколько популярных пакетных экосистем, плюс дает remote и virtual repositories. В AWS и Azure решения более раздроблены: для контейнеров один сервис, для пакетов другой. Это не делает их плохими, но архитектурно такая схема сложнее, если хочется один понятный центр управления артефактами. Мой вывод здесь основан на том, как сами провайдеры описывают свои сервисы и поддерживаемые форматы. ([Google Cloud Documentation][1])

Кроме того, Artifact Registry хорошо встраивается в типичный cloud-native pipeline внутри Google Cloud: Cloud Build, GKE, Cloud Run, IAM и Artifact Analysis. Поэтому для команды, которой важны единые правила доступа, единый способ публикации артефактов и более простая структура CI/CD, этот вариант кажется мне самым удобным. При этом я считаю, что для контейнеров как таковых и Amazon ECR, и Azure Container Registry тоже являются очень сильными решениями, особенно если компания уже полностью сидит на AWS или Azure. ([Google Cloud Documentation][1])

---

## Task 2 — Исследование serverless computing platforms

### 2.1 Основные serverless-платформы у разных провайдеров

#### AWS

Главный serverless compute-сервис в AWS — это **AWS Lambda**. Официальная документация AWS описывает Lambda как сервис, который запускает код по событиям и автоматически управляет вычислительными ресурсами. Lambda поддерживает managed runtimes, а также deployment в виде `.zip`-пакетов и container images. Сервис хорошо ложится на event-driven архитектуру: его удобно использовать с HTTP-вызовами, очередями, потоками событий и другими источниками событий AWS. Максимальный timeout стандартной функции составляет 900 секунд, то есть 15 минут. Оплата идет по числу запросов и времени выполнения. ([AWS Documentation][6])

Если говорить о производительности, то у Lambda остается проблема cold start, особенно для интерактивных сценариев. AWS предлагает несколько механизмов снижения задержек, в частности provisioned concurrency и SnapStart. В официальной документации прямо указано, что provisioned concurrency держит прединициализированные execution environments и ориентирован на уменьшение cold start latencies, а SnapStart позволяет заметно сократить время запуска. Поэтому Lambda особенно хорошо подходит для event-driven automation, интеграций, обработки сообщений и сравнительно небольших API. ([AWS Documentation][7])

#### GCP

В Google Cloud основная serverless-платформа — это **Cloud Run**, а для function-oriented сценариев используются **Cloud Run functions**. По документации Google, Cloud Run — это fully managed application platform, на которой можно запускать код, функцию или контейнер. Главное преимущество Cloud Run в том, что он не привязывает разработчика к строго функции как сущности: если приложение можно упаковать в контейнер, его можно развернуть в Cloud Run. Для популярных языков Google также поддерживает source-based deployment. Кроме обычных HTTP-сервисов, Cloud Run умеет работать в режиме services, jobs и worker pools. ([Google Cloud Documentation][8])

С точки зрения оплаты и ограничений Cloud Run выглядит очень гибко. Google указывает, что для сервисов действует pay-per-use модель с гранулярностью 100 мс, а стандартный timeout запроса равен 5 минутам и может быть увеличен до 60 минут. Для Cloud Run functions Google отдельно пишет про лимиты до 60 минут для HTTP-triggered функций и до 9 минут для event-driven функций. Масштабирование до нуля тоже поддерживается, но при scale-to-zero возможна дополнительная задержка старта, поэтому Google рекомендует minimum instances и startup CPU boost для уменьшения latency. На мой взгляд, это делает Cloud Run особенно удобным для HTTP API, микросервисов и backend-приложений. ([Google Cloud Documentation][8])

#### Azure

В Azure основными serverless-решениями я бы выделила **Azure Functions** и **Azure Container Apps**.

**Azure Functions** — это serverless compute-платформа для event-driven кода. Microsoft пишет, что она позволяет строить приложения с меньшими инфраструктурными затратами и поддерживает типичные модели триггеров и bindings. Официально поддерживаются C#, Java, JavaScript, PowerShell и Python, а через custom handlers можно использовать и другие языки, например Go или Rust-подобные сценарии. Для HTTP API Azure Functions тоже подходит, потому что HTTP trigger прямо заявлен как стандартный сценарий. Но здесь есть важное ограничение: независимо от настроек timeout, HTTP-triggered функция должна ответить максимум за 230 секунд из-за idle timeout Azure Load Balancer. ([Microsoft Learn][9])

По модели оплаты и масштабированию Azure Functions зависит от выбранного hosting plan. В Consumption-плане оплата идет только за реально использованные compute resources во время работы функции. В Premium-плане execution charge отсутствует, но оплата идет за выделенные core seconds и memory, а также улучшается ситуация с холодным стартом. Microsoft отдельно отмечает, что Flex Consumption сейчас является рекомендуемым serverless hosting plan для Azure Functions. ([Microsoft Learn][10])

**Azure Container Apps** — это serverless-платформа для контейнерных приложений. Microsoft описывает ее как способ запускать containerized applications без лишнего управления инфраструктурой. Container Apps особенно хорошо подходит для микросервисов, background workers и job-сценариев. Масштабирование здесь строится на KEDA, причем официальная документация отдельно подчеркивает поддержку event-driven scaling и jobs. Поэтому Container Apps выглядит как более “сервисный” и контейнерный вариант по сравнению с классическими Azure Functions. ([Microsoft Learn][11])

### 2.2 Сравнительная таблица

Ниже я свела основные различия serverless-платформ в одну таблицу. ([AWS Documentation][6])

| Провайдер | Основная serverless-платформа         | Поддерживаемые языки / runtimes                                                                               | Модель выполнения                                                | Cold start и масштабирование                                           | Оплата                                                            | Ограничения по времени                                                                  | Типичные сценарии                                             |
| --------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| AWS       | AWS Lambda                            | Managed runtimes, deployment через zip и container images                                                     | Event-driven, HTTP, очереди, streams, другие event sources       | Есть cold start; уменьшается через provisioned concurrency и SnapStart | По запросам и длительности выполнения                             | До 15 минут                                                                             | Event-driven automation, интеграции, обработка сообщений, API |
| GCP       | Cloud Run, Cloud Run functions        | Любой язык через контейнер; для популярных языков есть source-based deployment                                | HTTP services, jobs, worker pools, HTTP и event-driven functions | Scale to zero, minimum instances, startup CPU boost                    | Pay-per-use, гранулярность 100 мс                                 | Services: до 60 минут; functions: до 60 мин HTTP и до 9 мин event-driven                | REST API, микросервисы, web backends, jobs                    |
| Azure     | Azure Functions, Azure Container Apps | Functions: C#, Java, JavaScript, PowerShell, Python, custom handlers; Container Apps: любой язык в контейнере | Triggers and bindings, HTTP, event-driven containers, jobs       | Зависит от плана; в Container Apps масштабирование через KEDA          | Functions: зависит от hosting plan; Container Apps: pay-as-you-go | Для HTTP-triggered Functions ответ ограничен 230 сек; остальные лимиты зависят от плана | Serverless API, автоматизация, workflow, контейнерные сервисы |

### Какую platform я бы выбрала для REST API backend

Если речь идет именно о **REST API backend**, я бы выбрала **Google Cloud Run**.

Мне этот вариант кажется самым удобным, потому что Cloud Run ближе к привычной модели обычного backend-сервиса, чем классический FaaS. Можно взять приложение на любом языке, упаковать его в контейнер и развернуть как нормальный HTTP-сервис с autoscaling, HTTPS endpoint и оплатой по факту использования. При этом timeout можно увеличить до 60 минут, что заметно гибче, чем 15 минут у Lambda и особенно чем HTTP-ограничение в 230 секунд у Azure Functions. Для API и микросервисов это очень серьезный плюс. ([Google Cloud Documentation][8])

AWS Lambda я бы поставила на второе место, если backend сильно зависит от событийной модели AWS и интеграций внутри экосистемы AWS. Azure Functions тоже является сильным решением, особенно если проект уже строится вокруг Azure. Но если выбирать универсальный и удобный именно для REST API вариант, то Cloud Run, на мой взгляд, выглядит наиболее естественно: он дает serverless-масштабирование, но не заставляет мыслить приложение только как набор коротких функций. ([AWS Documentation][12])

### Рефлексия: основные плюсы и минусы serverless computing

Главные **плюсы** serverless-подхода — это снижение операционной нагрузки, автоматическое масштабирование и модель оплаты по фактическому использованию ресурсов. Все три провайдера подчеркивают, что разработчику не нужно напрямую управлять серверами, а платформа сама занимается запуском, масштабированием и инфраструктурной частью. Для API с непредсказуемой нагрузкой, событийных систем и автоматизации это действительно очень удобно. ([AWS Documentation][6])

Главные **минусы** — это cold starts, ограничения по времени выполнения, а также зависимость от конкретной платформы и ее модели. У Lambda есть ограничения по timeout и проблема холодного старта, у Cloud Run при scale-to-zero тоже может быть дополнительная задержка, а у Azure Functions есть достаточно жесткий лимит для HTTP-ответа. Поэтому serverless отлично подходит не для всего подряд, а прежде всего для stateless-приложений, событийных обработчиков, API и фоновых задач, где эти ограничения не становятся критичными. ([AWS Documentation][13])