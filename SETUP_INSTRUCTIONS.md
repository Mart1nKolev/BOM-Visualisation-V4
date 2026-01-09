# 🚀 Инструкции за пускане на BOM Visualizer в мрежа

## 📋 Какво ще постигнем:
- Всички в офиса виждат **едни и същи данни**
- Промените се запазват **автоматично**
- Работи на **всеки компютър** в локалната мрежа

---

## 🎯 Вариант 1: Пускане на Windows компютър (най-лесно)

### Стъпка 1: Подготовка
1. Копирай тези файлове в една папка:
   - `unified_bom_viewer.html`
   - `bom_data.json`
   - `server.py`
   - Папка `Screenshots/` (със снимките)

2. Инсталирай Python (ако нямаш):
   - Свали от: https://www.python.org/downloads/
   - При инсталация отметни "Add Python to PATH"

### Стъпка 2: Стартиране
1. Отвори папката с файловете
2. Shift + Right-click в папката → "Open PowerShell window here"
3. Напиши:
   ```powershell
   python server.py
   ```
4. Ще видиш:
   ```
   🌐 BOM Visualizer Server СТАРТИРАН!
   📂 Споделена папка: C:\...\Try for success 5
   
   🔗 Достъп до приложението:
      От този компютър:  http://localhost:8080/unified_bom_viewer.html
      От локалната мрежа: http://192.168.1.100:8080/unified_bom_viewer.html
   ```

5. **Копирай втория линк** (с IP адреса) и го изпрати на колегите!

### Стъпка 3: Използване
- Отвори линка в браузър
- Влез в Admin режим (парола: 2006)
- Класифицирай сборките
- Промените се **запазват автоматично** в `bom_data.json`
- Всички виждат промените веднага

### Важно:
- Компютърът трябва да е **включен**, за да работи
- **НЕ затваряй** PowerShell прозореца (там работи сървъра)
- За спиране натисни **Ctrl+C** в PowerShell

---

## 🏢 Вариант 2: Пускане на NAS (24/7 достъп)

### За Synology NAS:

#### Стъпка 1: Подготовка
1. Отвори **File Station** на NAS-а
2. Създай нова споделена папка: `BOM_Visualizer`
3. Качи файловете:
   - `unified_bom_viewer.html`
   - `bom_data.json`
   - `server.py`
   - Папка `Screenshots/`

#### Стъпка 2: Инсталиране на Python
1. Отвори **Package Center**
2. Инсталирай **Python 3**
3. Изчакай да завърши

#### Стъпка 3: Настройка на Task Scheduler
1. Отвори **Control Panel** → **Task Scheduler**
2. Create → **Triggered Task** → **User-defined script**
3. Настройки:
   - **Task Name:** BOM Visualizer Server
   - **User:** root
   - **Event:** Boot-up
   - **Enabled:** ✓
4. В **Task Settings** → **Run command:**
   ```bash
   /usr/bin/python3 /volume1/BOM_Visualizer/server.py > /volume1/BOM_Visualizer/server.log 2>&1 &
   ```
5. Запази

#### Стъпка 4: Стартиране
- Рестартирай NAS-а (или пусни task-а ръчно)
- Достъп: `http://<NAS-IP>:8080/unified_bom_viewer.html`

### За QNAP NAS:
Подобно на Synology, но:
- Инсталирай Python през **App Center**
- Използвай **Qboost** → **Auto-run** за автоматично стартиране

---

## 🔧 Вариант 3: Автоматично стартиране при Windows boot

### Стъпка 1: Създай batch файл
1. Създай нов текстов файл в папката: `start_server.bat`
2. Напиши в него:
   ```batch
   @echo off
   cd /d "C:\Users\user\Documents\Alibre Script Library\Try for success 5"
   python server.py
   pause
   ```
3. Запази и затвори

### Стъпка 2: Добави в Startup
1. Натисни **Win + R**
2. Напиши: `shell:startup`
3. Копирай `start_server.bat` в отворената папка
4. Готово! Сървърът ще стартира при всяко пускане на Windows

---

## 🧪 Тестване

### 1. Провери на собствения си компютър:
- Отвори: `http://localhost:8080/unified_bom_viewer.html`
- Трябва да се зареди приложението

### 2. Провери от друг компютър в мрежата:
- Отвори: `http://<IP-адрес>:8080/unified_bom_viewer.html`
- Трябва да работи

### 3. Тест на запазване:
- Влез в Admin режим (парола: 2006)
- Класифицирай 1 сборка
- Провери `bom_data.json` - трябва да има секция `"classification"`
- Refresh от друг компютър - трябва да видиш промяната

---

## ❓ Често срещани проблеми

### "Python не се разпознава като команда"
- Python не е инсталиран или не е добавен в PATH
- Решение: Инсталирай Python отново с опция "Add to PATH"

### "Не мога да достъпя от друг компютър"
- Firewall блокира порт 8080
- Решение: 
  1. Windows Defender Firewall → Advanced Settings
  2. Inbound Rules → New Rule → Port → TCP 8080 → Allow
  3. Готово

### "bom_data.json не се запазва"
- Проверка на permissions на файла
- Решение: Right-click → Properties → Security → Разреши "Write"

---

## 📞 Поддръжка

Ако имаш проблеми:
1. Провери дали сървърът е стартиран (виж PowerShell прозореца)
2. Провери дали всички файлове са в една папка
3. Провери дали `bom_data.json` съществува
4. Погледни логовете в PowerShell прозореца за грешки

**За повече помощ:** Провери `server.log` (ако пускаш на NAS)
