# Довідник DevOps

Цей посібник допоможе вам зрозуміти наш стек інфраструктури і те, як ми підтримуємо наші платформи. Хоча цей посібник не має вичерпних подробиць всіх операцій, його можна використовувати як довідник для розуміння систем.

Зв’яжіться з нами, якщо у вас виникнуть запитання, і ми з радістю роз’яснимо всі деталі.

## Керівництво з розгортання коду

Цей репозиторій постійно будується, тестується та розробляється на **окремих наборах інфраструктури (серверах, базах даних, CDN тощо)**.

Це передбачає три кроки, які потрібно виконувати послідовно:

1. Нові зміни (виправлення та функціональності) об’єднуються до нашої основної гілки (`main`) через запити на злиття.
2. Ці зміни проходять через ряд автоматизованих тестів.
3. Після проходження тестів ми випускаємо зміни (або оновлюємо їх, якщо потрібно) для розгортання у нашій інфраструктурі.

### Побудова бази коду: відображення гілок Git для розгортання

[`main`](https://github.com/freeCodeCamp/freeCodeCamp/tree/main) (гілка розробки за замовчуванням) зазвичай об’єднується до гілки [`prod-staging`](https://github.com/freeCodeCamp/freeCodeCamp/tree/prod-staging) раз на день та випускається в ізольовану інфраструктуру.

Це проміжний випуск для наших розробників і волонтерів. Він також відомий як «staging» або «beta».

Він ідентичний нашому робочому середовищу на `freeCodeCamp.org`, а також використовує окремі набори баз даних, сервери, вебпроксі тощо. Ця ізоляція дозволяє нам протестувати поточну розробку та функції у «виробничому» сценарії, не впливаючи на звичайних користувачів основних платформ freeCodeCamp.org.

Як тільки команда [`@freeCodeCamp/dev-team`](https://github.com/orgs/freeCodeCamp/teams/dev-team/members) задоволена змінами на проміжній платформі, ці зміни переносяться кожні декілька днів до гілки [`prod-current`](https://github.com/freeCodeCamp/freeCodeCamp/tree/prod-current).

Це кінцевий випуск, який переносить зміни до наших виробничих платформ на freeCodeCamp.org.

### Тестування змін: інтеграція та приймальне користувацьке тестування

Ми використовуємо різні рівні інтеграції та приймального тестування, щоб перевірити якість коду. Всі наші тести виконуються за допомогою такого програмного забезпечення, як [GitHub Actions CI](https://github.com/freeCodeCamp/freeCodeCamp/actions) та [Azure Pipelines](https://dev.azure.com/freeCodeCamp-org/freeCodeCamp).

Ми маємо модульні тести для тестування розв’язків завдань, API серверів та інтерфейсів користувача. Це допомагає протестувати інтеграцію різних компонентів.

> [!NOTE] Ми також в процесі написання тестів кінцевого користувача, що допоможе відтворити реальні сценарії, як-от оновлення електронної пошти або дзвінок до API чи сторонніх служб.

Ці тести допомагають запобігти повторенню проблем і гарантують, що ми не вводимо нову помилку під час роботи над іншою помилкою або функцією.

### Розгортання змін: надсилання змін до серверів

Ми налаштували безперервне програмне забезпечення доставки для надсилання змін до наших серверів розробки та виробництва.

Як тільки зміни відправлені в захищені гілки випуску, для гілки автоматично запускається конвеєр збірки. Конвеєри збірки відповідають за створення артефактів та їх зберігання в холодному сховищі для подальшого використання.

Конвеєр збірки працює для запуску відповідного конвеєра випуску, якщо він завершить успішний запуск. Конвеєри випуску відповідають за збір артефактів збірки, їх переміщення на сервери та запуск в експлуатацію.

Статуси збірок та випуски [доступні тут](#build-test-and-deployment-status).

## Запуск збірки, тесту та розгортання

Наразі лише команда розробників може надсилати зміни до виробничих гілок. Зміни до гілок `production-*` можна внести лише через швидке об’єднання до [`upstream`](https://github.com/freeCodeCamp/freeCodeCamp).

> [!NOTE] Найближчими днями ми вдосконалимо цей потік для кращого керування доступом і прозорості, щоб він здійснювався за допомогою запитів на злиття.

### Надсилання змін до проміжних застосунків

1. Налаштуйте віддалені гілки правильно.

   ```sh
   git remote -v
   ```

   **Результат:**

   ```
   origin   git@github.com:raisedadead/freeCodeCamp.git (fetch)
   origin   git@github.com:raisedadead/freeCodeCamp.git (push)
   upstream git@github.com:freeCodeCamp/freeCodeCamp.git (fetch)
   upstream git@github.com:freeCodeCamp/freeCodeCamp.git (push)
   ```

2. Переконайтеся, що гілка `main` чиста та синхронізована з головною гілкою.

   ```sh
   git checkout main
   git fetch --all --prune
   git reset --hard upstream/main
   ```

3. Переконайтеся, що GitHub CI передає гілку `main` до головної гілки.

   Тести [безперервної інтеграції](https://github.com/freeCodeCamp/freeCodeCamp/actions) повинні бути зеленими та проходити УСПІШНО для гілки `main`. Натисніть зелений прапорець біля хешу затвердження при перегляді коду гілки `main`.

    <details> <summary> Перевірка статусу на GitHub Actions (знімок екрана) </summary>
      <br>
      ![Перевірте статус збірки на GitHub Actions](https://raw.githubusercontent.com/freeCodeCamp/freeCodeCamp/main/docs/images/devops/github-actions.png)
    </details>

   Якщо у вас виникають проблеми, вам потрібно зупинитись і дослідити помилки.

4. Підтвердьте, що ви можете побудувати репозиторій локально.

   ```
   pnpm run clean-and-develop
   ```

5. Перенесіть зміни з `main` до `prod-staging` через швидке об’єднання

   ```
   git checkout prod-staging
   git merge main
   git push upstream
   ```

   > [!NOTE] Ви не зможете примусово надсилати зміни і якщо ви переписали історію, ці команди призведуть до помилки.
   > 
   > В такому випадку ви, можливо, зробили щось неправильно та повинні просто почати спочатку.

Попередні кроки автоматично запустять конвеєр збірки для гілки `prod-staging`. Як тільки збірка завершиться, артефакти будуть збережені як файли `.zip` в холодному сховищі для використання пізніше.

Конвеєр випуску запускається автоматично, коли новий артефакт доступний з підключеного конвеєра збірки. Для проміжних платформ цей процес не передбачає затвердження вручну, а артефакти надсилаються до клієнтських CDN та серверів API.

### Надсилання змін до виробничих застосунків

Процес здебільшого такий самий, як і для проміжних платформ, але містить декілька додаткових пунктів. Це потрібно, щоб переконатися, що ми нічого не порушуємо на freeCodeCamp.org, що можуть побачити сотні користувачів, які використовують його.

| НЕ виконуйте ці команди, якщо не переконались, що все працює на проміжній платформі. Не пропускайте жодних тестів, перш ніж продовжити процес. |
|:---------------------------------------------------------------------------------------------------------------------------------------------- |
|                                                                                                                                                |

1. Переконайтеся, що гілка `prod-staging` чиста та синхронізована з головною гілкою.

   ```sh
   git checkout prod-staging
   git fetch --all --prune
   git reset --hard upstream/prod-staging
   ```

2. Перенесіть зміни з `prod-staging` до `prod-current` через швидке об’єднання

   ```
   git checkout prod-current
   git merge prod-staging
   git push upstream
   ```

   > [!NOTE] Ви не зможете примусово надсилати зміни і якщо ви переписали історію, ці команди призведуть до помилки.
   > 
   > В такому випадку ви, можливо, зробили щось неправильно та повинні просто почати спочатку.

Попередні кроки автоматично запустять конвеєр збірки для гілки `prod-current`. Як тільки артефакт збірки готовий, він запустить конвеєр випуску.

**Додаткові кроки для персоналу**

Як тільки запуск випуску ініційовано, команда розробників отримає автоматизований лист про ручне втручання. Вони можуть _затвердити_ або _відхилити_ запуск випуску.

Якщо зміни працюють і вони протестовані на проміжній платформі, їх можна затвердити. Підтвердження потрібно надати протягом 4 годин після активації релізу перед автоматичним відхиленням. Персонал може повторно запустити відхилені запуски або чекати наступного циклу.

Для персоналу:

| Перевірте свою електронну пошту для прямого посилання або [перейдіть до панелі випусків](https://dev.azure.com/freeCodeCamp-org/freeCodeCamp/_release) після того, як завершиться збірка. |
|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                                                                                                                                                                                           |

Як тільки хтось з персоналу затвердить випуск, конвеєр відправить зміни до виробничої CDN та серверів API freeCodeCamp.org.

## Збірка, тест та статус розгортання

Ось поточний тест, збірка та статус розгортання кодової бази.

| Гілка                                                                            | Модульні тести                                                                                                                                                                                                                   | Інтеграційні тести                                                                                                                                                                                                       | Збірки та розгортання                                                                                                             |
|:-------------------------------------------------------------------------------- |:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |:--------------------------------------------------------------------------------------------------------------------------------- |
| [`main`](https://github.com/freeCodeCamp/freeCodeCamp/tree/main)                 | [![Node.js CI](https://github.com/freeCodeCamp/freeCodeCamp/workflows/Node.js%20CI/badge.svg?branch=main)](https://github.com/freeCodeCamp/freeCodeCamp/actions?query=workflow%3A%22Node.js+CI%22)                               | [![Cypress E2E Tests](https://img.shields.io/endpoint?url=https://dashboard.cypress.io/badge/simple/ke77ns/main&style=flat&logo=cypress)](https://dashboard.cypress.io/projects/ke77ns/analytics/runs-over-time)         | -                                                                                                                                 |
| [`prod-staging`](https://github.com/freeCodeCamp/freeCodeCamp/tree/prod-staging) | [![Node.js CI](https://github.com/freeCodeCamp/freeCodeCamp/workflows/Node.js%20CI/badge.svg?branch=prod-staging)](https://github.com/freeCodeCamp/freeCodeCamp/actions?query=workflow%3A%22Node.js+CI%22+branch%3Aprod-staging) | [![Cypress E2E Tests](https://img.shields.io/endpoint?url=https://dashboard.cypress.io/badge/simple/ke77ns/prod-staging&style=flat&logo=cypress)](https://dashboard.cypress.io/projects/ke77ns/analytics/runs-over-time) | [Azure Pipelines](https://dev.azure.com/freeCodeCamp-org/freeCodeCamp/_dashboards/dashboard/d59f36b9-434a-482d-8dbd-d006b71713d4) |
| [`prod-current`](https://github.com/freeCodeCamp/freeCodeCamp/tree/prod-staging) | [![Node.js CI](https://github.com/freeCodeCamp/freeCodeCamp/workflows/Node.js%20CI/badge.svg?branch=prod-current)](https://github.com/freeCodeCamp/freeCodeCamp/actions?query=workflow%3A%22Node.js+CI%22+branch%3Aprod-current) | [![Cypress E2E Tests](https://img.shields.io/endpoint?url=https://dashboard.cypress.io/badge/simple/ke77ns/prod-current&style=flat&logo=cypress)](https://dashboard.cypress.io/projects/ke77ns/analytics/runs-over-time) | [Azure Pipelines](https://dev.azure.com/freeCodeCamp-org/freeCodeCamp/_dashboards/dashboard/d59f36b9-434a-482d-8dbd-d006b71713d4) |
| `prod-next` (експериментальне, майбутнє)                                         | -                                                                                                                                                                                                                                | -                                                                                                                                                                                                                        | -                                                                                                                                 |

## Ранній доступ та бета-тестування

Ми вітаємо вас протестувати ці випуски в режимі **публічного бета-тестування** та отримати ранній доступ до майбутніх функціональностей платформи. Іноді ці функціональності/зміни також називають **наступними, бета, проміжними** тощо.

Ваші внески у вигляді зворотного зв’язку та повідомлень про проблеми допоможуть нам зробити платформу `freeCodeCamp.org` більш **стійкою**, **послідовною** та **стабільною** для кожного.

Ми вдячні за повідомлення про помилки, з якими ви стикаєтесь. Це допомагає покращити freeCodeCamp.org. Ви круті!

### Ідентифікація майбутньої версії платформ

Наразі публічне бета-тестування доступне на:

| Програма | Мова       | URL                                      |
|:-------- |:---------- |:---------------------------------------- |
| Навчання | Англійська | <https://www.freecodecamp.dev>           |
|          | Іспанська  | <https://www.freecodecamp.dev/espanol>   |
|          | Китайська  | <https://www.freecodecamp.dev/chinese>   |
| Новини   | Англійська | <https://www.freecodecamp.dev/news>      |
| Форум    | Англійська | <https://forum.freecodecamp.dev>         |
|          | Китайська  | <https://freecodecamp.dev/chinese/forum> |
| API      | -          | `https://api.freecodecamp.dev`           |

> [!NOTE] Назва домену відрізняється від **`freeCodeCamp.org`**. Це є навмисним, щоб відвернути індексацію пошукової системи і запобігти непорозумінь для користувачів платформи.
> 
> Наведений вище список не є повним списком усіх програм, які ми забезпечуємо. Також не всі мовні варіанти розгорнуті у проміжній версії, щоб зберегти ресурси.

### Ідентифікація поточної версії платформ

**Поточна версія платформи завжди доступна на [`freeCodeCamp.org`](https://www.freecodecamp.org).**

Команда розробників об’єднує зміни з гілки `prod-staging` до `prod-current`, коли вони випускають зміни. Верхнім затвердженням має бути те, що ви побачите на сайті.

Ви можете ідентифікувати точну розгорнуту версію, відвідавши збірку та журнали розгортання, доступні в розділі статусу. Або ж ви можете написати нам у [чаті](https://discord.gg/PRyKn3Vbay) для підтвердження.

### Відомі обмеження

Існують деякі відомі обмеження і компроміси при використанні бета-версії платформи.

- **Всі дані та особистий прогрес на бета-платформі НЕ будуть збережені чи перенесені до робочої версії**

  **Користувачі бета-версії матимуть окремий обліковий запис.** Бета-версія використовує іншу фізичну базу даних. Це дає можливість запобігти випадковій втраті даних чи змін. В разі потреби команда розробників може очистити базу даних цієї бета-версії.

- **Бета-платформи не надають жодних гарантій щодо працездатності та надійності**

  Очікується, що розгортання відбуватиметься часто та у швидких ітераціях, іноді декілька разів на день. В результаті іноді в бета-версії будуть неочікувані простої чи неробочі функціональності.

- **Щоб забезпечити ефективність виправлення, радимо не надсилати регулярних користувачів на цей вебсайт з метою перевірки.**

  Метою бета-сайту завжди було вдосконалити локальну розробку та тестування, нічого іншого. Це не майбутня версія платформи, а звичайний перегляд того, над чим ми працюємо.

- **Сторінка входу може відрізнятись від робочої версії**

  Ми використовуємо тестовий клієнт для freeCodeCamp.dev на Auth0, тому не можемо встановити власний домен. Це дозволяє зробити так, що всі зворотні виклики й сторінки авторизації з’являтимуться на домені за замовчуванням: `https://freecodecamp-dev.auth0.com/`. Це не впливає на функціональність та є максимально приближеним до робочої версії.

## Повідомлення про проблеми та зворотний зв’язок

Будь ласка, створюйте нові завдання для обговорення чи повідомлення про помилки.

Ви можете надіслати електронний лист на `dev[at]freecodecamp.org` у разі виникнення будь-яких запитів. Про вразливість безпеки потрібно повідомляти на `security[at]freecodecamp.org`, а не публічному журналі чи форумі.

## Керівництво з обслуговування сервера

> [!WARNING]
> 
> 1. Це керівництво стосується **лише персоналу freeCodeCamp**.
> 2. У цих інструкціях вказано не всю інформацію, тому будьте обережними.

Можливо, як персоналу вам надано доступ до наших хмарних провайдерів (Azure, Digital Ocean тощо).

Ось кілька зручних команд, які можна використовувати для роботи на віртуальних машинах (VM), наприклад, для оновлення технічного обслуговування або для загального ведення.

## Отримайте список віртуальних машин

> [!NOTE] Ви можете мати SSH-доступ до віртуальних машин, однак його недостатньо для отримання списку віртуальних машин. Вам також потрібен доступ до хмарних порталів.

### Azure

Встановіть Azure CLI `az`: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli

> **(Одноразово) Встановіть на macOS за допомогою [`homebrew`](https://brew.sh):**

```
brew install azure-cli
```

> **(Одноразовий) Вхід:**

```
az login
```

> **Отримайте список назв віртуальних машин та IP-адрес:**

```
az vm list-ip-addresses --output table
```

### Digital Ocean

Встановіть Digital Ocean CLI `doctl`: https://github.com/digitalocean/doctl#installing-doctl

> **(Одноразово) Встановіть на macOS за допомогою [`homebrew`](https://brew.sh):**

```
brew install doctl
```

> **(Одноразовий) Вхід:**

Автентифікація та зміна контексту: https://github.com/digitalocean/doctl#authenticating-with-digitalocean

```
doctl auth init
```

> **Отримайте список назв віртуальних машин та IP-адрес:**

```
doctl compute droplet list --format "ID,Name,PublicIPv4"
```

## Spin New Resources

We are working on creating our IaC setup, and while that is in works you can use the Azure portal or the Azure CLI to spin new virtual machines and other resources.

> [!TIP] No matter your choice of spinning resources, we have a few [handy cloud-init config files](https://github.com/freeCodeCamp/infra/tree/main/cloud-init) to help you do some of the basic provisioning like installing docker or adding SSH keys, etc.

## Зберігайте віртуальні машини в актуальному стані

Ви повинні встановлювати на віртуальну машину останні оновлення. Це гарантує, що віртуальна машина залатана найновішими виправленнями безпеки.

> [!WARNING] Перед запуском цих команд:
> 
> - Переконайтеся, що віртуальну машину повністю підготовлено та не виконується жодних кроків після встановлення.
> - Якщо ви оновлюєте пакети на віртуальній машині, яка вже обслуговує програму, переконайтеся, що програма зупинена/збережена. Оновлення пакетів спричинить стрибки у навантаженні мережі, памʼяті та/або центрального процесора, що призведе до відключення запущених програм.

Оновіть інформацію пакетів

```console
sudo apt update
```

Оновіть встановлені пакети

```console
sudo apt upgrade -y
```

Очистьте невикористані пакети

```console
sudo apt autoremove -y
```

## Працюйте над вебсерверами (проксі)

Ми запускаємо збалансовані екземпляри (Azure Load Balancer) для наших вебсерверів. На цих серверах працює NGINX, який перенаправляє весь трафік до freeCodeCamp.org із різних застосунків, що працюють на власних інфраструктурах.

Конфігурація NGINX доступна у [цьому репозиторію](https://github.com/freeCodeCamp/nginx-config).

### Перше завантаження

Підготовка віртуальних машин за допомогою коду

1. Завантажте і налаштуйте NGINX з репозиторію.

   ```console
   sudo su

   cd /var/www/html
   git clone https://github.com/freeCodeCamp/error-pages

   cd /etc/
   rm -rf nginx
   git clone https://github.com/freeCodeCamp/nginx-config nginx

   cd /etc/nginx
   ```

2. Встановіть сертифікати Cloudflare і конфігурацію головної програми.

   Отримайте оригінальні сертифікати зі сховища безпеки та встановіть їх в потрібних місцях.

   **АБО**

   Перемістіть наявні сертифікати:

   ```console
   # Local
   scp -r username@source-server-public-ip:/etc/nginx/ssl ./
   scp -pr ./ssl username@target-server-public-ip:/tmp/

   # Remote
   rm -rf ./ssl
   mv /tmp/ssl ./
   ```

   Оновіть головні конфігурації:

   ```console
   vi configs/upstreams.conf
   ```

   Додайте/оновіть вихідні IP-адреси програми.

3. Налаштуйте мережу та брандмауери.

   Налаштуйте брандмауери Azure та `ufw` відповідно до доступу до вихідних адрес.

4. Додайте віртуальну машину до балансувальника навантаження.

   За потреби налаштуйте та додайте правила до балансувальника навантаження. Можливо, також знадобиться додати віртуальну машину до пулу балансувальника навантаження.

### Журналювання та моніторинг

1. Перевірте стан служби NGINX за допомогою наступної команди:

   ```console
   sudo systemctl status nginx
   ```

2. Журналювання та моніторинг служб доступні на:

   NGINX Amplify: [https://amplify.nginx.com]('https://amplify.nginx.com') — наша поточна базова панель моніторингу. Ми працюємо над детальнішими показниками для кращого спостереження

### Оновлення екземплярів (обслуговування)

Налаштуйте зміни в екземплярах NGINX, які зберігаються на GitHub. Їх потрібно розгорнути в кожному екземплярі:

1. SSH в екземпляр і введіть sudo

```console
sudo su
```

2. Отримайте останній код конфігурації.

```console
cd /etc/nginx
git fetch --all --prune
git reset --hard origin/main
```

3. Протестуйте та повторно завантажте конфігурації [з Signals](https://docs.nginx.com/nginx/admin-guide/basic-functionality/runtime-control/#controlling-nginx).

```console
nginx -t
nginx -s reload
```

## Робота на екземплярах API

1. Встановіть інструменти збірки для бінарних файлів node (`node-gyp`) тощо.

```console
sudo apt install build-essential
```

### Перше завантаження

Підготовка віртуальних машин за допомогою коду

1. Встановіть Node LTS.

2. Встановіть pnpm глобально.

```console
npm install -g pnpm
```

3. Клонуйте freeCodeCamp, налаштуйте середовище та ключі.

```console
git clone https://github.com/freeCodeCamp/freeCodeCamp.git
cd freeCodeCamp
git checkout prod-current # or any other branch to be deployed
```

4. Створіть `.env` із безпечного сховища облікових даних.

5. Встановіть залежності

```console
pnpm install
```

6. Налаштуйте pm2 `logrotate` та запустіть під час завантаження

```console
pnpm pm2 install pm2-logrotate
pnpm pm2 startup
```

7. Побудуйте сервер

```console
pnpm prebuild && pnpm build:curriculum && pnpm build:server
```

8.  Запустіть екземпляри

```console
pnpm start:server
```

### Журналювання та моніторинг

```console
pnpm pm2 logs
```

```console
pnpm pm2 monit
```

### Оновлення екземплярів (обслуговування)

Зміни коду потрібно час від часу вносити в екземпляри API. Це може бути постійним оновленням або оновленням вручну. Останнє є обов’язковим при зміні залежностей або додаванні змінних середовища.

> [!ATTENTION] Наразі автоматизовані конвеєри не обробляють оновлення залежностей. Перед запуском будь-якого конвеєра розгортання нам потрібно виконати оновлення вручну.

#### 1. Оновлення вручну: використовується для оновлення залежностей, змінних середовища.

1. Зупиніть всі екземпляри

```console
pnpm pm2 stop all
```

2. Встановіть залежності

```console
pnpm install
```

3. Побудуйте сервер

```console
pnpm prebuild && pnpm build:curriculum && pnpm build:server
```

4. Запустіть екземпляри

```console
pnpm start:server && pnpm pm2 logs
```

#### 2. Постійне оновлення: використовується для логічних змін коду.

```console
pnpm reload:server && pnpm pm2 logs
```

> [!NOTE] Ми обробляємо постійні оновлення коду та логіки через конвеєри. Вам не потрібно запускати ці команди. Вони тут для документації.

#### 3. Оновлення Node

1. Встановіть нову версію Node

2. Оновіть pm2 для використання нової версії

```console
pnpm pm2 update
```

## Робота над екземплярами клієнта

1. Встановіть інструменти збірки для бінарних файлів node (`node-gyp`) тощо.

```console
sudo apt install build-essential
```

### Перше завантаження

Підготовка віртуальних машин за допомогою коду

1. Встановіть Node LTS.

2. Оновіть `npm`, встановіть PM2, налаштуйте `logrotate` та запустіть під час завантаження

   ```console
   npm i -g npm@8
   npm i -g pm2@4
   npm install -g serve@13
   pm2 install pm2-logrotate
   pm2 startup
   ```

3. Клонуйте конфігурацію клієнта, налаштування середовища та ключі.

   ```console
   git clone https://github.com/freeCodeCamp/client-config.git client
   cd client
   ```

   Запустіть екземпляри заповнювачів для вебклієнта, вони будуть оновлені артефактами з конвеєра Azure.

   > Завдання: це налаштування потрібно перемістити в сховище S3 або Azure Blob 
   > 
   > ```console
   >    echo "serve -c ../serve.json -p 50505 www" > client-start-primary.sh
   >    chmod +x client-start-primary.sh
   >    pm2 delete client-primary
   >    pm2 start  ./client-start-primary.sh --name client-primary
   >    echo "serve -c ../serve.json -p 52525 www" > client-start-secondary.sh
   >    chmod +x client-start-secondary.sh
   >    pm2 delete client-secondary
   >    pm2 start  ./client-start-secondary.sh --name client-secondary
   > ```

### Журналювання та моніторинг

```console
pm2 logs
```

```console
pm2 monit
```

### Оновлення екземплярів (обслуговування)

Зміни коду потрібно час від часу вносити в екземпляри API. Це може бути постійним оновленням або оновленням вручну. Останнє є обов’язковим при зміні залежностей або додаванні змінних середовища.

> [!ATTENTION] Наразі автоматизовані конвеєри не обробляють оновлення залежностей. Перед запуском будь-якого конвеєра розгортання нам потрібно виконати оновлення вручну.

#### 1. Оновлення вручну: використовується для оновлення залежностей, змінних середовища.

1. Зупиніть всі екземпляри

   ```console
   pm2 stop all
   ```

2. Встановіть чи оновіть залежності

3. Запустіть екземпляри

   ```console
   pm2 start all --update-env && pm2 logs
   ```

#### 2. Постійне оновлення: використовується для логічних змін коду.

```console
pm2 reload all --update-env && pm2 logs
```

> [!NOTE] Ми обробляємо постійні оновлення коду та логіки через конвеєри. Вам не потрібно запускати ці команди. Вони тут для документації.

## Робота над чат-серверами

Наші чат-сервери доступні з конфігурацією високої доступності, [рекомендованою в документації Rocket.Chat](https://docs.rocket.chat/installation/docker-containers/high-availability-install). Файл `docker-compose` можна знайти [тут](https://github.com/freeCodeCamp/chat-config).

Ми пропонуємо надлишкові екземпляри NGINX, які самі збалансовують навантаження (Azure Load Balancer) перед кластером Rocket.Chat. Файл конфігурації NGINX [доступний тут](https://github.com/freeCodeCamp/chat-nginx-config).

### Перше завантаження

Підготовка віртуальних машин за допомогою коду

**Кластер NGINX:**

1. Завантажте і налаштуйте NGINX з репозиторію.

   ```console
   sudo su

   cd /var/www/html
   git clone https://github.com/freeCodeCamp/error-pages

   cd /etc/
   rm -rf nginx
   git clone https://github.com/freeCodeCamp/chat-nginx-config nginx

   cd /etc/nginx
   ```

2. Встановіть сертифікати Cloudflare і конфігурацію головної програми.

   Отримайте оригінальні сертифікати зі сховища безпеки та встановіть їх в потрібних місцях.

   **АБО**

   Перемістіть наявні сертифікати:

   ```console
   # Local
   scp -r username@source-server-public-ip:/etc/nginx/ssl ./
   scp -pr ./ssl username@target-server-public-ip:/tmp/

   # Remote
   rm -rf ./ssl
   mv /tmp/ssl ./
   ```

   Оновіть головні конфігурації:

   ```console
   vi configs/upstreams.conf
   ```

   Додайте/оновіть вихідні IP-адреси програми.

3. Налаштуйте мережу та брандмауери.

   Налаштуйте брандмауери Azure та `ufw` відповідно до доступу до вихідних адрес.

4. Додайте віртуальну машину до балансувальника навантаження.

   За потреби налаштуйте та додайте правила до балансувальника навантаження. Можливо, також знадобиться додати віртуальну машину до пулу балансувальника навантаження.

**Кластер Docker:**

1. Завантажте і налаштуйте Docker з репозиторію

   ```console
   git clone https://github.com/freeCodeCamp/chat-config.git chat
   cd chat
   ```

2. Налаштуйте необхідні змінні середовища та IP-адреси екземплярів.

3. Запустіть сервер rocket-chat

   ```console
   docker-compose config
   docker-compose up -d
   ```

### Журналювання та моніторинг

1. Перевірте стан служби NGINX за допомогою наступної команди:

   ```console
   sudo systemctl status nginx
   ```

2. Перевірте стан запущених екземплярів docker за допомогою:

   ```console
   docker ps
   ```

### Оновлення екземплярів (обслуговування)

**Кластер NGINX:**

Налаштуйте зміни в екземплярах NGINX, які зберігаються на GitHub. Їх потрібно розгорнути в кожному екземплярі:

1. SSH в екземпляр і введіть sudo

   ```console
   sudo su
   ```

2. Отримайте останній код конфігурації.

   ```console
   cd /etc/nginx
   git fetch --all --prune
   git reset --hard origin/main
   ```

3. Протестуйте та повторно завантажте конфігурації [з Signals](https://docs.nginx.com/nginx/admin-guide/basic-functionality/runtime-control/#controlling-nginx).

   ```console
   nginx -t
   nginx -s reload
   ```

**Кластер Docker:**

1. SSH в екземпляр та перейдіть до шляху налаштування чату

   ```console
   cd ~/chat
   ```

2. Отримайте останній код конфігурації.

   ```console
   git fetch --all --prune
   git reset --hard origin/main
   ```

3. Отримайте останній образ docker для Rocket.Chat

   ```console
   docker-compose pull
   ```

4. Оновіть запущені екземпляри

   ```console
   docker-compose up -d
   ```

5. Переконайтесь, що екземпляри запущені

   ```console
   docker ps
   ```

6. Вилучіть зайві ресурси

   ```console
   docker system prune --volumes
   ```

   Вивід:

   ```console
   WARNING! This will remove:
     - all stopped containers
     - all networks not used by at least one container
     - all volumes not used by at least one container
     - all dangling images
     - all dangling build cache

   Are you sure you want to continue? [y/N] y
   ```

   Виберіть «так» (y), щоб видалити все, що не використовується. This will remove all stopped containers, all networks and volumes not used by at least one container, and all dangling images and build caches.

## Work on Contributor Tools

### Deploy Updates

ssh into the VM (hosted on Digital Ocean).

```console
cd tools
git pull origin master
pnpm install
pnpm run build
pm2 restart contribute-app
```

## Оновлення версій Node.js на віртуальних машинах

Отримайте список наразі встановлених версій node та npm

```console
nvm -v
node -v
npm -v

nvm ls
```

Встановіть останню версію Node.js LTS і перевстановіть всі глобальні пакети

```console
nvm install --lts --reinstall-packages-from=default
```

Перевірте встановлені пакети

```console
npm ls -g --depth=0
```

Зробіть псевдонім версії Node.js `default` для поточної LTS (закріплена до останньої основної версії)

```console
nvm alias default 16
```

(Необов’язково) Видаліть старі версії

```console
nvm uninstall <version>
```

> [!ATTENTION] For client applications, the shell script can't be resurrected between Node.js versions with `pm2 resurrect`. Deploy processes from scratch instead. This should become nicer when we move to a docker-based setup.
> 
> If using PM2 for processes you would also need to bring up the applications and save the process list for automatic recovery on restarts.

Отримайте інструкції/команди видалення за допомогою команди `unstartup` та використайте вивід, щоб видалити служби systemctl

```console
pm2 unstartup
```

Отримайте інструкції/команди встановлення за допомогою команди `startup` та використайте вивід, щоб додати служби systemctl

```console
pm2 startup
```

Швидкі команди для PM2 для перерахування, відновлення збережених процесів і т. д.

```console
pm2 ls
```

```console
pm2 resurrect
```

```console
pm2 save
```

```console
pm2 logs
```

## Встановлення та оновлення агентів Azure Pipeline

Див. https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-linux?view=azure-devops та дотримуйтесь інструкцій, щоб зупинити, видалити та перевстановити агентів. Загалом ви можете дотримуватись кроків, наведених нижче.

Вам знадобиться PAT, який можна взяти тут: https://dev.azure.com/freeCodeCamp-org/_usersSettings/tokens

### Встановлення агентів на цілях розгортання

Перейдіть до [Azure Devops](https://dev.azure.com/freeCodeCamp-org) та зареєструйте агента з нуля у необхідних [групах розгортання](https://dev.azure.com/freeCodeCamp-org/freeCodeCamp/_machinegroup).

> [!NOTE] Вам потрібно запустити сценарії в основному каталозі та переконатись, що не існує іншого каталогу `azagent`.

### Оновлення агентів

Наразі оновлення агентів вимагає їхнього видалення та переналаштування. Це необхідно для того, щоб вони правильно підбирали значення `PATH` та інші змінні оточення системи. Нам потрібно зробити це, щоб оновити Node.js на цільових віртуальних машинах розгортання.

1. Перейдіть та перевірте статус служби

   ```console
   cd ~/azagent
   sudo ./svc.sh status
   ```

2. Зупиніть службу

   ```console
   sudo ./svc.sh stop
   ```

3. Видаліть службу

   ```console
   sudo ./svc.sh uninstall
   ```

4. Видаліть агента з pipeline pool

   ```console
   ./config.sh remove
   ```

5. Видаліть файли конфігурації

   ```console
   cd ~
   rm -rf ~/azagent
   ```

Як тільки ви виконали вищевказані кроки, ви можете повторити ті самі кроки, що й при встановленні агента.

## Керівництво з розсилки листів

Ми використовуємо [інструмент CLI](https://github.com/freecodecamp/sendgrid-email-blast) для наших тижневих листів. Щоб підготувати та почати цей процес потрібно:

1. Увійдіть до DigitalOcean та почніть використовувати нові краплі в межах проєкту `Sendgrid`. Використайте знімок Ubuntu Sendgrid з найновішою датою. Він попередньо завантажений з інструментом CLI та сценарієм для отримання електронних пошт з бази даних. Зважаючи на поточний обсяг, трьох крапель достатньо для своєчасного надсилання листів.

2. Налаштуйте сценарій, щоб отримати віддалений доступ до списку електронних пошт.

   ```console
   cd /home/freecodecamp/scripts/emails
   cp sample.env .env
   ```

   Вам потрібно буде замінити значення заповнювачів у файлі `.env` на свої облікові дані.

3. Запустіть сценарій.

   ```console
   node get-emails.js emails.csv
   ```

   Це збереже список електронних пошт у файлі `emails.csv`.

4. Розділіть електронні пошти на декілька файлів, залежно від кількості потрібних крапель. Це найлегше зробити за допомогою `scp`, щоб локально отримати список електронних пошт та, використовуючи свій улюблений текстовий редактор, розділити їх на декілька файлів. Кожен файл потребує заголовку `email,unsubscribeId`.

5. Перейдіть до каталогу CLI за допомогою `cd /home/sendgrid-email-blast` та налаштуйте інструмент [згідно документації](https://github.com/freeCodeCamp/sendgrid-email-blast/blob/main/README.md).

6. Запустіть інструмент, щоб надіслати листи, дотримуючись [документації з користування](https://github.com/freeCodeCamp/sendgrid-email-blast/blob/main/docs/cli-steps.md).

7. Як тільки розсилка листів завершиться, переконайтеся, що усі повідомлення доставлено успішно, перш ніж знищити краплі.

## Керівництво з додавання екземплярів новин нових мов

### Зміни теми

Ми використовуємо власну [тему](https://github.com/freeCodeCamp/news-theme) для публікацій новин. Виконавши наступні зміни теми, можна додати нову мову.

1. Врахуйте інструкцію `else if` до [коду ISO нової мови](https://www.loc.gov/standards/iso639-2/php/code_list.php) у [`setup-locale.js`](https://github.com/freeCodeCamp/news-theme/blob/main/assets/config/setup-locale.js)
2. Створіть початкову папку конфігурації, створивши копію папки [`assets/config/en`](https://github.com/freeCodeCamp/news-theme/tree/main/assets/config/en) та змінивши її назву на код нової мови. (`en` —> `es` для іспанської)
3. Всередині папки нової мови змініть назви змінних у `main.js` та `footer.js` на відповідні скорочені коди мови (`enMain` —> `esMain` для іспанської)
4. Створіть копію [`locales/en.json`](https://github.com/freeCodeCamp/news-theme/blob/main/locales/en.json) та змініть її назву на код нової мови.
5. Додайте сценарії для щойно створених файлів конфігурації до [`partials/i18n.hbs`](https://github.com/freeCodeCamp/news-theme/blob/main/partials/i18n.hbs).
6. Додайте сценарій відповідної мови `day.js` з [cdnjs](https://cdnjs.com/libraries/dayjs/1.10.4) до [freeCodeCamp CDN](https://github.com/freeCodeCamp/cdn/tree/main/build/news-assets/dayjs/1.10.4/locale)

### Зміни інформаційної панелі Ghost

Оновіть активи публікацій, перейшовши на Ghost Dashboard > Settings > General та завантаживши [піктограму](https://github.com/freeCodeCamp/design-style-guide/blob/master/assets/fcc-puck-500-favicon.png), [логотип](https://github.com/freeCodeCamp/design-style-guide/blob/master/downloads/fcc_primary_large.png) та [зображення](https://github.com/freeCodeCamp/design-style-guide/blob/master/assets/fcc_ghost_publication_cover.png).
