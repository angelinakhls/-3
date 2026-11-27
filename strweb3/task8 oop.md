
# Защита задания 8 (Вариант 28): Пассажиры авиакомпании — поиск однофамильцев (ООП на прототипах и ES6 `class`) — Русский / English

> **Суть задания (Вариант 28):** Реализовать предметную область «Пассажиры авиакомпании» с поиском однофамильцев на заданном рейсе **двумя способами ООП**:
> 1. **Функциональный стиль + прототипное наследование** (`PersonProto` → `PassengerProto`).
> 2. **Синтаксис ES6 `class` / `extends`** (`PersonES6` → `PassengerES6`).
>
> В каждом базовом и дочернем классе реализовано не менее 5 методов: геттеры/сеттеры, добавление из формы, вывод всех объектов и вывод результата поиска однофамильцев.

---

## 1. Экранирование HTML и начальные данные / HTML Escaping and Initial Data

```javascript
(function () {
    'use strict';

    function escapeHtml(s) {
        return String(s || '')
            .replace(/&/g, '&amp;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;');
    }

    const INITIAL_PASSENGERS = [
        { lastName: 'Ковалёв', firstName: 'Андрей', patronymic: 'Сергеевич', flightNumber: 'B2-891' },
        { lastName: 'Смирнова', firstName: 'Елена', patronymic: 'Игоревна', flightNumber: 'B2-891' },
        { lastName: 'Ковалёв', firstName: 'Дмитрий', patronymic: 'Олегович', flightNumber: 'B2-891' },
        { lastName: 'Морозов', firstName: 'Илья', patronymic: 'Викторович', flightNumber: 'B2-715' },
        { lastName: 'Волкова', firstName: 'Анна', patronymic: 'Петровна', flightNumber: 'B2-715' },
        { lastName: 'Смирнова', firstName: 'Ольга', patronymic: 'Андреевна', flightNumber: 'B2-891' },
    ];
```

### Русский

* Весь скрипт обёрнут в **IIFE** (`(function () { ... })();`) со строгим режимом `'use strict';` для изоляции переменных от глобальной области видимости.
* **`escapeHtml(s)`** заменяет символы `&`, `<` и `>` на безопасные HTML-сущности перед вставкой пользовательских строк через `innerHTML` — это защищает страницу от поломки разметки и XSS-атак.
* **`INITIAL_PASSENGERS`** — стартовый массив объектов с фамилией, именем, отчеством и номером рейса. На рейсе `B2-891` специально добавлены однофамильцы (`Ковалёв` и `Смирнова`), чтобы сразу можно было проверить работу поиска.

### English

* The script is wrapped in an **IIFE** (`(function () { ... })();`) with `'use strict';` to isolate local variables from the global scope.
* **`escapeHtml(s)`** replaces `&`, `<`, and `>` with safe HTML entities before injecting text via `innerHTML`, protecting the DOM from broken tags and XSS.
* **`INITIAL_PASSENGERS`** is a seed array of passenger records (`lastName`, `firstName`, `patronymic`, `flightNumber`), including shared last names (`Ковалёв` and `Смирнова`) on flight `B2-891` for immediate testing.

---

## 2. Вариант 1: Функциональный стиль и прототипное наследование / Variant 1: Functional Style and Prototypal Inheritance

```javascript
    function PersonProto(lastName, firstName, patronymic) {
        this._lastName = lastName;
        this._firstName = firstName;
        this._patronymic = patronymic;
    }

    // 5+ методов базового класса (прототип)
    PersonProto.prototype.getLastName = function () {
        return this._lastName;
    };
    PersonProto.prototype.setLastName = function (val) {
        this._lastName = String(val || '').trim();
    };
    PersonProto.prototype.getFullName = function () {
        return `${this._lastName} ${this._firstName} ${this._patronymic}`.trim();
    };
    PersonProto.prototype.setFullName = function (lastName, firstName, patronymic) {
        this._lastName = String(lastName || '').trim();
        this._firstName = String(firstName || '').trim();
        this._patronymic = String(patronymic || '').trim();
    };
    PersonProto.prototype.describe = function () {
        return `Человек: ${this.getFullName()}`;
    };

    function PassengerProto(lastName, firstName, patronymic, flightNumber) {
        PersonProto.call(this, lastName, firstName, patronymic);
        this._flightNumber = String(flightNumber || '').trim().toUpperCase();
    }

    PassengerProto.prototype = Object.create(PersonProto.prototype);
    PassengerProto.prototype.constructor = PassengerProto;
```

### Русский

* **Базовый конструктор `PersonProto`:** функция-конструктор, вызываемая через `new`. Свойства `_lastName`, `_firstName`, `_patronymic` записываются в создаваемый объект `this` (подчёркивание `_` — условное соглашение об «инкапсулированных» полях).
* **5 методов на `PersonProto.prototype`:** методы выносятся в `prototype`, а не создаются внутри функции-конструктора, чтобы все экземпляры делили одну функцию в памяти:
  1. `getLastName()` — возвращает фамилию.
  2. `setLastName(val)` — задаёт фамилию с приведением к строке и обрезкой пробелов (`.trim()`).
  3. `getFullName()` — собирает полное ФИО.
  4. `setFullName(lastName, firstName, patronymic)` — обновляет сразу все три поля ФИО.
  5. `describe()` — формирует текстовое описание человека.
* **Наследник `PassengerProto`:**
  * `PersonProto.call(this, ...)` вызывает родительский конструктор в контексте нового пассажира `this`, чтобы унаследовать поля ФИО.
  * Добавляется собственное поле `this._flightNumber` в верхнем регистре (`.toUpperCase()`).
  * `PassengerProto.prototype = Object.create(PersonProto.prototype);` выстраивает цепочку прототипов: `экземпляр PassengerProto → PassengerProto.prototype → PersonProto.prototype → Object.prototype`.
  * `PassengerProto.prototype.constructor = PassengerProto;` восстанавливает свойство `constructor` после перезаписи прототипа.

### English

* **Base constructor `PersonProto`:** a constructor function called with `new`, assigning `_lastName`, `_firstName`, and `_patronymic` onto `this` (leading underscore indicates an internal property by convention).
* **5 methods on `PersonProto.prototype`:** attaching methods to `prototype` avoids duplicating function instances for every object:
  1. `getLastName()` — returns the last name.
  2. `setLastName(val)` — updates the last name after converting to string and trimming whitespace.
  3. `getFullName()` — returns the full name.
  4. `setFullName(...)` — updates all three name parts at once.
  5. `describe()` — returns a text summary of the person.
* **Child constructor `PassengerProto`:**
  * `PersonProto.call(this, ...)` invokes the parent constructor with the child's `this` context to initialize name properties.
  * Adds `_flightNumber` normalized to uppercase.
  * `PassengerProto.prototype = Object.create(PersonProto.prototype);` links the prototype chain to inherit base methods.
  * `PassengerProto.prototype.constructor = PassengerProto;` restores the `constructor` reference.

---

## 3. Методы наследника `PassengerProto` и алгоритм поиска однофамильцев / `PassengerProto` Methods and Namesake Search

```javascript
    // 5+ методов класса-наследника (прототип)
    PassengerProto.prototype.getFlightNumber = function () {
        return this._flightNumber;
    };
    PassengerProto.prototype.setFlightNumber = function (val) {
        this._flightNumber = String(val || '').trim().toUpperCase();
    };
    PassengerProto.prototype.describe = function () {
        return `${this.getFullName()} (рейс ${this._flightNumber})`;
    };
    PassengerProto.prototype.addFromForm = function (list, formEl) {
        const ln = formEl.querySelector('[name="lastName"]').value.trim();
        const fn = formEl.querySelector('[name="firstName"]').value.trim();
        const pt = formEl.querySelector('[name="patronymic"]').value.trim();
        const fl = formEl.querySelector('[name="flightNumber"]').value.trim().toUpperCase();
        if (!ln || !fn || !pt || !fl) return null;
        const p = new PassengerProto(ln, fn, pt, fl);
        list.push(p);
        return p;
    };
    PassengerProto.prototype.renderAll = function (list, containerEl) {
        if (!list.length) {
            containerEl.innerHTML = '<p>Список пассажиров пуст.</p>';
            return;
        }
        containerEl.innerHTML = `
            <ol>
                ${list.map((p) => `<li><strong>${escapeHtml(p.getFullName())}</strong> — рейс <code>${escapeHtml(p.getFlightNumber())}</code></li>`).join('')}
            </ol>
        `;
    };
    PassengerProto.prototype.renderResult = function (list, flightNumber, containerEl) {
        const target = String(flightNumber || '').trim().toUpperCase();
        const onFlight = list.filter((p) => p.getFlightNumber() === target);
        if (!onFlight.length) {
            containerEl.innerHTML = `<p>На рейсе <strong>${escapeHtml(target)}</strong> пассажиры не найдены.</p>`;
            return;
        }
        const groups = {};
        onFlight.forEach((p) => {
            const key = p.getLastName().toLowerCase();
            if (!groups[key]) groups[key] = [];
            groups[key].push(p);
        });
        const namesakes = Object.values(groups).filter((arr) => arr.length > 1).flat();
        if (!namesakes.length) {
            containerEl.innerHTML = `<p>На рейсе <strong>${escapeHtml(target)}</strong> однофамильцев нет (всего пассажиров: ${onFlight.length}).</p>`;
            return;
        }
        containerEl.innerHTML = `
            <p>На рейсе <strong>${escapeHtml(target)}</strong> найдены однофамильцы (${namesakes.length} чел.):</p>
            <ul>
                ${namesakes.map((p) => `<li>${escapeHtml(p.getFullName())}</li>`).join('')}
            </ul>
        `;
    };
```

### Русский

В дочернем классе `PassengerProto` реализовано **6 методов** (больше требуемых 5):
1. `getFlightNumber()` и `setFlightNumber(val)` — получение и изменение номера рейса.
2. `describe()` — **переопределённый метод** (полиморфизм): расширяет описание человека информацией о рейсе.
3. `addFromForm(list, formEl)` — читает поля формы, проверяет, что все 4 поля заполнены (если пусто — возвращает `null`), создаёт новый объект `PassengerProto`, добавляет его в массив `list` и возвращает созданный экземпляр.
4. `renderAll(list, containerEl)` — выводит нумерованный список `<ol>` всех пассажиров в переданный DOM-контейнер.
5. `renderResult(list, flightNumber, containerEl)` — **поиск однофамильцев на заданном рейсе**:
   * Сначала отбирает пассажиров нужного рейса: `list.filter((p) => p.getFlightNumber() === target)`.
   * Группирует их по фамилии в нижнем регистре (`toLowerCase()`), чтобы регистр букв не мешал поиску.
   * Оставляет только те группы, где пассажиров с одинаковой фамилией больше одного (`arr.length > 1`), и разворачивает их в плоский массив через `.flat()`.
   * Выводит найденных однофамильцев списком `<ul>` либо сообщает, что пассажиров/однофамильцев нет.

### English

`PassengerProto` defines **6 methods** (exceeding the minimum of 5):
1. `getFlightNumber()` and `setFlightNumber(val)` — getter and setter for the flight number.
2. `describe()` — **overridden method** (polymorphism) adding flight details to the description.
3. `addFromForm(list, formEl)` — reads form inputs, validates that all fields are non-empty (returning `null` if incomplete), creates a `PassengerProto` instance, pushes it into `list`, and returns it.
4. `renderAll(list, containerEl)` — renders an ordered list `<ol>` of all passengers inside `containerEl`.
5. `renderResult(list, flightNumber, containerEl)` — **namesake search on a given flight**:
   * Filters passengers matching the target flight (`target`).
   * Groups them by lowercase last name (`toLowerCase()`) for case-insensitive matching.
   * Keeps only groups with more than one person (`arr.length > 1`) and flattens them with `.flat()`, rendering the results or an informative message.

---

## 4. Вариант 2: Синтаксис ES6 `class` и `extends` / Variant 2: ES6 `class` and `extends`

```javascript
    class PersonES6 {
        constructor(lastName, firstName, patronymic) {
            this._lastName = lastName;
            this._firstName = firstName;
            this._patronymic = patronymic;
        }

        get lastName() {
            return this._lastName;
        }

        set lastName(val) {
            this._lastName = String(val || '').trim();
        }

        get fullName() {
            return `${this._lastName} ${this._firstName} ${this._patronymic}`.trim();
        }

        set fullName(str) {
            const parts = String(str || '').trim().split(/\s+/);
            this._lastName = parts[0] || '';
            this._firstName = parts[1] || '';
            this._patronymic = parts[2] || '';
        }

        describe() {
            return `Человек: ${this.fullName}`;
        }
    }

    class PassengerES6 extends PersonES6 {
        constructor(lastName, firstName, patronymic, flightNumber) {
            super(lastName, firstName, patronymic);
            this._flightNumber = String(flightNumber || '').trim().toUpperCase();
        }

        get flightNumber() {
            return this._flightNumber;
        }

        set flightNumber(val) {
            this._flightNumber = String(val || '').trim().toUpperCase();
        }

        describe() {
            return `${this.fullName} (рейс ${this._flightNumber})`;
        }

        addFromForm(list, formEl) {
            const ln = formEl.querySelector('[name="lastName"]').value.trim();
            const fn = formEl.querySelector('[name="firstName"]').value.trim();
            const pt = formEl.querySelector('[name="patronymic"]').value.trim();
            const fl = formEl.querySelector('[name="flightNumber"]').value.trim().toUpperCase();
            if (!ln || !fn || !pt || !fl) return null;
            const p = new PassengerES6(ln, fn, pt, fl);
            list.push(p);
            return p;
        }

        renderAll(list, containerEl) {
            if (!list.length) {
                containerEl.innerHTML = '<p>Список пассажиров пуст.</p>';
                return;
            }
            containerEl.innerHTML = `
                <ol>
                    ${list.map((p) => `<li><strong>${escapeHtml(p.fullName)}</strong> — рейс <code>${escapeHtml(p.flightNumber)}</code></li>`).join('')}
                </ol>
            `;
        }

        renderResult(list, flightNumber, containerEl) {
            const target = String(flightNumber || '').trim().toUpperCase();
            const onFlight = list.filter((p) => p.flightNumber === target);
            if (!onFlight.length) {
                containerEl.innerHTML = `<p>На рейсе <strong>${escapeHtml(target)}</strong> пассажиры не найдены.</p>`;
                return;
            }
            const map = new Map();
            onFlight.forEach((p) => {
                const key = p.lastName.toLowerCase();
                if (!map.has(key)) map.set(key, []);
                map.get(key).push(p);
            });
            const namesakes = Array.from(map.values()).filter((g) => g.length > 1).flat();
            if (!namesakes.length) {
                containerEl.innerHTML = `<p>На рейсе <strong>${escapeHtml(target)}</strong> однофамильцев нет (всего пассажиров: ${onFlight.length}).</p>`;
                return;
            }
            containerEl.innerHTML = `
                <p>На рейсе <strong>${escapeHtml(target)}</strong> найдены однофамильцы (${namesakes.length} чел.):</p>
                <ul>
                    ${namesakes.map((p) => `<li>${escapeHtml(p.fullName)}</li>`).join('')}
                </ul>
            `;
        }
    }
```

### Русский

* Вторая реализация делает ровно то же самое, но с использованием синтаксиса ES6 `class` и `extends` (под капотом он всё равно опирается на прототипы JavaScript, но выглядит чище и нагляднее).
* **Свойства `get` / `set`:** вызываются как обычные свойства без круглых скобок (`p.fullName` вместо `p.getFullName()`, `p.flightNumber = 'b2-891'` вместо вызова сеттера-функции). В `set fullName(str)` строка разбивается по пробелам через `.split(/\s+/)`.
* **`super(...)`:** в конструкторе `PassengerES6` обязательно вызывается до обращения к `this`, чтобы инициализировать родительскую часть объекта.
* **Коллекция `Map` в `renderResult`:** вместо обычного объекта для группировки по фамилиям используется `new Map()`, что удобно и безопасно при любых ключах.

### English

* The second implementation provides the same domain logic using ES6 `class` and `extends` syntax (built over JavaScript's prototype model with cleaner syntax).
* **`get` / `set` accessors:** accessed like regular properties rather than method calls (e.g., `p.fullName` or `p.flightNumber = ...`). `set fullName(str)` splits the string by whitespace using `.split(/\s+/)`.
* **`super(...)`:** called inside `PassengerES6`'s `constructor` before accessing `this` to initialize the parent class state.
* **`Map` in `renderResult`:** groups passengers by lowercase last name using a `Map` instance, then filters groups with more than one match (`g.length > 1`) and flattens them.

---

## 5. Переключение вкладок и привязка событий к DOM / Tab Switching and DOM Wiring

```javascript
    document.addEventListener('DOMContentLoaded', () => {
        const tabs = Array.from(document.querySelectorAll('.oop-tab-btn'));
        const panels = Array.from(document.querySelectorAll('.oop-panel'));
        if (!tabs.length) return;

        tabs.forEach((btn) => {
            btn.addEventListener('click', () => {
                const targetId = btn.dataset.target;
                tabs.forEach((t) => t.classList.toggle('is-active', t === btn));
                panels.forEach((p) => p.classList.toggle('is-active', p.id === targetId));
            });
        });

        // --- Инициализация Варианта 1 (Прототипы) ---
        const protoList = INITIAL_PASSENGERS.map(
            (x) => new PassengerProto(x.lastName, x.firstName, x.patronymic, x.flightNumber)
        );
        const protoHelper = protoList[0];
        const protoForm = document.getElementById('oop-proto-form');
        const protoAllEl = document.getElementById('oop-proto-all');
        const protoResEl = document.getElementById('oop-proto-result');
        const protoFlightInput = document.getElementById('oop-proto-flight-search');
        const protoCheckBtn = document.getElementById('oop-proto-check-btn');

        protoHelper.renderAll(protoList, protoAllEl);
        protoHelper.renderResult(protoList, protoFlightInput.value, protoResEl);

        protoForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const added = protoHelper.addFromForm(protoList, protoForm);
            if (added) {
                protoForm.reset();
                protoHelper.renderAll(protoList, protoAllEl);
                protoHelper.renderResult(protoList, protoFlightInput.value, protoResEl);
            }
        });

        protoCheckBtn.addEventListener('click', () => {
            protoHelper.renderResult(protoList, protoFlightInput.value, protoResEl);
        });

        // --- Инициализация Варианта 2 (class / extends) ---
        const es6List = INITIAL_PASSENGERS.map(
            (x) => new PassengerES6(x.lastName, x.firstName, x.patronymic, x.flightNumber)
        );
        const es6Helper = es6List[0];
        const es6Form = document.getElementById('oop-es6-form');
        const es6AllEl = document.getElementById('oop-es6-all');
        const es6ResEl = document.getElementById('oop-es6-result');
        const es6FlightInput = document.getElementById('oop-es6-flight-search');
        const es6CheckBtn = document.getElementById('oop-es6-check-btn');

        es6Helper.renderAll(es6List, es6AllEl);
        es6Helper.renderResult(es6List, es6FlightInput.value, es6ResEl);

        es6Form.addEventListener('submit', (e) => {
            e.preventDefault();
            const added = es6Helper.addFromForm(es6List, es6Form);
            if (added) {
                es6Form.reset();
                es6Helper.renderAll(es6List, es6AllEl);
                es6Helper.renderResult(es6List, es6FlightInput.value, es6ResEl);
            }
        });

        es6CheckBtn.addEventListener('click', () => {
            es6Helper.renderResult(es6List, es6FlightInput.value, es6ResEl);
        });
    });
})();
```

### Русский

* **Вкладки (`tabs` и `panels`):** при клике на кнопку вкладки считывается `btn.dataset.target`, и класс `is-active` переключается на выбранной кнопке и соответствующей панели (`p.id === targetId`), позволяя переключаться между реализацией на прототипах и реализацией на ES6 `class`.
* **Инициализация обоих списков:** `INITIAL_PASSENGERS.map(...)` превращает исходные данные в массивы экземпляров `PassengerProto` и `PassengerES6`.
* **Первичный вывод и обработчики:**
  * Сразу при загрузке вызываются `renderAll` и `renderResult`, чтобы пользователь сразу видел список пассажиров и результат проверки по умолчанию.
  * Отправка формы (`'submit'`) отменяется через `e.preventDefault()`, добавляет нового пассажира, сбрасывает поля формы (`form.reset()`) и обновляет оба блока вывода (полный список и результат поиска однофамильцев).
  * Кнопка проверки рейса (`checkBtn`) пересчитывает список однофамильцев для введённого номера рейса.

### English

* **Tabs (`tabs` and `panels`):** clicking a tab reads `btn.dataset.target` and toggles `.is-active` on the clicked tab and matching panel (`p.id === targetId`) to switch between the prototype and ES6 `class` implementations.
* **Initializing both lists:** `INITIAL_PASSENGERS.map(...)` maps raw objects into `PassengerProto` and `PassengerES6` instances.
* **Initial render and event handlers:**
  * Calls `renderAll` and `renderResult` right away so the full list and initial namesake check are visible immediately.
  * Form `'submit'` calls `e.preventDefault()`, adds a new passenger, resets the form (`form.reset()`), and refreshes both the full list and namesake search output.
  * The flight check button (`checkBtn`) re-runs the namesake search for the entered flight number.