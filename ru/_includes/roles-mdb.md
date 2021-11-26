На диаграмме показано, какие роли есть в сервисе и как они наследуют разрешения друг друга. Например, в `{{ roles-editor }}` входят все разрешения `{{ roles-viewer }}`. После диаграммы дано описание каждой роли.

![image](../_assets/mdb/service-roles-hierarchy.svg)

Роли, действующие в сервисе:

### Сервисные роли {#service-roles}

Роль | Разрешения
----- | -----
`{{ roles-mdb-admin }}` | Позволяет создавать и изменять кластеры управляемых баз данных.

### Роли других сервисов {{ yandex-cloud }} {#other-roles}

Роль | Разрешения
----- | -----
`{{ roles-cloud-owner }}` | Дает полный доступ к облаку и ресурсам в нем. Можно назначить только на облако.
`{{ roles-cloud-member }}` | Роль, необходимая для доступа к ресурсам в облаке всем, кроме [владельцев облака](../resource-manager/concepts/resources-hierarchy.md#owner) и [сервисных аккаунтов](../iam/concepts/users/service-accounts.md).
`{{ roles-vpc-public-admin }}` |  Позволяет управлять внешней связностью. Важно: если сеть и подсеть находятся в разных каталогах, то наличие этой роли проверяется на том каталоге, в котором находится *сеть*. Подробнее см. [Роли {{ vpc-name }}](../iam/concepts/access-control/roles.md#vpc-public-admin).

### Примитивные роли {#primitive-roles}

{% include [roles-primitive](roles-primitive.md) %}