---
title: "How to add a {{ captcha-full-name }} widget"
description: "Follow this guide to add a {{ captcha-name }} widget using the advanced method or as required."
---

# Add a {{ captcha-name }} widget

## Add a {{ captcha-name }} widget using the advanced method {#advanced}

You control how the widget loads using the `window.smartCaptcha` [object](../concepts/widget-methods.md#methods). The `onloadFunction` callback function is used for this in the guide:

1. Add the JS script to the user page. To do this, place the following code anywhere on the page, for example, inside the `<head>` tag:

   ```html
   <script
       src="https://smartcaptcha.yandexcloud.net/captcha.js?render=onload&onload=onloadFunction"
       defer
   ></script>
   ```

1. Add an empty container where you want to install your widget, to the page:

   ```html
   <div id="<container_ID>"></div>
   ```

   Where `--id` is an ad-hoc ID.

1. Add the code of the callback function to the page:

   ```html
   <script>
       function onloadFunction() {
       if (window.smartCaptcha) {
           const container = document.getElementById('<container_ID>');

           const widgetId = window.smartCaptcha.render(container, {
                sitekey: '<client_key>',
                hl: '<language>',
           });
       }
       }
   </script>
   ```

   Where:

   * `<container_ID>`: ID generated at the previous step.
   * `sitekey`: [Client-side key](../concepts/keys.md).
   * `hl`: Widget and challenge [language](../concepts/widget-methods.md#render).

   Add a check for existence of the `window.smartCaptcha` object to avoid an error when the function is called before the JS script loading is complete.

   {% include [info-container-height](../../_includes/smartcaptcha/info-container-height.md) %}

## Add a {{ captcha-name }} widget as required {#dynamic}

To add a widget and load CAPTCHA as required, use the following approach:

```js
window.onloadFunction = () => {
  if (window.smartCaptcha) {
    // Creating a CAPTCHA
  }
}

function handleScriptLoadingError() {
  // Handling errors
}

const scriptElement = document.createElement('script');
scriptElement.src = 'https://smartcaptcha.yandexcloud.net/captcha.js?render=onload&onload=onloadFunction';
scriptElement.onerror = handleScriptLoadingError;
document.body.appendElement(scriptElement);
```

## What's next {#whats-next}

* [Invisible CAPTCHA](../concepts/invisible-captcha.md).
* [Connecting a CAPTCHA in React](../concepts/react.md).
