# Лабораторная работа №2 со звёздочкой
## Задание
* Написать “плохой” Docker compose файл, в котором есть не менее трех “bad practices” по их написанию.
* Написать “хороший” Docker compose файл, в котором эти плохие практики исправлены.
* В хорошем файле настроить сервисы так, чтобы контейнеры в рамках этого compose-проекта поднимались вместе, но не "видели" друг друга по сети.

## Выполнили студенты
* Зюзин Владислав
* Недиков Михаил 
* Поскуряков Роман

## Установка Docker compose

Первым делом устанавливаем Docker, если ещё не установили, лучший гайд по установке представлен на видео по ссылке: https://yandex.ru/video/preview/3719090281561281090  
После этого устанавливаем Docker compose на ОС. В нашем случае это Linux, а именно Ubuntu:24.04, которую мы запускаем через VirtualBox.
Не забываем проверить обновления, благодаря вводу команды: 
```
sudo apt get update
```
Далее уже можно установить Docker compose, введём поочерёдно команды:
```
mkdir -p ~/.docker/cli-plugins/
```
```
curl -SL https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
```
```
chmod +x ~/.docker/cli-plugins/docker-compose
```
На рисунке ниже представлен код и его вывод, а так же проверка установки Docker compose

![1](1.png)

## Написание Docker compose файла

### Пара слов прр Docker compose

Docker Compose — это инструмент, предназначенный для удобного управления контейнерами Docker. Он особенно полезен для работы со средами, в которых задействовано несколько контейнеров, но также подходит для управления отдельными контейнерами. С его помощью можно изолировать окружения, одновременно запускать, пересобирать и останавливать контейнеры, отслеживать их состояние и пользоваться множеством других полезных функций, упрощающих работу с контейнерами.

### Как с ним работать?
```
version: '3.8'

services:
  container_1:
    image: ubuntu:22.04
    privileged: false
    volumes:
      - ./script1:/container_folder:ro
    command: >
      /bin/bash -c "
      apt-get update && 
      apt-get install -y bash iputils-ping &&
      bash /container_folder/MyScript1.bash"
      echo "I'm bad container_1"
    
  container_2:
    image: ubuntu:22.04
    privileged: false
    volumes:
      - ./script2:/container_folder:ro
    command: >
      /bin/bash -c "
      apt-get update && 
      apt-get install -y bash iputils-ping &&
      bash /container_folder/MyScript2.bash"
      echo "I'm bad container_2"
```
