
# Лабораторная работа по предмету "Облачные технологии и услуги"

## Техническое задание

1. **Написать “плохой” Dockerfile, в котором есть не менее трех “bad practices” по написанию докерфайлов**
   
2. **Написать “хороший” Dockerfile, в котором эти плохие практики исправлены**
   
3. **В Readme описать каждую из плохих практик в плохом докерфайле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат**
   
4. **В Readme описать 2 плохих практики по работе с контейнерами. ! Не по написанию докерфайлов, а о том, как даже используя хороший докерфайл можно накосячить именно в работе с контейнерами.**

## Начало работы

### **Плохой Dockerfile**

- Пишем простое приложение на Go

    ![alt text](<images/1.PNG>)

- Пишем плохой Dockerfile, используя “bad practices”

    ```
    FROM golang:latest

    RUN apt-get update
    RUN apt-get install -y golang

    COPY . /app

    RUN cd /app && go get -u github.com/labstack/echo/v4

    CMD ["go", "run", "/app/main.go"]
    ```

#

### Что не так?

#### 1. Использование последнего базового образа
    
- Тег latest не дает гарантии, что образ будет идентичным. Базовый образ может изменится, что в свою очередь приведет к проблемам с работой приложения.
 
#### 2. Установка зависимостей после копирования файлов

- Docker каждый раз при изменении кода будет переустанвливать все зависимости, даже если они не изменились.

#### 3. Запуск контейнера от root

- Из-за запуска от ```root``` контейнер становится уязвимым. При получении доступа к контейнеру любой человек получит привелегии 'root'.

#### 4. Копирование абсолютно всех файлов в контейнер

- Копирование всех файлов (включая ненужные) привеодит к вувелечению размера образа, следовательно, к времени сборки.
# 
- Создаем образ с помощью команды

    ```
    docker build -t flask-app .
    ```
    ![alt text](<images/2.PNG>)

    ![alt text](<images/3.png>)

#

### Хороший Dockerfile

- Пишем хороший Dockerfile, используя good practices

    ```
    FROM golang:1.23-alpine

    WORKDIR /app
    COPY go.mod go.sum ./
    RUN go mod download

    COPY main.go .

    RUN go build -o myapp .

    RUN addgroup -S appgroup && adduser -S appuser -G appgroup

    USER appuser

    CMD ["./myapp"]
    ```

#

### Почему Dockerfile хороший?


#### 1. Использование тега

- Используем конкртный тег ```golang:1.23-alpine```, что позволяет нам всегда работать с конкретной версией Go, обеспечивая стабильность. Также использование ```alpine``` уменьшает размер образа.

#### 2. Устанавка зависимостей до копирования файлов

- Сначала копируем файлы go.mod и go.sum и устанавливаем зависимости. Docker кэширует слои, поэтому при изменении только кода приложения не нужно заново устанавливать все зависимости. Это ускоряет время сборки и размер образа.

#### 3. Копирование только нужных файлов

- При копировании только необходимых файлов, мы исключаем попадание ненужных файлов, которые увеличивают размер и время сборки образа.

#### 4. Создание отдельного пользователя

- Создаем отделльного пользователя с ограниченными правами, чтобы не запускать контейнер от ```root``` и уменьшить риск атак.

#

- Создаем образ с помощью команды

    ```
    docker build -t flask-app-2 .
    ```

    ![alt text](<images/4.PNG>)

    ![alt text](<images/5.png>) 

#

### Вывод

Docker-образ, построенный на основе хорошего Dockerfile, весит значительно меньше, строится в несколько раз быстрее, а его стабильность, безопасность и удобство в использовании на голову выше, чем у Docker-образа, построенного на основе плохого Docerfile.  

## Плохие практики по работе с контейнерами

### 1. Неправильное управление ресурсами

- Проблема: 

    Неограниченное использование ресурсов (CPU, память) может привести к перегрузке хоста и снижению производительности. Контейнеры могут потреблять больше ресурсов, чем необходимо, что может негативно сказаться на других контейнерах и хосте.


- Решение: 

    Устанавливайте ограничения на использование ресурсов для контейнеров. Это поможет предотвратить перегрузку хоста и обеспечит более стабильную работу всех контейнеров.

    ```
     docker run --cpus="0.5" --memory="512m" myapp
    ```

### 2. Неправильное управление секретами
- Проблема: 
    
    Хранение секретов (например, паролей и ключей) внутри контейнеров или передача их через командную строку может привести к утечке данных.
    
- Решение: 

    Используйте ```Docker Secrets``` или другие механизмы для безопасного управления секретами. Это поможет защитить конфиденциальную информацию и предотвратить утечку данных.

    ```
    # Создание секрета
    echo "mysecretpassword" | docker secret create db_password -

    # Запуск контейнера с использованием секрета
    docker run --secret db_password myapp
    ```
