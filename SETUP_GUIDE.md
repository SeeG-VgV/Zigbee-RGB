# 🎯 ESP32-C6 ZigBee RGB Light - Полное руководство

## ✨ Что сделано

### ✅ Исправлены цвета
LED лента работает в RGB порядке, а не GRB. Модифицирован файл:
`managed_components/espressif__led_strip/src/led_strip_rmt_dev.c` (строки 32-36)

### ✅ Устранён boot loop
Добавлена проверка перед network steering в файле:
`main/esp_zb_light.c` (строки 59-65)

### ✅ Результат
- Цвета правильные (R→R, G→G, B→B)
- Мгновенный отклик на команды (без задержки 15-20 сек)
- Стабильная работа в ZigBee сети

---

## 🚀 Как прошить устройство

### 1. Подготовка
```bash
cd /Users/grigorij/My_ESP-IDF_Projects/ZigBee-RGB
```

### 2. Сборка
```bash
source /Users/grigorij/esp/v5.5.1/esp-idf/export.sh
idf.py build
```

### 3. Прошивка (первый раз - с очисткой)
```bash
# Полная очистка
idf.py -p /dev/tty.usbmodem12401 erase-flash

# Прошивка
cd build
/Users/grigorij/.local/bin/esptool.py --chip esp32c6 -b 460800 \
  --before default_reset --after hard_reset \
  --port /dev/tty.usbmodem12401 write_flash "@flash_args"
```

### 4. Просмотр логов
```bash
source /Users/grigorij/esp/v5.5.1/esp-idf/export.sh
idf.py monitor -p /dev/tty.usbmodem12401
```

---

## 🔄 Работа с Git

### Посмотреть текущий статус
```bash
cd /Users/grigorij/My_ESP-IDF_Projects/ZigBee-RGB
git status
git log --oneline
```

### Сохранить изменения
```bash
git add .
git commit -m "Описание что изменили"
git push
```

### Создать новую версию
```bash
git tag -a v1.1 -m "Version 1.1: Что добавили"
git push origin v1.1
```

### ⚠️ Откатить если что-то сломалось
```bash
# Вернуться к рабочей версии 1.0
git checkout v1.0

# Или просто откатить все изменения
git reset --hard origin/main
```

### 🆘 Полностью испортили? Скачать заново!
```bash
cd /Users/grigorij/My_ESP-IDF_Projects/
rm -rf ZigBee-RGB
git clone https://github.com/SeeG-VgV/Zigbee-RGB.git ZigBee-RGB
```

---

## 🔍 Отладка

### Если цвета неправильные
Проверьте файл: `managed_components/espressif__led_strip/src/led_strip_rmt_dev.c`

Строки 32-36 должны быть:
```c
rmt_strip->pixel_buf[start + 0] = red & 0xFF;    // RED первый!
rmt_strip->pixel_buf[start + 1] = green & 0xFF;
rmt_strip->pixel_buf[start + 2] = blue & 0xFF;
```

### Если устройство тормозит (boot loop)
Проверьте файл: `main/esp_zb_light.c`

В обработчике `ESP_ZB_BDB_SIGNAL_DEVICE_REBOOT` (строки 59-65) должно быть:
```c
if (esp_zb_bdb_is_factory_new() || esp_zb_get_short_address() == 0xFFFE) {
    ESP_LOGI(TAG, "Start network steering");
    esp_zb_bdb_start_top_level_commissioning(ESP_ZB_BDB_MODE_NETWORK_STEERING);
} else {
    ESP_LOGI(TAG, "Device already on network (Short Address: 0x%04hx)", 
             esp_zb_get_short_address());
}
```

### Правильные логи при работе
```
I (450) ESP_ZB_COLOR_DIMM_LIGHT: Device already on network (Short Address: 0xc1a1)
I (2750) ESP_ZB_COLOR_DIMM_LIGHT: Received message: endpoint(10), cluster(0x8)
I (2750) ESP_ZB_COLOR_DIMM_LIGHT: Light level changes to 255
```

### ❌ Неправильные логи (boot loop)
```
I (xxx) ESP_ZB_COLOR_DIMM_LIGHT: Start network steering
W (xxx) ESP_ZB_COLOR_DIMM_LIGHT: Network(0x7321) closed
I (xxx) ESP_ZB_COLOR_DIMM_LIGHT: Start network steering  <- повторяется!
W (xxx) ESP_ZB_COLOR_DIMM_LIGHT: Network(0x7321) closed
```

---

## 🔧 Переподключение к другой ZigBee сети

### 1. Очистить ZigBee storage
```bash
/Users/grigorij/.local/bin/esptool.py --chip esp32c6 \
  --port /dev/tty.usbmodem12401 \
  --before default_reset --after hard_reset \
  erase_region 0xf5000 0x1000
```

### 2. Перезагрузить устройство
Просто нажмите кнопку RESET или переподключите USB

### 3. Включить режим "Permit join" в Z2M
Устройство автоматически найдёт сеть и подключится

---

## 📊 Железо

- **Чип:** ESP32-C6 (QFN40 v0.0)
- **MAC:** 40:4c:ca:50:d7:64
- **LED:** WS2812/SK6812 RGB на GPIO 8 (RGB порядок!)
- **Порт:** /dev/tty.usbmodem12401
- **ZigBee:** Router, Endpoint 10, PAN ID 0x7321

---

## 🔗 Полезные ссылки

- **GitHub:** https://github.com/SeeG-VgV/Zigbee-RGB
- **Версия 1.0 (рабочая):** https://github.com/SeeG-VgV/Zigbee-RGB/releases/tag/v1.0

---

**Создано:** 14 ноября 2025  
**Версия:** 1.0 - Стабильная рабочая версия