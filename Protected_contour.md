# Курсовая работа "Защищённый контур" (Профессия Специалист по информационной безопасности)

## Легенда

Компания Х использует корпоративное сетевое приложение "Защищённый контур", позволяющее "безопасно" обмениваться корпоративной информацией (в том числе хранить персональные данные клиентов и информацию, отнесённую к коммерческой тайне). Вам предстоит проанализировать само приложение на уязвимости, а также настройки ОС на серверах, где это приложение функционирует.

Компания разработчик поставляет уже настроенный сервер в виде [OVA](https://drive.google.com/file/d/16uICxKQN91ZqjGa5a9AGml_WqG4oVXzp/view?usp=sharing) (md5 = 5935192d257a003b85584a84dec83367), который (по их заверениям) вполне безопасен и может использоваться "из коробки" (т.е. достаточно импортировать в среду виртуализации и можно запускать в боевой среде).

По информации, полученной от разработчика, цитата: "приложение содержит механизмы безопасности и может использоваться при построении ИСПД до 4 УЗ включительно и систем, обрабатывающих коммерческую тайну".

Документация на приложение разработчиком не предоставлена, поскольку, цитата: "интерфейс интуитивно понятный, обучающих материалов для пользователей не требуется".

Исходные коды так же не предоставлены.

Вас, как специалиста, попросили выполнить анализ данного приложения с точки зрения его реальной защищённости.

## Техническая информация

Цитата:
```
Логин/пароль пользователя в ОС OVA - system/system (после первого запуска необходимо сменить пароль).

Все файлы приложения располагаются в каталоге /opt/sk

Приложение предназначено для работы в ОС Linux x86_64

Приложение настроено в качестве сервиса systemd - sk.service

Приложение запускается на порту 8888 и использует протокол HTTP для своей работы, взаимодействие с приложением осуществляется через веб-интерфейс (посредством веб-браузера с хоста)

При импорте настроек виртуального образа в гипервизор, отличный от Virtual Box, может потребоваться дополнительная настройка сетевого интерфейса виртуального образа с помощью утилиты ip

При первом старте в приложении регистрируется пользователь с логином admin, пароль генерируется автоматически и записывается в файл /opt/sk/password.txt
```

## Задача

Подготовьте отчёт о:
1. Найденных несоответствиях системы требованиям нормативных документов*
1. Найденных "слабостях" (которые могут привести к уязвимостям) и предложение по необходимым мерам для их устранения (если такие меры возможно принять).

Формат отчёта - свободный, но обязательно должен включать указанные выше два пункта.

Примечание*: в части документов нужно:
1. ПДн: реализация мер по обеспечению безопасности в части ИАФ, УПД
1. КТ: "разрешение или запрет доступа к информации, составляющей коммерческую тайну" посредством механизмов разграничения доступа, встроенных в приложение (механизмы ОС и сторонних сервисов рассматривать не нужно)


# Отчет:
### Несоответствия системы требованиям нормативных документов.
- ИАФ.4 ИС не соответствует требованиям в части требования к паролям: любое количество попыток ввода неверного пароля не приводит к блокировке (так же относится к УПД.6).  
- ИАФ.5 ИС не соответствует требованиям в части защиты аутентификационной информации в процессе ее ввода: вводимые символы пароля не маскируются. (пример 1).
- УПД.3. Для доступа к системе используется незащищенный протокол http, данный протокол передает данные в открытом виде и позволяет их, например, перехватить. На примере из Wireshark видно данные пароль и логин. (пример 2).

Пример 1:

![Пример 1](https://github.com/TreninYI/InfoSec/assets/121427985/7525567b-81f5-488b-9d89-194227bcc769)

Пример 2:

![Пример 2](https://github.com/TreninYI/InfoSec/assets/121427985/40f94691-8816-40c6-82b2-95a8635da895)



## Уязвимости системы
1. Отсутствие шифрования траффика.
Приложение использует HTTP протокол без шифрования траффика, в том числе данных для аутентификации. Это значит, что в случае отслеживания траффика, например, с помощью «Wireshark», можно получить логин и пароль в нешифрованном виде и использовать их для аутентифиукации.
Рекомендуется использовать HTTPS протокол для шифрования траффика.
![image1](https://github.com/TreninYI/InfoSec/assets/121427985/8c8903e0-2786-4c9a-acc8-43cd7c09f5db)


2. Отсутствует механизма принудительной смены пароля для пользователя «system».  
Пароль для пользователя system является простым, с помощью какого-либо «инструмента подбора пароля» можно подобрать пароль, используя словарь. А после подбора пароля можно получить доступ к серверу, и подобрать пароль от учетной записи «admin». 
Рекомендуется принудительно заставить пользователя сменить пароль при первом входе.
3. Присутствует автозапоминание вводимых символов в поле пользователь и пароль.
Рекомендуется не использовать автозапоминание форм.
![image2](https://github.com/TreninYI/InfoSec/assets/121427985/ef589f58-82db-4f88-9668-8da6940e6fbf)

 4. Имея ссылку на документ, содержащий коммерческую тайну (http://10.0.0.6:8888/commercial/index.html), можно попасть на данную страницу не проходя авторизацию.  Скачать файл не удается, но удалить его можно не авторизировавшись. Так же можно просто загрузить любой тип файла, не авторизировавшись. (Для примера загружен скрипт одного из предыдущих дз)
Рекомендуется закрыть доступ не авторизованным лицам к адресу http://10.0.0.6:8888/commercial/index.html.
![image3](https://github.com/TreninYI/InfoSec/assets/121427985/35ca2cfb-6ae3-4ac5-87ae-a8349e6a6389)
![image3](https://github.com/TreninYI/InfoSec/assets/121427985/5e8612be-e9b3-43c9-a952-247ab25109e8)



