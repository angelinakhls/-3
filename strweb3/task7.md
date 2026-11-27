
# Защита задания 7: Расчёт возраста, определение дня недели и проверка совершеннолетия — Русский / English

> **Суть задания:** Реализовать форму проверки даты рождения, которая вычисляет точный возраст пользователя в полных годах, определяет день недели даты рождения для совершеннолетних (`18+`) и выводит предупреждение (`alert` и сообщение на странице) для несовершеннолетних.

---

## 1. Поиск элементов и справочник дней недели / Selecting Elements and Weekday Lookup

```javascript
//           ЗАДАНИЕ 7: Запрос даты рождения, расчёт лет, день недели / alert

const ageForm = document.getElementById('age-check-form');
const birthInput = document.getElementById('birth-date-input');
const ageResult = document.getElementById('age-check-result');

if (ageForm && birthInput && ageResult) {
    const WEEKDAYS_RU = [
        'воскресенье',
        'понедельник',
        'вторник',
        'среда',
        'четверг',
        'пятница',
        'суббота',
    ];
```

### Русский

* Через `document.getElementById` находятся три элемента: сама форма (`ageForm`), поле ввода даты (`birthInput`) и блок вывода результата (`ageResult`).
* Условие `if (ageForm && birthInput && ageResult)` защищает код от ошибок на тех страницах, где нет этой формы.
* Массив **`WEEKDAYS_RU`** сопоставляет числовой индекс дня недели, который возвращает метод JavaScript `Date.prototype.getDay()`, с русским названием: в JS отсчёт начинается с воскресенья (`0` — `'воскресенье'`, `1` — `'понедельник'`, ..., `6` — `'суббота'`).

### English

* Using `document.getElementById`, we retrieve the form (`ageForm`), the date input (`birthInput`), and the result container (`ageResult`).
* The guard `if (ageForm && birthInput && ageResult)` prevents runtime errors on pages without this form.
* **`WEEKDAYS_RU`** maps the numeric weekday returned by `Date.prototype.getDay()` to Russian weekday names, where Sunday is index `0` (`0` = `'воскресенье'`, `1` = `'понедельник'`, ..., `6` = `'суббота'`).

---

## 2. Парсинг даты и точный расчёт возраста / Date Parsing and Exact Age Calculation

```javascript
    ageForm.addEventListener('submit', (e) => {
        e.preventDefault();
        if (!birthInput.value) {
            ageResult.textContent = 'Пожалуйста, выберите дату рождения.';
            return;
        }
        const parts = birthInput.value.split('-').map(Number);
        const birthDate = new Date(parts[0], parts[1] - 1, parts[2]);
        const today = new Date();

        let age = today.getFullYear() - birthDate.getFullYear();
        const m = today.getMonth() - birthDate.getMonth();
        if (m < 0 || (m === 0 && today.getDate() < birthDate.getDate())) {
            age--;
        }

        if (age < 0 || age > 130) {
            ageResult.textContent = 'Укажите корректную дату рождения.';
            return;
        }
```

### Русский

* При отправке формы (`'submit'`) вызывается **`e.preventDefault()`**, чтобы страница не перезагружалась.
* Если дата не выбрана (`!birthInput.value`), выводится подсказка и обработчик завершается (`return`).
* Значение `<input type="date">` приходит в формате строки `'ГГГГ-ММ-ДД'`. Строка разбивается по дефису `.split('-')` и переводится в числа через `.map(Number)` — получаем массив `[год, месяц, день]`.
* Создаётся объект `new Date(parts[0], parts[1] - 1, parts[2])`: месяцы в конструкторе `Date` нумеруются с `0` (январь = `0`, декабрь = `11`), поэтому вычитаем `1` из номера месяца (`parts[1] - 1`) и создаём дату в **локальном часовом поясе** без сдвига на сутки из-за UTC.
* **Корректировка возраста с учётом дня рождения в текущем году:**
  * Сначала берётся простая разница лет: `today.getFullYear() - birthDate.getFullYear()`.
  * Далее проверяется, наступил ли уже день рождения в этом году: если текущий месяц меньше месяца рождения (`m < 0`) либо месяц совпадает (`m === 0`), но текущее число меньше числа рождения (`today.getDate() < birthDate.getDate()`), значит именин в этом году ещё не было и возраст уменьшается на единицу (`age--`).
* Проверка `if (age < 0 || age > 130)` отсекает даты из будущего и заведомо нереалистичные значения.

### English

* On form `'submit'`, **`e.preventDefault()`** stops the browser from reloading the page.
* If no date is selected (`!birthInput.value`), it displays a prompt and returns early.
* An `<input type="date">` value has the string format `'YYYY-MM-DD'`. Splitting by `'-'` and mapping to `Number` produces `[year, month, day]`.
* `new Date(parts[0], parts[1] - 1, parts[2])` constructs the date in **local time** (subtracting `1` from the month because JS months are `0`-indexed), avoiding UTC timezone off-by-one bugs.
* **Adjusting age for whether the birthday has occurred this year:**
  * Starts with `today.getFullYear() - birthDate.getFullYear()`.
  * Checks if today is before this year's birthday: if `m < 0`, or `m === 0 && today.getDate() < birthDate.getDate()`, it decrements `age` by `1` (`age--`).
* The `if (age < 0 || age > 130)` check rejects future dates and out-of-range ages.

---

## 3. Проверка совершеннолетия и вывод результата / Majority