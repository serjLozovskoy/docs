# {{ TF }} reference for {{ billing-name }}

{% include [terraform-ref-intro](../_includes/terraform-ref-intro.md) %}

## Resources {#resources}

The following {{ TF }} provider resources are supported for {{ billing-name }}:

| **{{ TF }} resource** | **{{ yandex-cloud }} resource** |
| --- | --- |
| [yandex_billing_cloud_binding]({{ tf-provider-resources-link }}/billing_cloud_binding) | Linking a [cloud](../resource-manager/concepts/resources-hierarchy.md#cloud) to a [billing account](./concepts/billing-account.md) |

## Data sources {#data-sources}

{{ billing-name }} supports the following {{ TF }} provider data sources:

| **{{ TF }} data source** | **Description** |
| --- | --- |
| [yandex_billing_cloud_binding]({{ tf-provider-datasources-link }}/datasource_billing_cloud_binding) | Information about linking a [cloud](../resource-manager/concepts/resources-hierarchy.md#cloud) to a [billing account](./concepts/billing-account.md) |
