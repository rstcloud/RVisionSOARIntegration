# Обогащение индикаторов в R-Vision SOAR
## Принцип работы
В R-Vision SOAR связанные с инцидентом индикаторы хранятся в 
отдельном поле `iocs` Данное поле недоступно в виде тега и изменить 
его содержимое напрямую невозможно. Также данное поле имеет фиксированную 
структуру, в которую невозможно добавить новые поля.  
В связи с этим для отображения доп. информации необходимо создать новое поле.
Данное поле будет хранить индикатор и все доп. данные.  
Интеграция работает по следующей схеме:
- Сценарий реагирования использует Коннектор.
- Коннектор делает GET-запрос к FastAPI-сервису, работающему в контейнере, передав ID инцидента в параметре запроса.
- Сервис, получив запрос от Коннектора, делает запрос поля `iocs` инцидента через API R-Vision SOAR.
- Получив индикаторы, Сервис запрашивает их в RST Cloud API.
- Собрав все данные Сервис отправляет их в виде запроса на обновление заданного в конфиге поля инцидента.
## Развертывание в docker
1. Сгенерировать API-токен в R-Vision SOAR.
2. Создать новое поле инцидента `iocs_threats` в R-Vision SOAR.
3. Указать что данное поле имеет тим Массив.
4. Создать следующую схему для данного поля: 
``` 
Тип: Текст ----------- Тег: ioc
Тип: Несколько строк - Тег: tags
Тип: Несколько строк - Тег: threats
Тип: Число ----------- Тег: score
Тип: Дата ------------ Тег: first_seen
Тип: Дата ------------ Тег: last_seen
Тип: Несколько строк - Тег: links
```
5. Создать REST-Коннектор и указать URL `http://<IP>:9080/ioc?identifier={{tag.IDENTIFIER}}`, метод GET, где IP - адрес расположения сервиса. Если сервис будет работать на хосте SOAR, то 127.0.0.1
6. Выставить в коннекторе таймаут 60 сек.
7. Переименовать `conf/config.yml.sample` в `conf/config.yml`.
8. Вписать в `config.yml` адрес R-Vision SOAR и токены.
9. Собрать образ `docker build -t rstintegration .`
10. В `docker-compose.yml` для volume указать расположение директорий `conf` и `log`
11. Запустить контейнер `docker-compose up -d`

## Развертывание в systemd
1. Установить python 3.7
2. Выполнить
```pip3.7 install -r requirements.txt
cd /opt
git clone https://github.com/rstcloud/RVisionSOARIntegration.git
cd RVisionSOARIntegration
mkdir logs
mv conf/config.yml.sample conf/config.yml
cp rvisionrstintegration.service /etc/systemd/system/
```
3. Отредактировать `conf/config.yml`
4. `systemctl enable rvisionrstintegration`
5. `systemctl start rvisionrstintegration`