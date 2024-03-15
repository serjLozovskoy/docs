# Создать проект

Для создания проекта нужна роль `{{ roles-speechsense-admin }}` или `{{ roles-speechsense-editor }}` в пространстве.

{% note info %}

Перед созданием проекта [создайте подключение](../connection/create.md).

{% endnote %}

Чтобы создать проект:

1. Откройте [главную страницу]({{ link-speechsense-main }}) {{ speechsense-name }}.
1. Перейдите в нужное пространство.
1. Нажмите кнопку ![create](../../../_assets/console-icons/folder-plus.svg) **Создать проект**.
1. Введите имя и при необходимости описание проекта.
1. В блоке **Подключение** нажмите **Добавить подключение** и выберите нужный пункт из списка.
1. (Опционально) Добавьте правила фильтрации диалогов на основе метаданных из подключения. Фильтрация невозможна для полей с типом **Дата**.
1. Нажмите кнопку **Создать проект**.