# Docker образ для bitrix, с установленной node-js и @bitrix/cli <br>
~ Рабочая папка **/src**, файлы которые могут потребоваться:
  - `bitrixsetup.php` - для разработки с нуля
  - `restore.php` - для развертывания сайта с последующими доработками

### Запуск проекта
  - Склонируйте данный репозиторий
  - Удалите папку .git в корне
  - Создайте файл `.env`, в нем  требуется определить хост, имя пользователя, пароль и название бд по примеру .env.example
  - Запустите
    
        docker-compose up
  - Откройте в браузере `localhost` и выбрать один из нужных файлов
  - При установке указать данные бд указанные в файлике `.env`

# Первоначальная настройка проекта Bitrix с Git
### Связывание с Git
  - После установки системы, нужно проинициализировать репозиторий

        git init
    
  - И связать его с git

        git remote add origin ****

  - Обновить локальные данные

        git pull
    
  - Сделать первый комит

        git commit -m "Create repository"



### Инструкция для деплоя проекта через гит
  
  - Заполнить все секретные переменные `Settings -> Security -> Actions -> Secrets`

      | Name          | Description                 |
      | -----------   | -----------                 |
      | MAIN_HOST     | Адрес хоста sftp            |
      | MAIN_USER     | Имя пользователя sftp       |
      | MAIN_PASSWORD | Пароль пользователя sftp    |
      | MAIN_FOLDER   | Папка проекта               |
      | DB_USER       | Имя пользователя бд         |
      | DB_DATABASE   | Название базы данных        |
      | DB_PASSWORD   | Пароль пользователя бд      |

  - Создать событие в `actions`

        name: Deploy
        on:
          push:
            branches: ["main"]
          pull_request:
            branches: ["main"]
        
        jobs:
          deploy:
            runs-on: ubuntu-latest
            steps:
              - uses: actions/checkout@v2
              # Setup key and update files
              - run: set -eu
              - run: mkdir "$HOME/.ssh"
              - run: echo "${{ secrets.MAIN_KEY }}" > "$HOME/.ssh/key"
              - run: chmod 600 "$HOME/.ssh/key"
              - run: rsync -e "ssh -i $HOME/.ssh/key -o StrictHostKeyChecking=no" --archive --compress --delete --exclude='/bitrix' . ${{ secrets.MAIN_USER }}@${{ secrets.MAIN_HOST }}:${{ secrets.MAIN_FOLDER }}
              # Update db by ssh client
              - name: SSH Remote Commands
                uses: appleboy/ssh-action@v1.0.3
                with:
                  host: ${{ secrets.MAIN_HOST }}
                  username: ${{ secrets.MAIN_USER }}
                  key: ${{ secrets.MAIN_KEY }}
                  script: |
                    cd ${{ secrets.MAIN_FOLDER }}
                    mysql -u ${{ secrets.DB_USER }} -p ${{ secrets.DB_DATABASE }} < ./dump.sql --password='${{ secrets.DB_PASSWORD }}'
                    rm -rf ${{ secrets.MAIN_FOLDER }}/bitrix/cache/*

### Для https:// соединения
- Создать сертификат

        openssl req -x509 -out localhost.crt -keyout localhost.key \                                                 
        -newkey rsa:2048 -nodes -sha256 \
        -subj '/CN=localhost' -extensions EXT -config <( \
         printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
- Прописать в `docker-compose.yml` nginx -> ports :
  
        - 443:443
  
- Прописать в `docker-compose.yml` nginx -> volumes :
  
      - ./nginx/localhost.crt:/etc/nginx/localhost.crt
      - ./nginx/localhost.key:/etc/nginx/localhost.key

- Прописать в `nginx.conf` http -> server :

        listen 443 ssl http2;
        ssl_certificate /etc/nginx/localhost.crt;
        ssl_certificate_key /etc/nginx/localhost.key;

### Правила
- *На каждую новую задачку надо создавать отдельную ветку от dev*
- *По завершении вливать в dev и удалять*
- *Если необходимы быстрые фиксы, можно создать ветку hot_fixes от main*
- *После тестирования на dev вливать на main*
- *При изменение бд делать дамп находясь внутри папки `/src` через команду:

        docker exec -it <container_id> mysqldump -u <user_name> --password=<password> <db_name> > ./dump.sql

- *Ручной разворот бд из дампа (*на Windows флаг `-it` менять на `-i`*)

        docker exec -it <container_id> mysql -u <db_user> -p <db_name> < ./dump.sql --password='<db_password>'

###### Дампы с разных ОС отличаются, для просмотра используется Files changed из Github

###### При локальном развертовании dump.sql читстить кэш битрикса
