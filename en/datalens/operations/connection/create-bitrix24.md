---
title: "How to create a Bitrix24 connection"
description: "Follow this guide to create a Bitrix24 connection."
---

# Creating a Bitrix24 connection

To create a Bitrix24 connection:


{% include [datalens-workbooks-collections-note](../../../_includes/datalens/operations/datalens-workbooks-collections-note.md) %}


1. Go to the [connections page]({{ link-datalens-main }}/connections).
1. Click **Create connection**.
1. Under **Partner connections**, select the **Bitrix24** connection.
1. Specify the connection parameters:

   * **Portal**: URL of your Bitrix24 portal in `test.bitrix24.com` format.
   * **Token**: Get a secret key in Bitrix24 by selecting **CRM** → **Analytics** → **BI analytics** in the **Yandex DataLens** tab. For more information, see [this guide](https://helpdesk.bitrix24.ru/open/17402692).
   * Leave the **Automatically create a dashboard, charts, and a dataset on the connection** option enabled if you need a folder with a standard set of datasets and charts and a ready-made dashboard.

1. Click **Create connection**.
1. Enter a name for the connection and click **Create**.

{% include [datalens-check-host](../../../_includes/datalens/operations/datalens-check-host.md) %}
