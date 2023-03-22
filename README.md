## GitLab CI/CD Runner простой проект для новичка


### Немного теории

**CI/CD** - методика непрерывной интеграции и доставки контента

Разделяется два процесса - **CI (Continuous Integration) и CD (Continuous Delivery)**

Представляет собой автоматизированный процесс от разработки и тестов до разворачивания в продакшн

**CI** - практика разработки программного обеспечения, выполнение сборок, выявление дефектов и решение проблем

**CD** - подход к разработке программного обеспечения, передача стабильного ПО в эксплуатацию

**Runner** - по, предназначенное для автоматизизации выполнения заданий

Например
Производятся обновления в проектах - **git push**
Последующее внесение изменений на серверах, например синхронизация файлов

Предполагается, что **GitLab** уже установлен, если нет, воспользуйтесь предыдущими материалами
https://www.youtube.com/watch?v=DZuP7QawWc4
http://snakeproject.ru/rubric/article.php?art=gitlab_1_27012022
https://www.youtube.com/watch?v=NI3xLFOgo0M
http://snakeproject.ru/rubric/article.php?art=gitlab_1_28012022

У меня уже был создан тестовый проект **test**, на нем и буду издеваться


### Установка GitLab Runner

**Настройка репозитория Debian / Ubuntu:**
$ curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash

**Установка раннера Debian / Ubuntu:**
$ sudo apt install gitlab-runner

**Настройка репозитория Red Hat / CentOS:**
$ curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash

**Установка раннера Red Hat / CentOS:**
$ sudo yum install gitlab-runner

**Запуск службы:**
$ sudo systemctl enable gitlab-runner --now gitlab-runner


### Связка (Регистрация) Runner

**Runner** необходимо связать с проектом в **GitLab**, узнаем настройки

В web панели **GitLab** - страница репозитория - **Settings - CI/CD - Runners - Expand**  
Specific runners  
These runners are specific to this project.  
Set up a specific Runner for a project  
Install GitLab Runner and ensure it's running.  
Register the runner with this URL:  
`http://127.0.0.1:80/`  
And this registration token:  
`hKhcXHxAmqvd-QAmxc-m`  

Видим параметры раннера **(url и token)**, они сейчас понадобятся в командной строке

Общее использование раннеров разными проектами можно отключить

Под надписью **Enable shared runners for this project** есть переключатель

**В командной строке:**
$ sudo gitlab-runner register

**Адрес сервера с GitLab:**
Enter the GitLab instance URL (for example, https://gitlab.com/):
`http://127.0.0.1:80/ `

**Токен для регистрации раннера:**
Enter the registration token:
`hKhcXHxAmqvd-QAmxc-m`

**Описание в свободной форме:**
Enter a description for the runner:
`test runner`

**Теги для раннера, на свой вкус (будут использоваться для отправки раннеру задания):**
Enter tags for the runner (comma-separated):
`test, api`

**Исполнитель из списка вариантов:**
Enter an executor:
`shell`

$ sudo systemctl restart gitlab-runner

**Проверка (должно быть is_alive):**
$ sudo gitlab-runner verify


### Настройка Runner

Перезагрузим страницу с параметрами раннера

Ниже появилась строка с элементом, заходим справа от названия по "изображению редактирования"

#### Выставляем 3 галки на следующих вариантах:

Если обработчик заданий остановлен, то не принимает новых заданий
###### Paused Runners don't accept new jobs

Может запускать задания без тегов
###### Indicates whether this runner can pick jobs without tags

Если обработчик заданий заблокирован, его нельзя назначать для других проектов
###### When a runner is locked, it cannot be assigned to other projects

Runner запускается только на защищенных ветках нам не надо!
###### This runner will only run on pipelines triggered on protected branches

**Save Changes** и переходим к настройке проекта файлом **.gitlab-ci.yml**


### CI/CD простой проект

В прошлый раз сам проект test скачал с помощью git clone в прапку /tmp  
$ cd /tmp && git clone git@gitlab.localdomain:user/test.git && cd /tmp/test  
$ date > test.txt && git add test.txt && git commit -m "To push" && git push 
 
Теперь наша задача, чтоб при каждом **git push** запускался автоматически сценарий **yml**

Простейший формат сценария yml:
```
stages: 
  - имя_шага_1
  - имя_шага_2
название_задачи_1
  stage: имя_шага_1
  tags:
    - имя_тега_1
    - имя_тега_2
  script:
    - команда_1
    - команда_2
  stage: имя_шага_2
  script:
    - команда_1
    - команда_2
  artifacts:
      paths:
        - что_вернуть_на_сервер_с_gitlab
```


stages описывает шаги разворачивания и очередность выполнения сценария

Задачи выполняются в рамках каждого шага, должны иметь:
указание этапа stage
список команд для выполнения в script

Во время выполнения (по умолчанию в домашней папке) на сервере создается директория build

Раннер клонирует туда последние исходники проекта, далее переходит в папку с исходниками

После, выполняются команды из списка в script

**Теги для GitLab и теги для Git - два разных понятия**

Вы можете указать для задания тег, если раннер с этим связанным тегом доступен, он выполнит задание

Как мы помним, в командной строке своему раннеру я задал 2 тега **test и api**

Если у раннера при регистрации не указан тег из задачи, то он ее не выполнет

Таким образом можно создать разделение выполнения задач определенными раннерами

**Т.к. сервер у меня один, а хочется пример с исполнением ssh команды типа на удаленку, "эмулируем":**
```
$ sudo su gitlab-runner  
$ ssh-keygen  
$ cat ~/.ssh/id_rsa.pub  > ~/.ssh/authorized_keys
```

Теперь мы сможем использовать функционал типа для "удаленных серверов" laugh

**Наш простой pipeline .gitlab-ci.yml:**
```
stages:
  - build_and_test
  - deploy

variables:
  test_file_remote: /tmp/test_job_remote.txt
  test_file: /tmp/test_job.txt
  git_file: /tmp/test/test.txt 

test_job_ci:
  stage:  build_and_test
  tags:
    - test
    - api
  script:
    - echo "Project dir - "$CI_PROJECT_DIR > $test_file
    - echo "Project name - "$CI_PROJECT_NAME >> $test_file
    - test $git_file 
    - cat $git_file >> $test_file
    - ssh gitlab-runner@127.0.0.1 cat $test_file > $test_file_remote

test_job_cd:
  stage:  deploy
  tags:
    - api
  script:
    - if [ -e $test_file_remote ]; then cat $test_file_remote; else echo "Where my file?!" > $test_file_remote; fi
    - mkdir build
    - cp $test_file_remote build/
  artifacts:
      paths:
        - build/
```

**Переменные использованы из списка:**
https://docs.gitlab.com/ee/ci/variables/index.html

**Создаем pipeline:**
Страница проекта - **CI/CD - Pipelines - Editor - Create New CI/CD Pipeline**
Откроется редактор **.gitlab-ci.yml** - изменяем - **Commit Changes**

Сохраняем, ожидаем несколько секунд, переходим в **jobs**

В **Jobs** должен быть выведен результат выполнения сценария и ход выполнения работы

В **Pipelines** - отображаются все задачи - **успех Passed**

В **artifacts** мы указываем то, что вернется на сервер **GitLab**

Артефакты можно скачать в **CI/CD - Pipelines** или **CI/CD - Jobs** в задачах в виде архива

Физически артефакты хранятся в директории билда проекта, отображается в **job** логах выполнения

При каждом **git push** данный **pipeline** будет автоматически запускаться

###### Для того, чтоб в будущем учетка gitlab-runner могла запускать привелигированные команды, добавьте ее в sudo:
```
$ sudo visudo
gitlab-runner ALL=(ALL) NOPASSWD: ALL
```


###### Глобальные переменные можно добавить в Страница проекта - Settings - CI/CD - Variables - Expand - Add Variable 

 
