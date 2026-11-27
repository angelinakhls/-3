
# Защита задания 9: Дополнительные Web API (Синтез речи, Геолокация, Батарея) — Русский / English

> **Суть задания:** Продемонстрировать работу со встроенными браузерными интерфейсами (**Web APIs**), которые дают веб-странице доступ к возможностям устройства пользователя:
> 1. **Web Speech API (`speechSynthesis`)** — озвучивание введённого текста голосом.
> 2. **Geolocation API (`navigator.geolocation`)** — определение географических координат устройства.
> 3. **Battery Status API (`navigator.getBattery`)** — получение процента заряда и статуса зарядки аккумулятора.

---

## 1. Синтез речи / Speech Synthesis API (`window.speechSynthesis`)

```javascript
        const speechText = document.getElementById('api-speech-text');
        const speechSpeakBtn = document.getElementById('api-speech-speak');
        const speechStopBtn = document.getElementById('api-speech-stop');
        const speechStatus = document.getElementById('api-speech-status');

        if (speechSpeakBtn && speechText) {
            speechSpeakBtn.addEventListener('click', () => {
                if (!('speechSynthesis' in window)) {
                    speechStatus.textContent = 'Web Speech API не поддерживается в данном браузере.';
                    return;
                }
                window.speechSynthesis.cancel();
                const utterance = new SpeechSynthesisUtterance(speechText.value || 'Добро пожаловать на Мебельную фабрику!');
                utterance.lang = 'ru-RU';
                utterance.rate = 1.0;
                utterance.onstart = () => { speechStatus.textContent = 'Идёт озвучивание текста...'; };
                utterance.onend = () => { speechStatus.textContent = 'Озвучивание завершено.'; };
                window.speechSynthesis.speak(utterance);
            });
            if (speechStopBtn) {
                speechStopBtn.addEventListener('click', () => {
                    if ('speechSynthesis' in window) window.speechSynthesis.cancel();
                    speechStatus.textContent = 'Озвучивание остановлено.';
                });
            }
        }
```

### Русский

* **Поиск элементов:** Находим поле ввода текста (`speechText`), кнопки «Озвучить» (`speechSpeakBtn`) и «Остановить» (`speechStopBtn`), а также строку для вывода статуса (`speechStatus`).
* **Проверка поддержки API:** Конструкция `if (!('speechSynthesis' in window))` проверяет, есть ли в глобальном объекте браузера `window` встроенный движок синтеза речи. Если браузер устарел и не поддерживает его — выводим сообщение и выходим (`return`).
* **Очистка очереди (`window.speechSynthesis.cancel()`):** Перед запуском новой фразы принудительно сбрасываем предыдущую речь, чтобы фразы не накладывались друг на друга в очереди при повторном клике.
* **Создание речевого высказывания (`new SpeechSynthesisUtterance(...)`):**
  * В конструктор передаётся текст из поля `speechText.value`, а если поле пустое — подставляется приветственная фраза по умолчанию (`|| 'Добро пожаловать на Мебельную фабрику!'`).
  * `utterance.lang = 'ru-RU'` — указываем русский язык синтезатора, чтобы корректно читалась кириллица.
  * `utterance.rate = 1.0` — стандартная скорость речи (`1.0` = нормальный темп).
* **События жизненного цикла фразы:**
  * `utterance.onstart` — срабатывает в момент начала озвучивания и обновляет текст статуса.
  * `utterance.onend` — срабатывает, когда синтезатор дочитал фразу до конца.
* **Запуск и остановка:** Метод `window.speechSynthesis.speak(utterance)` запускает озвучивание, а кнопка «Остановить» снова вызывает `window.speechSynthesis.cancel()`.

### English

* **Selecting elements:** We find the text input (`speechText`), "Speak" (`speechSpeakBtn`) and "Stop" (`speechStopBtn`) buttons, and the status paragraph (`speechStatus`).
* **Feature detection:** `if (!('speechSynthesis' in window))` checks whether the browser's global `window` object supports the speech synthesis engine. If not, it shows a message and exits (`return`).
* **Clearing the queue (`window.speechSynthesis.cancel()`):** Before speaking a new phrase, we cancel any active speech so phrases do not pile up in a queue when clicked multiple times.
* **Creating an utterance (`new SpeechSynthesisUtterance(...)`):**
  * We pass `speechText.value` or a default greeting (`|| 'Добро пожаловать на Мебельную фабрику!'`) if the field is empty.
  * `utterance.lang = 'ru-RU'` sets the synthesizer language to Russian.
  * `utterance.rate = 1.0` sets the normal speaking speed.
* **Speech lifecycle events:**
  * `utterance.onstart` fires when audio playback begins and updates the status text.
  * `utterance.onend` fires once the phrase finishes playing.
* **Start and stop:** `window.speechSynthesis.speak(utterance)` starts speaking, while the "Stop" button calls `window.speechSynthesis.cancel()`.

---

## 2. Геолокация / Geolocation API (`navigator.geolocation`)

```javascript
        const geoBtn = document.getElementById('api-geo-btn');
        const geoOut = document.getElementById('api-geo-output');
        if (geoBtn && geoOut) {
            geoBtn.addEventListener('click', () => {
                if (!('geolocation' in navigator)) {
                    geoOut.textContent = 'Geolocation API не поддерживается вашим браузером.';
                    return;
                }
                geoOut.textContent = 'Запрашиваем координаты...';
                navigator.geolocation.getCurrentPosition(
                    (pos) => {
                        const lat = pos.coords.latitude.toFixed(5);
                        const lon = pos.coords.longitude.toFixed(5);
                        const acc = Math.round(pos.coords.accuracy);
                        geoOut.innerHTML = `Широта: <strong>${lat}</strong>, Долгота: <strong>${lon}</strong> (точность ~${acc} м).`; }, (err) => { geoOut.textContent = `Не удалось получить координаты: ${err.message}`;
                    },
                    { timeout: 8000 }
                );
            });
        }
```

### Русский

* **Проверка поддержки:** `if (!('geolocation' in navigator))` проверяет наличие объекта геолокации в `navigator` (объекте с информацией о браузере и системе).
* **Асинхронный запрос координат (`getCurrentPosition`):** Получение геопозиции занимает время (нужно разрешение пользователя и данные от GPS/Wi-Fi/IP), поэтому метод работает асинхронно и принимает **три аргумента**:
  1. **Коллбэк успеха `(pos) => { ... }`** — вызывается, когда пользователь разрешил доступ и координаты получены:
     * `pos.coords.latitude` — широта, округлённая до 5 знаков после запятой (`.toFixed(5)`).
     * `pos.coords.longitude` — долгота, округлённая до 5 знаков.
     * `pos.coords.accuracy` — погрешность определения в метрах, округлённая до целого (`Math.round`).
  2. **Коллбэк ошибки `(err) => { ... }`** — срабатывает, если пользователь нажал «Запретить», отключён GPS или истекло время ожидания; выводит текст причины (`err.message`).
  3. **Объект настроек `{ timeout: 8000 }`** — ограничивает максимальное время ожидания ответа 8 секундами (`8000` мс), чтобы запрос не висел бесконечно.

### English

* **Feature detection:** `if (!('geolocation' in navigator))` checks if `geolocation` is available inside the `navigator` object.
* **Asynchronous location request (`getCurrentPosition`):** Getting coordinates takes time and user permission, so this method runs asynchronously and takes **three arguments**:
  1. **Success callback `(pos) => { ... }`** — runs once permission is granted and coordinates are ready:
     * `pos.coords.latitude` — latitude formatted to 5 decimal places (`.toFixed(5)`).
     * `pos.coords.longitude` — longitude formatted to 5 decimal places.
     * `pos.coords.accuracy` — accuracy radius in meters rounded with `Math.round`.
  2. **Error callback `(err) => { ... }`** — runs if the user denies permission or location fails, displaying `err.message`.
  3. **Options object `{ timeout: 8000 }`** — limits the wait time to 8 seconds (`8000` ms) so the request does not hang forever.

---

## 3. Состояние батареи / Battery Status API (`navigator.getBattery`)

```javascript
        const batBtn = document.getElementById('api-battery-btn');
        const batOut = document.getElementById('api-battery-output');
        if (batBtn && batOut) {
            batBtn.addEventListener('click', () => {
                if (typeof navigator.getBattery !== 'function') {
                    batOut.textContent = 'Battery Status API не поддерживается или ограничен в этом браузере.';
                    return;
                }
                navigator.getBattery().then((battery) => {
                    const level = Math.round(battery.level * 100);
                    const charging = battery.charging ? 'Да (заряжается)' : 'Нет (от аккумулятора)';
                    batOut.innerHTML = `Уровень заряда: <strong>${level}\%</strong> · Зарядка: <strong>${charging}</strong>.`;
                }).catch(() => {
                    batOut.textContent = 'Не удалось получить сведения о батарее.';
                });
            });
        }
    });
})();
```

### Русский

* **Проверка поддержки:** `typeof navigator.getBattery !== 'function'` проверяет, существует ли метод `getBattery` в браузере (например, в браузерах на движке Chromium он работает, а в Firefox или Safari может быть отключён в целях приватности).
* **Работа через промис (`Promise`):** Метод `navigator.getBattery()` возвращает **Promise** (обещание), который разрешается объектом `battery`:
  * В блоке **`.then((battery) => { ... })`**:
    * `battery.level` хранит уровень заряда в виде дроби от `0.0` до `1.0` (например, `0.84`). Умножаем на `100` и округляем через `Math.round()`, получая привычные проценты (`84%`).
    * `battery.charging` — логическое свойство (`true` или `false`), показывающее, подключено ли зарядное устройство. Через тернарный оператор (`? :`) переводим его в понятный текст.
  * В блоке **`.catch(() => { ... })`** — перехватываем возможную ошибку, если система заблокировала доступ к датчику питания.

### English

* **Feature detection:** `typeof navigator.getBattery !== 'function'` checks whether `getBattery` is available in the current browser (it works in Chromium-based browsers, while Firefox and Safari often restrict it for privacy).
* **Working with Promises:** `navigator.getBattery()` returns a **Promise** that resolves with a `battery` manager object:
  * Inside **`.then((battery) => { ... })`**:
    * `battery.level` is a decimal between `0.0` and `1.0` (for example, `0.84`). Multiplying by `100` and rounding with `Math.round()` converts it to a percentage (`84%`).
    * `battery.charging` is a boolean (`true` or `false`) indicating whether the device is plugged in, formatted into human-readable text with a ternary operator (`? :`).
  * Inside **`.catch(() => { ... })`** — catches any rejection if the system blocks access to battery data.