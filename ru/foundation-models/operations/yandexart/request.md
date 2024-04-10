# Сгенерировать изображение с помощью {{ yandexart-name }}

{% include notitle [preview-stage](../../../_includes/foundation-models/yandexgpt/preview.md) %}

С помощью нейросети {{ yandexart-name }} вы можете генерировать изображения в [асинхронном режиме](../../concepts/index.d#working-mode).


## Перед началом работы {#before-begin}

{% list tabs group=instructions %}

- API {#api}

  Чтобы воспользоваться примерами запросов через API, установите [cURL](https://curl.haxx.se) для отправки API-запросов и утилиту [jq](https://github.com/jqlang/jq) для работы с файлами JSON.

  Получите данные для аутентификации в API, как описано на странице [{#T}](../../api-ref/authentication.md).

{% endlist %}

## Сгенерируйте изображение {#generate-text}

{% note info %}

Чтобы повышать качество генерируемых ответов, {{ yandexart-name }} логирует промты пользователей. Не передавайте в запросах чувствительную информацию и персональные данные.

{% endnote %}

{% list tabs group=instructions %}

- API {#api}

  1. Создайте файл с телом запроса (например, `prompt.json`):

     ```json
     {
     "modelUri": "art://<идентификатор_каталога>/yandex-art/latest",
     "generationOptions": {
       "seed": 17
     },
     "messages": [
       {
         "weight": 1,
         "text": "узор из цветных пастельных суккулентов разных сортов, hd full wallpaper, четкий фокус, множество сложных деталей, глубина кадра, вид сверху"
       }
     ]
     }
     ```

     Где:

     {% include [api-parameters](../../../_includes/foundation-models/yandexgpt/api-parameters.md) %}

  1. Отправьте запрос нейросети с помощью метода [ImageGenerationAsync.generate](../../image-generation/api-ref/ImageGenerationAsync/generate.md), выполнив команду:

     ```bash
     curl --request POST \
       -H "Authorization: Bearer <значение_IAM-токена>" \
       -d "@prompt.json" \
       "https://llm.{{ api-host }}/foundationModels/v1/imageGenerationAsync"  
     ```

     Где:
 
     * `<значение_IAM-токена>` — IAM-токен вашего аккаунта.
     * `prompt.json` — файл в формате JSON, содержащий параметры запроса.
     
     В ответе сервис вернет идентификатор вашего запроса:

     ```json
     {
     "id":"fbveu1sntj**********","description":"","createdAt":null,"createdBy":"","modifiedAt":null,"done":false,"metadata":null}
     ```

  1. Генерация изображения занимает некоторое время. Подождите 10 секунд и отправьте запрос, чтобы получить результат генерации. Если изображение готово, результат вернется в [кодировке Base64](https://en.wikipedia.org/wiki/Base64) и будет записан в файл `image.jpeg`.
  
     ```bash
     curl -X GET -H "Authorization: Bearer <значение_IAM-токена>" https://llm.api.cloud.yandex.net:443/operations/<идентификатор_запроса> | jq -r '.response | .image' | base64 -d > image.jpeg
     ```

     Где:

     * `<значение_IAM-токена>` — IAM-токен, полученный [перед началом работы](#before-begin).
     * `<идентификатор_запроса>` — значение поля `id`, полученное в ответе на запрос генерации.

{% endlist %}
