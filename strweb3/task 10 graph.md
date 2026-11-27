
# Защита задания 10 (Вариант 28): График разложения $\ln(1 + x)$ в ряд Тейлора (Chart.js) — Русский / English

> **Суть задания (Вариант 28):** Вычислить значения функции $\ln(1 + x)$ с помощью разложения в степенной ряд Тейлора $\sum_{n=1}^{\infty} \frac{(-1)^{n-1} x^n}{n}$ при $\vert{}x\vert{} < 1$ с заданной точностью $\varepsilon$. Сформировать таблицу результатов (`x`, `n`, `F(x)`, `Math F(x)`, `eps`), построить два графика разных цветов на одной координатной плоскости с помощью **Chart.js**, добавить легенду, подписи осей, пользовательскую аннотацию, пошаговую анимацию построения и кнопку сохранения графика в **PNG**.

---

## 1. Вычисление суммы ряда Тейлора / Taylor Series Calculation (`computeLnSeries`)

```javascript
(function () {
    'use strict';

    function computeLnSeries(x, eps, maxTerms = 500) {
        let sum = 0;
        let n = 0;
        for (let k = 1; k <= maxTerms; k++) {
            const term = Math.pow(-1, k - 1) * Math.pow(x, k) / k;
            sum += term;
            n = k;
            if (Math.abs(term) < eps) break;
        }
        return { fx: sum, n };
    }
```

### Русский

* Весь скрипт обёрнут в **IIFE** (`(function () { ... })();`) со строгим режимом `'use strict';` для изоляции переменных от глобальной области видимости.
* Функция **`computeLnSeries(x, eps, maxTerms = 500)`** вычисляет приближённое значение $\ln(1 + x)$ в точке `x` с заданной точностью `eps`:
  * `sum` — накопитель суммы ряда; `n` — счётчик просуммированных членов ряда.
  * В цикле `for` (от `k = 1` до `maxTerms = 500` как страховки от бесконечного цикла при очень маленьком `eps`) на каждом шаге вычисляется очередной член знакочередующегося ряда по формуле варианта 28:
    $$\text{term}_k = \frac{(-1)^{k-1} \cdot x^k}{k}$$
  * Член прибавляется к `sum`, а условие **`if (Math.abs(term) < eps) break;`** останавливает суммирование, как только модуль очередного слагаемого становится меньше заданной погрешности `eps`.
  * Функция возвращает объект `{ fx: sum, n }` — приближённое значение функции и количество членов ряда, которое потребовалось для достижения нужной точности.

### English

* The script is wrapped in an **IIFE** (`(function () { ... })();`) with `'use strict';` to isolate variables from the global scope.
* The **`computeLnSeries(x, eps, maxTerms = 500)`** function calculates an approximate value of $\ln(1 + x)$ at `x` with precision `eps`:
  * `sum` accumulates the series sum; `n` counts how many terms were added.
  * Inside the `for` loop (up to `maxTerms = 500` as a safety limit), each iteration calculates the next term of the alternating series:
    $$\text{term}_k = \frac{(-1)^{k-1} \cdot x^k}{k}$$
  * The term is added to `sum`, and **`if (Math.abs(term) < eps) break;`** stops the loop once the absolute value of the current term drops below `eps`.
  * It returns an object `{ fx: sum, n }` containing the approximated value and the number of terms needed.

---

## 2. Пользовательский плагин аннотации для Chart.js / Custom Annotation Plugin (`annotationPlugin`)

```javascript
    // Пользовательский плагин аннотации для Chart.js (вертикальная линия x = 0 и текстовая аннотация)
    const annotationPlugin = {
        id: 'lr3Annotation',
        afterDraw(chart, args, opts) {
            const { ctx, chartArea, scales } = chart;
            if (!chartArea || !scales.x || !scales.y) return;

            ctx.save();
            const zeroIdx = chart.data.labels.findIndex((lbl) => Math.abs(Number(lbl)) < 1e-6);
            if (zeroIdx >= 0) {
                const xPos = scales.x.getPixelForValue(zeroIdx);
                ctx.beginPath();
                ctx.setLineDash([6, 4]);
                ctx.strokeStyle = '#d97706';
                ctx.lineWidth = 1.5;
                ctx.moveTo(xPos, chartArea.top);
                ctx.lineTo(xPos, chartArea.bottom);
                ctx.stroke();
                ctx.setLineDash([]);

                ctx.fillStyle = '#b45309';
                ctx.font = 'bold 12px Manrope, sans-serif';
                ctx.fillText('x = 0, ln(1) = 0', xPos + 6, chartArea.top + 18);
            }

            if (opts && opts.note) {
                const boxX = chartArea.left + 12;
                const boxY = chartArea.bottom - 38;
                ctx.fillStyle = 'rgba(29, 77, 66, 0.9)';
                ctx.fillRect(boxX, boxY, 270, 28);
                ctx.fillStyle = '#ffffff';
                ctx.font = '12px Manrope, sans-serif';
                ctx.fillText(opts.note, boxX + 10, boxY + 18);
            }
            ctx.restore();
        },
    };
```

### Русский

Чтобы не подключать внешние библиотеки плагинов, я написала собственный плагин **`annotationPlugin`** для Chart.js с идентификатором `id: 'lr3Annotation'`:

* Хук **`afterDraw`** вызывается библиотекой автоматически сразу после отрисовки осей и линий графика на элементе `<canvas>`.
* Из объекта `chart` извлекаются контекст рисования `ctx`, границы области графика `chartArea` и шкалы координат `scales`. Вызовы `ctx.save()` и `ctx.restore()` сохраняют и восстанавливают настройки кисти Canvas, чтобы не испортить остальные элементы графика.
* **Вертикальная линия в нуле:** с помощью `findIndex` ищется индекс метки `x = 0`. Через `scales.x.getPixelForValue(zeroIdx)` индекс переводится в точную пиксельную координату `xPos`. Затем пунктирной линией (`setLineDash([6, 4])`) рисуется вертикальная ось от `chartArea.top` до `chartArea.bottom` с подписью `'x = 0, ln(1) = 0'`.
* **Информационная плашка:** в левом нижнем углу области графика рисуется прямоугольник (`fillRect`) с текстом из параметров плагина (`opts.note`), где выводится текущая точность `eps` и максимальное расхождение `max |Δ|` между рядом и точной функцией.

### English

Instead of loading external plugin libraries, I created a custom Chart.js plugin **`annotationPlugin`** with `id: 'lr3Annotation'`:

* The **`afterDraw`** hook runs automatically right after Chart.js draws the axes and lines on the `<canvas>`.
* It extracts the 2D drawing context `ctx`, plot boundaries `chartArea`, and coordinate `scales`. `ctx.save()` and `ctx.restore()` isolate canvas styling changes so they do not affect other chart elements.
* **Vertical line at zero:** `findIndex` locates the label for `x = 0`, and `scales.x.getPixelForValue(zeroIdx)` converts that index into an exact pixel position `xPos`. A dashed vertical line (`setLineDash([6, 4])`) is drawn from `chartArea.top` to `chartArea.bottom` with the label `'x = 0, ln(1) = 0'`.
* **Summary badge:** In the bottom-left corner of the chart area, it draws a filled rectangle (`fillRect`) displaying `opts.note` — the current `eps` and maximum error `max |Δ|` between the series and `Math.log`.

---

## 3. Табулирование функции и заполнение HTML-таблицы / Tabulating Values and Rendering the HTML Table

```javascript
    document.addEventListener('DOMContentLoaded', () => {
        const canvas = document.getElementById('ln-series-chart');
        if (!canvas) return;

        const form = document.getElementById('chart-params-form');
        const epsInput = document.getElementById('chart-eps');
        const stepInput = document.getElementById('chart-step');
        const tbody = document.getElementById('series-table-tbody');
        const saveBtn = document.getElementById('chart-save-btn');
        let chartInstance = null;

        function buildChartAndTable() {
            const eps = Math.max(1e-7, Math.min(0.5, Number(epsInput.value) || 0.001));
            const step = Math.max(0.05, Math.min(0.4, Number(stepInput.value) || 0.1));

            const rows = [];
            for (let x = -0.8; x <= 0.9001; x += step) {
                const rx = Number(x.toFixed(3));
                if (Math.abs(rx) >= 1) continue;
                const { fx, n } = computeLnSeries(rx, eps);
                const mathFx = Math.log(1 + rx);
                rows.push({ x: rx, n, fx, mathFx, eps });
            }

            tbody.innerHTML = rows.map((r) => `
                <tr>
                    <td>${r.x.toFixed(2)}</td>
                    <td>${r.n}</td>
                    <td>${r.fx.toFixed(6)}</td>
                    <td>${r.mathFx.toFixed(6)}</td>
                    <td>${r.eps}</td>
                </tr>
            `).join('');
```

### Русский

* При загрузке страницы находятся `<canvas>`, форма параметров (`epsInput`, `stepInput`), тело таблицы `tbody` и кнопка сохранения `saveBtn`. В переменной `chartInstance` хранится ссылка на текущий экземпляр графика.
* В функции **`buildChartAndTable()`**:
  * Считываются и ограничиваются безопасными рамками точность `eps` (от `1e-7` до `0.5`) и шаг `step` (от `0.05` до `0.4`).
  * Цикл `for` проходит по аргументу `x` на интервале сходимости ряда $|x| < 1$ (от `-0.8` до `0.9` с шагом `step`). Округление `Number(x.toFixed(3))` убирает погрешность двоичной арифметики плавающей точки (когда вместо `0.3` получается `0.30000000000000004`).
  * Для каждой точки вызывается `computeLnSeries(rx, eps)` и эталонная функция **`Math.log(1 + rx)`**, а результат сохраняется в массив `rows`.
  * Через `.map()` и `tbody.innerHTML` автоматически генерируются строки `<tr>` с 5 столбцами по заданию: `x`, `n`, `F(x)`, `Math F(x)` и `eps`.

### English

* On `DOMContentLoaded`, the script locates the `<canvas>`, parameter inputs (`epsInput`, `stepInput`), table body `tbody`, and save button `saveBtn`. `chartInstance` holds the active chart reference.
* Inside **`buildChartAndTable()`**:
  * `eps` and `step` are read from inputs and clamped within safe bounds.
  * The `for` loop iterates over `x` inside the convergence interval $|x| < 1$ (from `-0.8` to `0.9`). `Number(x.toFixed(3))` avoids JavaScript floating-point precision artifacts.
  * For each point, it computes `computeLnSeries(rx, eps)` and the built-in **`Math.log(1 + rx)`**, storing the results in `rows`.
  * Using `.map()` and `tbody.innerHTML`, it populates the table rows `<tr>` with all 5 required columns: `x`, `n`, `F(x)`, `Math F(x)`, and `eps`.

---

## 4. Настройка Chart.js: два графика, анимация и сохранение в PNG / Chart.js Setup, Animation, and PNG Export

```javascript
            if (typeof Chart === 'undefined') {
                return;
            }

            const labels = rows.map((r) => r.x.toFixed(2));
            const seriesValues = rows.map((r) => Number(r.fx.toFixed(6)));
            const mathValues = rows.map((r) => Number(r.mathFx.toFixed(6)));
            const maxDiff = rows.reduce((m, r) => Math.max(m, Math.abs(r.fx - r.mathFx)), 0);

            if (chartInstance) {
                chartInstance.destroy();
            }

            const totalDuration = 1600;
            const delayBetweenPoints = totalDuration / Math.max(1, labels.length);

            chartInstance = new Chart(canvas.getContext('2d'), {
                type: 'line',
                data: {
                    labels,
                    datasets: [
                        {
                            label: 'Разложение в ряд F(x) = Σ(-1)^(n-1)·x^n/n',
                            data: seriesValues,
                            borderColor: '#1d4d42',
                            backgroundColor: 'rgba(29, 77, 66, 0.14)',
                            borderWidth: 3,
                            pointRadius: 4,
                            tension: 0.25,
                        },
                        {
                            label: 'Точная функция Math.log(1 + x)',
                            data: mathValues,
                            borderColor: '#e11d48',
                            backgroundColor: 'rgba(225, 29, 72, 0.10)',
                            borderWidth: 2.5,
                            borderDash: [7, 5],
                            pointRadius: 3,
                            tension: 0.25,
                        },
                    ],
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    animation: {
                        x: {
                            type: 'number',
                            easing: 'easeOutQuart',
                            duration: delayBetweenPoints,
                            from: NaN,
                            delay(ctx) {
                                return ctx.type === 'data' ? ctx.dataIndex * delayBetweenPoints : 0;
                            },
                        },
                    },
                    plugins: {
                        title: {
                            display: true,
                            text: 'Вариант 28: Сравнение ряда Тейлора F(x) и Math.log(1 + x) при |x| < 1',
                            font: { size: 15, weight: 'bold' },
                        },
                        legend: {
                            display: true,
                            position: 'top',
                        },
                        tooltip: {
                            mode: 'index',
                            intersect: false,
                        },
                        lr3Annotation: {
                            note: `eps = ${eps} \vert{} max \vert{}Δ\vert{} =${maxDiff.toExponential(2)}`,
                        },
                    },
                    scales: {
                        x: {
                            title: { display: true, text: 'Аргумент x (|x| < 1)', font: { weight: 'bold' } },
                            grid: { color: 'rgba(148, 163, 184, 0.25)' },
                        },
                        y: {
                            title: { display: true, text: 'Значение функции y = ln(1 + x)', font: { weight: 'bold' } },
                            grid: { color: 'rgba(148, 163, 184, 0.25)' },
                        },
                    },
                },
                plugins: [annotationPlugin],
            });
        }

        if (form) {
            form.addEventListener('submit', (e) => {
                e.preventDefault();
                buildChartAndTable();
            });
        }

        if (saveBtn) {
            saveBtn.addEventListener('click', () => {
                if (!chartInstance) return;
                const link = document.createElement('a');
                link.download = 'chart_variant28_ln_1_plus_x.png';
                link.href = chartInstance.toBase64Image('image/png', 1.0);
                link.click();
            });
        }

        buildChartAndTable();
    });
})();
```

### Русский

1. **Подготовка данных и очистка старого графика:**
   * Из массива `rows` формируются три массива для Chart.js: подписи оси X (`labels`), точки ряда (`seriesValues`) и точки эталонной функции (`mathValues`), а через `.reduce()` вычисляется максимальная ошибка `maxDiff`.
   * Если пользователь изменил `eps` или шаг и нажал «Перестроить», старый экземпляр графика обязательно удаляется через **`chartInstance.destroy()`**, иначе на одном `<canvas>` наложатся два графика и начнут мерцать при наведении мыши.
2. **Два набора данных (`datasets`) на одной оси:**
   * Первый набор — сумма ряда `F(x)`, рисуется сплошной тёмно-зелёной линией (`#1d4d42`).
   * Второй набор — точная функция `Math.log(1 + x)`, рисуется пунктирной малиновой линией (`#e11d48`, `borderDash: [7, 5]`), чтобы при совпадении кривых обе линии оставались хорошо различимыми.
3. **Пошаговая анимация (`animation`):**
   * Общее время отрисовки `totalDuration = 1600` мс делится на количество точек (`delayBetweenPoints`). Функция `delay(ctx)` задаёт задержку пропорционально номеру точки (`ctx.dataIndex * delayBetweenPoints`), благодаря чему линии плавно рисуются слева направо.
4. **Оформление (`plugins` и `scales`):**
   * Включены заголовок графика (`title`), верхняя легенда (`legend`), общая всплывающая подсказка сразу для обеих кривых при наведении (`tooltip.mode = 'index'`), подписи обеих осей (`scales.x.title`, `scales.y.title`) и наш плагин `annotationPlugin`.
5. **Сохранение графика в PNG (`saveBtn`):**
   * При клике на кнопку сохранения вызывается встроенный метод Chart.js **`chartInstance.toBase64Image('image/png', 1.0)`**, который кодирует текущее изображение холста в Base64-ссылку формата PNG в максимальном качестве (`1.0`).
   * Динамически создаётся временная ссылка `<a>` с атрибутом `download = 'chart_variant28_ln_1_plus_x.png'`, и программный вызов `link.click()` запускает скачивание файла в браузере.

### English

1. **Data preparation and cleanup:**
   * `rows` is mapped into `labels`, `seriesValues`, and `mathValues`, while `.reduce()` finds the maximum error `maxDiff`.
   * Before redrawing with new parameters, **`chartInstance.destroy()`** removes the previous chart instance so two charts do not overlap on the same `<canvas>`.
2. **Two datasets on the same axes (`datasets`):**
   * Dataset 1 (`F(x)` series) uses a solid dark green line (`#1d4d42`).
   * Dataset 2 (`Math.log(1 + x)`) uses a dashed crimson line (`#e11d48`, `borderDash: [7, 5]`) so both curves remain clearly visible even when they overlap closely.
3. **Progressive animation (`animation`):**
   * `totalDuration = 1600` ms is divided across the number of points (`delayBetweenPoints`). The `delay(ctx)` callback staggers each point by `ctx.dataIndex * delayBetweenPoints`, drawing the curves smoothly from left to right.
4. **Chart elements (`plugins` and `scales`):**
   * Configures the chart `title`, top `legend`, index-based hover `tooltip`, axis titles (`scales.x.title`, `scales.y.title`), and registers `annotationPlugin`.
5. **Saving the chart as PNG (`saveBtn`):**
   * Clicking the save button calls **`chartInstance.toBase64Image('image/png', 1.0)`** to export the canvas as a full-quality PNG Data URL.
   * A temporary `<a>` element with `download = 'chart_variant28_ln_1_plus_x.png'` is created and triggered via `link.click()` to download the image file.