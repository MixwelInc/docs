# Обновление версии {{ PG }}

Вы можете обновить кластер {{ mpg-name }} до любой поддерживаемой версии.

{% note info %}

* Недоступно обновление версий кластера {{ mpg-name }}, оптимизированных для работы с системой <q>1С:Предприятие</q>. Название таких версий заканчивается на `-1с`.
* Недоступно обновление обычной версии кластера до версий для <q>1С:Предприятие</q> (например, с версии 10 на версию 10-1с).

{% endnote %}

Обновление доступно только на следующую версию относительно текущей, например, с версии 11 на 12. Обновление до более поздних версий производится поэтапно. Например, обновление версии PostgreSQL с 11 до 13 выполняется в такой последовательности: 11 → 12 → 13.

## Перед обновлением {#before-update}

Перед обновлением кластера убедитесь, что это не нарушит работу ваших приложений:

1. Посмотрите [историю изменений](https://www.postgresql.org/docs/release/) для версий {{ PG }}, до которых вы собираетесь обновить кластер, и проверьте, не повлияют ли какие-то из них на работу приложений или установленных [расширений](cluster-extensions.md) {{ PG }}.
1. Попробуйте обновить тестовый кластер (его можно [развернуть](cluster-backups.md#restore), например, из резервной копии основного кластера).
1. [Выполните резервное копирование](cluster-backups.md#create-backup) основного кластера непосредственно перед обновлением.

## Обновить кластер {#start-update}

{% note alert %}

* После обновления СУБД вернуть кластер к предыдущей версии невозможно.
* Успешность обновления версии {{ PG }} зависит от многих факторов, в том числе от настроек кластера и данных, хранящихся в базах. Рекомендуется сначала [обновить тестовый кластер](#before-update), который использует те же данные и настройки.

{% endnote %}

{% list tabs %}

- Консоль управления

  1. Перейдите на страницу каталога и выберите сервис **{{ mpg-name }}**.
  1. Выберите нужный кластер в списке и нажмите кнопку **Изменить кластер**.
  1. В поле **Версия** выберите номер новой версии.
  1. Нажмите кнопку **Сохранить изменения**.

  После запуска обновления кластер переходит в статус **UPDATING**. Дождитесь окончания операции и проверьте версию кластера.

- CLI

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  1. Получите список ваших кластеров {{ PG }} командой:

     ```bash
     {{ yc-mdb-pg }} cluster list
     ```

  1. Получите информацию о нужном кластере и проверьте версию {{ PG }}, указанную в свойстве `config.version`:

     ```bash
     {{ yc-mdb-pg }} cluster get <идентификатор или имя кластера>
     ```

  1. Запустите обновление {{ PG }}:

     ```bash
     {{ yc-mdb-pg }} cluster update <идентификатор или имя кластера> \
        --postgresql-version <номер новой версии>
     ```

  После запуска обновления кластер переходит в статус **UPDATING**. Дождитесь окончания операции и проверьте версию кластера.

- Terraform

    1. Откройте актуальный конфигурационный файл {{ TF }} с планом инфраструктуры.

       О том, как создать такой файл, см. в разделе [{#T}](cluster-create.md).

    1. Добавьте в блок `cluster_config` нужного кластера {{ mpg-name }} поле `version` или измените его значение, если оно уже существует:

       ```hcl
       resource "yandex_mdb_postgresql_cluster" "<имя кластера>" {
         ...
         cluster_config {
           version = "<версия PostgreSQL>"
         }
       }
       ```

    1. Проверьте корректность настроек.

         {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

    1. Подтвердите изменение ресурсов.

         {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

   Подробнее см. в [документации провайдера {{ TF }}]({{ tf-provider-mpg }}).

- API

  Воспользуйтесь методом API [update](../api-ref/Cluster/update.md) и передайте в запросе:

    * Идентификатор кластера в параметре `clusterId`. Его можно получить со [списком кластеров в каталоге](./cluster-list.md#list-clusters).
    * Номер версии {{ PG }} в параметре `configSpec.version`.
    * Список полей конфигурации кластера, подлежащих изменению, в параметре `updateMask`.

        {% include [updateMask note](../../_includes/mdb/note-api-updatemask.md) %}

{% endlist %}

## Примеры {#examples}

Допустим, нужно обновить кластер с версии 11 до версии 12.

{% list tabs %}

- CLI

   1. Чтобы получить список кластеров и узнать их идентификаторы и имена, выполните команду:

      ```bash
      {{ yc-mdb-pg }} cluster list
      ```

      ```text
      +----------------------+---------------+---------------------+--------+---------+
      |          ID          |     NAME      |     CREATED AT      | HEALTH | STATUS  |
      +----------------------+---------------+---------------------+--------+---------+
      | c9q8p8j2gaih8iti42mh |   postgre406  | 2021-10-23 12:44:17 | ALIVE  | RUNNING |
      +----------------------+---------------+---------------------+--------+---------+
      ```

   1. Чтобы получить информацию о кластере с именем `postgre406`, выполните команду:

      ```bash
      {{ yc-mdb-pg }} cluster get postgre406
      ```

      ```text
        id: c9q8p8j2gaih8iti42mh
        ...
        config:
          version: "11"
          ...
      ```

   1. Для обновления кластера `postgre406` до версии 12, выполните команду:

      ```bash
      {{ yc-mdb-pg }} cluster update postgre406 --postgresql-version 12
      ```

{% endlist %}