
# Защита задания 4 (Вариант 28): Генератор `<input type="url">` — Русский / English

> **Суть задания (Вариант 28):** Реализовать генерацию элемента `<input type="url">` по нажатию на флажок (чекбокс) с возможностью интерактивно менять его атрибуты (`name`, `placeholder`, `value`, `required`, `pattern`, `readonly`), добавлять сколько угодно таких полей, удалять выбранные, проводить кастомную валидацию и сохранять состояние в `localStorage` после обновления страницы.

---

## 1. Изоляция скрипта и функция экранирования / Script Isolation and Escaping (`escapeAttr`)

```javascript
(function () {
    'use strict';

    function escapeAttr(val) {
        return String(val || '')
            .replace(/&/g, '&amp;')
            .replace(/"/g, '&quot;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;');
    }
```

### Русский

* Весь скрипт обёрнут в **IIFE** (`(function () { ... })();`) со строгим режимом `'use strict';`, чтобы изолировать переменные от глобальной области видимости.
* Функция **`escapeAttr(val)`** защищает HTML-разметку от поломки (XSS-инъекций и кривых кавычек). Поскольку пользователь сам вводит значения для атрибутов `value`, `placeholder` или `pattern`, спецсимволы вроде кавычек `"` или угловых скобок `<` `>` могут сломать тег `<input>`. Эта функция заменяет опасные символы на безопасные HTML-сущности (`&quot;`, `&lt;`, `&gt;`, `&amp;`).

### English

* The script is wrapped in an **IIFE** (`(function () { ... })();`) with `'use strict';` to isolate local variables from the global scope.
* The **`escapeAttr(val)`** function protects the HTML markup from breaking. Because the user types custom text into attributes like `value`, `placeholder`, or `pattern`, special characters such as quotes `"` or angle brackets `<` `>` could break the `<input>` tag. This helper replaces them with safe HTML entities (`&quot;`, `&lt;`, `&gt;`, `&amp;`).

---

## 2. Инициализация, `localStorage` и модель данных / Initialization, `localStorage`, and Data Model

```javascript
    document.addEventListener('DOMContentLoaded', () => {
        const genCheckbox = document.getElementById('url-gen-checkbox');
        const genAddMoreBtn = document.getElementById('url-gen-add-more');
        const genListWrap = document.getElementById('url-gen-list');
        const URL_STORAGE_KEY = 'lr3_v28_url_elements';

        if (genCheckbox && genListWrap) {
            let items = [];
            try {
                const raw = localStorage.getItem(URL_STORAGE_KEY);
                if (raw) items = JSON.parse(raw);
            } catch (_) {
                items = [];
            }

            function saveItems() {
                localStorage.setItem(URL_STORAGE_KEY, JSON.stringify(items));
            }

            function createDefaultItem() {
                const num = items.length + 1;
                return {
                    id: Date.now() + Math.floor(Math.random() * 1000),
                    name: `site_url_${num}`,
                    placeholder: '[https://mebelfabrika.by/catalog.html](https://mebelfabrika.by/catalog.html)',
                    value: '[https://mebelfabrika.by](https://mebelfabrika.by)',
                    required: true,
                    pattern: 'https?://.+\\..+',
                    readonly: false,
                };
            }
```

### Русский

* После загрузки DOM (`DOMContentLoaded`) скрипт находит три главных элемента: главный флажок (`genCheckbox`), кнопку «Добавить ещё» (`genAddMoreBtn`) и контейнер для карточек (`genListWrap`).
* В массиве **`items`** хранится состояние всех сгенерированных полей. При загрузке страницы блок `try...catch` проверяет `localStorage` по ключу `'lr3_v28_url_elements'`: если там есть сохранённый JSON, он восстанавливается через `JSON.parse(raw)`. Если хранилище пустое или данные повреждены, `items` остаётся пустым массивом `[]`.
* **`saveItems()`** — вспомогательная функция, которая превращает массив `items` в JSON-строку (`JSON.stringify`) и записывает её в `localStorage`. Она вызывается при любом изменении, добавлении или удалении поля.
* **`createDefaultItem()`** — фабричная функция, создающая объект нового поля со стандартными значениями по Варианту 28:
  * `id` — уникальный идентификатор (текущее время в миллисекундах + случайное число), чтобы точно отличать карточки при удалении.
  * `name`, `placeholder`, `value` — предустановленные строковые значения.
  * `pattern: 'https?://.+\\..+'` — регулярное выражение, требующее протокол `http://` или `https://` и наличие точки в домене.
  * `required: true` и `readonly: false` — булевы флаги для логических HTML-атрибутов.

### English

* After the DOM loads (`DOMContentLoaded`), the script finds three main elements: the master checkbox (`genCheckbox`), the "Add more" button (`genAddMoreBtn`), and the list container (`genListWrap`).
* The **`items`** array holds the data model for all generated inputs. On page load, a `try...catch` block checks `localStorage` under `'lr3_v28_url_elements'`: if saved JSON exists, it restores the array with `JSON.parse(raw)`. If storage is empty or corrupted, `items` defaults to `[]`.
* **`saveItems()`** is a helper function that converts `items` into a JSON string (`JSON.stringify`) and saves it to `localStorage`. It runs whenever an input is added, edited, or deleted.
* **`createDefaultItem()`** creates a new object with default attributes for Variant 28:
  * `id` — a unique ID (current timestamp + random number) to identify each card reliably during deletion.
  * `name`, `placeholder`, `value` — preset string attributes.
  * `pattern: 'https?://.+\\..+'` — a regular expression requiring `http://` or `https://` and a dot in the domain name.
  * `required: true` and `readonly: false` — boolean flags for HTML boolean attributes.

---

## 3. Отрисовка карточек и панели атрибутов / Rendering Cards and Attribute Controls (`renderUrlItems`)

```javascript
            function renderUrlItems() {
                genCheckbox.checked = items.length > 0;
                if (genAddMoreBtn) {
                    genAddMoreBtn.style.display = items.length > 0 ? 'inline-flex' : 'none';
                }

                if (!items.length) {
                    genListWrap.innerHTML = '<p class="muted">Отметьте флажок выше, чтобы сгенерировать поле <code>&lt;input type="url"&gt;</code> и панель настройки его атрибутов.</p>';
                    return;
                }

                genListWrap.innerHTML = items.map((item, idx) => `
                    <div class="url-gen-item" data-id="${item.id}">
                        <div class="url-gen-preview">
                            <strong>Элемент #${idx + 1}:</strong>
                            <input
                                type="url"
                                class="generated-url-input"
                                name="${escapeAttr(item.name)}"
                                placeholder="${escapeAttr(item.placeholder)}"
                                value="${escapeAttr(item.value)}"
                                ${item.pattern ? `pattern="${escapeAttr(item.pattern)}"` : ''}
                                ${item.required ? 'required' : ''}
                                ${item.readonly ? 'readonly' : ''}
                            >
                            <button type="button" class="button secondary url-validate-btn">Проверить валидность</button>
                            <button type="button" class="button url-delete-btn" style="background:#b91c1c;color:#fff;">Удалить</button>
                        </div>
                        <div class="url-validate-msg" style="margin-bottom:0.65rem;font-size:0.88rem;"></div>
                        <div class="url-gen-attrs">
                            <label>
                                name
                                <input type="text" data-attr="name" value="${escapeAttr(item.name)}">
                            </label>
                            <label>
                                placeholder
                                <input type="text" data-attr="placeholder" value="${escapeAttr(item.placeholder)}">
                            </label>
                            <label>
                                value (значение)
                                <input type="text" data-attr="value" value="${escapeAttr(item.value)}">
                            </label>
                            <label>
                                pattern (шаблон regex)
                                <input type="text" data-attr="pattern" value="${escapeAttr(item.pattern)}">
                            </label>
                            <label class="inline-check">
                                <input type="checkbox" data-attr="required" ${item.required ? 'checked' : ''}>
                                required
                            </label>
                            <label class="inline-check">
                                <input type="checkbox" data-attr="readonly" ${item.readonly ? 'checked' : ''}>
                                readonly
                            </label>
                        </div>
                    </div>
                `).join('');
```

### Русский

Функция **`renderUrlItems()`** синхронизирует экран с массивом `items`:

* Состояние главного флажка `genCheckbox.checked` и видимость кнопки «Добавить ещё» (`genAddMoreBtn`) зависят от того, есть ли элементы в массиве (`items.length > 0`).
* Если массив пуст (`!items.length`), выводится текст-подсказка, и функция завершается (`return`).
* Если элементы есть, метод `.map()` превращает каждый объект `item` в HTML-карточку `.url-gen-item` с атрибутом `data-id="${item.id}"`:
  1. **Верхняя часть (`.url-gen-preview`):** сам сгенерированный `<input type="url">`, куда через шаблонные строки и `escapeAttr` подставляются все 6 требуемых атрибутов (`name`, `placeholder`, `value`, `pattern`, `required`, `readonly`), а также кнопки «Проверить валидность» и «Удалить».
  2. **Блок сообщения (`.url-validate-msg`):** пустой контейнер для вывода результата проверки.
  3. **Панель настроек (`.url-gen-attrs`):** 4 текстовых поля и 2 чекбокса для управления каждым атрибутом. У каждого контрола есть атрибут `data-attr="..."` (например, `data-attr="placeholder"`), чтобы в JavaScript одним универсальным циклом понимать, за какой именно атрибут отвечает это поле.

### English

The **`renderUrlItems()`** function syncs the screen with the `items` array:

* The main checkbox state (`genCheckbox.checked`) and the visibility of the "Add more" button (`genAddMoreBtn`) depend on whether `items.length > 0`.
* If the array is empty (`!items.length`), it displays a hint message and returns early.
* Otherwise, `.map()` turns each `item` object into a `.url-gen-item` card with `data-id="${item.id}"`:
  1. **Top preview (`.url-gen-preview`):** the generated `<input type="url">` populated with all 6 required attributes (`name`, `placeholder`, `value`, `pattern`, `required`, `readonly`) via `escapeAttr`, plus the "Validate" and "Delete" buttons.
  2. **Status box (`.url-validate-msg`):** an empty container for validation feedback.
  3. **Attribute controls (`.url-gen-attrs`):** 4 text inputs and 2 checkboxes to edit every attribute. Each control has a `data-attr="..."` attribute so JavaScript knows which property to update.

---

## 4. Двусторонняя синхронизация, кастомная валидация и удаление / Two-Way Sync, Custom Validation, and Deletion

```javascript
                genListWrap.querySelectorAll('.url-gen-item').forEach((card) => {
                    const id = Number(card.dataset.id);
                    const model = items.find((x) => x.id === id);
                    const liveInput = card.querySelector('.generated-url-input');
                    const msgEl = card.querySelector('.url-validate-msg');

                    // Синхронизация ручного ввода в самом <input type="url">
                    liveInput.addEventListener('input', () => {
                        model.value = liveInput.value;
                        const valAttrInp = card.querySelector('[data-attr="value"]');
                        if (valAttrInp) valAttrInp.value = liveInput.value;
                        saveItems();
                    });

                    // Интерактивная настройка атрибутов
                    card.querySelectorAll('[data-attr]').forEach((ctrl) => {
                        const attr = ctrl.dataset.attr;
                        const evName = ctrl.type === 'checkbox' ? 'change' : 'input';
                        ctrl.addEventListener(evName, () => {
                            if (ctrl.type === 'checkbox') {
                                model[attr] = ctrl.checked;
                                liveInput[attr] = ctrl.checked;
                                if (ctrl.checked) liveInput.setAttribute(attr, '');
                                else liveInput.removeAttribute(attr);
                            } else {
                                model[attr] = ctrl.value;
                                if (attr === 'value') {
                                    liveInput.value = ctrl.value;
                                } else if (ctrl.value) {
                                    liveInput.setAttribute(attr, ctrl.value);
                                } else {
                                    liveInput.removeAttribute(attr);
                                }
                            }
                            saveItems();
                        });
                    });

                    // Кастомная проверка валидности URL + pattern
                    card.querySelector('.url-validate-btn').addEventListener('click', () => {
                        liveInput.setCustomValidity('');
                        const val = liveInput.value.trim();
                        if (model.required && !val) {
                            liveInput.setCustomValidity('Поле обязательно для заполнения.');
                        } else if (model.pattern && val) {
                            try {
                                const re = new RegExp(`^(?:${model.pattern})$`);
                                if (!re.test(val)) {
                                    liveInput.setCustomValidity(`Значение не соответствует шаблону: ${model.pattern}`);
                                }
                            } catch (_) {
                                liveInput.setCustomValidity('Некорректное регулярное выражение в атрибуте pattern.');
                            }
                        }
                        const ok = liveInput.checkValidity();
                        liveInput.classList.toggle('input-valid', ok);
                        liveInput.classList.toggle('input-invalid', !ok);
                        msgEl.textContent = ok
                            ? `Валидация пройдена: «${val}» (name="${model.name}").`
                            : `Ошибка валидации: ${liveInput.validationMessage}`;
                        msgEl.style.color = ok ? '#15803d' : '#dc2626';
                    });

                    // Удаление элемента
                    card.querySelector('.url-delete-btn').addEventListener('click', () => {
                        items = items.filter((x) => x.id !== id);
                        saveItems();
                        renderUrlItems();
                    });
                });
            }
```

### Русский

После отрисовки карточек цикл `.forEach` проходит по каждой карточке `card`, находит её объект в массиве `items` по `id` и навешивает 4 обработчика:

1. **Прямой ввод в сам `<input type="url">` (`liveInput`):**
   * При вводе текста срабатывает `'input'`: новое значение записывается в `model.value`, дублируется в нижнее поле настройки `value` и сохраняется в `localStorage` через `saveItems()`.
2. **Интерактивная настройка атрибутов (`[data-attr]`):**
   * Для чекбоксов (`required`, `readonly`) используется событие `'change'`, для текстовых полей — `'input'`.
   * Если это чекбокс, скрипт обновляет свойство `liveInput[attr]` и добавляет/удаляет HTML-атрибут через `setAttribute` / `removeAttribute` (например, при включении `readonly` поле сразу блокируется для ввода).
   * Если это текстовое поле, обновляется `liveInput.value` (для атрибута `value`) либо вызывается `setAttribute(attr, ctrl.value)` (для `name`, `placeholder`, `pattern`), а если поле очистили — атрибут удаляется через `removeAttribute`.
3. **Кастомная валидация (Constraint Validation API):**
   * При клике на «Проверить валидность» сначала сбрасывается предыдущая ошибка через `liveInput.setCustomValidity('')`.
   * Далее проверяются два кастомных правила: если стоит `required` и строка пустая — задаётся ошибка; если задан `pattern` и строка не пустая — создаётся регулярное выражение `new RegExp(...)` и проверяется совпадение через `re.test(val)` (блок `try...catch` защищает от падения, если пользователь ввёл сломанный regex).
   * Затем вызывается встроенный метод браузера **`liveInput.checkValidity()`**, который одновременно проверяет и стандартный формат URL (`type="url"`), и наши кастомные ошибки `setCustomValidity`.
   * По результату `ok` (`true`/`false`) переключаются CSS-классы рамки (`input-valid` / `input-invalid`) и выводится зелёное или красное сообщение с текстом из `liveInput.validationMessage`.
4. **Удаление элемента:**
   * При клике на кнопку «Удалить» метод `items.filter((x) => x.id !== id)` оставляет в массиве все элементы, кроме текущего, сохраняет обновлённый список и заново вызывает `renderUrlItems()`.

### English

After rendering the cards, `.forEach` loops over each `card`, finds its matching object in `items` by `id`, and attaches 4 listeners:

1. **Direct typing in `<input type="url">` (`liveInput`):**
   * On `'input'`, the typed text updates `model.value`, mirrors into the `value` setting box below, and saves to `localStorage` via `saveItems()`.
2. **Interactive attribute editing (`[data-attr]`):**
   * Checkboxes (`required`, `readonly`) listen to `'change'`, while text boxes listen to `'input'`.
   * For checkboxes, it updates `liveInput[attr]` and toggles the HTML attribute with `setAttribute` / `removeAttribute` (so checking `readonly` immediately locks the input).
   * For text inputs, it updates `liveInput.value` or calls `setAttribute` / `removeAttribute` for `name`, `placeholder`, and `pattern`.
3. **Custom validation (Constraint Validation API):**
   * Clicking "Validate" first clears old errors with `liveInput.setCustomValidity('')`.
   * It checks if `required` is set on an empty value, or tests the input against `new RegExp(...)` when `pattern` is set (wrapped in `try...catch` in case the user types an invalid regex).
   * Then **`liveInput.checkValidity()`** runs both native `type="url"` checks and our custom validity rules.
   * Depending on `ok` (`true`/`false`), it toggles `input-valid` / `input-invalid` classes and displays a green or red status message using `liveInput.validationMessage`.
4. **Element deletion:**
   * Clicking "Delete" filters out the current item using `items.filter((x) => x.id !== id)`, saves the updated array, and re-renders the list with `renderUrlItems()`.

---

## 5. Управление главным флажком и кнопкой «Добавить ещё» / Master Checkbox and "Add More" Button

```javascript
            genCheckbox.addEventListener('change', () => {
                if (genCheckbox.checked) {
                    items.push(createDefaultItem());
                    saveItems();
                    renderUrlItems();
                } else {
                    items = [];
                    saveItems();
                    renderUrlItems();
                }
            });

            if (genAddMoreBtn) {
                genAddMoreBtn.addEventListener('click', () => {
                    items.push(createDefaultItem());
                    saveItems();
                    renderUrlItems();
                });
            }

            renderUrlItems();
        }
```

### Русский

* **`genCheckbox` (`'change'`):** если пользователь поставил галочку (`checked === true`), в массив `items` добавляется первый элемент через `createDefaultItem()`; если снял галочку — массив очищается (`items = []`). После этого данные сохраняются и перерисовываются.
* **`genAddMoreBtn` (`'click'`):** позволяет добавлять неограниченное количество дополнительных полей `<input type="url">`, каждый раз добавляя новый объект в `items`, сохраняя в `localStorage` и вызывая `renderUrlItems()`.
* Финальный вызов **`renderUrlItems()`** при старте страницы сразу отрисовывает элементы, которые были загружены из `localStorage` после обновления страницы.

### English

* **`genCheckbox` (`'change'`):** when checked (`true`), it pushes the first element via `createDefaultItem()` into `items`; when unchecked (`false`), it clears `items = []`, saves, and re-renders.
* **`genAddMoreBtn` (`'click'`):** lets the user add as many extra `<input type="url">` elements as needed by pushing new items, saving to `localStorage`, and calling `renderUrlItems()`.
* The final **`renderUrlItems()`** call on page load immediately renders any saved elements restored from `localStorage`.