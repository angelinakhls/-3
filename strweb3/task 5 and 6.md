
# Защита заданий 5 и 6: Пагинация каталога и 3D-эффект объёма/параллакса — Русский / English

> **Суть заданий:**
> * **Задание 5 (Пагинация):** Разбить карточки товаров в каталоге на страницы (по умолчанию по 3 элемента), добавить выпадающий список для выбора количества отображаемых карточек, кнопки навигации («Назад», номера страниц, «Вперёд») и счётчик показанных элементов.
> * **Задание 6 (3D-эффект объёма и параллакса):** Реализовать интерактивный 3D-наклон карточки при движении курсора мыши (`perspective`, `rotateX`, `rotateY`), световой блик, следующий за курсором (`radial-gradient`), и параллакс-сдвиг фотографии внутри карточки.

---

## 1. Инициализация и 3D-эффект объёма / параллакса (Задание 6) / Setup and 3D Tilt & Parallax Effect (Task 6)

```javascript
/* Задания 5 и 6: Пагинация каталога (макс. 3 элемента + выбор количества) и 3D-эффект объёма/параллакса при наведении */
(function () {
    'use strict';

    document.addEventListener('DOMContentLoaded', () => {
        const grid = document.querySelector('.catalog-grid');
        if (!grid) return;

        const cards = Array.from(grid.querySelectorAll('.catalog-card'));

        //Задание 6
        cards.forEach((card) => {
            const glare = document.createElement('div');
            glare.className = 'card-glare';
            card.appendChild(glare);

            const img = card.querySelector('img');

            card.addEventListener('mousemove', (e) => {
                const rect = card.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                const centerX = rect.width / 2;
                const centerY = rect.height / 2;

                const rotateX = ((y - centerY) / centerY) * -10;
                const rotateY = ((x - centerX) / centerX) * 10;

                card.classList.add('is-tilting');
                card.style.transform = `perspective(900px) rotateX(${rotateX.toFixed(2)}deg) rotateY(${rotateY.toFixed(2)}deg) scale3d(1.03, 1.03, 1.03)`;

                const pctX = ((x / rect.width) * 100).toFixed(1);
                const pctY = ((y / rect.height) * 100).toFixed(1);
                glare.style.background = `radial-gradient(circle at ${pctX}\%${pctY}%, rgba(255,255,255,0.34), rgba(255,255,255,0) 62%)`;

                if (img) {
                    const moveX = ((x - centerX) / centerX) * -6;
                    const moveY = ((y - centerY) / centerY) * -6;
                    img.style.transform = `scale(1.07) translate(${moveX.toFixed(1)}px,${moveY.toFixed(1)}px)`;
                }
            });

            card.addEventListener('mouseleave', () => {
                card.classList.remove('is-tilting');
                card.style.transform = 'perspective(900px) rotateX(0deg) rotateY(0deg) scale3d(1, 1, 1)';
                glare.style.background = 'none';
                if (img) {
                    img.style.transform = 'scale(1) translate(0, 0)';
                }
            });
        });
```

### Русский

* **Изоляция и поиск карточек:** Код обёрнут в **IIFE** со строгим режимом `'use strict';`. После загрузки DOM (`DOMContentLoaded`) находится сетка каталога `.catalog-grid` (если её нет на текущей странице — скрипт завершается). Все карточки `.catalog-card` собираются в массив `cards` с помощью `Array.from`.
* **Создание слоя светового блика:** Для каждой карточки через `document.createElement('div')` динамически создаётся прозрачный слой `.card-glare` и добавляется внутрь карточки (`card.appendChild(glare)`). Также сохраняется ссылка на картинку `img` внутри карточки.
* **Расчёт 3D-наклона при движении мыши (`'mousemove'`):**
  1. `card.getBoundingClientRect()` получает координаты и размеры карточки на экране.
  2. `x = e.clientX - rect.left` и `y = e.clientY - rect.top` вычисляют координаты курсора **внутри самой карточки** (от левого верхнего угла).
  3. `centerX` и `centerY` — координаты центра карточки.
  4. Вычисляются углы поворота от `-10°` до `+10°`:
     * `rotateX = ((y - centerY) / centerY) * -10` — наклон по вертикали (минус нужен, чтобы при наведении на верхний край карточка наклонялась «назад», а при наведении на нижний — «вперёд»).
     * `rotateY = ((x - centerX) / centerX) * 10` — наклон по горизонтали (влево-вправо).
  5. Через `card.style.transform` применяется 3D-трансформация: `perspective(900px)` задаёт глубину 3D-пространства, `rotateX` и `rotateY` наклоняют карточку, а `scale3d(1.03, 1.03, 1.03)` слегка приподнимает (увеличивает) её.
* **Световой блик (`glare`):** Координаты курсора переводятся в проценты (`pctX`, `pctY`), и в ту же точку помещается центр радиального градиента `radial-gradient(...)`, создавая реалистичный блик на поверхности карточки.
* **Параллакс картинки (`img`):** Если в карточке есть изображение, оно сдвигается в **противоположную** от курсора сторону на величину до `6px` (`* -6`) и слегка увеличивается (`scale(1.07)`, чтобы при сдвиге не появлялись пустые края). Это создаёт эффект глубины между фоном карточки и фотографией.
* **Возврат в исходное состояние (`'mouseleave'`):** Когда курсор уходит с карточки, углы сбрасываются в `0deg`, масштаб возвращается в `1`, а блик убирается (`glare.style.background = 'none'`).

### English

* **Setup and card collection:** Wrapped in an **IIFE** with `'use strict';`. On `DOMContentLoaded`, it locates `.catalog-grid` and converts all `.catalog-card` elements into a standard array `cards` via `Array.from`.
* **Glare overlay creation:** For each card, `document.createElement('div')` creates a `.card-glare` overlay and appends it via `card.appendChild(glare)`. It also caches the inner `img` element.
* **3D tilt calculation on `'mousemove'`:**
  1. `card.getBoundingClientRect()` retrieves the card's screen bounds.
  2. `x = e.clientX - rect.left` and `y = e.clientY - rect.top` find the cursor position relative to the card's top-left corner.
  3. `centerX` and `centerY` find the center of the card.
  4. Tilt angles (`-10°` to `+10°`) are calculated: `rotateX` tilts vertically (inverted with `-10` for natural 3D perspective) and `rotateY` tilts horizontally.
  5. `card.style.transform` applies `perspective(900px)`, `rotateX`, `rotateY`, and a subtle `scale3d(1.03, 1.03, 1.03)` zoom.
* **Dynamic glare (`glare`):** Cursor coordinates are converted into percentages (`pctX`, `pctY`) to position a `radial-gradient` light reflection directly under the mouse.
* **Image parallax (`img`):** The image shifts in the opposite direction of the cursor (`* -6` pixels) while scaled up (`scale(1.07)` so edges stay covered), creating depth between the card frame and the photo.
* **Reset on `'mouseleave'`:** When the mouse leaves the card, rotations reset to `0deg`, scale returns to `1`, and the glare is cleared.

---

## 2. Пагинация каталога (Задание 5) / Catalog Pagination (Task 5)

```javascript
        // Задание 5
        const paginationContainer = document.getElementById('catalog-pagination');
        const perPageSelect = document.getElementById('catalog-per-page');
        if (!paginationContainer || !cards.length) return;

        let perPage = Number(perPageSelect ? perPageSelect.value : 3) || 3;
        let currentPage = 1;

        function renderCatalogPage() {
            const total = cards.length;
            const effectivePerPage = perPage === 0 ? total : perPage;
            const totalPages = Math.max(1, Math.ceil(total / effectivePerPage));
            if (currentPage > totalPages) currentPage = totalPages;

            const startIdx = (currentPage - 1) * effectivePerPage;
            const endIdx = startIdx + effectivePerPage;

            cards.forEach((card, idx) => {
                card.style.display = (idx >= startIdx && idx < endIdx) ? '' : 'none';
            });

            const shownFrom = total ? startIdx + 1 : 0;
            const shownTo = Math.min(total, endIdx);

            let btns = `<button type="button" class="lr3-page-btn" data-page="${currentPage - 1}" ${currentPage <= 1 ? 'disabled' : ''}>← Назад</button>`;
            for (let p = 1; p <= totalPages; p++) {
                btns += `<button type="button" class="lr3-page-btn ${p === currentPage ? 'is-active' : ''}" data-page="${p}">${p}</button>`;
            }
            btns += `<button type="button" class="lr3-page-btn" data-page="${currentPage + 1}" ${currentPage >= totalPages ? 'disabled' : ''}>Вперёд →</button>`;

            paginationContainer.innerHTML = `
                <span class="lr3-page-info">Показано ${shownFrom}–${shownTo} из ${total} изделий (стр. ${currentPage} из ${totalPages})</span>
                <div class="lr3-pagination-buttons">${btns}</div>
            `;

            paginationContainer.querySelectorAll('.lr3-page-btn').forEach((btn) => {
                btn.addEventListener('click', () => {
                    const p = Number(btn.dataset.page);
                    if (p >= 1 && p <= totalPages && p !== currentPage) {
                        currentPage = p;
                        renderCatalogPage();
                        grid.scrollIntoView({ behavior: 'smooth', block: 'start' });
                    }
                });
            });
        }

        if (perPageSelect) {
            perPageSelect.addEventListener('change', () => {
                perPage = Number(perPageSelect.value);
                currentPage = 1;
                renderCatalogPage();
            });
        }

        renderCatalogPage();
    });
})();
```

### Русский

* **Состояние пагинации:**
  * Находятся контейнер пагинации `paginationContainer` и выпадающий список выбора количества карточек `perPageSelect`.
  * `perPage` — сколько карточек показывать на одной странице (по умолчанию `3` согласно заданию).
  * `currentPage = 1` — номер текущей открытой страницы.
* **Расчёт границ и показ карточек (`renderCatalogPage`):**
  1. `effectivePerPage = perPage === 0 ? total : perPage` — если в селекте выбран пункт «Показать все» (значение `0`), на странице отображаются сразу все карточки (`total`).
  2. `totalPages = Math.max(1, Math.ceil(total / effectivePerPage))` — общее количество страниц вычисляется делением с округлением вверх (`Math.ceil`).
  3. Вычисляется диапазон индексов для текущей страницы: начальный индекс `startIdx = (currentPage - 1) * effectivePerPage` и конечный `endIdx = startIdx + effectivePerPage`.
  4. Цикл `cards.forEach` проходит по всем карточкам: если индекс карточки попадает в полуинтервал `[startIdx; endIdx)`, её свойство `style.display` очищается (`''`, возвращая стандартное отображение сетки), иначе ставится `'none'` (карточка скрывается).
* **Генерация кнопок и счётчика:**
  * Формируется HTML-строка кнопок `btns`: кнопка «← Назад» (блокируется атрибутом `disabled`, если открыта первая страница `currentPage <= 1`), кнопки с номерами страниц от `1` до `totalPages` (активной странице добавляется класс `is-active`) и кнопка «Вперёд →» (блокируется на последней странице).
  * В `paginationContainer.innerHTML` выводится текстовая сводка (`Показано 1–3 из 9 изделий`) и блок кнопок.
* **Обработка кликов по страницам и смены количества:**
  * На каждую кнопку `.lr3-page-btn` вешается клик: скрипт читает целевой номер страницы из `btn.dataset.page`, обновляет `currentPage`, перерисовывает страницу через `renderCatalogPage()` и плавно прокручивает экран к началу каталога с помощью **`grid.scrollIntoView({ behavior: 'smooth', block: 'start' })`**.
  * При изменении значения в выпадающем списке `perPageSelect` (`'change'`) обновляется `perPage`, текущая страница сбрасывается на первую (`currentPage = 1`), и каталог перестраивается.
  * Финальный вызов `renderCatalogPage()` сразу разбивает каталог на страницы при загрузке.

### English

* **Pagination state:**
  * Finds `paginationContainer` and the items-per-page dropdown `perPageSelect`.
  * `perPage` stores how many items appear per page (`3` by default). `currentPage` starts at `1`.
* **Calculating page slices (`renderCatalogPage`):**
  1. `effectivePerPage = perPage === 0 ? total : perPage` handles a "Show all" option (`0`) by displaying all items at once.
  2. `totalPages` is computed using `Math.ceil(total / effectivePerPage)` (at least `1`).
  3. `startIdx` and `endIdx` define the slice of card indices visible on `currentPage`.
  4. `cards.forEach` shows cards whose index falls within `[startIdx, endIdx)` (`display = ''`) and hides the rest (`display = 'none'`).
* **Building buttons and status info:**
  * Builds `btns` containing a "← Back" button (`disabled` on page 1), numbered page buttons (`1..totalPages` with `is-active` on the current page), and a "Next →" button (`disabled` on the last page).
  * Updates `paginationContainer.innerHTML` with the item range summary and page buttons.
* **Page navigation and per-page selector events:**
  * Clicking any `.lr3-page-btn` reads `btn.dataset.page`, updates `currentPage`, calls `renderCatalogPage()`, and smoothly scrolls back to the top of the catalog grid via **`grid.scrollIntoView({ behavior: 'smooth', block: 'start' })`**.
  * Changing `perPageSelect` (`'change'`) updates `perPage`, resets `currentPage` to `1`, and re-renders the catalog.
  * The final `renderCatalogPage()` call initializes the paginated view on page load.