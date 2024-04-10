# Фундаментальные модели в {{ ml-platform-name }}

{{ ml-platform-full-name }} предоставляет возможность работать с [фундаментальными моделями](../../../glossary/ml-models.md#foundation), чтобы вы могли использовать их для решения своих задач и при необходимости дообучать на своих данных. [Дообучение](../../../glossary/ml-models.md#fine-tuning) производится по методу Fine-tuning, результаты дообучения хранятся в {{ ml-platform-name }}.

{% note info %}

Дообучение фундаментальных моделей находится на стадии [Preview](../../../overview/concepts/launch-stages.md).

{% endnote %}

## Модели, доступные для дообучения {#available-models}

[В разделе  **Фундаментальные модели**]({{ link-datasphere-main }}/foundation-models/ygpt) вы можете найти модели, развернутые в {{ yandex-cloud}}. Их можно использовать в {{ ml-platform-name }} как есть или дообучить на своих данных, чтобы ответы моделей точнее отражали специфику ваших задач. 

Сейчас для дообучения доступна модель {{ gpt-pro }}. Вы сможете обращаться к дообученной модели из проекта {{ ml-platform-name }} и через [API сервиса {{ yagpt-full-name }}](../../../yandexgpt/api-ref/authentication.md).

{% note warning %}

Модели на базе {{ gpt-lite }} (созданные до 27 марта 2024 года) перестанут работать 29 апреля 2024 года.

{% endnote %}

## Данные для дообучения {{ gpt-pro }} {#yagpt-tuning}

{% include [fine-tuning-file-requirements](../../../_includes/datasphere/fine-tuning-file-requirements.md) %}

В интерфейсе {{ ml-platform-name }} создайте новую дообученную фундаментальную модель, введите инструкцию для модели, задайте темп обучения и загрузите данные. Дообучение займет некоторое время.

## Возможности дообучения {#tuning-abilities}

{% include [tuning-abilities](../../../_includes/foundation-models/yandexgpt/tuning-abilities.md) %}

## Запросы к дообученной модели {#requests}

Обращаться к дообученной модели можно через интерфейс {{ ml-platform-name }} Playground или через [API v1](../../../yandexgpt/text-generation/api-ref/index.md) в синхронном режиме из {{ ml-platform-name }} и других приложений. Запросы в Playground осуществляются от имени пользователя, у которого есть флаг доступа к модели. Отправлять запрос через Playground можно в оригинальную или дообученную модель, чтобы сравнить результаты.

Для отправки запросов через API добавьте пользовательский или сервисный аккаунт, от имени которого будут выполняться запросы, в список участников проекта {{ ml-platform-name }} . Аккаунт должен иметь роль `ai.languageModels.user`.
