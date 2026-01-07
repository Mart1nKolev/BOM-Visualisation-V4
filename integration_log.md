# Дневник на интеграцията: bom_classifier.html → unified_bom_viewer.html

Този документ проследява всички промени, направени при интегрирането на функционалности от `bom_classifier.html` в `unified_bom_viewer.html`.

---

## 🎯 Интеграция #1: Снимки с tooltip и modal при hover/click върху имена на детайли

**Дата:** 5 декември 2025  
**Статус:** ✅ Завършена и работеща

### Описание
Добавена е функционалност за показване на снимки при интеракция с имена на детайли:
- При **hover** (посочване с мишка) - показва се малък tooltip със снимка
- При **click** (кликване) - отваря се modal прозорец с голяма снимка

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. CSS стилове (добавени ~линии 370-550)
```css
/* Tooltip за снимки */
.image-tooltip {
    color: #007bff;
    text-decoration: underline;
    cursor: pointer;
    position: relative;
}

.tooltip-image {
    position: fixed;
    z-index: 10000;
    background: white;
    border: 2px solid #007bff;
    border-radius: 8px;
    padding: 10px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.3);
    max-width: 400px;
    pointer-events: none;
}

.tooltip-image img {
    max-width: 100%;
    height: auto;
    display: block;
}

/* Modal за снимки */
.image-modal {
    display: none;
    position: fixed;
    z-index: 9999;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,0.8);
}

.modal-content {
    background-color: #2c3e50;
    margin: 5% auto;
    padding: 0;
    border-radius: 10px;
    width: 80%;
    max-width: 900px;
    box-shadow: 0 4px 20px rgba(0,0,0,0.5);
}
```

#### 2. JavaScript функции (добавени)

**`convertImagePath(path)`**
- Конвертира абсолютни Windows пътища към относителни web пътища
- Пример: `C:\Users\...\Parts\30_01_S1.png` → `images/Parts/30_01_S1.png`

**`createImageTooltip(name, imagePath)`**
- Генерира HTML span елемент с tooltip/modal функционалност
- **Ключова логика:** Ако `imagePath` е празен, автоматично генерира път: `Screenshots/Parts/{name}.png`
- Escape-ва специални символи в имената
- Връща сини подчертани текстове с event handlers

**`tryLoadImage(imagePath, callback, attempt)`**
- Интелигентно зареждане на снимки с 3 fallback опции
- Опитва варианти: `{name}.png`, `{name}_1_.png`, `{name}_2_.png`

**`showTooltip(event, imagePath)`**
- Показва tooltip на позицията на мишката
- Интелигентно позициониране (избягва ръбовете на екрана)

**`hideTooltip()`**
- Премахва tooltip при излизане на мишката

**`showImageModal(name, imagePath)`**
- Отваря modal прозорец с голяма снимка
- Показва съобщение ако снимката липсва

**`closeImageModal()`**
- Затваря modal прозореца

#### 3. HTML структура (добавена преди `</body>`)
```html
<!-- Modal за показване на снимки -->
<div id="imageModal" class="image-modal" onclick="closeImageModal()">
    <div class="modal-content" onclick="event.stopPropagation()">
        <div class="modal-header">
            <h2 id="modalTitle">Снимка</h2>
            <span class="close" onclick="closeImageModal()">×</span>
        </div>
        <div class="modal-body">
            <img id="modalImage" src="" alt="Снимка на детайл">
            <p id="noImageMessage">Няма налично изображение за този елемент</p>
        </div>
    </div>
</div>
```

#### 4. Модификации на съществуващи функции

**`generateDetailsParts()` - Таб "Детайли"**
- **Режим "Всички детайли (общо)":** Променено от `${part.name}` на `${createImageTooltip(part.name, '')}`
- **Режим "По възли":** Променено от `${part.name}` на `${createImageTooltip(part.name, part.imagePath)}`
- Добавени debug съобщения за проследяване

**`filterByObject()` - Таб "По възли"**
- Добавено `imagePath: item.imagePath || ''` при създаване на filteredAssemblies обекти
- Запазва imagePath информацията за всеки елемент

**`generateAssemblyTable(assemblies, title)` - Таб "По възли"**
- Добавена проверка: детайли (не асемблита) получават `createImageTooltip`
- Асемблитата остават с обикновен текст (без tooltip)
- Логика: `const displayName = assembly.isSubAssembly ? assembly.name : createImageTooltip(assembly.name, assembly.imagePath || '');`

### Локация на снимките
- **Папка:** `Screenshots/Parts/`
- **Формат:** PNG файлове
- **Именуване:** Точно име на детайла (напр. `30_01_S1.png`, `30_02_S1.png`)

### Тествани табове
✅ **Детайли** - Режим "Всички детайли (общо)" - снимките се показват  
✅ **Детайли** - Режим "По възли" - снимките се показват  
✅ **По възли** - Филтриране по обект - снимките се показват за детайли (не за асемблита)

### Технически детайли
- Използва се `file://` протокол (локални файлове)
- CORS ограничения заобиколени с FileReader API за JSON зареждане
- Снимките се зареждат директно от относителен път
- Fallback механизъм при липсващи снимки

### Следваща стъпка
Очаква се указание за следващата функционалност, която да се интегрира от `bom_classifier.html`.

---

## 🎨 Подобрение #1.1: Коригиране на позицията на tooltip

**Дата:** 8 декември 2025  
**Статус:** ✅ Завършено

### Описание
Tooltip-ът първоначално се показваше залепен за левия ръб на таблицата. Направени са множество итерации за да се постигне идеалното позициониране - центрирано в бялото поле с леко отстояние от ръба.

### Решение
- **Премахнати CSS класове** - tooltip-ът вече не използва `.tooltip-image` клас
- **Всички стилове inline** - задават се директно през JavaScript за избягване на CSS конфликти и кеширане
- **Изчисляване на позиция в пиксели** - вместо `transform: translateX()` използва директно `left: Xpx`
- **`!important` флаг** - използва `style.cssText` с `!important` за принудително приложение

### Финални настройки на позициониране
```javascript
// При залепване в левия ръб
tooltip.style.cssText = 'left: 15px !important; right: auto !important;';

// При централно позициониране
const centerPos = relativeLeft + (relativeWidth / 2) - (tooltipWidth / 2) + 10; // +10px offset
tooltip.style.cssText = `left: ${centerPos}px !important; right: auto !important;`;

// При залепване в десния ръб
tooltip.style.cssText = 'left: auto !important; right: 10px !important;';
```

### Резултат
✅ Tooltip се показва центрирано в бялото поле с 10-15px отстояние от ръбовете  
✅ Не се отрязва от таблицата  
✅ Изглежда идентично на `bom_classifier.html`

---

## 🔧 Подобрение #1.2: Поддръжка на снимки за асемблита

**Дата:** 8 декември 2025  
**Статус:** ✅ Завършено

### Проблем
Детайли като "GE - Pizzato", които са част от асемблита ("GE - Pizzato Komplekt"), не показваха снимки. Файловете за тези детайли се намираха в `Screenshots/Assemblies/` вместо `Screenshots/Parts/`, и имаха допълнителен суфикс в името (`GE - Pizzato Komplekt_1_.png`).

### Решение

#### 1. Добавен параметър `isAssembly` към `createImageTooltip()`
```javascript
function createImageTooltip(name, imagePath, isAssembly = false) {
    if (!imagePath || imagePath.trim() === '') {
        const folder = isAssembly ? 'Assemblies' : 'Parts';
        imagePath = `Screenshots/${folder}/${name}.png`;
    }
    // ...
}
```

#### 2. Подобрена `tryLoadImage()` функция с интелигентни fallback опции

**Последователност на опитите (5 нива):**
1. **Опит 1:** Оригинален път (Parts или Assemblies според `isAssembly`)
2. **Опит 2:** Алтернативна папка (Parts ↔ Assemblies)
3. **Опит 3:** С добавен ` Komplekt_1_` суфикс
4. **Опит 4:** С добавен `_1_` суфикс
5. **Опит 5:** Обратно в другата папка с `_1_`

**Пример за "GE - Pizzato" (isAssembly=false):**
- `Screenshots/Parts/GE - Pizzato.png` ❌
- `Screenshots/Assemblies/GE - Pizzato.png` ❌
- `Screenshots/Assemblies/GE - Pizzato Komplekt_1_.png` ✅ **Намерена!**

#### 3. Обновени всички извиквания на `createImageTooltip()`
- `generateAssemblyTable()` - подава `assembly.isSubAssembly`
- `generateDetailsParts()` (global) - винаги `false` (детайли)
- `generateDetailsParts()` (byObject) - винаги `false` (детайли)

### Резултат
✅ Снимки се зареждат както от `Parts/`, така и от `Assemblies/`  
✅ Автоматично пробва различни варианти на имена (с/без Komplekt, с/без _1_)  
✅ Работи за всички комбинации детайл/асембли  
✅ Детайлни Console логове за debugging (✅/❌ емоджита)

---

## 🌐 Интеграция #2: Речник за превод на имена

**Дата:** 8 декември 2025  
**Статус:** ✅ Завършена

### Описание
Добавен речник за автоматичен превод на имена на детайли и асемблита от латиница на български език. Имената се преобразуват в реално време при показване в таблиците, tooltip-овете и modal прозорците.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. Речник за превод (`translationDictionary`)

```javascript
const translationDictionary = {
    // Основни компоненти
    'Komplekt': 'Комплект',
    'Bufer': 'Буфер',
    'Planka': 'Планка',
    'Anker': 'Анкер',
    'Pizzato': 'Пизато',
    'Reze': 'Резе',
    'Service': 'Сервиз',
    
    // Винтове и крепежи
    'M4x16': 'М4х16',
    'M4x35': 'М4х35',
    'M8x20': 'М8х20',
    'M10x80': 'М10х80',
    'M12x30': 'М12х30',
    
    // Общи думи
    'ALL': 'ВСИЧКИ',
    'GE': 'ГЕ'
};
```

#### 2. Функция за превод (`translateName()`)

```javascript
function translateName(name) {
    let translatedName = name;
    
    // Преводим всяка дума от речника
    Object.keys(translationDictionary).forEach(englishWord => {
        const bulgarianWord = translationDictionary[englishWord];
        // Използваме regex за замяна на цели думи (не части от думи)
        const regex = new RegExp('\\b' + englishWord.replace(/[.*+?^${}()|[\]\\]/g, '\\$&') + '\\b', 'g');
        translatedName = translatedName.replace(regex, bulgarianWord);
    });
    
    return translatedName;
}
```

**Особености:**
- Използва regex за замяна на **цели думи** (с `\b` word boundary)
- Escape-ва специални regex символи
- Case-sensitive - поддържа както главни, така и малки букви

#### 3. Интеграция в показването на имена

**В `createImageTooltip()`:**
```javascript
const translatedName = translateName(name);
return `<span class="image-tooltip">
            ${translatedName}  // Показва преведеното име
        </span>`;
```

**В `showImageModal()`:**
```javascript
modalTitle.textContent = translateName(name);  // Преведено заглавие в modal
```

### Примери на превод

| Оригинал | Преведено |
|----------|-----------|
| `GE - Pizzato Komplekt` | `ГЕ - Пизато Комплект` |
| `ALL Bufer Service` | `ВСИЧКИ Буфер Сервиз` |
| `Anker M10x80 Komplekt` | `Анкер М10х80 Комплект` |
| `Planka Bufer` | `Планка Буфер` |
| `30_01_S1` | `30_01_S1` (без промяна) |

### Резултат
✅ Всички имена на детайли/асемблита се показват на български  
✅ Преводът работи в таблиците, tooltip-овете и modal прозорците  
✅ Запазени са оригиналните имена за paths и идентификатори  
✅ Лесно разширяем речник - добавяне на нови думи става в един обект

### Следваща стъпка
Очаква се указание за следващата функционалност, която да се интегрира от `bom_classifier.html`.

---

## 📊 Интеграция #3: Разделяне на "Всички части" на Детайли и Сборки

**Дата:** 8 декември 2025  
**Статус:** ✅ Завършена

### Описание
Табът "📊 Всички части" сега показва две отделни секции вместо една обща таблица:
1. **📦 Детайли (единични части)** - всички части без "Komplekt" в името
2. **🔧 Сборки (групи от детайли)** - всички части с "Komplekt" или "комплект" в името

Това прави прегледа по-ясен и улеснява анализа на BOM структурата.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. Модифицирана функция `generatePartsSummary()` (линии ~2105-2215)

**Логика за разделяне:**
```javascript
// Разделяме на детайли и сборки
const singleParts = sortedParts.filter(part => 
    !part.name.includes('Komplekt') && !part.name.includes('комплект')
);
const assemblies = sortedParts.filter(part => 
    part.name.includes('Komplekt') || part.name.includes('комплект')
);
```

#### 2. Нова HTML структура с две таблици

**Общо заглавие:**
```html
<h3>Общо обобщение на всички части 
    (${sortedParts.length} общо: ${singleParts.length} детайла + ${assemblies.length} сборки)
</h3>
```

**Секция за детайли:**
```html
<div style="margin-bottom: 30px;">
    <h4 style="color: #007bff; margin: 20px 0 10px 0;">
        📦 Детайли (единични части) - ${singleParts.length} бр.
    </h4>
    <table class="data-table">
        <!-- Колони: Име на част, Общо количество, Брой обекти, Обекти, Спецификация материал -->
    </table>
</div>
```

**Секция за сборки:**
```html
<div style="margin-bottom: 30px;">
    <h4 style="color: #28a745; margin: 20px 0 10px 0;">
        🔧 Сборки (групи от детайли) - ${assemblies.length} бр.
    </h4>
    <table class="data-table">
        <!-- Колони: Име на част, Общо количество, Брой обекти, Обекти, Спецификация материал -->
    </table>
</div>
```

#### 3. Визуално оформление

**Цветно кодиране:**
- Детайли: **Синьо** (#007bff) 📦
- Сборки: **Зелено** (#28a745) 🔧

**Разстояние между секциите:**
- `margin-bottom: 30px` - ясно разграничени таблици

#### 4. Добавени CSS стилове за `.data-table` клас

```css
.data-table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 10px;
    background: white;
    border-radius: 8px;
    overflow: visible;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.data-table th {
    background: linear-gradient(135deg, #2c3e50, #3498db);
    color: white;
    padding: 15px 20px;
    text-align: left;
    font-weight: bold;
}

.data-table td {
    padding: 12px 20px;
    border-bottom: 1px solid #dee2e6;
}

.data-table tr:nth-child(even) {
    background-color: #f8f9fa;
}

.data-table tr:hover {
    background-color: #e3f2fd;
}
```

### Подобрения на таблиците

#### Увеличено разстояние между колони
- **Хоризонтален padding:** Увеличен от 15px на **20px**
- **`.bom-table td`:** `padding: 12px 20px`
- **`.bom-table th`:** `padding: 15px 20px`
- **`.data-table td`:** `padding: 12px 20px`
- **`.data-table th`:** `padding: 15px 20px`

#### Центрирани стойности в числовите колони
**За таблицата на детайлите:**
```html
<th style="text-align: center;">Общо количество</th>
<th style="text-align: center;">Брой обекти</th>
<!-- В редовете: -->
<td style="text-align: center;">${part.totalQuantity}</td>
<td style="text-align: center;">${part.objects.size}</td>
```

**За таблицата на сборките:**
```html
<th style="text-align: center;">Общо количество</th>
<th style="text-align: center;">Брой обекти</th>
<!-- В редовете: -->
<td style="text-align: center;">${part.totalQuantity}</td>
<td style="text-align: center;">${part.objects.size}</td>
```

### Резултат
✅ Детайлите и сборките са ясно разделени  
✅ Лесно се вижда колко части от всеки тип има  
✅ Цветното кодиране помага за бърза ориентация  
✅ Увеличеното разстояние между колони подобрява четливостта  
✅ Центрираните числа изглеждат по-професионално  
✅ Съвместимост със съществуващите функционалности (снимки, tooltips, превод)

### Следваща стъпка
Очаква се указание за следващата функционалност, която да се интегрира от `bom_classifier.html`.

---

## 🔽 Интеграция #4: Expand/Collapse функционалност за сборки с рекурсивно разгъване

**Дата:** 8 декември 2025  
**Статус:** ✅ Завършена и работеща

### Описание
Добавена е функционалност за разгъване и свиване на сборки с "+" бутони, показваща детайлите и подасемблитата вътре във всяка сборка. Функционалността е рекурсивна - можеш да разгънеш подасемблитата вътре в други сборки.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. Модифицирана HTML структура за сборките в таблицата

**Добавен бутон "+" за всяка сборка:**
```html
<td style="text-align: center;">
    <button onclick="toggleAssemblyDetailsGlobal(index, assemblyName, objects, quantity)" 
            id="expand-btn-${index}"
            style="background: #28a745; color: white; border: none; width: 25px; height: 25px; 
                   border-radius: 50%; cursor: pointer; font-weight: bold;">+</button>
</td>
```

**Добавен скрит ред за детайлите:**
```html
<tr id="details-row-${index}" style="display: none;">
    <td colspan="6" style="padding: 0; border: 1px solid #ddd;">
        <div id="details-content-${index}" style="padding: 15px; background: #f8f9fa;">
            <!-- Детайлите ще се зареждат динамично -->
        </div>
    </td>
</tr>
```

#### 2. Нова функция `toggleAssemblyDetailsGlobal()`

```javascript
function toggleAssemblyDetailsGlobal(assemblyIndex, assemblyName, objectsStr, totalQuantity) {
    const detailsRow = document.getElementById(`details-row-${assemblyIndex}`);
    const detailsContent = document.getElementById(`details-content-${assemblyIndex}`);
    const button = document.getElementById(`expand-btn-${assemblyIndex}`);
    
    if (detailsRow.style.display === 'none') {
        // Показваме детайлите
        detailsRow.style.display = 'table-row';
        button.innerHTML = '−';
        button.style.background = '#dc3545'; // Червен
        
        // Зареждаме детайлите
        loadAssemblyDetailsGlobal(assemblyName, objectsStr.split(','), detailsContent, totalQuantity);
    } else {
        // Скриваме детайлите
        detailsRow.style.display = 'none';
        button.innerHTML = '+';
        button.style.background = '#28a745'; // Зелен
    }
}
```

#### 3. Нова функция `loadAssemblyDetailsGlobal()`

**Логика за извличане на директни деца:**
```javascript
function loadAssemblyDetailsGlobal(assemblyName, objectsList, container, totalQuantity) {
    const allParts = new Map();
    const allSubAssemblies = new Map();
    
    window.bomData.forEach(bom => {
        if (objectsList.includes(bom.objectName)) {
            bom.flatBOM.forEach(item => {
                const pathParts = item.path.split('/');
                
                // Намираме индекса на нашата сборка в path-а
                let assemblyIndex = -1;
                for (let i = 0; i < pathParts.length; i++) {
                    const pathPart = pathParts[i].replace(/<\d+>/g, '').trim();
                    if (pathPart === assemblyName) {
                        assemblyIndex = i;
                        break;
                    }
                }
                
                if (assemblyIndex >= 0) {
                    // Случай 1: Path завършва на сборката (детайли)
                    const lastPart = pathParts[pathParts.length - 1];
                    const lastPartClean = lastPart.replace(/<\d+>/g, '').trim();
                    
                    if (lastPartClean === assemblyName && item.name !== '<подасембли>') {
                        // Добавяме детайла
                    }
                    
                    // Случай 2: Path има още един елемент (подасемблита)
                    if (pathParts.length === assemblyIndex + 2 && item.name === '<подасембли>') {
                        // Добавяме подасемблито
                    }
                }
            });
        }
    });
}
```

#### 4. Рекурсивна функция `toggleSubAssembly()`

**За подасемблитата вътре в разгънатите детайли:**
```javascript
function toggleSubAssembly(containerId, assemblyName, objectsStr, button) {
    const container = document.getElementById(containerId);
    
    if (container.style.display === 'none') {
        container.style.display = 'block';
        button.innerHTML = '−';
        button.style.background = '#dc3545';
        
        // Рекурсивно зареждане - извиква loadAssemblyDetailsGlobal()
        loadAssemblyDetailsGlobal(assemblyName, objectsStr.split(','), container, 0);
    } else {
        container.style.display = 'none';
        button.innerHTML = '+';
        button.style.background = '#28a745';
    }
}
```

#### 5. HTML структура за детайлите (единен списък)

**Детайли и подасемблита в една колона:**
```javascript
allItems.forEach((item, idx) => {
    const subItemId = `sub-${Date.now()}-${idx}`;
    
    if (item.isAssembly) {
        // Подасембли с + бутон
        html += `
            <div>
                <button onclick="toggleSubAssembly('${subItemId}', ...)">+</button>
                <span>🏗️ ${translateName(item.name)}</span>
            </div>
            <div id="${subItemId}" style="display: none;">
                <!-- Вложени детайли -->
            </div>
        `;
    } else {
        // Обикновен детайл
        html += `<div>${createImageTooltip(...)}</div>`;
    }
});
```

#### 6. Коригирана логика за aggregation (само level 1)

**Показване само на главни сборки:**
```javascript
window.bomData.forEach(bom => {
    if (bom.flatBOM && bom.flatBOM.length > 0) {
        bom.flatBOM.forEach(item => {
            // Само level 1 елементи
            if (item.level === 1) {
                let partName;
                if (item.name === '<подасембли>') {
                    // Вземаме името от path-а
                    const pathParts = item.path.split('/');
                    const lastPart = pathParts[pathParts.length - 1];
                    partName = lastPart.replace(/<\d+>/g, '').trim();
                } else {
                    partName = item.name.replace(/<\d+>/g, '').trim();
                }
                // Добавяме в partsSummary
            }
        });
    }
});
```

#### 7. Добавен `flatBOM` в `window.bomData`

**При зареждане на данните:**
```javascript
return {
    objectName: obj.objectName,
    sourceFile: obj.sourceFile,
    assemblies: obj.assemblies || [],
    flatBOM: obj.flatBOM || [],  // ДОБАВЕНО
    allParts: allParts
};
```

### Визуално оформление

**Бутони:**
- Зелен "+" = свито състояние
- Червен "−" = разгънато състояние
- Кръгла форма, 25px × 25px

**Вложени детайли:**
- Сиво поле (`background: #f8f9fa`)
- Padding: 15px
- Border: 1px solid #ddd

**Структура:**
- Детайли: С отместване от 30px, сини и подчертани (tooltips)
- Подасемблита: Italic, 🏗️ икона, със собствени "+" бутони

### Примери на използване

**Пример 1: Planka Bufer Komplekt**
```
[+] Planka Bufer Komplekt (1 бр.)
    ↓ (след клик)
    🔧 Детайли:
        30_04_S1         1 бр.
        Bufer            1 бр.
        [+] 🏗️ M12x30 Komplekt   1 бр.
        [+] 🏗️ M8x20 Komplekt    2 бр.
```

**Пример 2: M8x20 Komplekt (вложен)**
```
[+] 🏗️ M8x20 Komplekt (2 бр.)
    ↓ (след клик)
    🔧 Детайли:
        Bolt M8x20 DIN933    2 бр.
        Washer M8 DIN125     2 бр.
        Nut M8 DIN934        2 бр.
```

### Резултат
✅ Всяка сборка има "+" бутон за разгъване  
✅ Показват се само level 1 сборки в главния списък  
✅ Вложените сборки (M8x20, M12x30) се виждат само при разгъване на родителя  
✅ Рекурсивна функционалност - можеш да разгънеш подасемблитата  
✅ Детайли и подасемблита в един списък (не две колони)  
✅ Визуална индикация със зелени/червени бутони  
✅ Tooltips за детайлите, превод на имената  
✅ Правилна йерархия според BOM структурата

### Следваща стъпка
Очаква се указание за следващата функционалност, която да се интегрира от `bom_classifier.html`.

---

## 📑 Интеграция #5: Обединяване на "По възли" и "Всички елементи" табове

**Дата:** 2024  
**Цел:** Опростяване на интерфейса чрез обединяване на два таба в един с dropdown меню

### Проблем
Имаме два отделни таба:
- `📊 По възли` - с dropdown за филтриране по обект
- `📃 Всички елементи` - показва всички елементи без филтриране

Това създава дублиране на функционалност и излишни бутони в интерфейса.

### Решение
Обединяваме двата таба като добавим "📃 Всички елементи" като опция в dropdown-а на таба "По възли".

### Промени

#### 1. Премахване на отделния таб бутон
```html
<!-- ПРЕДИ: 7 таб бутона -->
<button onclick="showTab('overview')">🏠 Преглед</button>
<button onclick="showTab('byObject')">📊 По възли</button>
<button onclick="showTab('allItems')">📃 Всички елементи</button>  <!-- ПРЕМАХНАТО -->
<button onclick="showTab('allParts')">📦 Всички части</button>
...

<!-- СЛЕД: 6 таб бутона -->
<button onclick="showTab('overview')">🏠 Преглед</button>
<button onclick="showTab('byObject')">📊 По възли</button>
<button onclick="showTab('allParts')">📦 Всички части</button>
...
```

#### 2. Добавяне на опция в dropdown-а
```html
<!-- В таб "По възли" -->
<select id="objectFilter" onchange="filterByObject()">
    <option value="">Всички възли</option>
    <option value="__ALL_ITEMS__">📃 Всички елементи</option>  <!-- НОВО -->
    <!-- Динамични опции за обекти... -->
</select>
```

#### 3. Модифициране на filterByObject() функцията
```javascript
function filterByObject() {
    const selectedObject = document.getElementById('objectFilter').value;
    currentFilter = selectedObject;
    
    // Ако е избрано "Всички елементи" от dropdown-а
    if (selectedObject === "__ALL_ITEMS__") {
        let allAssemblies = [];
        
        bomData.objects.forEach(obj => {
            if (obj.flatBOM) {
                obj.flatBOM.forEach(item => {
                    allAssemblies.push({
                        path: item.path,
                        name: item.name,
                        level: item.level,
                        quantity: item.quantity,
                        objectSource: item.objectSource,
                        hasRAL: item.name.includes('RAL'),
                        ralType: item.name.includes('RAL') ? item.name.match(/RAL\d*/)?.[0] : null,
                        isSubAssembly: item.isAssembly,
                        cleanName: item.name
                    });
                });
            } else {
                obj.assemblies.forEach(assembly => {
                    allAssemblies.push({
                        ...assembly,
                        objectSource: obj.objectName
                    });
                });
            }
        });
        
        const tableHTML = generateAssemblyTable(allAssemblies, 'Всички елементи');
        document.getElementById('byObjectContent').innerHTML = tableHTML;
        return;
    }
    
    // Останалата логика за "Всички възли" и конкретни обекти...
}
```

### Как работи

1. **Всички възли** (празна стойност в dropdown)
   - Показва всички елементи от всички обекти
   - Старо поведение запазено

2. **📃 Всички елементи** (стойност `__ALL_ITEMS__`)
   - Показва ВСИЧКИ елементи включително части
   - Използва `flatBOM` структурата
   - Същото съдържание като старият таб "Всички елементи"

3. **Конкретен обект** (име на обект)
   - Филтрира само елементите от избрания обект
   - Старо поведение запазено

### Ползи
✅ По-чист интерфейс с един по-малко бутон  
✅ Логично групиране - "Всички елементи" е вариант на "По възли"  
✅ Запазена функционалност - нищо не е загубено  
✅ По-лесна навигация с dropdown меню  

### Следваща стъпка
Очаква се указание за следващата функционалност, която да се интегрира от `bom_classifier.html`.

---

## 🌳 Интеграция #6: Реорганизация на "Дърво на детайлите" и автоматично извличане на възли

**Дата:** 2024  
**Цел:** Преименуване на таб, премахване на dropdown меню и интелигентно извличане на главни асемблита като възли

### Проблем
Табът "По възли" имаше:
1. Объркващо име - не беше ясно какво представлява
2. Dropdown с подменюта които бяха излишни след обединението
3. Колоната "Обект" показваше техническото име `Alibre_BOM_Hierarchical` вместо реалните възли (главни асемблита)
4. Надпис с брояч на елементи който заемаше място

### Решение
Преименуване на таба в "Дърво на детайлите", премахване на dropdown-а и автоматично извличане на главните асемблита от йерархията.

### Промени

#### 1. Преименуване на таба и заглавието
```html
<!-- ПРЕДИ -->
<button onclick="showTab('byObject')">🏢 По възли</button>
<h3>🏢 Данни по възли</h3>

<!-- СЛЕД -->
<button onclick="showTab('byObject')">🌳 Дърво на детайлите</button>
<h3>🌳 Информация по възли</h3>
```

#### 2. Премахване на dropdown меню
```html
<!-- ПРЕДИ -->
<div id="byObjectTab" class="tab-content">
    <h3>🏢 Данни по възли</h3>
    <div class="object-filter">
        <label>Избери обект:</label>
        <select id="objectFilter" class="filter-select" onchange="filterByObject()">
            <option value="">🏢 Всички възли</option>
            <option value="__ALL_ITEMS__">📃 Всички елементи</option>
        </select>
    </div>
    <div id="byObjectContent"></div>
</div>

<!-- СЛЕД -->
<div id="byObjectTab" class="tab-content">
    <h3>🌳 Информация по възли</h3>
    <div id="byObjectContent"></div>
</div>
```

#### 3. Опростена функция filterByObject()
```javascript
// ПРЕДИ: Сложна логика с dropdown и филтриране
function filterByObject() {
    const selectedObject = document.getElementById('objectFilter').value;
    currentFilter = selectedObject;
    
    if (selectedObject === "__ALL_ITEMS__") { ... }
    if (selectedObject === "") { ... }
    else { ... }
}

// СЛЕД: Директно показване на всички елементи
function filterByObject() {
    // Винаги показваме всички елементи (няма dropdown)
    let filteredAssemblies = [];
    
    bomData.objects.forEach(obj => {
        if (obj.flatBOM) {
            obj.flatBOM.forEach(item => {
                filteredAssemblies.push({ /* ... */ });
            });
        }
    });
    
    const tableHTML = generateAssemblyTable(filteredAssemblies);
    document.getElementById('byObjectContent').innerHTML = tableHTML;
}
```

#### 4. Премахване на заглавие с брояч
```javascript
// ПРЕДИ: Заглавие с брой елементи
function generateAssemblyTable(assemblies, title) {
    let tableHTML = `
        <h4>${title} (${assemblies.length})</h4>
        <table class="bom-table">

// СЛЕД: Директно таблица без заглавие
function generateAssemblyTable(assemblies) {
    let tableHTML = `
        <table class="bom-table">
```

#### 5. Промяна на колона от "Обект" на "Възел"
```html
<!-- ПРЕДИ -->
<th>Обект</th>

<!-- СЛЕД -->
<th>Възел</th>
```

#### 6. Автоматично извличане на главен асембли (възел)
```javascript
function extractMainAssembly(path) {
    // Извлича главния асембли от път като: "Alibre_BOM_Hierarchical/ALL Bufer Service Komplekt/..."
    // Връща вторият елемент (главният асембли)
    if (!path) return '';
    
    const parts = path.split('/');
    if (parts.length >= 2) {
        return parts[1]; // Вторият елемент е главният асембли
    }
    return parts[0]; // Fallback ако няма "/" в пътя
}

// Използване в таблицата
assemblies.forEach(assembly => {
    const mainAssembly = extractMainAssembly(assembly.path);
    
    tableHTML += `
        <tr>
            <td><strong>${mainAssembly}</strong></td>
            <!-- ... -->
        </tr>
    `;
});
```

### Как работи извличането на възел

**Структура на път:**
```
Alibre_BOM_Hierarchical/ALL Bufer Service Komplekt/30_03_S1
├── [0] Alibre_BOM_Hierarchical  ← Технически обект (игнорираме)
├── [1] ALL Bufer Service Komplekt  ← ГЛАВЕН АСЕМБЛИ (ВЪЗЕЛ)
└── [2] 30_03_S1  ← Детайл
```

**Примери:**
- `Alibre_BOM_Hierarchical/ALL Bufer Service Komplekt/30_03_S1` → **ALL Bufer Service Komplekt**
- `Alibre_BOM_Hierarchical/Anker M10x80 Komplekt/Bolt` → **Anker M10x80 Komplekt**
- `Alibre_BOM_Hierarchical/Planka Bufer Komplekt/M8x20/Washer` → **Planka Bufer Komplekt**

### Ползи
✅ По-ясно име на таба - "Дърво на детайлите" е интуитивно  
✅ По-чист интерфейс без излишни dropdown менюта  
✅ Колоната "Възел" показва реални имена на главни асемблита  
✅ Автоматично разпознаване - работи с произволен брой проекти/възли  
✅ Премахнато дублиране на информация  
✅ По-малко конфигурация - всичко работи автоматично  

### Следваща стъпка
Очаква се указание за следващата функционалност, която да се интегрира от `bom_classifier.html`.

---

## 🔐 Интеграция #7: Администраторски и Потребителски режим (Фаза 1)

**Дата:** 10 декември 2025  
**Статус:** ✅ Завършена и работеща

### Описание
Добавена е функционалност за разделяне на достъпа до редакция на данни чрез админ режим с парола. Това е първата фаза от цялостната имплементация на класификацията "Цех vs На обекта".

### Цел на Фаза 1
Създаване на toggle switch за превключване между потребителски (read-only) и администраторски режим, защитен с парола.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. CSS стилове (добавени ~570 реда)

**Admin Mode Toggle Switch:**
```css
.admin-toggle-container {
    position: absolute;
    left: 20px;
    top: 50%;
    transform: translateY(-50%);
    display: flex;
    align-items: center;
    gap: 10px;
    background: rgba(255,255,255,0.15);
    padding: 10px 15px;
    border-radius: 25px;
    backdrop-filter: blur(10px);
}

.toggle-switch {
    position: relative;
    width: 50px;
    height: 26px;
    display: inline-block;
}

.toggle-slider {
    position: absolute;
    cursor: pointer;
    background-color: rgba(255,255,255,0.3);
    transition: 0.4s;
    border-radius: 26px;
}

input:checked + .toggle-slider {
    background-color: #28a745;
}
```

**Admin Mode Indicator:**
```css
.admin-mode-indicator {
    position: absolute;
    right: 180px;
    top: 20px;
    background: #28a745;
    color: white;
    padding: 8px 16px;
    border-radius: 20px;
    font-size: 14px;
    font-weight: bold;
    display: none;
}

.admin-mode-indicator.active {
    display: flex;
}
```

**Password Modal:**
```css
.password-modal {
    display: none;
    position: fixed;
    z-index: 10000;
    background-color: rgba(0,0,0,0.6);
}

.password-modal.active {
    display: flex;
    justify-content: center;
    align-items: center;
}

.password-modal-content {
    background: white;
    padding: 30px;
    border-radius: 15px;
    box-shadow: 0 10px 40px rgba(0,0,0,0.3);
    max-width: 400px;
    animation: slideIn 0.3s ease;
}
```

#### 2. HTML структура

**Toggle Switch в Header:**
```html
<div class="admin-toggle-container">
    <span class="admin-toggle-label">👤 Админ режим</span>
    <label class="toggle-switch">
        <input type="checkbox" id="adminModeToggle" onchange="toggleAdminMode()">
        <span class="toggle-slider"></span>
    </label>
</div>
```

**Admin Mode Indicator:**
```html
<div class="admin-mode-indicator" id="adminModeIndicator">
    <span>🔓</span>
    <span>АДМИН РЕЖИМ</span>
</div>
```

**Password Modal:**
```html
<div class="password-modal" id="passwordModal">
    <div class="password-modal-content">
        <div class="password-modal-header">
            <h2>🔐 Admin режим</h2>
            <button class="password-modal-close" onclick="closePasswordModal()">&times;</button>
        </div>
        <div class="password-modal-body">
            <p>Моля, въведете паролата за достъп до админ режим:</p>
            <input type="password" id="passwordInput" class="password-input" 
                   placeholder="Въведете парола..." 
                   onkeypress="handlePasswordKeyPress(event)">
            <div class="password-error" id="passwordError">
                ❌ Грешна парола. Моля, опитайте отново.
            </div>
        </div>
        <div class="password-modal-footer">
            <button class="password-modal-button secondary" onclick="closePasswordModal()">Откажи</button>
            <button class="password-modal-button primary" onclick="validatePassword()">Влез</button>
        </div>
    </div>
</div>
```

#### 3. JavaScript функции

**Променливи:**
```javascript
let isAdminMode = false;
const ADMIN_PASSWORD = "2006";
```

**Основни функции:**
- `loadAdminModeState()` - зарежда запазено състояние от localStorage при стартиране
- `toggleAdminMode()` - обработва кликване на toggle switch
- `showPasswordModal()` - показва modal за парола с focus на input полето
- `closePasswordModal()` - затваря modal и връща toggle в изключено състояние ако не е admin
- `validatePassword()` - проверява дали паролата е "2006"
- `handlePasswordKeyPress(event)` - обработва Enter за потвърждаване
- `enterAdminMode()` - активира admin режим и запазва в localStorage
- `exitAdminMode()` - деактивира admin режим и запазва в localStorage
- `updateUIForAdminMode()` - контролира видимостта и активността на admin контроли

**Интеграция при зареждане:**
```javascript
window.addEventListener('load', function() {
    console.log('🚀 Window load event triggered!');
    
    // Зареждаме admin mode състояние
    loadAdminModeState();
    
    console.log('🔍 Starting detectSharedMode...');
    detectSharedMode();
    // ...
});
```

#### 4. RAL Редакция защита

**HTML промени:**
- Всички бутони и полета получиха `class="admin-only-control"`
- Добавено информационно съобщение `class="user-mode-message"`

**Защитени функции:**
- `updateRalEntry()` - проверява за admin режим преди промяна
- `addNewRalEntry()` - блокира добавянето в user режим
- `deleteRalEntry()` - блокира изтриването в user режим
- `saveRalData()` - блокира запазването в user режим

**UI контрол:**
```javascript
function updateUIForAdminMode() {
    const adminControls = document.querySelectorAll('.admin-only-control');
    const userMessages = document.querySelectorAll('.user-mode-message');
    
    adminControls.forEach(control => {
        if (isAdminMode) {
            control.disabled = false;
            control.style.opacity = '1';
            control.style.cursor = 'pointer';
        } else {
            control.disabled = true;
            control.style.opacity = '0.5';
            control.style.cursor = 'not-allowed';
        }
    });
    
    userMessages.forEach(msg => {
        msg.style.display = isAdminMode ? 'none' : 'inline';
    });
}
```

### Как работи

**User режим (по подразбиране):**
- Toggle switch е изключен
- Няма зелен индикатор
- Всички RAL контроли са disabled и полупрозрачни
- Показва се информационно съобщение
- При опит за промяна - alert съобщение и презареждане на таблицата

**Admin режим (след парола "2006"):**
- Toggle switch е включен (зелен)
- Показва се зелен индикатор "🔓 АДМИН РЕЖИМ" горе вдясно
- Всички RAL контроли са активни и редактируеми
- Съобщението се скрива
- Състоянието се запазва в localStorage

**Парола защита:**
- Паролата е hardcoded в JavaScript: `"2006"`
- При грешна парола - червено подсветяване на input полето
- Error съобщение изчезва след 2 секунди
- Enter клавиш работи за потвърждаване

### Тестване

✅ Toggle switch в горния ляв ъгъл на header-а  
✅ Кликване на toggle показва password modal  
✅ Въвеждане на правилна парола "2006" активира admin режим  
✅ Зеленият индикатор се показва горе вдясно  
✅ RAL полетата стават редактируеми  
✅ Презареждане на страницата запазва режима  
✅ Изключване на toggle деактивира admin режим  
✅ RAL полетата отново стават disabled  

### Следваща стъпка
**Фаза 2:** Добавяне на нов таб "🏭 Сглобяване" с две секции (Цех/Обект) и localStorage функционалност

---

## 🏭 Интеграция #8: Нов таб "Сглобяване" с localStorage (Фаза 2)

**Дата:** 10 декември 2025  
**Статус:** ✅ Завършена и работеща

### Описание
Създаден е нов таб "🏭 Сглобяване" с основна структура за класификация на сборките според мястото на сглобяване (Цех vs Обект). Това е втората фаза от цялостната имплементация на класификационната система.

### Цел на Фаза 2
Създаване на основната структура на таба с две секции, празно състояние и интеграция със съществуващата мрежова инфраструктура за съхранение на данни.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. HTML структура (~50 реда)

**Таб бутон в navigation:**
```html
<button class="tab-button" onclick="showTab('assemblyClassification')">🏭 Сглобяване</button>
```
Поставен между "⚙️ Крепежи" и "🎨 RAL елементи"

**Структура на таба:**
```html
<div id="assemblyClassificationTab" class="tab-content">
    <h3>🏭 Класификация на сглобяването</h3>
    
    <!-- Бутони за управление (само admin) -->
    <div class="classification-controls admin-only-control">
        <button onclick="startClassificationWizard()">🏭 Започни класификация</button>
        <button onclick="exportClassification()">📥 Експорт JSON</button>
        <button onclick="importClassification()">📤 Импорт JSON</button>
        <input type="file" id="classificationFileInput" accept=".json">
    </div>
    
    <!-- Съобщение за user режим -->
    <div class="user-mode-message">
        ℹ️ Класификацията на сборките е достъпна само в админ режим
    </div>
    
    <!-- Секция 1: Сглобяване в цеха -->
    <div class="classification-section workshop-section">
        <h4>🏭 Сглобяване в цеха <span id="workshopCount">0</span></h4>
        <div id="workshopContent"></div>
    </div>
    
    <!-- Секция 2: Сглобяване на обекта -->
    <div class="classification-section external-section">
        <h4>🏗️ Сглобяване на обекта <span id="externalCount">0</span></h4>
        <div id="externalContent"></div>
    </div>
</div>
```

#### 2. CSS стилове (~100 реда)

**Основни стилове за секциите:**
```css
.classification-section {
    background: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.classification-content {
    margin-top: 15px;
}
```

**Празно състояние:**
```css
.empty-classification {
    text-align: center;
    padding: 40px;
    color: #6c757d;
    background: #f8f9fa;
    border-radius: 8px;
    border: 2px dashed #dee2e6;
}

.empty-classification-icon {
    font-size: 48px;
    margin-bottom: 15px;
}
```

**Стилове за сборки:**
```css
.assembly-item {
    padding: 12px 15px;
    margin: 8px 0;
    background: #f8f9fa;
    border-radius: 6px;
    border-left: 4px solid #007bff;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: all 0.2s;
}

.assembly-item:hover {
    background: #e9ecef;
    transform: translateX(3px);
}

.workshop-section .assembly-item {
    border-left-color: #28a745;
}

.external-section .assembly-item {
    border-left-color: #dc3545;
}
```

**Бутони за преместване:**
```css
.move-button {
    padding: 5px 12px;
    background: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 13px;
    transition: all 0.2s;
}

.move-button:hover {
    background: #0056b3;
}
```

#### 3. JavaScript функции (~200+ реда)

**Глобална променлива:**
```javascript
let assemblyClassification = {
    workshop: [],    // Пътища на сборки в цеха
    external: [],    // Пътища на сборки на обекта
    timestamp: null
};
```

**Зареждане на класификация:**
```javascript
async function loadAssemblyClassification() {
    console.log('📥 Зареждане на класификация...');
    
    // Използва съществуващата loadSharedUserData()
    const userData = await loadSharedUserData();
    
    // Филтрира classification_* ключове
    const workshop = [];
    const external = [];
    
    for (let key in userData) {
        if (key.startsWith('classification_')) {
            const path = key.replace('classification_', '');
            const category = userData[key];
            
            if (category === 'workshop') {
                workshop.push(path);
            } else if (category === 'external') {
                external.push(path);
            }
        }
    }
    
    if (workshop.length > 0 || external.length > 0) {
        assemblyClassification = {
            workshop: workshop,
            external: external,
            timestamp: new Date().toISOString()
        };
        console.log('✅ Класификация заредена:', assemblyClassification);
    }
    
    return assemblyClassification;
}
```

**Запазване на класификация:**
```javascript
async function saveAssemblyClassification(data) {
    console.log('💾 Запазване на класификация...', data);
    
    // Изчистваме старите classification_* записи
    const userData = await loadSharedUserData();
    for (let key in userData) {
        if (key.startsWith('classification_')) {
            await saveUserDataChange(key, null);
        }
    }
    
    // Запазваме workshop сборките
    for (let path of data.workshop) {
        await saveUserDataChange('classification_' + path, 'workshop');
    }
    
    // Запазваме external сборките
    for (let path of data.external) {
        await saveUserDataChange('classification_' + path, 'external');
    }
    
    assemblyClassification = data;
    console.log('✅ Класификация запазена');
}
```

**Генериране на таба:**
```javascript
async function generateAssemblyClassificationTab() {
    console.log('🏭 Генериране на таб за класификация...');
    
    await loadAssemblyClassification();
    
    const workshopContent = document.getElementById('workshopContent');
    const externalContent = document.getElementById('externalContent');
    const workshopCount = document.getElementById('workshopCount');
    const externalCount = document.getElementById('externalCount');
    
    // Празно състояние
    if (assemblyClassification.workshop.length === 0 && 
        assemblyClassification.external.length === 0) {
        const emptyState = `
            <div class="empty-classification">
                <div class="empty-classification-icon">🏭</div>
                <h4>Няма класифицирани сборки</h4>
                <p>Класификацията помага да се организират сборките според мястото на сглобяване.</p>
                <p style="margin-top: 15px; font-weight: 500;">
                    ${isAdminMode ? 
                        'Натиснете бутона "🏭 Започни класификация" за да започнете.' : 
                        'Влезте в админ режим за да започнете класификация.'}
                </p>
            </div>
        `;
        workshopContent.innerHTML = emptyState;
        externalContent.innerHTML = '';
        workshopCount.textContent = '0';
        externalCount.textContent = '0';
        return;
    }
    
    // Генериране на съдържание за workshop/external сборките
    // ... (вижте пълния код за HTML генериране)
    
    workshopCount.textContent = assemblyClassification.workshop.length;
    externalCount.textContent = assemblyClassification.external.length;
}
```

**Помощна функция:**
```javascript
function findAssemblyByPath(path) {
    if (!bomData || !bomData.objects) return null;
    
    for (let obj of bomData.objects) {
        if (obj.flatBOM) {
            for (let item of obj.flatBOM) {
                if (item.path === path) {
                    return {
                        name: item.name,
                        path: item.path,
                        quantity: item.quantity,
                        level: item.level
                    };
                }
            }
        }
    }
    return null;
}
```

**Placeholder функции за следващи фази:**
```javascript
function startClassificationWizard() {
    alert('🏭 Wizard за класификация ще бъде имплементиран във Фаза 3');
}

function exportClassification() {
    alert('📥 Експорт на класификация ще бъде имплементиран във Фаза 6');
}

function importClassification() {
    document.getElementById('classificationFileInput').click();
}

function handleClassificationImport(event) {
    alert('📤 Импорт на класификация ще бъде имплементиран във Фаза 6');
}

function moveAssemblyToExternal(path) {
    alert('→ Преместване ще бъде имплементирано във Фаза 4');
}

function moveAssemblyToWorkshop(path) {
    alert('→ Преместване ще бъде имплементирано във Фаза 4');
}
```

#### 4. Интеграция с admin режим

**Обновена `updateUIForAdminMode()`:**
```javascript
function updateUIForAdminMode() {
    // ... съществуващ код за RAL контроли ...
    
    // Контролираме класификация бутоните
    const classificationControls = document.querySelectorAll('.classification-controls');
    classificationControls.forEach(control => {
        control.style.display = isAdminMode ? 'block' : 'none';
    });
    
    // Презареждаме класификацията ако е отворен табът
    if (document.getElementById('assemblyClassificationTab').classList.contains('active')) {
        generateAssemblyClassificationTab();
    }
}
```

**Обновена `showTab()` функция:**
```javascript
function showTab(tabName) {
    // ... съществуващ код ...
    
    if (tabName === 'assemblyClassification') {
        generateAssemblyClassificationTab();
    }
}
```

#### 5. Интеграция с мрежовата инфраструктура

**Използване на съществуващи функции:**
- ✅ `loadSharedUserData()` - за зареждане на класификации
- ✅ `saveUserDataChange(key, value)` - за запазване на всяка класификация
- ✅ Автоматично работи в мрежов/локален режим
- ✅ Real-time споделяне между потребители

**Формат на данните:**
```javascript
// В localStorage/network се запазва:
{
    "classification_Alibre_BOM_Hierarchical/Assembly1": "workshop",
    "classification_Alibre_BOM_Hierarchical/Assembly2": "external",
    // ...
}
```

### Как работи

**User режим:**
- Вижда таба "🏭 Сглобяване"
- Вижда празното състояние с инструкция: "Влезте в админ режим за да започнете класификация."
- Няма бутони за управление (скрити)
- Ако има данни - вижда списъците без бутони за преместване

**Admin режим:**
- Вижда бутоните за управление:
  - "🏭 Започни класификация" (placeholder за Фаза 3)
  - "📥 Експорт JSON" (placeholder за Фаза 6)
  - "📤 Импорт JSON" (placeholder за Фаза 6)
- Вижда бутони "[→ На обекта]" и "[→ В цеха]" при всяка сборка (placeholder за Фаза 4)
- Празното състояние показва: "Натиснете бутона '🏭 Започни класификация' за да започнете."

**Празно състояние:**
- Икона 🏭
- Заглавие "Няма класифицирани сборки"
- Описание на функционалността
- Различни инструкции според режима (admin/user)

**Съхранение:**
- Използва `saveUserDataChange('classification_' + path, category)`
- Работи автоматично в мрежов/локален режим
- Real-time споделяне между потребители
- Всяка сборка се запазва поотделно за гъвкавост

### Тестване

✅ Нов таб "🏭 Сглобяване" между "Крепежи" и "RAL"  
✅ Кликване на таба показва празното състояние  
✅ Празното състояние има красив дизайн с икона  
✅ В user режим - инструкция за влизане в admin режим  
✅ В admin режим - бутони за управление се показват  
✅ В admin режим - инструкция за стартиране на wizard  
✅ Placeholder alert-и работят за бъдещи функции  
✅ Презареждане при смяна на admin режим  
✅ Бройките се обновяват (0/0 в празно състояние)  

### Следваща стъпка
**Фаза 3:** Имплементация на modal wizard за класификация на level 1 сборки с автоматично наследяване за level 2+

---

## 🎯 Интеграция #9: Modal wizard за класификация на сборки (Фаза 3)

**Дата:** 12 декември 2025  
**Статус:** ✅ Завършена и работеща

### Описание
Добавена е пълна функционалност за класификация на сборки чрез интерактивен wizard:
- Modal прозорец с прогрес бар и навигация
- Класификация на level 1 сборки с два бутона: "🏭 В цеха" / "🏗️ На обекта"
- Автоматично наследяване на класификацията за всички деца (level 2+)
- Запазване в localStorage/мрежа с real-time споделяне
- Извличане на реални имена от пътища (fix за `<подасембли>`)
- Премахване на CORS грешки при file:// протокол

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. HTML структура за wizard modal (преди `</body>`)
```html
<!-- Classification Wizard Modal -->
<div class="wizard-modal" id="classificationWizard">
    <div class="wizard-modal-content">
        <!-- Header -->
        <div class="wizard-header">
            <h2>🏭 Класификация на сборки</h2>
            <button class="wizard-close-btn" onclick="closeClassificationWizard()">&times;</button>
        </div>
        
        <!-- Progress Bar -->
        <div class="wizard-progress-container">
            <div class="wizard-progress-text">
                Сборка <span id="wizardCurrentIndex">1</span> от <span id="wizardTotalCount">0</span>
            </div>
            <div class="wizard-progress-bar">
                <div class="wizard-progress-fill" id="wizardProgressFill"></div>
            </div>
        </div>
        
        <!-- Body - динамично съдържание -->
        <div class="wizard-body" id="wizardBody"></div>
        
        <!-- Footer - Navigation -->
        <div class="wizard-footer">
            <button class="wizard-nav-btn" id="wizardPrevBtn" onclick="wizardPrevious()">← Назад</button>
            <button class="wizard-nav-btn wizard-skip-btn" id="wizardSkipBtn" onclick="wizardSkip()">Прескочи</button>
            <button class="wizard-nav-btn" id="wizardNextBtn" onclick="wizardNext()">Напред →</button>
        </div>
    </div>
</div>
```

#### 2. CSS стилове за wizard (~280 реда)
```css
/* Wizard Modal Styles */
.wizard-modal {
    display: none;
    position: fixed;
    top: 0; left: 0;
    width: 100%; height: 100%;
    background: rgba(0, 0, 0, 0.7);
    z-index: 10000;
    justify-content: center;
    align-items: center;
}

.wizard-modal.active { display: flex; }

.wizard-modal-content {
    background: white;
    border-radius: 15px;
    width: 90%; max-width: 700px;
    max-height: 90vh;
    overflow: hidden;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
    animation: wizardSlideIn 0.3s ease-out;
}

@keyframes wizardSlideIn {
    from { transform: translateY(-50px); opacity: 0; }
    to { transform: translateY(0); opacity: 1; }
}

/* Progress bar, buttons, styling... */
```

#### 3. JavaScript функции

**`getLevel1Assemblies()`** - Извличане на главни сборки
```javascript
function getLevel1Assemblies() {
    const level1Assemblies = [];
    
    // Обхожда всички обекти и филтрира level 1 елементи
    bomData.objects.forEach(obj => {
        if (obj.flatBOM && obj.flatBOM.length > 0) {
            obj.flatBOM.forEach(item => {
                if (item.level === 1) {
                    // Извлича име от път ако е <подасембли>
                    let displayName = item.name;
                    if (!displayName || displayName === '<подасембли>') {
                        displayName = extractNameFromPath(item.path);
                    }
                    
                    level1Assemblies.push({
                        path: item.path,
                        cleanName: displayName,
                        objectName: obj.name,
                        quantity: item.quantity || 1
                    });
                }
            });
        }
    });
    
    return level1Assemblies;
}
```

**`startClassificationWizard()`** - Стартиране на wizard
- Получава всички level 1 сборки
- Инициализира индекси и класификации
- Показва modal-а
- Зарежда първата сборка

**`showWizardAssembly(index)`** - Показване на сборка
- Обновява прогрес бара
- Генерира HTML със снимка, име, детайли
- Показва два големи бутона за избор
- Контролира навигационните бутони

**`classifyAssembly(category)`** - Класифициране на сборка
- Запазва избора (workshop/external)
- Извиква `autoClassifyChildren()` за автоматично класифициране на деца
- Активира бутон "Напред"
- Автоматично преминава към следващата сборка
- При последна сборка → завършва wizard-а

**`autoClassifyChildren(parentPath, category)`** - Автоматична класификация
- Обхожда всички елементи в flatBOM
- Намира деца със `startsWith(parentPath + '/')` и `level > 1`
- Класифицира всички деца в същата категория
- Логва броя класифицирани деца

**`completeWizard()`** - Финален екран
- Показва резултати (X в цеха, Y на обекта)
- Бутон "Запази и затвори"

**`saveAndCloseWizard()`** - Запазване на класификациите
```javascript
async function saveAndCloseWizard() {
    // Групира по категория
    const classificationData = { workshop: [], external: [] };
    Object.keys(wizardClassifications).forEach(path => {
        const category = wizardClassifications[path];
        if (category === 'workshop') classificationData.workshop.push(path);
        else if (category === 'external') classificationData.external.push(path);
    });
    
    // Запазва всяка класификация поотделно
    for (const path of classificationData.workshop) {
        await saveUserDataChange('classification_' + path, 'workshop');
    }
    for (const path of classificationData.external) {
        await saveUserDataChange('classification_' + path, 'external');
    }
    
    // Затваря wizard-а и презарежда таба
    closeClassificationWizard();
    await loadAssemblyClassification();
    generateAssemblyClassificationTab();
}
```

**Навигационни функции:**
- `wizardPrevious()` - Връща към предишната сборка
- `wizardNext()` - Преминава към следващата
- `wizardSkip()` - Прескача сборка (с confirmation)
- `closeClassificationWizard()` - Затваря modal-а

#### 4. Подобрения на съществуващи функции

**`findAssemblyByPath(path)`** - Поправка за `<подасембли>`
```javascript
function findAssemblyByPath(path) {
    // Помощна функция за извличане на име от път
    function extractNameFromPath(path) {
        const lastPart = path.split('/').pop();
        return lastPart.replace(/<\d+>$/, ''); // Премахва <1>, <2> и т.н.
    }
    
    // Търси в flatBOM
    for (let item of obj.flatBOM) {
        if (item.path === path) {
            // Ако името е <подасембли>, извлича го от пътя
            let displayName = item.name;
            if (!displayName || displayName === '<подасембли>') {
                displayName = extractNameFromPath(item.path);
            }
            return { name: displayName, path, quantity, level };
        }
    }
}
```

**`detectSharedMode()`** - Премахване на CORS грешки
```javascript
async function detectSharedMode() {
    // Проверка за file:// протокол ПРЕДИ fetch
    if (window.location.protocol === 'file:') {
        isSharedMode = false;
        updateNetworkStatus(false, "Локален режим");
        console.log('📱 Локален режим - файлът е отворен директно (file://)');
        console.log('💡 За мрежов режим отворете файла през HTTP сървър');
        return; // Без fetch заявка!
    }
    
    // Продължава с HTTP/HTTPS проверка...
}
```

### Как работи

**1. Стартиране на wizard:**
- Admin влиза в таб "🏭 Сглобяване"
- Кликва "🏭 Започни класификация"
- Wizard modal се отваря с първата level 1 сборка

**2. Класификация на сборка:**
- Виж име, обект, количество
- Кликни "🏭 В цеха" ИЛИ "🏗️ На обекта"
- Автоматично се класифицират ВСИЧКИ деца (level 2, 3, 4...)
- Прогрес барът се обновява
- Автоматично преминава към следващата

**3. Навигация:**
- "← Назад" - връща към предишна сборка
- "Прескочи" - оставя некласифицирана
- "Напред →" - към следващата (активен след класификация)

**4. Завършване:**
- Показва резултати: X в цеха, Y на обекта
- "Запази и затвори" → запазва в localStorage/мрежата
- Таб "🏭 Сглобяване" се обновява с резултатите

**5. Persistence:**
- Всяка класификация се запазва като `classification_{path}` = category
- При refresh - данните се зареждат автоматично
- В мрежов режим - споделят се real-time между потребители

### Тестване

✅ Wizard modal се отваря правилно  
✅ Прогрес бар показва "Сборка X от Y"  
✅ **Реални имена** вместо `<подасембли>` (извлечени от път)  
✅ Бутони "В цеха" / "На обекта" работят  
✅ Автоматична класификация на деца (22+ елемента)  
✅ Навигация (Назад/Напред/Прескочи) работи  
✅ Финален екран показва резултати  
✅ Запазване в localStorage/мрежа работи  
✅ След refresh - данните се запазват  
✅ Таб "🏭 Сглобяване" показва класифицираните сборки  
✅ **Без CORS грешки** в конзолата при file:// протокол  

### Ключови подобрения

1. **Извличане на име от път:**
   - Проблем: BOM данните съдържат `<подасембли>` вместо реално име
   - Решение: Извличаме последната част от пътя и премахваме `<1>`, `<2>` и т.н.
   - Пример: `ALL Bufer/Planka Bufer<1>` → `Planka Bufer`

2. **Премахване на CORS грешки:**
   - Проблем: При отваряне на file:// fetch към `/get_user_data` хвърля грешка
   - Решение: Проверка на протокола ПРЕДИ fetch заявка
   - Резултат: Чиста конзола без червени грешки

3. **Автоматична класификация на деца:**
   - Ефективност: Един клик класифицира главна сборка + всички подсборки
   - Логика: `path.startsWith(parentPath + '/')` и `level > 1`
   - Лог: Показва колко деца са класифицирани

### Следваща стъпка
**Фаза 4:** Йерархично дърво с [+] expand/collapse бутони за визуализация на подсборки

---

## 🎯 Интеграция #10: Йерархично дърво за класификация - Стъпка 1 (Фаза 4)

**Дата:** 23 декември 2025  
**Статус:** ⏳ В процес - Стъпка 1/5 завършена

### Описание
Започната имплементация на йерархично дърво с expand/collapse функционалност за визуализация на детайли и подсборки в класифицираните сборки.

**Цел на Фаза 4:** Вместо прост flat списък със сборки, показваме йерархична структура с възможност за разгъване/сгъване на подсборки и преместване между категории.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### Стъпка 1: Анализ на данните и функция за извличане на деца ✅

**`getAssemblyChildren(parentPath)`** - Нова функция за анализ на структурата
```javascript
function getAssemblyChildren(parentPath) {
    const children = {
        details: [],      // Детайли (няма деца)
        subassemblies: [] // Подсборки (има деца)
    };
    
    // 1. Намира родителя и неговото ниво в flatBOM
    // 2. Извлича всички директни деца (level = parentLevel + 1)
    // 3. За всяко дете проверява дали има собствени деца
    // 4. Разделя на details (няма деца) и subassemblies (има деца)
    // 5. Извлича реални имена от пътища (fix за <подасембли>)
    
    return children;
}
```

**Логика за разделяне:**
- **Детайл** = елемент БЕЗ деца в flatBOM (обикновени части: болтове, плочи, и т.н.)
- **Подсборка** = елемент С деца в flatBOM (komplekt-и които имат собствени части)

**Филтриране на деца:**
```javascript
// Намира директни деца
item.path.startsWith(parentPath + '/') && 
item.level === parentLevel + 1

// Проверява дали елемент има деца (подсборка)
const hasChildren = bomData.objects.some(obj => 
    obj.flatBOM.some(item => 
        item.path.startsWith(child.path + '/') && 
        item.level === child.level + 1
    )
);
```

**Извличане на имена:**
```javascript
function extractNameFromPath(path) {
    const lastPart = path.split('/').pop();
    return lastPart.replace(/<\d+>$/, ''); // Премахва <1>, <2> и т.н.
}
```

**Връщан резултат:**
```javascript
{
    details: [
        { path: '...', name: 'Planka буфер', quantity: 1, level: 2 },
        { path: '...', name: 'Anker M10x80', quantity: 1, level: 2 },
        ...
    ],
    subassemblies: [
        { path: '...', name: 'Planka буфер Komplekt', quantity: 1, level: 2 },
        { path: '...', name: 'GE - Pizzato Komplekt', quantity: 1, level: 2 },
        ...
    ]
}
```

### Тестване на Стъпка 1

**Команда в конзолата:**
```javascript
const testPath = "ALL Bufer Service Komplekt";
const children = getAssemblyChildren(testPath);
console.log("Детайли:", children.details);
console.log("Подсборки:", children.subassemblies);
```

**Очакван резултат:**
```
📊 Деца на ALL Bufer Service Komplekt: {детайли: 6, подсборки: 4}

Детайли: Array(6)
  0: {path: "ALL Bufer Service Komplekt/Planka буфер<1>", name: "Planka буфер", quantity: 1, level: 2}
  1: {path: "ALL Bufer Service Komplekt/Anker M10x80<1>", name: "Anker M10x80", quantity: 1, level: 2}
  ...

Подсборки: Array(4)
  0: {path: "ALL Bufer Service Komplekt/Planka буфер Komplekt<1>", name: "Planka буфер Komplekt", quantity: 1, level: 2}
  1: {path: "ALL Bufer Service Komplekt/GE - Pizzato Komplekt<1>", name: "GE - Pizzato Komplekt", quantity: 1, level: 2}
  ...
```

### Тестове проведени

✅ Функцията намира правилно директните деца на родителска сборка  
✅ Правилно разделя на детайли (6 елемента) и подсборки (4 елемента)  
✅ Извлича реални имена от пътища (няма `<подасембли>`)  
✅ Връща структуриран обект готов за HTML генерация  
✅ Логва информация в конзолата за debugging  

### Следваща стъпка
**Стъпка 2/5:** HTML генериране на йерархично дърво с [+] бутони за expand/collapse

---

## 🎯 Интеграция #11: Йерархично дърво за класификация - Стъпка 2 (Фаза 4)

**Дата:** 24 декември 2025  
**Статус:** ✅ Завършена

### Описание
Имплементирано е реалното HTML генериране на йерархичното дърво в таб „🏭 Сглобяване“, включително базова expand/collapse логика.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. Рендер на дърво за сборки
- Добавен renderer, който визуализира сборка → директни деца (детайли и подсборки).
- Показва „+“ бутон само за елементи, които са реални сборки (подсборки).

#### 2. Expand/Collapse handler
- Добавена функция за toggle на node (показване/скриване на контейнера с деца).

### Тестване
✅ Root сборки се визуализират като дърво  
✅ Разгъване/сгъване работи на основно ниво  
✅ „+“ бутон не се показва за детайли

### Следваща стъпка
**Стъпка 3/5:** Рекурсивно рендериране за подсборки (дърво в дълбочина)

---

## 🎯 Интеграция #12: Йерархично дърво за класификация - Стъпка 3 (Фаза 4)

**Дата:** 24 декември 2025  
**Статус:** ✅ Завършена

### Описание
Надградено е дървото да работи рекурсивно (подсборки вътре в подсборки), така че expand/collapse да функционира на произволна дълбочина.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. Рекурсивен renderer
- Рендер функцията извиква сама себе си за всяка подсборка.
- Всеки node има отделен контейнер за деца.

#### 2. Expand/collapse за под-нива
- Toggle логиката е приложена за всеки node в дървото, а не само за root.

### Тестване
✅ Подсборките се разгъват/сгъват коректно  
✅ Няма визуални „загуби“ на елементи при навигация в дървото

### Следваща стъпка
**Стъпка 4/5:** Сортиране на елементите за по-добра четимост

---

## 🎯 Интеграция #13: Йерархично дърво за класификация - Стъпка 4 (Фаза 4)

**Дата:** 26 декември 2025  
**Статус:** ✅ Завършена

### Описание
Добавено е „умно“ сортиране на деца (детайли/подсборки), за да се показват в предвидим и човешки ред.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. Smart sort
- Числово водени имена (напр. започващи с число) се сортират числово.
- Останалите се сортират по locale (`bg`) за по-добро поведение на кирилица.

### Тестване
✅ Елементите вече излизат в стабилен, четим ред

### Следваща стъпка
**Стъпка 5/5:** Edge-case fixes в извличането на деца + стабилен DOM id

---

## 🎯 Интеграция #14: Йерархично дърво за класификация - Стъпка 5 (Фаза 4)

**Дата:** 28 декември 2025  
**Статус:** ✅ Завършена

### Описание
Финализирано е дървото с ключови корекции за реалните quirks в BOM данните и стабилност на expand/collapse.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. Fix: липсващи деца при „същия path, по-високо level“
- Обработено е представянето на деца, които могат да се появяват със същия `path` като родителя, но с по-висок `level`.

#### 2. Fix: липсващи root детайли (същия path/level, различно име)
- Разширена е root логиката, така че „сиблинг“ редове със същия `path` и `level` да не изчезват.

#### 3. Fix: duplicate DOM ids
- Въведени са уникални id-та per-node (counter/генератор), за да няма колизии и „неработещ“ expand.

### Тестване
✅ Всички очаквани детайли/подсборки се виждат  
✅ Expand/collapse работи стабилно  
✅ Няма колизии в DOM id-та

---

## 🎨 Подобрение #14.1: Секции в дървото (детайли/подсборки/крепежи)

**Дата:** 29 декември 2025  
**Статус:** ✅ Завършено

### Описание
Добавено е групиране в дървото на:
- „Детайли“
- „Подсборки“
- „Крепежи необходими за сглобяването“

Целта е по-ясна визуална структура на root ниво и вътре в всяка сборка.

### Файлове променени
- `unified_bom_viewer.html`

### Резултат
✅ По-четима структура във всеки възел на дървото

---

## 🔩 Подобрение #14.2: По-надеждна детекция на крепежи + Cyrillic/Latin lookalikes

**Дата:** 29 декември 2025  
**Статус:** ✅ Завършено

### Описание
Детекцията на „крепежи“ беше направена по-устойчива и независима от думата „Komplekt“, защото някои реални подсборки съдържат „Komplekt“.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени
- Разпознаване на крепежи по:
    - начало с `M`/`М` (латиница/кирилица)
    - „Anker/Анкер“
    - ключови думи (bolt/nut/washer/spring washer и т.н.)
- Нормализация на „lookalike“ букви (кирилица/латиница), за да се избегнат пропуски.

### Резултат
✅ Крепежите се групират по-надеждно във всяка сборка

---

## 🧹 Подобрение #14.3: Намаляване на debug шума (toggle за логове)

**Дата:** 30 декември 2025  
**Статус:** ✅ Завършено

### Описание
Намалени са подробните `console.log` съобщения чрез debug flag/toggle, за да не се пълни конзолата при нормална работа.

### Файлове променени
- `unified_bom_viewer.html`

### Резултат
✅ По-чиста конзола при нормално ползване  
✅ Възможност за включване на логове при debugging

---

## 🖼️ Интеграция #15: Снимки (tooltip/modal) и в assembly tree + подобрен fallback за Assemblies

**Дата:** 2 януари 2026  
**Статус:** ✅ Завършена

### Описание
Добавена е функционалност имената в assembly tree да са интерактивни (както в таб „Детайли“) и да показват снимки при hover/click.

### Файлове променени
- `unified_bom_viewer.html`

### Детайлни промени

#### 1. Интерактивни имена в дървото
- Преизползвана е `createImageTooltip(...)` логиката при рендер на елементите в дървото.

#### 2. Подобрена логика за fallback при търсене на снимки
- Коригирана е стратегията за намиране на файлове като `*_1_.png` в `Screenshots/Assemblies/`.
- Предпочита да изчерпи „варианти на името в същата папка“, преди да сменя папката.

### Тестване
✅ Assemblies снимки от `Screenshots/Assemblies/` се показват коректно (вкл. `*_1_.png`)  
✅ Детайлите продължават да работят с `Screenshots/Parts/`

---

## ✅ Обобщение: Фаза 4 приключена

Към 6 януари 2026 Фаза 4 е реализирана изцяло:
- Йерархично дърво с рекурсивен expand/collapse
- Устойчива логика за извличане на деца (quirks в BOM)
- Умно сортиране
- Секции „Детайли / Подсборки / Крепежи“
- По-надеждна детекция на крепежи
- Debug toggle за логове
- Снимки (tooltip/modal) и в дървото, с коректен fallback за assemblies


---

## 🎯 Интеграция #16: Визуална връзка при независима класификация + Преместване (Фаза 5 и 5.5)

**Дата:** 7 януари 2026  
**Статус:** ✅ Завършена

### Описание
Имплементирана е пълна функционалност за откриване и визуализация на "сирачета" (подсборки класифицирани в различна категория от родителя им), както и възможност за ръчно преместване на сборки между категории.

### Файлове променени
- ``unified_bom_viewer.html``

### Детайлни промени

#### 1. Orphan Detection функции (Фаза 5 Стъпка 2)
Добавени са две ключови функции за откриване на връзки:

**``findParentAssembly(assemblyPath)``** - намира родителската сборка
- Извлича parent path
- Проверява дали родителят е root level (skip ако няма '/')
- Проверява дали родителят е класифициран
- Връща null ако родителят не е визуализиран в UI

**``findOrphanAssemblies(assemblyPath)``** - намира "сирачетата"
- Намира категорията на родителя
- Извлича децата чрез ``getAssemblyChildren()``
- Сравнява категории и връща несъответствията

**Ключова логика:**
- Root level родители (без '/' в пътя) се игнорират
- Orphan warning се показва само ако родителят Е визуализиран в UI
- Множество итерации за перфектна точност на detection-а

#### 2. UI елементи за orphan визуализация (Фаза 5 Стъпка 3)

**HTML генериране:**
- ⚠️ Икона до името на родителската сборка
- Hover tooltip с текст: "Част от [Parent] (в цеха/на обекта)"
- Read-only жълта секция със заглавие "⚠️ Свързани подсборки в друга категория:"
- Списък с всички orphan деца
- Бутон ``[👁️ Виж]`` за всяко дете

**CSS стилове:**
- ``.assembly-orphan-icon`` - ⚠️ икона със ``cursor: help``
- ``.assembly-trace-tooltip`` - Hover tooltip стил
- ``.assembly-trace`` - Жълта секция с border
- ``.assembly-trace-list`` - Списък със orphan деца
- ``.assembly-view-btn`` - Син [👁️ Виж] бутон
- ``.assembly-highlight`` - Жълта highlight анимация 3 сек

#### 3. Scroll и highlight функционалност (Фаза 5 Стъпка 4)

**``scrollToAssembly(assemblyPath, targetCategory)``** - scroll към сборка
- Намира елемента В ПРАВИЛНАТА СЕКЦИЯ (за дублирани имена)
- Разгъва всички родители
- Smooth scroll до центъра на екрана
- Highlight анимация за 3 секунди

**``expandAncestors(assemblyPath, containerSelector)``** - разгъване на родители
- Извлича всички ancestor пътища
- Намира всеки ancestor В ПРАВИЛНАТА СЕКЦИЯ
- Разгъва children div и променя expand button на '−'

**``highlightAssembly(element)``** - highlight анимация
- Добавя ``.assembly-highlight`` class
- CSS keyframe анимация (yellow → transparent)
- Премахва class след 3000ms

**Критична подробност:** Category-aware scrolling предотвратява грешки при дублирани имена (напр. "M12x30 Комплект" може да се среща в workshop вложен в parent И external като самостоятелен).

#### 4. Ръчно преместване на сборки (Фаза 5.5)

Имплементирани са реални функции вместо placeholder-и:

**``moveAssemblyToExternal(path)``** - премества от workshop към external
- Проверява дали съществува в workshop
- Премахва от workshop масива
- Добавя в external масива
- Запазва чрез ``saveAssemblyClassification()``
- Refresh на UI чрез ``generateAssemblyClassificationTab()``

**``moveAssemblyToWorkshop(path)``** - премества от external към workshop
- Същата логика в обратна посока

**UI бутони:**
- В workshop секция: ``[→ На обекта]`` бутон (видим само в админ режим)
- В external секция: ``[← В цеха]`` бутон (видим само в админ режим)

#### 5. Валидация при смяна на JSON файл

Критична функционалност за предотвратяване на "призрачни" данни:

**``validateAndCleanClassification()``** - валидира класификацията
- Събира всички валидни assembly paths от текущия BOM
- Филтрира workshop/external масиви
- Премахва невалидни пътища (от стари JSON файлове)
- Запазва изчистената класификация

**Интеграция:** Функцията се извиква автоматично в ``displayBOMData()`` след ``loadAllUserData()``.

**Проблем, който решава:** При смяна на JSON файл, старите класификации (напр. крепежи от друг проект) остават в localStorage и се показват грешно в новия проект.

### Тестване
✅ ⚠️ Икона се показва САМО когато родител е визуализиран в UI и е в различна категория  
✅ Tooltip показва коректно име и категория на родителя  
✅ Read-only списък показва всички orphan деца с правилни категории  
✅ Бутон [👁️ Виж] scroll-ва точно до дете-то в правилната секция (дори при дублирани имена)  
✅ Highlight анимация работи 3 секунди с yellow fade  
✅ Бутони [→ На обекта] и [← В цеха] преместват коректно между категории  
✅ UI се refresh-ва автоматично след преместване  
✅ При смяна на JSON файл, невалидни класификации се изчистват автоматично  

### Резултат
✅ Пълна визуализация на cross-category връзки  
✅ Интуитивна навигация с един клик  
✅ Възможност за ръчна корекция на класификация  
✅ Надеждно управление на данни при смяна на проекти  

---

## ✅ Обобщение: Фаза 5 и 5.5 приключени

Към 7 януари 2026 Фаза 5 е реализирана изцяло с допълнителни подобрения:
- Orphan detection с интелигентна логика (root level check, visualization check)
- ⚠️ Икона + Tooltip + Read-only списък
- [👁️ Виж] функция с category-aware scrolling и highlight анимация
- Ръчно преместване между категории (Фаза 5.5)
- Автоматична валидация при смяна на JSON файл