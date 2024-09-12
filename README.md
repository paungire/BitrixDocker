# Docker образ для bitrix, с установленной node-js и @bitrix/cli <br>
~ Рабочая папка **/src**, файлы которые могут потребоваться:
  - `bitrixsetup.php` - для разработки с нуля
  - `restore.php` - для развертывания сайта с последующими доработками

### Запуск проекта
  - В файлике `docker-compose.yml` определить хост, имя пользователя, пароль и название бд
  - Запустить
    
        docker-compose up
  - При установке указать эти данные

# Первоначальная настройка проекта Bitrix с Git
### Инструкция для деплоя проекта через гит
  - Создать доп ветку `dev`
  - Заполнить все секретные переменные `Settings -> Security -> Actions -> Variable`

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

        name: PushMain
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
              # Setup key
              - run: set -eu
              - run: mkdir "$HOME/.ssh"
              - run: echo "${{ secrets.MAIN_KEY }}" > "$HOME/.ssh/key"
              - run: chmod 600 "$HOME/.ssh/key"
              - run: rsync -e "ssh -i $HOME/.ssh/key -o StrictHostKeyChecking=no" --archive --compress --delete --exclude='/bitrix' . ${{ secrets.MAIN_USER }}@${{ secrets.MAIN_HOST }}:${{ secrets.MAIN_FOLDER }}
              
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
