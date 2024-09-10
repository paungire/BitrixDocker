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
              - name: SSH Remote Commands
                uses: appleboy/ssh-action@v1.0.3
                with:
                  host: ${{ secrets.MAIN_HOST }}
                  username: ${{ secrets.MAIN_USER }}
                  password: ${{ secrets.MAIN_PASSWORD }}
                  script: |
                    cd ${{ secrets.MAIN_FOLDER }};
                    git checkout main;
                    git pull;

### Правила
- *На каждую новую задачку надо создавать отдельную ветку от dev*
- *По завершении вливать в dev и удалять*
- *Если необходимы быстрые фиксы, можно создать ветку hot_fixes от main*
- *После тестирования на dev вливать на main*
