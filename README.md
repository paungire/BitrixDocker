# Docker образ для bitrix, с установленной node-js и @bitrix/cli <br>
~ Рабочая папка **/src**, файлы которые могут потребоваться:
  - `bitrixsetup.php` - для разработки с нуля
  - `restore.php` - для развертывания сайта с последующими доработками

### Запуск проекта
  - В файлике `docker-compose.yml` определить хост, имя пользователя, пароль и название бд
  - Запустить
    
        docker-compose up
  - Открыть в браузере `localhost` и выбрать один из нужных файлов
  - При установке указать данные бд указанные в файлике `docker-compose.yml`
  - ##### По окончанию первоначальной установки обязательно удалить папку .git чтобы отвязаться от гита

# Первоначальная настройка проекта Bitrix с Git
### Связывание с Git

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


### Правила
- *На каждую новую задачку надо создавать отдельную ветку от dev*
- *По завершении вливать в dev и удалять*
- *Если необходимы быстрые фиксы, можно создать ветку hot_fixes от main*
- *После тестирования на dev вливать на main*
- *При изменение бд делать дамп находясь внутри папки `/src` через команду:

        docker exec -it <container_id> mysqldump -u <user_name> --password=<password> <db_name> > ./dump.sql

- *Ручной разворот бд из дампа

        mysql -u <db_user> -p <db_name> < ./dump.sql --password='<db_password>'       
