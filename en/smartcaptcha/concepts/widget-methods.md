---
title: "{{ captcha-full-name }} widget installation methods"
description: "You can use the automatic or advanced method to connect the {{ captcha-name }} widget. Depending on the connection method, different widget parameter transmission methods are used."
---

# Widget installation methods {{ captcha-full-name }}

There are two methods to connect a {{ captcha-name }} widget:

* Automatic
* Advanced

Depending on the connection method, different widget parameter transmission methods are used.

## Automatic connection method {#common-method}

In the automatic method, a JS script that loads the widget on the user page is added to the page by a link:

```html
<script src="https://smartcaptcha.yandexcloud.net/captcha.js" defer></script>
```

After uploading, the JS script searches for all containers suitable for loading a widget into them, and draws widgets in them.

You can load the widget using the `div` elements with the `smart-captcha` class:

```html
<div class="smart-captcha"></div>
```

Parameters for rendering the widget are set via container data attributes.

Example of a container with parameters for rendering a widget:

```html
<div
  class="smart-captcha"
  data-sitekey="<client_part_key>"
  data-hl="<language>"
></div>
```

Where:

| Data attribute | Value | Default value |
| --------------- | ---------------------------------------------------------------------------- | --------------------------- |
| `data-sitekey` | `string` | None |
| `data-hl` | `'ru'` \| `'en'` \| `'be'` \| `'kk'` \| `'tt'` \| `'uk'` \| `'uz'` \| `'tr'` | `window.navigator.language` |
| `data-callback` | `string` | None |

{% cut "Example of embedding the widget" %}

```html
<html>
  <head>
    <meta charset="UTF-8">
    <title>Authentication form</title>
    <script>
      function callback(token) {
        console.log(callback);
      }
    </script>
    <script src="https://smartcaptcha.yandexcloud.net/captcha.js" async defer></script>
  </head>

  <body>
    <p>Enter your username and password:</p>
    <form id="auth_form" action="" method="POST">
      <input id="csrf_token" name="csrf_token" type="hidden" />
      <label for="username">Name</label>
      <input id="username" type="text" name="user" />
      <label for="userpass">Password</label>
      <input id="userpass" type="password" name="password" />
      <div
        id="captcha-container"
        class="smart-captcha"
        data-sitekey="<client_part_key>"
        data-hl="en"
        data-callback="callback"
      ></div>
      <input type="submit" value="Submit" />
    </form>
  </body>
</html>
```

{% endcut %}

## Advanced connection method {#extended-method}

In the advanced method, a JS script that loads the widget on the user page is added to the page by a link:

```html
<script src="https://smartcaptcha.yandexcloud.net/captcha.js?render=onload&onload=onloadFunction"></script>
```

In the `onload` parameter, a function is provided that contains parameters for rendering the widget. In this example, this is the `onloadFunction` function.

After loading the JS script, an access to the `window.smartCaptcha` object is shown along with methods for the widget.

{% cut "Sample onloadFunction code" %}

```html
<div id="<container_ID>"></div>

<script>
  function onloadFunction() {
    if (window.smartCaptcha) {
      const container = document.getElementById('<container_ID>');

      const widgetId = window.smartCaptcha.render(container, {
        sitekey: '<client_part_key>',
        hl: '<language>',
      });
    }
  }
</script>
```

{% endcut %}


## window.smartCaptcha methods {#methods}


### render method {#render}

The `render` method is used to draw the widget.

```ts
(
  container: HTMLElement | string,
  params: {
      sitekey: string;
      callback?: (token: string) => void;
      hl?: 'ru' | 'en' | 'be' | 'kk' | 'tt' | 'uk' | 'uz' | 'tr';
      test?: boolean;
      webview?: boolean;
      invisible?: boolean;
      shieldPosition?: `top-left` | `center-left` | `bottom-left` | `top-right` | `center-right` | `bottom-right`;
      hideShield?: boolean;
  }
) => WidgetId;
```

Where:

* `container`: Container for the widget. You can transmit a DOM element or ID of the container.
* `params`: Object with parameters for the CAPTCHA:
   * `sitekey`: [Client-side key](./keys.md).
   * `callback`: Handler function.
   * `hl`: Widget language.
   * `test`: Running CAPTCHA in test mode. The user will always get a challenge. Use this property for debugging and testing only.
   * `webview`: Running CAPTCHA in **WebView**. You can use it to make user response validation more precise when adding CAPTCHA to mobile apps via **WebView**.
   * `invisible`: [Invisible CAPTCHA](./invisible-captcha.md).
   * `shieldPosition`: Position of the [data processing notice section](./invisible-captcha.md#data-processing-notice).
   * `hideShield`: Hide the [data processing notice section](./invisible-captcha.md#data-processing-notice).

{% include [warning-hideshield](../../_includes/smartcaptcha/warning-hideshield.md) %}

The method returns `widgetId`, that is, a unique widget ID.

### GetResponse method {#getresponse}

The `getResponse` method returns the current value of the user token.

```ts
(widgetId: WidgetId | undefined) => string;
```

`widgetId` argument: Unique widget ID. If no argument is provided, the result of the first rendered widget is returned.

### Execute method {#execute}

The `execute` method starts user validation. It is used to initiate the invisible CAPTCHA test at a certain event, e.g., when the user clicks the **Submit** button of an authentication form.

```ts
(widgetId: WidgetId | undefined) => void;
```

`widgetId` argument: Unique widget ID. If the argument is not provided, the test will be run on the first rendered widget.

### reset method {#reset}

The `reset` method resets the widget to the initial state.

```ts
(widgetId: WidgetId | undefined) => void;
```

`widgetId` argument: Unique widget ID. If no argument has been passed, the first rendered widget is reset.

### destroy method {#destroy}

The `destroy` method deletes widgets and any listeners they create.

```ts
(widgetId: WidgetId | undefined) => void;
```

The `widgetId` argument is a unique widget ID. If the argument is not passed, the first rendered widget will be deleted.

### subscribe method {#subscribe}

The `subscribe` method enables you to subscribe listeners to certain widget events.
For instance, you can listen for the opening and closing of a challenge pop-up window. This may be useful for displaying the keyboard on mobile devices.

```ts
(widgetId: widgetId, event: SubscribeEvent, callback: Function) =>
  UnsubscribeFunction;
```

Where:

* `widgetId`: Unique widget ID.
* `event`: Event:

   ```ts
   type SubscribeEvent =
   | 'challenge-visible'
   | 'challenge-hidden'
   | 'network-error'
   | 'javascript-error'
   | 'success'
   | 'token-expired';
   ```

   Event descriptions:

   | Event | When it occurs | Handler signature |
   | ------------------- | ---------------------------------------------| ------------------------- |
   | `challenge-visible` | Opening the challenge pop-up window | `() => void` |
   | `challenge-hidden` | Closing the challenge pop-up window | `() => void` |
   | `network-error` | A network error occurred | `() => void` |
   | `javascript-error` | A critical JS error occurred | `(error: { filename: string, message: string,`<br>`col: number, line: number }) => void` |
   | `success` | Successful user validation | `(token: string) => void` |
   | `token-expired` | Invalidated verification token | `() => void` |

* `callback`: Listener function:

   ```ts
   UnsubscribeFunction = () => void;
   ```

{% include [javascript-error](../../_includes/smartcaptcha/javascript-error.md) %}

Usage example:

```html
<div id="container"></div>

<script
  src="https://smartcaptcha.yandexcloud.net/captcha.js?render=onload&onload=onloadFunction"
  async
  defer
></script>

<script>
  function onloadFunction() {
    if (window.smartCaptcha) {
      const container = document.getElementById('container');
      const widgetId = window.smartCaptcha.render(container, {
        /* params */
      });

      const unsubscribe = window.smartCaptcha.subscribe(
        widgetId,
        'challenge-visible',
        () => console.log('challenge is visible')
      );
    }
  }
</script>
```

{% cut "Example of embedding the widget" %}

```html
<form>
  <div id="captcha-container"></div>
  <button id="smartcaptcha-demo-submit" disabled="1">Submit</button>
</form>

<script
  src="https://smartcaptcha.yandexcloud.net/captcha.js?render=onload&onload=smartCaptchaInit"
  defer
></script>

<script>
  function callback(token) {
    console.log(token);
    if (token) {
      document
        .getElementById('smartcaptcha-demo-submit')
        .removeAttribute('disabled');
    } else {
      document
        .getElementById('smartcaptcha-demo-submit')
        .setAttribute('disabled', '1');
    }
  }

  function smartCaptchaInit() {
    if (!window.smartCaptcha) {
      return;
    }

    window.smartCaptcha.render('captcha-container', {
      sitekey: '<client_part_key>',
      callback: callback,
    });
  }

  function smartCaptchaReset() {
    if (!window.smartCaptcha) {
      return;
    }

    window.smartCaptcha.reset();
  }

  function smartCaptchaGetResponse() {
    if (!window.smartCaptcha) {
      return;
    }

    var resp = window.smartCaptcha.getResponse();
    console.log(resp);
    alert(resp);
  }
</script>
```

{% endcut %}
