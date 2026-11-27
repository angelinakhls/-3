
# Защита кода переключателя темы (`Theme Toggle`) — Русский / English

---

## 1. Изоляция кода и строгий режим / Code Isolation and Strict Mode

```javascript
/* Задание 2: Переключатель светлой/тёмной темы с сохранением в localStorage */
(function () {
    'use strict';

    const STORAGE_KEY = 'factory_site_theme';
```

### Русский

Весь скрипт обёрнут в **IIFE** (`Immediately Invoked Function Expression` — немедленно вызываемую функцию) `(function () { ... })();`:

* Это нужно для **изоляции области видимости**: переменные и функции внутри (`STORAGE_KEY`, `applyTheme`) не попадают в глобальный объект `window` и не конфликтуют с другими скриптами на странице.
* `'use strict';` — включает **строгий режим** JavaScript, который запрещает небезопасные конструкции (например, случайное создание глобальных переменных без `const` или `let`) и помогает сразу находить ошибки.
* `const STORAGE_KEY = 'factory_site_theme';` — константа с именем ключа, под которым выбранная тема будет сохраняться в памяти браузера (`localStorage`). Вынесение строки в константу защищает от опечаток.

### English

The entire script is wrapped in an **IIFE** (`Immediately Invoked Function Expression`) `(function () { ... })();`:

* This provides **scope isolation**: variables and functions inside (`STORAGE_KEY`, `applyTheme`) stay local, do not pollute the global `window` object, and cannot conflict with other scripts on the page.
* `'use strict';` — turns on JavaScript **strict mode**, which blocks unsafe code (like accidentally creating global variables without `const` or `let`) and helps catch bugs early.
* `const STORAGE_KEY = 'factory_site_theme';` — a constant holding the key name used to save the chosen theme in browser storage (`localStorage`). Using a constant prevents typos.

---

## 2. Функция применения темы / Applying the Theme (`applyTheme`)

```javascript
    function applyTheme(theme) {
        const isDark = theme === 'dark';
        document.documentElement.setAttribute('data-theme', isDark ? 'dark' : 'light');

        const btn = document.getElementById('theme-toggle-btn');
        if (btn) {
            btn.setAttribute('aria-pressed', String(isDark));
            btn.innerHTML = isDark
                ? '<span aria-hidden="true">☀️</span> Светлая тема'
                : '<span aria-hidden="true">🌙</span> Тёмная тема';
        }
    }
```

### Русский

Функция `applyTheme(theme)` отвечает за визуальное переключение темы на странице и обновление самой кнопки. Она принимает один параметр **`theme`** (строку `'dark'` или `'light'`):

* `const isDark = theme === 'dark';` — проверяет, передана ли тёмная тема, и сохраняет результат (`true` или `false`) в логическую переменную `isDark`.
* `document.documentElement.setAttribute('data-theme', isDark ? 'dark' : 'light');` — `document.documentElement` ссылается на корневой тег `<html>`. Через `setAttribute` я записываю в него атрибут `data-theme="dark"` или `data-theme="light"`. В CSS к этому атрибуту привязаны переменные цветов (например, `[data-theme="dark"] { --bg: #121212; }`), поэтому вся страница мгновенно меняет цвета.
* `const btn = document.getElementById('theme-toggle-btn');` — находит кнопку переключения темы по её `id`. Проверка `if (btn)` защищает код от ошибки, если на какой-то странице этой кнопки нет.
* `btn.setAttribute('aria-pressed', String(isDark));` — обновляет атрибут доступности для скринридеров (программ чтения с экрана), переводя булево значение `true`/`false` в строку `'true'`/`'false'`.
* `btn.innerHTML = ...` — меняет иконку и подпись внутри кнопки через тернарный оператор (`? :`): если сейчас включена тёмная тема, кнопка предлагает переключиться на светлую (`☀️ Светлая тема`), и наоборот (`🌙 Тёмная тема`). Атрибут `aria-hidden="true"` скрывает эмодзи от скринридеров, чтобы они читали только текст.

### English

The `applyTheme(theme)` function handles changing the page theme visually and updating the button itself. It takes one parameter **`theme`** (the string `'dark'` or `'light'`):

* `const isDark = theme === 'dark';` — checks if the dark theme was passed and stores the result (`true` or `false`) in the boolean variable `isDark`.
* `document.documentElement.setAttribute('data-theme', isDark ? 'dark' : 'light');` — `document.documentElement` points to the root `<html>` tag. Using `setAttribute`, I set the attribute `data-theme="dark"` or `data-theme="light"`. In CSS, color variables depend on this attribute (for example, `[data-theme="dark"] { --bg: #121212; }`), so the whole page updates its colors instantly.
* `const btn = document.getElementById('theme-toggle-btn');` — finds the theme button by its `id`. The `if (btn)` check prevents errors if the button does not exist on a page.
* `btn.setAttribute('aria-pressed', String(isDark));` — updates the accessibility attribute for screen readers by converting the boolean `true`/`false` into a string `'true'`/`'false'`.
* `btn.innerHTML = ...` — updates the icon and label inside the button using a ternary operator (`? :`): when dark mode is active, the button offers to switch to light mode (`☀️ Светлая тема`), and vice versa (`🌙 Тёмная тема`). The `aria-hidden="true"` attribute hides the emoji from screen readers so they only read the text.

---

## 3. Загрузка сохранённой темы и клик по кнопке / Loading Saved Theme and Button Click

```javascript
    document.addEventListener('DOMContentLoaded', function () {
        const saved = localStorage.getItem(STORAGE_KEY) || 'light';
        applyTheme(saved);

        const btn = document.getElementById('theme-toggle-btn');
        if (btn) {
            btn.addEventListener('click', function () {
                const current = document.documentElement.getAttribute('data-theme') === 'dark' ? 'dark' : 'light';
                const next = current === 'dark' ? 'light' : 'dark';
                localStorage.setItem(STORAGE_KEY, next);
                applyTheme(next);
            });
        }
    });
})();
```

### Русский

Этот блок запускается при загрузке страницы и связывает `localStorage` с кнопкой переключения:

* `document.addEventListener('DOMContentLoaded', ...)` — ждёт, пока браузер полностью построит DOM-дерево HTML-страницы, чтобы кнопка `theme-toggle-btn` точно существовала к моменту поиска.
* `const saved = localStorage.getItem(STORAGE_KEY) || 'light';` — проверяет, сохранял ли пользователь тему ранее в `localStorage`. Если пользователь зашёл впервые и в хранилище пусто (`null`), оператор ИЛИ (`||`) подставляет светлую тему `'light'` по умолчанию.
* `applyTheme(saved);` — сразу применяет загруженную тему при открытии страницы.
* Далее я нахожу кнопку `btn` и, если она есть (`if (btn)`), вешаю на неё обработчик события клика `addEventListener('click', ...)`:
  1. `const current = ...` — считываю текущую активную тему прямо из атрибута `data-theme` тега `<html>`.
  2. `const next = current === 'dark' ? 'light' : 'dark';` — вычисляю противоположную тему: если была тёмная, следующей станет светлая, и наоборот.
  3. `localStorage.setItem(STORAGE_KEY, next);` — записываю новый выбор пользователя в `localStorage`, чтобы тема не сбрасывалась при перезагрузке страницы или переходе на другие страницы сайта.
  4. `applyTheme(next);` — вызываю функцию обновления интерфейса, которая меняет атрибут `data-theme` на `<html>` и перерисовывает текст кнопки.

### English

This block runs when the page loads and connects `localStorage` with the toggle button:

* `document.addEventListener('DOMContentLoaded', ...)` — waits until the browser finishes building the HTML DOM tree, ensuring the `theme-toggle-btn` element exists before we search for it.
* `const saved = localStorage.getItem(STORAGE_KEY) || 'light';` — checks if the user saved a theme earlier in `localStorage`. If it is a first visit and storage is empty (`null`), the OR operator (`||`) falls back to `'light'` by default.
* `applyTheme(saved);` — applies the loaded theme immediately when the page opens.
* Next, I find the button `btn` and, if it exists (`if (btn)`), attach a `'click'` event listener with `addEventListener('click', ...)`:
  1. `const current = ...` — reads the currently active theme directly from the `data-theme` attribute of the `<html>` tag.
  2. `const next = current === 'dark' ? 'light' : 'dark';` — determines the opposite theme: if it was dark, the next one is light, and vice versa.
  3. `localStorage.setItem(STORAGE_KEY, next);` — saves the user's new choice in `localStorage` so the theme stays active after refreshing or navigating to other pages.
  4. `applyTheme(next);` — calls the UI update function to change the `data-theme` attribute on `<html>` and update the button text.