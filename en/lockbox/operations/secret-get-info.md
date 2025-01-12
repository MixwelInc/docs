---
title: "How to get information about a secret in {{ lockbox-full-name }}"
---

# Getting information about a secret, its contents, and access rights

You can get detailed [information about a secret](#secret-info) and [secret contents](#secret-contents) and [view access rights to a secret](#secret-access).

## Getting information about a secret {#secret-info}

{% list tabs group=instructions %}

- Management console {#console}

   1. In the [management console]({{ link-console-main }}), select the folder the secret belongs to.
   1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_lockbox }}**.
   1. In the left-hand menu, select **{{ ui-key.yacloud.lockbox.label_section-secrets }}**.
   1. Click the name of the secret you need.

- CLI {#cli}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   1. See a description of the CLI command to get information about a secret:

      ```bash
      yc lockbox secret get --help
      ```

   1. Get information about a secret by specifying its name or ID:

      ```bash
      yc lockbox secret get <secret_name_or_ID>
      ```

      Result:

      ```bash
      id: e6qi98vtdva1********
      folder_id: b1go79qlt1tp********
      created_at: "2023-11-03T15:28:18.909Z"
      name: test-secret
      kms_key_id: abj765aos682********
      status: ACTIVE
      current_version:
        id: e6q7nvojsgmk********
        secret_id: e6qi98vtdva1********
        created_at: "2023-11-03T15:28:18.909Z"
        status: ACTIVE
        payload_entry_keys:
          - example-key
      ```

- API {#api}

   To get information about a secret, use the [get](../api-ref/Secret/get.md) REST API method for the [Secret](../api-ref/Secret/index.md) resource or the [SecretService/Get](../api-ref/grpc/secret_service.md#Get) gRPC API call.

{% endlist %}

## Getting the contents of a secret {#secret-contents}

{% list tabs group=instructions %}

- Management console {#console}

   1. In the [management console]({{ link-console-main }}), select the folder the secret belongs to.
   1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_lockbox }}**.
   1. In the left-hand menu, select **{{ ui-key.yacloud.lockbox.label_section-secrets }}**.
   1. Click the name of the secret you need.
   1. Under **{{ ui-key.yacloud.lockbox.label_secret-versions-section }}**, click the secret version you need.

- CLI {#cli}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   1. View a description of the CLI command to get the contents of a secret:

      ```bash
      yc lockbox payload get --help
      ```

   1. Get the contents of a secret by specifying its name or ID:

      ```bash
      yc lockbox payload get <secret_name_or_ID>
      ```

      Result:

      ```bash
      version_id: e6q7nvojsgmk********
      entries:
        - key: example-key
          text_value: example-value
      ```

- API {#api}

   To get the secret contents, use the [get](../api-ref/Payload/get.md) REST API method for the [Payload](../api-ref/Payload/index.md) resource or the [PayloadService/Get](../api-ref/grpc/payload_service.md#Get) gRPC API call.

{% endlist %}

## Viewing permissions to a secret {#secret-access}

{% list tabs group=instructions %}

- Management console {#console}

   1. In the [management console]({{ link-console-main }}), select the folder the secret belongs to.
   1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_lockbox }}**.
   1. In the left-hand menu, select **{{ ui-key.yacloud.lockbox.label_section-secrets }}**.
   1. Click the name of the secret you need.
   1. In the left-hand panel, select the ![image](../../_assets/console-icons/persons.svg) **{{ ui-key.yacloud.common.resource-acl.label_access-bindings }}** section.

- CLI {#cli}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   1. View a description of the CLI command to view access permissions to a secret:

      ```bash
      yc lockbox secret list-access-bindings --help
      ```

   1. View access permissions to a secret by specifying its name or ID:

      ```bash
      yc lockbox secret list-access-bindings <secret_name_or_ID>
      ```

      Result:

      ```bash
      +---------+---------------+----------------------+
      | ROLE ID | SUBJECT TYPE  |      SUBJECT ID      |
      +---------+---------------+----------------------+
      | viewer  | federatedUser | ajej2i98kcjd******** |
      +---------+---------------+----------------------+
      ```

- API {#api}

   To view access permissions to a secret, use the [ListAccessBindings](../api-ref/Secret/listAccessBindings.md) REST API method for the [Secret](../api-ref/Secret/index.md) resource or the [SecretService/ListAccessBindings](../api-ref/grpc/secret_service.md#ListAccessBindings) gRPC API call.

{% endlist %}