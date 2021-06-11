## Laboratory work IX

Данная лабораторная работа посвещена изучению процесса создания артефактов на примере **Github Release**

```sh
$ open https://help.github.com/articles/creating-releases/
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab09** на сервисе **GitHub**
- [x] 2. Ознакомиться со ссылками учебного материала
- [x] 3. Получить токен для доступа к репозиториям сервиса **GitHub**
- [x] 4. Выполнить инструкцию учебного материала
- [x] 5. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
# Устанавливаем значение переменных окружения
# Указываем токен с доступом к репозиторию (и дополнительно к gist)
$ export GITHUB_TOKEN=********************************
# Указываем имя пользователя Github
$ export GITHUB_USERNAME=tishchka
# Указываем используемый пакетный менеджер
$ export PACKAGE_MANAGER=apt
# Указываем, в какой утилите будет создаваться ключ
$ export GPG_PACKAGE_NAME=gpg
```

```sh
# for *-nix system
# Устанавливаем утилиту xclip, предоставляющую доступ к буферу обмена Х из коммандной строки
$ $PACKAGE_MANAGER install xclip
# Создаем комадну gsed и связываем ее с sed (потоковым редактором)
$ alias gsed=sed
# Связывание команды pbcopy с записью данных в секцию clipboard
$ alias pbcopy='xclip -selection clipboard'
# Связывание команды pbpaste с чтением данных из секции clipboard
$ alias pbpaste='xclip -selection clipboard -o'
```

```sh
# Переходим в рабочую директорию
$ cd ${GITHUB_USERNAME}/workspace
# Запоминаем текущий каталог в виртуальном стеке каталогов
$ pushd .
# Исполняем команду из файла scripts/activate
$ source scripts/activate
# Скачивание и установка пакета для Go, для работы с релизами Github (github-release)
$ go get github.com/aktau/github-release
```

```sh
# Копируем файлы lab08 из удаленного репозитория github в папку projects/lab09
$ git clone https://github.com/${GITHUB_USERNAME}/lab08 projects/lab09
# Переход в директорию lab09
$ cd projects/lab09
# Удаление ссылки на старый удаленный репозиторий
$ git remote remove origin
# Добавляем ссылку на новый репозиторий
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab09
```

```sh
# Редактируем картинку для Travis
$ gsed -i 's/lab08/lab09/g' README.md
```

```sh
# Устанавливаем программу GPG для шифрования информации и создания электронных цифровых подписей
$ $PACKAGE_MANAGER install ${GPG_PACKAGE_NAME}
# Проверяем на наличие ранее созданных ключей(подписей), путем показа существующих ключей
$ gpg --list-secret-keys --keyid-format LONG
# Гененрируем ключ, выбираем нужные параметры, которые предложит программа
$ gpg --full-generate-key
# Проверяем, что ключ создался
$ gpg --list-secret-keys --keyid-format LONG
$ gpg -K ${GITHUB_USERNAME}
# Создание переменной окружения с сохраненным в ней публичным ключом
$ GPG_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep ssb | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}')
# Создание переменной окружения с сохраненным в ней секретным ключом
$ GPG_SEC_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep sec | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}')
$ gpg --armor --export ${GPG_KEY_ID} | pbcopy
$ pbpaste
$ open https://github.com/settings/keys
# Добавляем секретный ключ в настройки Github
$ git config user.signingkey ${GPG_SEC_KEY_ID}
$ git config gpg.program gpg
```

```sh
# Настраиваем скрипт для добавления сообщения к тегу
$ test -r ~/.bash_profile && echo 'export GPG_TTY=$(tty)' >> ~/.bash_profile
$ echo 'export GPG_TTY=$(tty)' >> ~/.profile
```

```sh
# Генерируем пакет с раширением .tar.gz
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
$ cmake --build _build --target package
```

```sh
# Логинемся в travis
$ travis login --auto
$ travis enable
```

```sh
# Редактируем tag
$ git tag -s v0.1.0.0
# Верефецируем тег
$ git tag -v v0.1.0.0
# Деменстрируем изменения
$ git show v0.1.0.0
# Отправляем измения на Github
$ git push origin master --tags
```

```sh
# Выводим версию github-release
$ github-release --version
$ github-release info -u ${GITHUB_USERNAME} -r lab09
# Создание релиза
$ github-release release \
    --user ${GITHUB_USERNAME} \
    --repo lab09 \
    --tag v0.1.0.0 \
    --name "libprint" \
    --description "my first release"
```

```sh
$ export PACKAGE_OS=`uname -s` PACKAGE_ARCH=`uname -m` 
$ export PACKAGE_FILENAME=print-${PACKAGE_OS}-${PACKAGE_ARCH}.tar.gz
# Добавление артефакта с указанием ОС и архитектуры, на которых происходила компиляция библиотеки
$ github-release upload \
    --user ${GITHUB_USERNAME} \
    --repo lab09 \
    --tag v0.1.0.0 \
    --name "${PACKAGE_FILENAME}" \
    --file _build/*.tar.gz
```

```sh
$ github-release info -u ${GITHUB_USERNAME} -r lab09
# Скачивание релиза
$ wget https://github.com/${GITHUB_USERNAME}/lab09/releases/download/v0.1.0.0/${PACKAGE_FILENAME}
# Просмотр содержимого архива
$ tar -ztf ${PACKAGE_FILENAME}
```

## Report

```sh
$ popd
$ export LAB_NUMBER=09
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gistup -m "lab${LAB_NUMBER}"
```

## Links

- [Create Release](https://help.github.com/articles/creating-releases/)
- [Get GitHub Token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
- [Signing Commits](https://help.github.com/articles/signing-commits-with-gpg/)
- [Go Setup](http://www.golangbootcamp.com/book/get_setup)
- [github-release](https://github.com/aktau/github-release)

```
Copyright (c) 2015-2021 The ISC Authors
```
