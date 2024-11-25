**ЛАБОРАТОРНАЯ РАБОТА №2 DOCKERFILE** 

**Цель работы** 

Научиться создавать и тестировать Dockerfile и Docker Compose, выявлять и исправлять ошибки, улучшать практики работы с контейнерами. ![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.017.png)

**Задания** 

1. Создать "плохой" Dockerfile с минимум 3 ошибками. 
1. Исправить его, создав "хороший" Dockerfile. 
1. Подготовить README с описанием ошибок и исправлений. 
1. Привести примеры плохих практик в работе с контейнерами. 

**Со звездочкой:** Создать "плохой" и "хороший" Docker Compose файл с аналогичными ошибками и исправлениями. ![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.018.png)

**Выполненные шаги** 

**1. Написание "плохого" Dockerfile** 

**Содержимое "плохого" Dockerfile:** 

FROM ubuntu:latest 

USER root 

RUN apt-get update && apt-get install -y python3 pip COPY . /app 

CMD ["bash"] 

**Ошибки:** 

1. Используется полный образ Ubuntu, который большой и содержит ненужные компоненты. 
1. Контейнер работает от имени пользователя root, что угрожает безопасности. 
1. Устанавливаются пакеты без указания версий, что делает контейнер нестабильным. ![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.020.png)


**2. Написание "хорошего" Dockerfile** 

**Содержимое "хорошего" Dockerfile:** 

FROM python:3.9-slim RUN useradd -m appuser 

RUN apt-get update && apt-get install -y --no-install-recommends \     python3=3.9.\* \ 

`    `python3-pip=20.3.\* && \ 

`    `rm -rf /var/lib/apt/lists/\* 

WORKDIR /app 

COPY requirements.txt /app/ 

RUN pip install --no-cache-dir -r requirements.txt 

COPY . /app 

USER appuser 

CMD ["python3", "app.py"] 

**Исправления:** 

1. Использован минимизированный базовый образ python:3.9-slim, который значительно уменьшает размер. 
1. Добавлен пользователь appuser для повышения безопасности. 
1. Указаны версии устанавливаемых пакетов, что повышает стабильность сборки. 
1. Удален кэш APT после установки, что уменьшает размер образа. 

**3. Тестирование Dockerfile** 

1. **Сборка "плохого" Dockerfile:** 

   docker build -t bad-container -f Dockerfile.bad . docker run --rm -p 80:80 bad-container

   **Результат:** 

- Контейнер собрался, но занимает много места. 
- Работает от имени root, что создает риски безопасности. 
2. **Сборка "хорошего" Dockerfile:** 

   docker build -t good-container -f Dockerfile.good . docker run --rm -p 8080:80 good-container 

   **Результат:** 

- Контейнер меньше по размеру. 
- Работает от имени пользователя appuser, безопаснее. 

**4. Примеры плохих практик работы с контейнерами ![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.025.png)**

1. **Хранение данных внутри контейнера.** 
- Решение: Использовать volumes для сохранения данных между запусками. 
2. **Открытие всех портов контейнера.** 
- Решение: Открывать только минимально необходимые порты. 

**5. Создание "плохого" Docker Compose** 

**Содержимое "плохого" Docker Compose файла:** 

version: '3.9' 

services: 

`  `app: 

`    `build: . 

`    `ports: 

- "80:80" 

`    `environment: 

`      `DATABASE\_URL: "postgres://user:password@localhost:5432/db"   database: 

`    `image: postgres:latest 

`    `environment: 

`      `POSTGRES\_USER: user 

`      `POSTGRES\_PASSWORD: password 

`      `POSTGRES\_DB: db 

`    `ports: 

- "5432:5432" 

**Ошибки:** 

1. Все сервисы находятся в одной сети, что создает угрозу безопасности. 
1. Конфиденциальные данные хранятся в файле Compose. 
1. Все порты базы данных открыты. ![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.028.png)

**6. Создание "хорошего" Docker Compose** 

**Содержимое "хорошего" Docker Compose файла:** 

version: '3.9' 

services: 

`  `app: 

`    `build: . 

`    `ports: 

- "8080:80" 

`    `networks: 

- app\_network 

`    `environment: 

- DATABASE\_URL=postgres://user:password@database:5432/db 

`  `database: 

`    `image: postgres:13 

`    `networks: 

- db\_network 

`    `environment: 

`      `POSTGRES\_USER: user 

`      `POSTGRES\_PASSWORD: password       POSTGRES\_DB: db 

networks: 

`  `app\_network: 

`    `driver: bridge   db\_network: 

`    `driver: bridge 

**Исправления:** 

1. Сервисы разделены на разные сети. 
1. Конфиденциальные данные вынесены в файл .env. 
1. Открыт только порт приложения. 

**7. Тестирование Docker Compose** 

**Запуск "плохого" Compose проекта:** 

docker-compose -f docker-compose.bad.yml up --build 

**Результат:** Проект работает, но сеть не изолирована, конфиденциальные данные видны. 

**Запуск "хорошего" Compose проекта:** 

docker-compose -f docker-compose.good.yml --env-file .env up --build 

**Результат:** Проект изолирован, данные защищены. ![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.034.png)

**Скриншоты** 

![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.035.png)

Рисунок 8. Плохой Dockerfile**  

![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.036.png)

Рисунок 9. Хороший Dockerfile**  

![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.037.png)

Рисунок 10. Тестирование работы контейнера**  

![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.038.png)

Рисунок 11. Плохой Compose файл  

![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.039.jpeg)

Рисунок 12. Хороший Compose файл**  

![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.040.png)

Рисунок 13. Тестирование изоляции  **![](Aspose.Words.0c9b463e-662d-4820-8aac-50db791a7838.041.png)**

**Итог** 

1. Написаны и протестированы "плохой" и "хороший" Dockerfile. 
1. Подготовлены "плохой" и "хороший" Docker Compose файлы. 
1. Все исправления зафиксированы в README. 
1. Тесты показали безопасность и эффективность "хороших" файлов.