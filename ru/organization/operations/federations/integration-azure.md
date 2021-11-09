# Аутентификация с помощью Azure Active Directory

С помощью [федерации удостоверений](../../add-federation.md) вы можете использовать [Azure Active Directory](https://azure.microsoft.com/ru-ru/services/active-directory/) (далее — Azure AD) для аутентификации пользователей в организации.

Настройка аутентификации состоит из следующих этапов:

1. [Создание и настройка SAML-приложения в Azure AD](#azure-settings).

1. [Создание и настройкa федерации в {{org-full-name}}](#yc-settings).

1. [Настройка системы единого входа (SSO)](#sso-settings).

1. [Проверка аутентификации](#test-auth).

## Перед началом {#before-you-begin}

Чтобы воспользоваться инструкциями в этом разделе, вам понадобится учетная запись Azure с активной подпиской.

## Создание и настройка SAML-приложения в Azure AD {#azure-settings}

### Создайте SAML-приложение и скачайте сертификат

В роли поставщика удостоверений (IdP) выступает SAML-приложение в Azure AD. Начните создавать приложение и скачайте сертификат:

1. Перейдите на [портал Azure AD](https://portal.azure.com/).

1. В разделе **Службы Azure** выберите **Azure Active Directory**.

1. На панели слева выберите **Корпоративные приложения**.

1. Нажмите **Новое приложение**.

1. На странице **Обзор коллекции Azure AD** нажмите **Создайте собственное приложение**.

1. В открывшемся окне:

   1. Введите название приложения.

   1. Выберите вариант **Integrate any other application you don't find in the gallery (Non-gallery)**.

   1. Нажмите кнопку **Создать**.

1.  На открывшейся странице **Обзор** на панели слева выберите **Единый вход**.

1. Выбрите метод единого входа **SAML**.

1. На странице **Вход на основе SAML** в разделе **3. Сертификат подписи SAML** скачайте сертификат (Base64). С помощью него поставщик удостоверений подписывает сообщение о том, что пользователь прошел аутентификацию.

Не закрывайте страницу: данные IdP-сервера понадобятся при [создании и настройке федерации](#yc-settings).

### Добавьте пользователей

Добавьте пользователей на IdP-сервер:

1. Перейдите на страницу [Корпоративные приложения](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps).

1. Выберите созданное SAML-приложение.

1. На панели слева выберите **Пользователи и группы**.

1. Нажмите **Добавить пользователя или группу**.

1. В поле **Пользователи** нажмите **Не выбрано**.

1. В открывшемся окне отметьте пользователей и нажмите кнопку **Выбрать**.

1. Нажмите кнопку **Назначить**.

## Создание и настройкa федерации в {{org-full-name}} {#yc-settings}

### Создайте федерацию

{% list tabs %}

- Консоль управления

  1. Перейдите в сервис [{{org-full-name}}]({{link-org-main}}).

  1. На панели слева выберите раздел [Федерации]({{link-org-federations}}) ![icon-federation](../../../_assets/organization/icon-federation.png).

  1. Нажмите кнопку **Создать федерацию**.

  1. Задайте имя федерации. Имя должно быть уникальным в каталоге.

  1. При необходимости добавьте описание.

  1. В поле **Время жизни cookie** укажите время, в течение которого браузер не будет требовать у пользователя повторной аутентификации.

  1. В поле **IdP Issuer** вставьте ссылку, которая указана в поле **Идентификатор Azure AD** на странице **Вход на основе SAML** в Azure AD. Формат ссылки:

      ```
      https://sts.windows.net/<ID SAML-приложения>/
      ```

  1. В поле **Ссылка на страницу для входа в IdP** вставьте ссылку, которая указана в поле **URL-адрес входа** на странице **Вход на основе SAML** в Azure AD. Формат ссылки:

      ```
      https://login.microsoftonline.com/<ID SAML-приложения>/saml2
      ```

  1. Включите опцию **Автоматически создавать пользователей**, чтобы пользователь после аутенцификации автоматически добавлялся в организацию. Если опция отключена, федеративных пользователей потребуется [добавить вручную](../../add-account.md#add-user-sso).

  1. Нажмите кнопку **Создать федерацию**.

- CLI

    {% include [cli-install](../../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

    1. Посмотрите описание команды создания федерации:

        ```
        yc organization-manager federation saml create --help
        ```

    1. Создайте федерацию:

        ```
        yc organization-manager federation saml create --name my-federation \
            --organization-id abcdef12a1abcdefgh1a \
            --auto-create-account-on-login \
            --cookie-max-age 12h \
            --issuer "https://sts.windows.net/1a1234a1-123-12ab-a12a1-ab1a1a12a123/" \
            --sso-binding POST \
            --sso-url "https://login.microsoftonline.com/1a1234a1-123-12ab-a12a1-ab1a1a12a123/saml2"
        ```

        Где:

        * `name` — имя федерации. Имя должно быть уникальным в каталоге.

        * `organization-id` — идентификатор организации. 

        * `auto-create-account-on-login` — флаг, который активирует автоматическое создание новых пользователей в облаке после аутентификации на IdP-сервере. 
        Опция упрощает процесс заведения пользователей, но созданный таким образом пользователь не сможет выполнять никаких операций с ресурсами в облаке. Исключение — те ресурсы, на которые назначены роли системной группе `allUsers` или `allAuthenticatedUsers`.

            Если опцию не включать, то пользователь, которого не добавили в облако, не сможет войти в консоль управления, даже если пройдет аутентификацию на вашем IdP-сервере. В этом случае вы можете управлять белым списком пользователей, которым разрешено пользоваться ресурсами {{ yandex-cloud }}.

        * `cookie-max-age` — время, в течение которого браузер не должен требовать у пользователя повторной аутентификации.

        * `issuer` — идентификатор IdP-сервера, на котором должна происходить аутентификация.

            Используйте ссылку, которая указана в поле **Идентификатор Azure AD** на странице **Вход на основе SAML** в Azure AD. Формат ссылки:
            ```
            https://sts.windows.net/<ID SAML-приложения>/
            ```

        * `sso-url` — URL-адрес страницы, на которую браузер должен перенаправить пользователя для аутентификации.

            Используйте ссылку, которая указана в поле **URL-адрес входа** на странице **Вход на основе SAML** в Azure AD. Формат ссылки:

            ```
            https://login.microsoftonline.com/<ID SAML-приложения>/saml2
            ```

        * `sso-binding` — укажите тип привязки для Single Sign-on. Большинство поставщиков поддерживают тип привязки `POST`.

- API

    1. [Получите идентификатор каталога](../../../resource-manager/operations/folder/get-id.md), в котором вы будете создавать федерацию.

    1. Создайте файл с телом запроса, например `body.json`:

        ```json
        {
          "folderId": "a1ab1abc12a1a123abcd",
          "name": "my-federation",
          "organizationId": "abcdef12a1abcdefgh1a",
          "autoCreateAccountOnLogin": true,
          "cookieMaxAge":"43200s",
          "issuer": "https://sts.windows.net/1a1234a1-123-12ab-a12a1-ab1a1a12a123/",
          "ssoUrl": "https://login.microsoftonline.com/1a1234a1-123-12ab-a12a1-ab1a1a12a123/saml2",
          "ssoBinding": "POST"
        }
        ```

        Где:

        * `folderId` — идентификатор каталога.

        * `name` — имя федерации. Имя должно быть уникальным в каталоге.

        * `organizationId` — идентификатор организации. 

        * `autoCreateAccountOnLogin` — флаг, который активирует автоматическое создание новых пользователей в облаке после аутентификации на IdP-сервере. Опция упрощает процесс заведения пользователей, но созданный таким образом пользователь не сможет выполнять никаких операций с ресурсами в облаке. Исключение — те ресурсы, на которые назначены роли системной группе `allUsers` или `allAuthenticatedUsers`.

            Если опцию не включать, то пользователь, которого не добавили в облако, не сможет войти в консоль управления, даже если пройдет аутентификацию на вашем IdP-сервере. В этом случае вы можете управлять белым списком пользователей, которым разрешено пользоваться ресурсами {{ yandex-cloud }}.

        * `cookieMaxAge` — время, в течениt которого браузер не должен требовать у пользователя повторной аутентификации.

        * `issuer` — идентификатор IdP-сервера, на котором должна происходить аутентификация.

            Используйте ссылку, которая указана в поле **Идентификатор Azure AD** на странице **Вход на основе SAML** в Azure AD. Формат ссылки:

            ```
            https://sts.windows.net/<ID SAML-приложения>/
            ```

        * `ssoUrl` — URL-адрес страницы, на которую браузер должен перенаправить пользователя для аутентификации.

            Используйте ссылку, которая указана в поле **URL-адрес входа** на странице **Вход на основе SAML** в Azure AD. Формат ссылки:

            ```
            https://login.microsoftonline.com/<ID SAML-приложения>/saml2
            ```

        * `ssoBinding` — укажите тип привязки для Single Sign-on. Большинство поставщиков поддерживают тип привязки `POST`.

    1. Создайте федерацию с помощью метода [create](../../api-ref/Federation/create.md):

        ```bash
        $ curl -X POST \
          -H "Content-Type: application/json" \
          -H "Authorization: Bearer <IAM-токен>" \
          -d '@body.json' \
          https://organization-manager.api.cloud.yandex.net/organization-manager/v1/saml/federations
           {
            "done": true,
            "metadata": {
            "@type": "type.googleapis.com/yandex.cloud.organization-manager.v1.saml.CreateFederationMetadata",
            "federationId": "abcdefg0abc0ab0a00ab"
           },
        ...
        ```

    В ответе, в свойстве `federationId`, будет указан идентификатор созданной федерации, сохраните его. Этот идентификатор понадобится на следующих шагах.

{% endlist %}

### Добавьте сертификаты

При аутентификации у сервиса {{org-name}} должна быть возможность проверить сертификат IdP-сервера. Для этого добавьте [скачанный ранее](#azure-settings) сертификат в федерацию:

{% list tabs %}

- Консоль управления

  1. На панели слева выберите раздел [Федерации]({{link-org-federations}}) ![icon-federation](../../../_assets/organization/icon-federation.png).

  1. Нажмите имя федерации, для которой нужно добавить сертификат.

  1. Внизу страницы нажмите кнопку **Добавить сертификат**.

  1. Введите название и описание сертификата.

  1. Выберите способ добавления сертификата:

      * Чтобы добавить сертификат в виде файла, нажмите **Выбрать файл** и укажите путь к нему.

      * Чтобы вставить скопированное содержимое сертификата, выберите способ **Текст** и вставьте содержимое.

  1. Нажмите кнопку **Добавить**.

- CLI

  {% include [cli-install](../../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  1. Посмотрите описание команды добавления сертификата:

      ```
      yc organization-manager federation saml certificate create --help
      ```

  1. Добавьте сертификат для федерации, указав путь к файлу сертификата:

      ```
      yc organization-manager federation saml certificate create \
        --federation-id abcdefg0abc0ab0a00ab \
        --name "my-certificate" \
        --certificate-file certificate.cer
      ```

- API

  Воспользуйтесь методом [create](../../api-ref/Certificate/create.md) для ресурса [Certificate](../../api-ref/Certificate/index.md):

  1. Сформируйте тело запроса. В свойстве `data` укажите содержимое сертификата:

      ```json
      {
        "federationId": "abcdefg0abc0ab0a00ab",
        "name": "my-certificate",
        "data": "-----BEGIN CERTIFICATE..."
      }
      ```

  2. Отправьте запрос на добавление сертификата:

      ```bash
      $ export IAM_TOKEN=CggaATEVAgA...
      $ curl -X POST \
          -H "Content-Type: application/json" \
          -H "Authorization: Bearer ${IAM_TOKEN}" \
          -d '@body.json' \
          "https://organization-manager.api.cloud.yandex.net/organization-manager/v1/saml/certificates"
      ```

{% endlist %}

{% note tip %}

Чтобы аутентификация не прерывалась в тот момент, когда у очередного сертификата закончился срок действия, рекомендуется добавлять в федерацию несколько сертификатов — текущий и те, которые будут использоваться после текущего. Если один сертификат окажется недействительным, Yandex.Cloud попробует проверить подпись другим сертификатом.

{% endnote %}

### Получите ссылку для входа в консоль

Когда вы настроите аутентификацию с помощью федерации, пользователи смогут войти в консоль управления по ссылке, в которой содержится идентификатор федерации. Эту же ссылку необходимо будет указать при [настройке системы единого входа (SSO)](#sso-settings).

Получите ссылку:

1. Скопируйте идентификатор федерации:

    1. На панели слева выберите раздел [Федерации]({{link-org-federations}}) ![icon-federation](../../../_assets/organization/icon-federation.png).

    1. Скопируйте идентификатор федерации, для которой вы настраиваете доступ.

2. Сформируйте ссылку с помощью полученного идентификатора:

    `{{ link-console-main }}/federations/<ID федерации>`

## Настройка системы единого входа (SSO) {#sso-settings}

### Добавьте ссылку для входа в консоль

Когда вы создали федерацию и получили ссылку для входа в консоль, завершите создание SAML-приложения в Azure AD:

1. Откройте страницу настроек SAML-приложения **Вход на основе SAML**.

1. В разделе **1. Базовая конфигурация SAML** укажите сведения о {{ yandex-cloud }}, выступающем в роли поставщика услуг. 
  
   Для этого в полях **Идентификатор (сущности)** и **URL-адрес ответа (URL-адрес службы обработчика утверждений)** введите полученную ранее ссылку для входа в консоль.

1. Нажмите **Сохранить**. 

### Настройте сопоставление атрибутов пользователей

После аутентификации пользователя IdP-сервер отправит в {{ yandex-cloud }} SAML-сообщение, которое будет содержать:

* информацию об успешной аутентификации;

* атрибуты пользователя, такие как идентификатор Name ID, имя и адрес электронной почты.

Вы можете настроить сопоставление между атрибутами SAML-сообщения и персональными данными, которые хранятся на IdP-сервере. Для этого на странице **Вход на основе SAML** в разделе **2. Утверждения и атрибуты пользователя** нажмите **Изменить**.

Типы персональных данных, которые поддерживает {{ org-full-name }} для Azure AD, приведены ниже.

Данные пользователя | Комментарий | Атрибуты приложений
------------------- | ----------- | -------------------
Уникальный идентификатор пользователя (Name ID) | Обязательный атрибут.<br> По умолчанию в Azure AD в качестве источника атрибута используется User Principal Name (UPN) в формате `<login>_<domain>#EXT#@<supplier>.onmicrosoft.com`. При добавлении пользователей в федерацию вручную такой формат Name ID не поддерживается. Рекомендуется в Azure AD изменить источник атрибута: вместо UPN `user.userprincipalname` выбрать адрес электронной почты `user.mail`.| Утверждение **Уникальный идентификатор пользователя (ID)**
Фамилия | Отображается в сервисах {{yandex-cloud}}. | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`
Имя | Отображается в сервисах {{yandex-cloud}}. | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`
Полное имя | Отображается в сервисах {{yandex-cloud}}.<br>Пример: `Иван Иванов` | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`
Почта | Используется для отправки уведомлений из сервисов {{yandex-cloud}}.<br>Пример:&nbsp;`ivanov@example.com` | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

### Добавьте пользователей в организацию

Если при [создании федерации](#yc-settings) вы не включили опцию **Автоматически создавать пользователей**, федеративных пользователей нужно добавить в организацию вручную.

Для этого вам понадобятся пользовательские Name ID. Их возвращает IdP-сервер вместе с ответом об успешной аутентификации.

{% list tabs %}

- Консоль управления

  1. [Войдите в аккаунт]({{link-passport}}) администратора организации.

  1. Перейдите в сервис [{{org-full-name}}]({{link-org-main}}).

  1. На левой панели выберите раздел [Пользователи]({{link-org-users}}) ![icon-users](../../../_assets/organization/icon-users.png).

  1. В правом верхнем углу нажмите на стрелку возле кнопки **Добавить пользователя**. Выберите пункт **Добавить федеративных пользователей**.

  1. Выберите федерацию, из которой необходимо добавить пользователей.

  1. Перечислите Name ID пользователей, разделяя их переносами строк.

  1. Нажмите кнопку **Добавить**. Пользователи будут подключены к организации.

- CLI

  {% include [cli-install](../../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  1. Посмотрите описание команды добавления пользователей:

      ```
      yc organization-manager federation saml add-user-accounts --help
      ```

  1. Добавьте пользователей, перечислив их Name ID через запятую:

      ```
      yc organization-manager federation saml add-user-accounts --id abcdefg0abc0ab0a00ab \
        --name-ids=alice@example.com,bob@example.com,charlie@example.com
      ```

      Где:

      * `id` — идентификатор федерации.

      * `name-ids` — Name ID пользователей.

- API

  Чтобы добавить пользователей федерации в облако:

  1.  Сформируйте файл с телом запроса, например `body.json`. В теле запроса укажите массив Name ID пользователей, которых необходимо добавить:

      ```json
      {
        "nameIds": [
          "alice@example.com",
          "bob@example.com",
          "charlie@example.com"
        ]
      }
      ```
  1.  Отправьте запрос, указав в параметрах идентификатор федерации вместо `<ID федерации>`:

      ```bash
      $ curl -X POST \
        -H "Content-Type: application/json" \
        -H "Authorization: Bearer <IAM-токен>" \
        -d '@body.json' \
        https://organization-manager.api.cloud.yandex.net/organization-manager/v1/saml/federations/<ID федерации>:addUserAccounts
      ```

{% endlist %}

## Проверка аутентификации {#test-auth}

Когда вы закончили настройку SSO, протестируйте, что все работает:

1. Откройте браузер в гостевом режиме или режиме инкогнито.

1. Перейдите по [ссылке для входа в консоль](#yc-settings), полученной ранее. Браузер должен перенаправить вас на страницу аутентификации Microsoft.

1. Введите данные для аутентификации и нажмите кнопку **Далее**.

После успешной аутентификации IdP-сервер перенаправит вас обратно по ссылке для входа в консоль, а после — на главную страницу консоли. В правом верхнем углу вы cможете увидеть, что вошли в консоль от имени федеративного пользователя.

#### Что дальше {#what-is-next}

* [Назначьте роли добавленным пользователям](../../roles.md#add-role).