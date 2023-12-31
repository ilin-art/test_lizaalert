 # Описание модели данных

Звездочкой (*) отмечены обязательные поля.
В качестве уникального идентификатора, первичного ключа моделей используется автоматически создаваемое Django для моделей поле **id** типа BigAutoField.

## Аккаунт

Используется встроенная модель [User](https://docs.djangoproject.com/en/3.2/ref/contrib/auth/) из django.contrib.auth.models.
Поля email, first_name и last_name заполнять данными из user_info Яндекс ID: 
`first_name: user_info['first_name']`, `last_name: user_info['last_name']`, `email: user_info['default_email']`.
Полю username принять решение заполнять яндекс.id или login. Сохранить возможность изменения пользователем полей first_name и last_name в профиле пользователя.
Роли (Permission) назначаются c использованием моделей и методов, предоставляемых django.contrib.auth.models.

### Используемые роли
**Главный администратор (main_admin)** - (все права + может удалять курсы). Выборочно назначает роли и дает им права.

**Администратор (admin)** — загружает материалы, добавляет студентов, подтверждает статусы студентов, может переключаться на "вид студента".
Администратор может поставить ограничение для преподавателей:
- преподаватель может редактировать другие направления;
- преподаватель может редактировать только в рамках своего направления.

**Преподаватель (teacher)** — загружает материалы, видит, кто проходит курс, и кто его уже прошёл. 
Выполняет проверку тестов.

**Пользователь (volunteer)** — доброволец, который проходит курсы.

У пользователя есть признак «Направление» (может быть несколько направлений) и уровень «Новичок/Бывалый/Профессионал». 
Пользователь может по ним фильтровать доступные курсы + видит все открытые курсы.

При первоначальной регистрации признак «Направление» пуст. 
По умолчанию при первой регистрации для пользователя устанавливается уровень «Новичок».
Для пользователей с уровнем «Новичок» одобрение (активация) учетной записи автоматическая, 
без подтверждения администратора.
Новому пользователю открыты все курсы с уровнем «Новичок».

Если новый пользователь вводит уровень «Бывалый/Профессионал» — это отправляется на подтверждение администратору, 
и только после подтверждения администратором указанный уровень закрепляется в профиле.
Предусмотреть сброс флага подтверждения уровня пользователя и отправку уведомления администратору 
в случае изменения пользователем в профиле уровня на более высокий.

Регион - важная информация для пользователя, по нему открываются курсы этого региона.

_Примечания:_

_1. Следует предусмотреть вывод подтверждения пользователю в случае попытки установить более низкий
уровень, например Бывалый при текущем уровне Профессионал._
_2. Обсудить с заказчиком необходимость запрета смены региона в профиле пользователя, например, 
пользователь может изменить регион для доступа к курсам, которые отсутствуют в его регионе или определить 
процедуру смены региона пользователем._


### Аутентификация и авторизация

Аутентификация пользователя на платформе с помощью OAuth2 через Яндекс ID.
Дальнейшая авторизация на платформе с использованием JWT токенов (access_token и refresh_token), 
формируемых после успешной аутентификации через OAuth2.
Токены Яндекс ID не храним, поскольку обращение к сервисам Яндекса не предполагается.

Авторизация в системе с использованием JWT токенов (access_token и refresh_token). 
Реализовать с использованием [плагина](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/index.html) аутентификации JWT для Django REST Framework.
Определиться с плагином для аутентификации с помощью OAuth2 через Яндекс ID.
Роли передаются в виде списка в поле roles payload access-токена.
E-mail пользователя берется из user_info Яндекс ID и не может быть изменен пользователем в профиле самостоятельно.
В этом случае нет необходимости использовать Celery для подтверждения e-mail.


**Для всех таблиц ниже используется отдельная схема БД Postgresql (например, content)**

## Волонтер

Имя модели - **Volunteer**
Таблица БД - **volunteers**

Поля модели:

- **user*** - пользователь, тип OneToOneField, отношение один-к-одному к модели **User**
- **phone_number*** - номер телефона, тип PhoneNumberField*
- **birth_date*** - дата рождения, тип DateField
- **location*** - географический регион, тип ForeignKey (один ко многим) к модели **Location** (Географический регион)
- **direction** - направление, тип ForeignKey (один ко многим) к модели **Direction** (Направление)
- **call_sign** - позывной на форуме, тип CharField, длина 50
- **photo** - путь к фотографии волонтера и ее миниатюре, тип ThumbnailerImageField**
- **level** - уровень, тип ManyToManyField (многие ко многим) к модели **Level** (Уровень), через модель **VolunteerLevel**
- **badges** - значки, тип ManyToManyField (многие ко многим) к модели **Badge** (Значки), через модель **VolunteerBadge**
- **courses** - курсы, тип ManyToManyField (многие ко многим) к модели **Сourse** (Курсы), через модель **VolunteerCourse**
- **created_at*** - дата создания записи об учащемся, тип DateTimeField, сделать автоматическое проставление текущего времени
- **updated_at*** - дата обновления записи об учащемся, тип DateTimeField с таймстамп, сделать автоматическое проставление текущего времени

*Использовать библиотеку [django-phonenumber-field[phonenumbers]==6.0.0](https://pypi.org/project/django-phonenumber-field/6.0.0/)
**Использовать библиотеку [easy-thumbnails](https://pypi.org/project/easy-thumbnails/2.7.2/)

## Уровень

Имя модели - **Level**
Таблица БД - **levels**

Поля модели:

- **name*** - наименование уровня, тип CharField, длина 20, (выбор из Новичок, Бывалый, Профессионал)
- **description** - описание уровня и условий его достижения, тип TextField

## Учащиеся <-> Уровень (многие-ко-многим)

Имя модели - **VolunteerLevel**
Таблица БД  - **volunteers_levels**

Поля модели:

- **volunteer*** - тип ForeignKey к модели волонтера **Volunteer**
- **level*** - тип ForeignKey к модели уровень **Level**
- **confirmed*** - статус подтверждения волонтера Администратором, тип BooleanField, по умолчанию False
- **who_confirmed** - кто подтвердил статус пользователя, тип ForeignKey (один ко многим) к модели **User**
- **created_at*** - дата создания записи связи учащийся - уровень, тип DateTimeField, сделать автоматическое проставление текущего времени
- **updated_at*** - дата обновления записи связи учащийся - уровень, тип DateTimeField с таймстамп, сделать автоматическое проставление текущего времени

Создать уникальный индекс (UniqueConstraint) по полям [volunteer, level] чтобы конкретный уровень был привязан к пользователю только один раз.

## Географический регион

Имя модели - **Location**
Таблица БД - **locations**

Поля модели:

- **code** - код региона, тип SmallIntegerField
- **region*** - наименование региона, тип CharField, длина 120

## Направление

Имя модели - **Direction**
Таблица БД - **directions**

Поля модели:

- **title*** - наименование направления, тип CharField, длина 120
- **description** - описание направления, тип TextField

## Значок

Имя модели - **Badge**
Таблица БД - **badges**

Поля модели:

- **name*** - наименование значка, тип CharField, длина 40
- **description** - описание значка и условий его получения, тип TextField

## Учащиеся <-> Значки (многие-ко-многим)

Имя модели - **VolunteerBadge**
Таблица БД  - **volunteers_badges**

Поля модели:

- **volunteer*** - тип ForeignKey к модели волонтера **Volunteer**
- **badge*** - тип ForeignKey к модели значка **Badge**
- **created_at*** - дата создания записи о связи волонтер - значок, тип DateTimeField, сделать автоматическое проставление текущего времени

## Учащиеся <-> Курсы (многие-ко-многим)

Имя модели - **VolunteerCourse**
Таблица БД  - **volunteers_courses**

Поля модели:

- **volunteer*** - тип ForeignKey к модели волонтер **Volunteer**
- **course*** - тип ForeignKey к модели курса **Course**
- **status*** - статус курса, тип CharField, длина 20, выбор из Активный, Пройден, Вы записаны
- **assessment*** - оценка по результатам прохождения курса, тип FloatField, валидация диапазона: от 0 до 100,0. Если курс не проходился, то 0.
- **created_at*** - дата создания записи о связи волонтер - курс, тип DateTimeField, сделать автоматическое проставление текущего времени

Создать уникальный индекс (UniqueConstraint) по полям [volunteer, course, status] чтобы конкретный курс был привязан к пользователю только с одним из возможных статусов.

Реализовать метод модели создания записей в таблице volunteers_lessons по всем урокам курса при изменении status на Вы записаны.

## Сертификат

Имя модели - **Sertificate**
Таблица БД - **sertificates** 

Поля модели:

- **volunteer*** - тип ForeignKey к модели волонтер **Volunteer**
- **course*** - тип ForeignKey к модели курса **Course**
- **final_assessment*** - итоговая оценка по результатам прохождения курса, тип FloatField, валидация диапазона: от (_уточнить нижний процент_) до 100,0.
- **sertificate_path*** - путь к файлу с сертификатом, тип FileField
- **issued** - руководитель курса, кто выдал сертификат, тип ForeignKey (один ко многим) к модели **User**
- **created_at*** - дата создания записи о сертификате, тип DateTimeField, сделать автоматическое проставление текущего времени

## Курс

Имя модели - **Course**
Таблица БД - **courses**

Поля модели:

- **title*** - название курса, тип CharField, длина 120
- **direction*** - направление, тип CharField, длина 60, выбор из заданного списка (_уточнить список направлений_)
- **location** - географический регион, ManyToManyField (многие ко многим) к модели **Location** (Географический регион), через модель **CourseLocation**
- **level*** - уровень, тип ForeignKey (один ко многим) к модели **Level** (Уровень)
- **format*** - формат курса, длина 60, выбор из заданного списка (_уточнить список форматов_)
- **start_date*** - дата начала курса, тип DateField
- **cover_path** - путь к обложке курса, тип FileField
- **short_description*** - краткое описание курса, тип CharField, длина 120
- **full_description*** - полное описание курса, тип TextField
- **skills** - тип ManyToManyField (многие ко многим) к модели **Skill** (Навыки), через модель **CourseSkill**
- **faq** - тип ManyToManyField (многие ко многим) к модели **FAQ** (Часто задаваемые вопросы), через модель **CourseFAQ**
- **contents** - ManyToManyField (многие ко многим) к модели **Chapter** (Главы), через модель **CourseChapter**
- **duration** - продолжительность курса (вычисляемое поле на основании информации о продолжительности всех уроков курса), тип SmallIntegerField
- **number_of_classes** - количество занятий (вычисляемое поле на основании количества уроков в курсе), тип SmallIntegerField
- **attempts_limit*** - количество попыток на прохождение курса. Если задано 0, то количество попыток не ограничено. Тип SmallIntegerField, валидация от 0 до 10 (_уточнить возможную верхнюю границу диапазона_)
- **user_created*** - пользователь создавший курс, тип ForeignKey (один ко многим) к модели **User**
- **user_modified*** - последний редактировавший курс, тип ForeignKey (один ко многим) к модели **User**
- **ready*** - статус готовности готов/не готов, тип BooleanField
- **visible*** - статус видимости курса вкл./выкл., тип BooleanField
- **published*** - статус публикации курса, тип BooleanField
- **created_at*** - дата создания записи о курсе, тип DateTimeField, сделать автоматическое проставление текущего времени
- **updated_at*** - дата обновления записи о курсе, тип DateTimeField, сделать автоматическое проставление текущего времени

## Навыки

Имя модели - **Skill**
Таблица БД - **skills**

Поля модели:

- **title*** - название навыка, тип CharField, длина 120
- **description*** - описание навыка, тип CharField, длина 255
- **user_created*** - пользователь создавший описание навыка, тип ForeignKey (один ко многим) к модели **User**
- **user_modified*** - последний редактировавший описания навыка, тип ForeignKey (один ко многим) к модели **User**
- **created_at*** - дата создания записи о навыке, тип DateTimeField, сделать автоматическое проставление текущего времени
- **updated_at*** - дата обновления записи о навыке, тип DateTimeField, сделать автоматическое проставление текущего времени

## Курсы <-> Географический регион (многие-ко-многим)

Имя модели - **CourseLocation**
Таблица БД - courses_locations

Поля модели:

- **course*** - тип ForeignKey к модели курса **Course**
- **location*** - тип ForeignKey к модели навыка **Location**
- **created_at*** - дата создания записи о связи курс - географический регион, тип DateTimeField, сделать автоматическое проставление текущего времени

Создать уникальный индекс (UniqueConstraint) по полям [course, location], чтобы конкретный курс был привязан к конкретному региону один раз.

## Курсы <-> Навыки (многие-ко-многим)

Имя модели - **CourseSkill**
Таблица БД - courses_skills

Поля модели:

- **course*** - тип ForeignKey к модели курса **Course**
- **skill*** - тип ForeignKey к модели навыка **Skill**
- **created_at*** - дата создания записи о связи курс - навык, тип DateTimeField, сделать автоматическое проставление текущего времени

## FAQ

Имя модели - **FAQ**
Таблица БД - **faq**

Поля модели:

- **question*** - вопрос, тип CharField, длина 120
- **answer*** - ответ на вопрос, тип CharField, длина 255
- **user_created*** - пользователь создавший ответ, тип ForeignKey (один ко многим) к модели **User**
- **user_modified*** - пользователь последний редактировавший ответ, тип ForeignKey (один ко многим) к модели **User**
- **created_at*** - дата создания записи об ответе на вопрос, тип DateTimeField, сделать автоматическое проставление текущего времени
- **updated_at*** - дата обновления записи об ответе на вопрос, тип DateTimeField, сделать автоматическое проставление текущего времени

## Курсы <-> FAQ (многие-ко-многим)

Имя модели - **CourseFAQ**
Таблица БД - **courses_faq**

Поля модели:

- **course*** - тип ForeignKey к модели курса **Course**
- **faq*** - тип ForeignKey к модели часто задаваемых вопросов **FAQ**
- **created_at*** - дата создания записи о связи курс - ответ на вопрос, тип DateTimeField, сделать автоматическое проставление текущего времени

## Глава

Имя модели - **Chapter**
Таблица БД - **capters**

Поля модели:

- **title** - название главы, тип CharField, длина 120 _(необязательно, если уроки не делятся на главы)_
- **lessons** - тип ManyToManyField (многие ко многим) к модели **Lesson** (Урок), через модель **СhapterLesson**
- **user_created*** - пользователь создавший главу, тип ForeignKey (один ко многим) к модели **User**
- **user_modified*** - пользователь последний редактировавший главу, тип ForeignKey (один ко многим) к модели **User**
- **created_at*** - дата создания записи о главе, тип DateTimeField, сделать автоматическое проставление текущего времени
- **updated_at*** - дата обновления записи о главе, тип DateTimeField, сделать автоматическое проставление текущего времени

Примечание: _поле **lessons** тип ManyToManyField имеет смысл, только если предполагается, что одни и те же уроки 
могут быть привязаны к разным курсам (главам в рамках курса). Например, какие-либо вводные уроки или правила безопасности. 
Если нет, то достаточно будет типа ForeignKey от каждого урока к главе (один-ко-многим)_

## Курсы <-> Главы (многие-ко-многим)

Имя модели - **CourseChapter**
Таблица БД - **courses_chapters**

Поля модели:

- **course*** - тип ForeignKey к модели курса **Course**
- **chapter*** - тип ForeignKey к модели главы урока **Chapter**
- **order_number*** - порядковый номер главы, тип SmallIntegerField, проверка нижнего дипазона >= 1
- **created_at*** - дата создания записи о связи курс - глава урока, тип DateTimeField, сделать автоматическое проставление текущего времени

Создать уникальный индекс (UniqueConstraint) по полям [course, chapter, order_number] чтобы конкретная глава в рамках курса имела уникальный порядковый номер.
_Требует уточнения, так как в случае множественного изменения порядка глав в курсе изменения необходимо будет проводить в одной транзакции с отключенным контролем конфиликтов до завершения транзакции_

## Уроки

Имя модели - **Lesson**
Таблица БД - **lessons**

Поля модели:

- **title*** - название урока, тип CharField, длина 120
- **description** - описание урока, тип TextField (_требует уточнения: как хранить форматированный текст в БД и в каком виде отдавать в frontend_)
- **type*** - тип урока, тип CharField, длина 20, выбор из перечня Урок, Видеоурок, Вебинар, Тест (Квиз)
- **tags** - ключевые слова урока, тип CharField, длина 255
- **duration*** - продолжительность урока, минут, тип SmallIntegerField
- **user_created*** - пользователь создавший урок, тип ForeignKey (один ко многим) к модели **User**
- **user_modified*** - последний редактировавший урок, тип ForeignKey (один ко многим) к модели **User**
- **ready*** - статус готовности готов/не готов, тип BooleanField
- **visible*** - статус видимости урока вкл./выкл., тип BooleanField
- **published*** - статус публикации урока, опубликован - да/нет,тип BooleanField
- **additional*** - дополнительный урок - да/нет, тип BooleanField
- **diploma*** - дипломный урок - да/нет, тип BooleanField
- **created_at*** - дата создания записи об уроке, тип DateTimeField, сделать автоматическое проставление текущего времени
- **updated_at*** - дата обновления записи об уроке, тип DateTimeField, сделать автоматическое проставление текущего времени

## Главы <-> Уроки (многие-ко-многим)

Имя модели - **ChapterLesson**
Таблица БД - **chapters_lessons**

Поля модели:

- **chapter*** - тип ForeignKey к модели главы урока **Chapter**
- **lesson*** - тип ForeignKey к модели курса **Lesson**
- **order_number*** - порядковый номер урока в главе, тип SmallIntegerField, проверка нижнего дипазона >= 1
- **created_at*** - дата создания записи о связи курс - глава урока, тип DateTimeField, сделать автоматическое проставление текущего времени

Создать уникальный индекс (UniqueConstraint) по полям [chapter, lesson, order_number] чтобы конкретный урок в рамках главы имел уникальный порядковый номер. Необходимо для сортировки.
_Требует уточнения, так как в случае множественного изменения порядка уроков в главе или добавления урока между имеющимися уроками изменения необходимо будет проводить по всем урокам в одной транзакции с отключенным контролем конфиликтов до завершения транзакции_

## Видеоурок

Имя модели - **VideoLesson**
Таблица БД - **video_lessons**

Поля модели:

- **lesson*** - урок с описанием, тип ForeignKey (один-ко-многим) к модели урока **Lesson**
- **video*** - путь к файлу с видео, тип FileField (_требует уточнения_)
- **user_created*** - пользователь создавший видеоурок (загрузивший видео), тип ForeignKey (один ко многим) к модели **User**
- **user_modified*** - последний редактировавший видеоурок (обновивший видео), тип ForeignKey (один ко многим) к модели **User**
- **created_at*** - дата создания записи о видеоуроке, тип DateTimeField, сделать автоматическое проставление текущего времени
- **updated_at*** - дата обновления записи о видеоуроке, тип DateTimeField, сделать автоматическое проставление текущего времени

## Вебинар

Имя модели - **Webinar**
Таблица БД - **webinars**

Поля модели:

- **lesson*** - урок с описанием, тип ForeignKey (один-ко-многим) к модели урока **Lesson**
- **url_webinar*** - ссылка на вебинар, тип URLField
- **start_date*** - дата и время начала вебинара, тип DateTimeField
- **user_created*** - пользователь создавший ссылку на вебинар, тип ForeignKey (один ко многим) к модели **User**
- **user_modified*** - последний редактировавший ссылку на вебинар, тип ForeignKey (один ко многим) к модели **User**
- **created_at*** - дата создания записи о ссылке на вебинар, тип DateTimeField, сделать автоматическое проставление текущего времени
- **updated_at*** - дата обновления записи о ссылке на вебинар, тип DateTimeField, сделать автоматическое проставление текущего времени

## Волонтер <-> Урок (многие-ко-многим)

Имя модели - **VolunteerLesson**
Таблица БД - **volunteers_lessons**

Поля модели:

- **volunteer*** - тип ForeignKey к модели волонтера **Volunteer**
- **lesson*** - тип ForeignKey к модели урока **Lesson**
- **passed*** - статус прохождения урока, тип BooleanField, по умолчанию False
- **start_date** - дата начала (первое отрытие урока), тип DateTimeField
- **end_date** - дата завершения урока (переход к следующему уроку или тесту), тип DateTimeField
- **created_at*** - дата создания записи о связи волонтер - урок, тип DateTimeField, сделать автоматическое проставление текущего времени
- **updated_at*** - дата обновления записи о связи волонтер - урок, тип DateTimeField, сделать автоматическое проставление текущего времени

Создать уникальный индекс (UniqueConstraint) по полям [volunteer, lesson, passed] чтобы конкретный урок для пользователя имел уникальный статус прохождения. 

При активации курса с помощью метода модели **VolunteerCourse** инициировать создание записей в БД по всем урокам курса с заполнением поля passed значением по умолчанию. 
