# Практическая работа 10. Анализ данных Docker. Jupyter+Postgres.

```все команды выполняются из git bash```

## Запускаем  контейнер
Многие компании имеют в докере свои репозитории, которые они постоянно обновляют: `Jupyter`, `Postgres`. Чтобы запустить блокноты Jupyter в докере, нам нужно найти правильную команду docker cli. Мы можем найти его на [Jupyter's "Read the docks" page](https://jupyter-docker-stacks.readthedocs.io/en/latest/).

Запустите следующую команду в терминале

```cmd
docker run -p 8888:8888 jupyter/scipy-notebook:33add21fab64
```
> Эта команда извлекает изображение ```jupyter/scipy-notebook``` с тегом ```33add21fab64`` из Docker Hub, если оно еще не присутствует на локальном хосте.
> Затем он запускает контейнер, на котором работает сервер Jupyter Notebook, и предоставляет серверу хост-порт ``8888``. Журналы сервера появляются в терминале.
> При посещении ```http://<hostname>:8888/?token=<token>``` в браузере загружается страница панели управления Jupyter Notebook, где имя хоста — это имя компьютера, на котором запущен Docker, а token — это напечатанный секретный токен. в консоли. Контейнер остается нетронутым при перезапуске после выхода сервера ноутбука.

# Как попасть в  контейнер
Пока  контейнер работает, откройте второй терминал и введите следующую команду:

```cmd
docker ps
```
Вы должны увидеть следующее:

![](./img/02.png)

Скопируйте **идентификатор контейнера**, он понадобится вам на следующем шаге.

Что, если я захочу попасть в свой контейнер? Например, установить библиотеки для работы с данными, базами данных.

Вам нужно войти во второй терминал (если вы закрыли его после предыдущей команды, просто откройте новый):

```cmd
docker exec -it <mycontainer> bash
```

```<mycontainer>``` is you ***container id***

Получим:

![](./img/03.png)

Проверим каталог:

```cmd
ls
```
В каталоге увидим папку **work**.

![](./img/04.png)

Также можно проверить, установлен ли на нем Python::

```cmd
python
```

Он должен быть предварительно установлен в  контейнере..

# Как скопировать файлы в контейнер

Скопируем файл **csv** в наш контейнер.
Команда копирования описана в [официальной документации] (https://docs.docker.com/engine/reference/commandline/cp/) и обсуждается в [вопросе Stakoverflow] (https://stackoverflow.com/questions/22907231 /как-копировать-файлы-с-хоста-в-докер-контейнер)

Загрузите [файл](./files/SampleSuperstore.csv) на компьютер.

Общий формат команды *copy*:

```cmd
docker cp foo.txt <container_id>:/foo.txt
```

Первый *foo.txt* — это путь к файлу на  компьютере (тот, который вы скачали выше). Второй *foo.txt* — это будущий путь к скопированному файлу в контейнере.

Чтобы проверить местоположение, в котором вы находитесь после запуска контейнера, введите следующую команду изнутри докера:

```cmd
pwd
``` 
Скорее всего, ответ будет ```/home/[username]```.
Мы будем использовать этот путь в качестве префикса для будущего местоположения файла, который мы хотим скопировать в наш контейнер.
- `[username]` - имя учетной записи, под которой вошли в ОС.

Полный путь будет следующим ```/home/[username]/Sample Superstore.csv```

We are ready to copy the file. 
The following command should be entered from teh terminal of you local machine. NOT from inside of the container. You may need to open a new terminal.

```cmd
docker cp ./files/SampleSuperstore.csv <container_id>:/home/jovyan/files/SampleSuperstore.csv
```

Теперь проверьте, скопирован ли файл. Напишите команду внутри контейнера:

```cmd
ls
```

Вы должны увидеть ```SampleSuperstore.csv```:

![](./img/05.png)

Проверить, работает ли блокнот `Jupyter` в контейнере и загружает ли данные.

В терминале, где  запустили контейнер, щелкните одну из двух ссылок под надписью Jupyter Notebook работает по адресу: **Jupyter Notebook is running at:**

![](./img/06.png)

Затем создайте `notebook` и введите следующий код:

```python
import pandas as pd
df = pd.read_csv('SampleSuperstore.csv')
df.head()
```
Если вы видите следующий вывод, то все работает отлично:

![](./img/07.png)

# Подключение Volume

## **Задача:**
 Требуется скопировать данные обратно на локальный компьютер после того, как поработали над ними в блокноте Jupyter в контейнере. По аналогии с предыдущей задачей можем скопировать файл из контейнера на локальную машину.

Воспользунмся командой *docker cli*:

```cmd
docker cp <container_id>:/foo.txt foo.txt
```
Однако если у нас будет больше файлов/блокнотов, ваш проект может стать довольно большим, что делать в таком случае. Копирование файлов вручную окажется неэффективным.

## **Решение:**

Для решения данной проблемы воспользуемся технологией **volume**. Это работает так: подключаете жесткий диск к контейнеру. Таким «жестким диском» может быть просто папка на локальной машине. Эта папка будет доступна и на локальной машине и в контейнере.

В терминале перейдите в папку/каталог, к которому вы хотите смонтировать тома, и получите путь:

```cmd
pwd
```
Выберем каталог в контейнере, который должен быть подключен к вашей папке на локальном компьютере. Я выбрал ```/home/[username]/```

Затем введите команду, которая запускает контейнер и присоединяет к нему **volume**.

```cmd
docker run -v <path to your folder on the local computer>:/home/jovyan/ -p 8888:8888 jupyter/scipy-notebook:33add21fab64
``` 
Или:

```cmd
docker run -v /${PWD}:/home/[username]/ -p 8888:8888 jupyter/scipy-notebook:33add21fab64
```

# How to run terminal commands automatically at the launch

Why this is important? 

For example you are launching Jupyter notebook contaienr and want specific libraries be pre-installed.

Originally we launched the container ```scipy-notebook:33add21fab64```. This container was not created from the scratch, per se. It was created by somebody else, realeased on docker marketplace and we are just launching it on our machine.

What's cool about docker, we can borrow someone's image(base image), for example, ```scipy-notebook:33add21fab64```, and tune it up to fit our needs.

We tune up the image(base image) with the **dockerfile** helper file. This file contains the description on the logic of your container.

Create in the folder, where you have been launching the image from, the file named **Dockerfile** and insert the following code inside of it.

```docker
FROM jupyter/scipy-notebook:33add21fab64

RUN pip install pandas
```

*Run* command above will be launched in the terminal in the container once the container is launched.

Now, how we would launch the docker after we have created the docker file?


First, we need to enter in the terminal the command:

```cmd
docker build .
```

The dot(.) in the code above is the location of the *dockerfile*.

What we need from log/output on the screen is the container id. You can find it either:
- after the words " => => writing image"
- after the words "successfullu buit"

Previously we launched the container with

```cmd
docker run -v /${PWD}:/home/jovyan/ -p 8888:8888 jupyter/scipy-notebook:33add21fab64
```

We need to modify the command above and insert the container id we got above insted of the base image name.

```cmd
docker run -v /${PWD}:/home/jovyan/ -p 8888:8888 <container id>
```

# Docker Compose

Why we might need it? To launch several services at the same time. In stead of bulky docker cli commands we will utilize configuration file with much simplier syntaxis.

Create **docker-compose.yml** file in the folder from which you launch your conatiner.

Fill in the docker compose file the following way.
```yml
version: "3.1"
services:
  jupyter:
    build:
      context: ./
      dockerfile: Dockerfile
    volumes:
      - ./:/home/jovyan/
    ports:
      - 8888:8888
```

*Context* above is the location of dockerfile.

To launch the container:

```cmd
docker-compose up
```

# Launch postgress along with Jupyter


From now on adding containers becomes much easier task, since we have started using docker-compose.

Modify the **docker-compose.yml** file to include db:

```yml
version: "3.1"
services:
  jupyter:
    build:
      context: ./
      dockerfile: Dockerfile
    volumes:
      - ./:/home/jovyan/
    ports:
      - 8888:8888
  db:
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    ports:
      - 5432:5432
```

As well modify the **Dockerfile** to include new library ```psycopg2```:

```
FROM jupyter/scipy-notebook:33add21fab64

RUN pip3 install pandas psycopg2     

```


To launch the container:

```cmd
docker-compose up --build
```


Open/create the notebook in your container and enter and execute the following script:

```python
import psycopg2
conn = psycopg2.connect("host=db dbname=postgres user=postgres password=postgres")
```

If no errors - cool, let's continue!

# Load Data to DB

Stackoverflow [question](https://stackoverflow.com/questions/2987433/how-to-import-csv-file-data-into-a-postgresql-table) is helping as usual.

There are two methods. We will use the option which employs ```sqlalchemy library```.

Open/create the notebook in your container and enter and execute the following script:

```python
from sqlalchemy import create_engine
engine = create_engine('postgresql://postgres:postgres@db:5432/postgres')

df.to_sql('superstoredata', engine)

df_pg = pd.read_sql_query('Select * from superstoredata', con=engine)

df_pg.head()

```

# Add Volume to save DB data permanently on disk

if your containers have been runnning:

To stop containers from running
```cmd
docker-compose stop
```

To delete containers
```cmd
docker-compose stop
```

Adjust the *docker-compose.yml*:

```yml
version: "3.1"
services:
  jupyter:
    build:
      context: ./
      dockerfile: Dockerfile
    volumes:
      - ./:/home/jovyan/
    ports:
      - 8888:8888
  db:
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - 5432:5432
volumes:
  pgdata:
```

Now run:

```cmd
docker-compose up --build
```

Execute all of your python scripts that write data to the database.

Again, to stop containers from running
```cmd
docker-compose stop
```

To delete containers
```cmd
docker-compose down
```

Type in command to check existing volumes on your local machine:

```cmd
docker volume ls
```

You should see  ```docker_pgdata```.

Theoretically we can backup our database and have it as a file and open it in DBeaver.

To remove volume from your local machine:

```cmd
docker volume rm <volume name>
```
