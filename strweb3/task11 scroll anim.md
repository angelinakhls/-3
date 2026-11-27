
# Защита задания 11 (Вариант 28): Анимации при скроллинге и Web Animations API — Русский / English

> **Суть задания:** Реализовать две анимации на странице мебельной фабрики:
> 1. **Скролл-анимация:** вращающиеся стулья, положение и поворот которых привязаны к прокрутке страницы (работают вперёд при скролле вниз и в обратном порядке при скролле вверх).
> 2. **Дополнительная анимация:** анимация на базе современного браузерного **Web Animations API** (`Element.animate()`) с интерактивным управлением воспроизведением.

---

## 1. Инициализация скролл-анимации и определение направления / Scroll Animation Setup and Direction Tracking

```javascript
(function () {
    'use strict';

    document.addEventListener('DOMContentLoaded', () => {
        // --- 1. Скролл-анимация: вращающиеся стулья (вперёд и назад) ---
        const section = document.getElementById('chairs-scroll-section');
        const chairs = Array.from(document.querySelectorAll('.scroll-chair'));
        const badge = document.getElementById('scroll-progress-badge');

        if (section && chairs.length) {
            let lastScrollY = window.scrollY;
            let directionLabel = 'ожидание прокрутки';
```

### Русский

* Весь скрипт обёрнут в **IIFE** (`(function () { ... })();`) со строгим режимом `'use strict';`, чтобы изолировать локальные переменные от глобальной области видимости.
* После загрузки DOM (`DOMContentLoaded`) скрипт находит три элемента: секцию-триггер `section`, коллекцию всех элементов стульев `.scroll-chair` (сразу превращая её в обычный массив через `Array.from`) и плашку статуса `badge`.
* Проверка `if (section && chairs.length)` защищает код от ошибок на страницах, где этого блока нет.
* Переменная **`lastScrollY`** запоминает предыдущую координату вертикальной прокрутки (`window.scrollY`), чтобы при следующем движении сравнить старую и новую позицию и понять, куда крутит пользователь — вниз или вверх.

### English

* The script is wrapped in an **IIFE** (`(function () { ... })();`) with `'use strict';` to isolate local variables from the global scope.
* Once the DOM loads (`DOMContentLoaded`), the script finds three elements: the trigger section `section`, all chair elements `.scroll-chair` (converted into a standard array via `Array.from`), and the status badge `badge`.
* The `if (section && chairs.length)` check prevents errors on pages that do not contain this section.
* The **`lastScrollY`** variable stores the previous vertical scroll position (`window.scrollY`) so the next scroll event can compare old and new positions to detect scrolling down or up.

---

## 2. Расчёт прогресса прокрутки и трансформация стульев / Calculating Scroll Progress and Transforming Chairs (`updateChairsOnScroll`)

```javascript
            function updateChairsOnScroll() {
                const currentY = window.scrollY;
                if (currentY > lastScrollY) directionLabel = 'вниз ↓ (прямое вращение)';
                else if (currentY < lastScrollY) directionLabel = 'вверх ↑ (обратное вращение)';
                lastScrollY = currentY;

                const rect = section.getBoundingClientRect();
                const vh = window.innerHeight || 800;
                // Прогресс от 0 (секция входит снизу) до 1 (секция уходит вверх)
                const rawProgress = (vh - rect.top) / (vh + rect.height);
                const progress = Math.max(0, Math.min(1, rawProgress));

                chairs.forEach((chair, idx) => {
                    const dir = idx % 2 === 0 ? 1 : -1;
                    const speed = 1 + idx * 0.25;
                    const angle = (progress - 0.5) * 540 * dir * speed;
                    const shiftY = Math.sin(progress * Math.PI * 2 + idx) * 28;
                    const shiftX = (progress - 0.5) * 90 * dir;
                    const scale = 0.85 + Math.sin(progress * Math.PI) * 0.28;

                    chair.style.transform = `translate3d(${shiftX.toFixed(1)}px, ${shiftY.toFixed(1)}px, 0) rotate(${angle.toFixed(1)}deg) scale(${scale.toFixed(3)})`; });  if (badge) { badge.textContent = `Прогресс скролла: ${Math.round(progress * 100)}% · Направление: ${directionLabel}`; } }  window.addEventListener('scroll', updateChairsOnScroll, { passive: true }); window.addEventListener('resize', updateChairsOnScroll, { passive: true }); updateChairsOnScroll(); } ```  ### Русский  Функция **`updateChairsOnScroll()`** пересчитывает координаты и углы поворота при каждом движении полосы прокрутки:  1. **Определение направления:** сравнивается текущая позиция `currentY` с `lastScrollY`. Если `currentY > lastScrollY` — страница едет вниз (прямое вращение); если меньше — вверх (обратное вращение). После этого `lastScrollY` обновляется. 2. **Вычисление нормализованного прогресса (`progress` от `0` до `1`):** * Метод **`section.getBoundingClientRect()`** возвращает размеры секции и её текущее расстояние до верхнего края экрана (`rect.top`). * `vh` (`window.innerHeight`) — высота видимой области окна браузера. * Формула `(vh - rect.top) / (vh + rect.height)` превращает положение секции в число: когда секция только появляется снизу экрана, `rawProgress = 0`; когда полностью уходит за верхний край экрана, `rawProgress = 1`. * Функция `Math.max(0, Math.min(1, rawProgress))` жёстко ограничивает значение диапазоном `[0; 1]`. Поскольку прогресс жёстко привязан к положению секции на экране, при прокрутке вверх `progress` плавно уменьшается от `1` до `0`, автоматически проигрывая всю анимацию в обратном порядке. 3. **Математика движения каждого стула (`.forEach`):** * `dir` — направление вращения: чётные стулья (`idx \% 2 === 0`) крутятся по часовой стрелке (`1`), нечётные — против (`-1`). * `speed` — индивидуальный коэффициент скорости, чтобы стулья двигались не синхронно, а с красивым параллакс-эффектом. * `angle` — угол поворота в градусах (до `540°`, то есть полтора оборота). * `shiftY` и `shiftX` — плавное покачивание по синусоиде (`Math.sin`) по вертикали и горизонтальное расхождение. * `scale` — пульсация масштаба: в середине экрана (`progress = 0.5`, где `sin(π/2) = 1`) стулья максимально увеличиваются, а по краям уменьшаются. * Все преобразования применяются через **`translate3d(...) rotate(...) scale(...)`** — использование `translate3d` переносит отрисовку на видеокарту (GPU) для максимальной плавности. 4. **Подписка на события:** обработчики навешиваются на `'scroll'` и `'resize'` (изменение размера окна) с флагом **`{ passive: true }`**, который сообщает браузеру, что скрипт не будет вызывать `preventDefault()`, благодаря чему прокрутка страницы не тормозит.  ### English  The **`updateChairsOnScroll()`** function recalculates positions and rotation angles on every scroll step:  1. **Direction tracking:** It compares `currentY` with `lastScrollY`. If `currentY > lastScrollY`, the user scrolls down (forward rotation); if smaller, the user scrolls up (reverse rotation). Then `lastScrollY` is updated. 2. **Normalized scroll progress (`progress` from `0` to `1`):** * **`section.getBoundingClientRect()`** gets the section's height and its distance from the top of the viewport (`rect.top`). * `vh` (`window.innerHeight`) is the viewport height. * The formula `(vh - rect.top) / (vh + rect.height)` maps the section's position to a number: `0` when it enters from the bottom, and `1` when it exits at the top. * `Math.max(0, Math.min(1, rawProgress))` clamps the value between `0` and `1`. Because `progress` depends directly on scroll position, scrolling back up smoothly decreases `progress` from `1` to `0`, naturally reversing the animation. 3. **Motion math for each chair (`.forEach`):** * `dir` — rotation direction: even chairs (`idx \% 2 === 0`) rotate clockwise (`1`), odd ones counter-clockwise (`-1`). * `speed` — individual speed multiplier so chairs move at slightly different rates. * `angle` — rotation angle in degrees (up to `540°`). * `shiftY` and `shiftX` — smooth wave motion using `Math.sin` vertically and horizontal shift. * `scale` — subtle zoom effect peaking at the center of the screen (`progress = 0.5`). * All transforms are applied via **`translate3d(...) rotate(...) scale(...)`** to trigger GPU hardware acceleration for smooth performance. 4. **Event listeners:** Attached to `'scroll'` and `'resize'` with **`{ passive: true }`**, telling the browser that `preventDefault()` will not be called so scrolling remains smooth.  ---  ## 3. Дополнительная анимация через Web Animations API / Additional Animation with Web Animations API (`Element.animate`)  ```javascript // --- 2. Дополнительная анимация по своему вкусу (Web Animations API) --- const waapiBox = document.getElementById('waapi-chair-box'); if (waapiBox && typeof waapiBox.animate === 'function') { const animation = waapiBox.animate( [ { transform: 'translateX(-140px) rotate(0deg) scale(0.85)', opacity: 0.7 }, { transform: 'translateX(0px) rotate(180deg) scale(1.18)', opacity: 1, offset: 0.5 }, { transform: 'translateX(140px) rotate(360deg) scale(0.85)', opacity: 0.7 }, ], { duration: 3200, iterations: Infinity, direction: 'alternate', easing: 'ease-in-out', } );  const btnPlay = document.getElementById('waapi-play'); const btnPause = document.getElementById('waapi-pause'); const btnReverse = document.getElementById('waapi-reverse'); const speedRange = document.getElementById('waapi-speed'); const speedOut = document.getElementById('waapi-speed-val');  if (btnPlay) btnPlay.addEventListener('click', () => animation.play()); if (btnPause) btnPause.addEventListener('click', () => animation.pause()); if (btnReverse) btnReverse.addEventListener('click', () => animation.reverse()); if (speedRange) { speedRange.addEventListener('input', () => { const rate = Number(speedRange.value) \vert{}\vert{} 1; animation.playbackRate = rate; if (speedOut) speedOut.textContent = `${rate.toFixed(1)}×`;
                });
            }
        }
    });
})();
```

### Русский

Во второй части используется нативный браузерный **Web Animations API (WAAPI)** — метод **`element.animate(keyframes, options)`**, который объединяет производительность CSS-анимаций с полным программным контролем из JavaScript:

1. **Проверка поддержки:** `typeof waapiBox.animate === 'function'` проверяет, поддерживает ли браузер WAAPI.
2. **Первый аргумент — массив ключевых кадров (`keyframes`):**
   * Начало (`0%`): сдвиг влево на `-140px`, угол `0°`, масштаб `0.85`, прозрачность `0.7`.
   * Середина (`offset: 0.5`, то есть `50%` времени): центр (`0px`), поворот на `180°`, увеличение до `1.18`, полная непрозрачность `1`.
   * Конец (`100%`): сдвиг вправо на `140px`, полный оборот `360°`, уменьшение до `0.85`.
3. **Второй аргумент — объект параметров (`options`):**
   * `duration: 3200` — длительность одного прохода `3.2` секунды (`3200` мс).
   * `iterations: Infinity` — бесконечное повторение цикла.
   * `direction: 'alternate'` — маятниковый режим: дойдя до правого края, элемент плавно едет обратно влево без резкого скачка.
   * `easing: 'ease-in-out'` — плавное ускорение в начале и замедление в конце.
4. **Интерактивное управление объектом `Animation`:**
   * Метод `waapiBox.animate()` возвращает контроллер `animation`, к которому привязаны элементы управления на странице:
     * Кнопка **Play** вызывает `animation.play()` (запуск/возобновление).
     * Кнопка **Pause** вызывает `animation.pause()` (пауза).
     * Кнопка **Reverse** вызывает `animation.reverse()` (инверсия направления прямо на ходу).
     * Ползунок скорости (`speedRange`) при событии `'input'` меняет свойство **`animation.playbackRate`**, позволяя ускорять или замедлять анимацию в реальном времени и отображая множитель скорости (например, `1.5×`).

### English

The second part uses the native **Web Animations API (WAAPI)** — the **`element.animate(keyframes, options)`** method, which combines CSS animation performance with full JavaScript playback control:

1. **Feature check:** `typeof waapiBox.animate === 'function'` verifies that the browser supports WAAPI.
2. **First argument — `keyframes` array:**
   * Start (`0%`): shifted left `-140px`, `0°` rotation, `0.85` scale, `0.7` opacity.
   * Middle (`offset: 0.5`, i.e., `50%` progress): centered at `0px`, rotated `180°`, scaled up to `1.18`, full opacity `1`.
   * End (`100%`): shifted right `140px`, full `360°` rotation, scaled back to `0.85`.
3. **Second argument — `options` object:**
   * `duration: 3200` — one cycle lasts `3.2` seconds (`3200` ms).
   * `iterations: Infinity` — loops forever.
   * `direction: 'alternate'` — ping-pong mode: the element smoothly reverses back instead of jumping to the start.
   * `easing: 'ease-in-out'` — smooth acceleration and deceleration.
4. **Interactive `Animation` object controls:**
   * `waapiBox.animate()` returns an `animation` controller connected to UI buttons:
     * **Play** calls `animation.play()`.
     * **Pause** calls `animation.pause()`.
     * **Reverse** calls `animation.reverse()` to flip direction on the fly.
     * The speed slider (`speedRange`) listens to `'input'` and updates **`animation.playbackRate`**, speeding up or slowing down the animation in real time while displaying the multiplier (e.g., `1.5×`).