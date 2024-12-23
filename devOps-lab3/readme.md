
# Лабораторная работа № 3 - CI/CD

##

## Введение


В рамках лабораторной работы необходимо:

1. Написать “плохой” CI/CD файл, который работает, но в нем есть не менее пяти “bad practices” по написанию CI/CD.
2. Написать “хороший” CI/CD, в котором эти плохие практики исправлены.
3. В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат.


## Практическая часть

Для начала напишем простое приложение на Go и применим для него CI/CD с пятью плохими пркатиками.

### Плохой CI/CD
```
name: CI/CD Pipeline - Bad Practices

on:
  push:

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Setup Go
        uses: actions/setup-go@v2

      - name: Install dependencies
        run: go mod tidy

      - name: Run unit tests
        run: go test ./... -v || true  # Тесты могут упасть, но пайплайн продолжит работу

      - name: Build and deploy
        run: |
          go build -o my-app main.go
          echo "Deploying to production..."
```

![](images/bad1.PNG)
![](images/bad2.PNG)

### Описание:

1. Выполнение CI/CD при любом пуше в репозиторий. Если работа ведется в другой ветке, то пуш в нее все равно вызовет пайплайн, что увеличивает время сборки:


2. Использование ```ubuntu-latest```. В будущем это может привести к конфликтам в версиях фреймворков и библиотек. 
  
3. Нет кеширования зависимостей. При каждом запуске все зависимости будут устанавливаться заново, что значительно увеличивает время сборки.

4. Игнорирование ошибок тестов. Если тесты не прошли, то пайплайн все равно продолжит работу, что может быть опасно в дальнейшем.

5. Объединение сборки, тестов и деплоя в один шаг. Если  како-то из этих пунктов провалится, то невозможно будет определить, где была ошибка, что сильно усложняет отладку.

### Хороший CI/CD

```
name: CI/CD Pipeline - Good Practices

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Setup Go
        uses: actions/setup-go@v2
        with:
          go-version: '1.22'

      - name: Cache Go modules
        uses: actions/cache@v2
        with:
          path: |
            ~/.cache/go-build
            ~/.cache/go-mod
          key: ${{ runner.os }}-go-mod-${{ hashFiles('**/go.mod') }}
          restore-keys: |
            ${{ runner.os }}-go-mod-

      - name: Install dependencies
        run: go mod tidy
        
      - name: Run unit tests
        run: go test ./... -v  

  deploy:
    runs-on: ubuntu-22.04
    needs: test  
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Deploy to production
        run: echo "Deploying to production environment..."
```

![](images/good1.PNG)
![](images/good2.PNG)
![](images/good3.PNG)

### Описание:

1. Пайплайн теперь запускается только при пуше в main. Теперь не будет тратиться лишнее время на запуск сборки, когда нам это не нужно.

2. Четко указана версия ```ubuntu-22.04```. Это позволит избежать конфликтов между версиями библиотек и фреймворков. 

3. Добавлено кеширование зависимостей. Теперь при повторной сборке нам не нужно будет ждать кучу времени.

4. Теперь мы не игнорируем тесты и запускаем их с флагом ```-v```, чтобы в случае чего получить подробную информацию.

5. Разделение на jobs'ы. Теперь код более структурирован и понятен для чтения и отладки.  

## Вывод
В ходе выполнения данной лабораторной работы я впервые столкнулся с пайпалайнами и их написанием. Тема показалась мне довольно интересной и полезной, так что я уверен, что знания, полученные в процессе, окажутся полезными в будущем.
