# Rubber Ducky con Arduino UNO R3

Convierte tu **Arduino UNO R3** en un **USB Rubber Ducky** — un teclado HID que ejecuta pulsaciones a velocidad de máquina en cuanto lo conectás.


## Cómo funciona

El Arduino UNO R3 tiene **dos chips**:

| Chip | USB nativo | Rol |
|------|------------|-----|
| **ATmega328P** | ❌ | Ejecuta el sketch con el payload |
| **ATmega16U2** | ✅ | Se flashea con firmware de **teclado HID** |

El 328P le envía teclas por Serial (UART, pines 0/1).  
El 16U2 las traduce a reportes HID USB.  
Windows lo ve como un teclado real — confianza total del kernel.

```
[328P] ──Serial──→ [16U2 firmware teclado] ──USB HID──→ [PC]
   ↑                                                        │
   │                                                        ↓
  payload                    Win+R → URL → Enter, etc.
```

## Requisitos

| Software | Link |
|----------|------|
| **dfu-programmer** | https://github.com/dfu-programmer/dfu-programmer/releases/tag/v1.1.0 |
| **Zadig** (drivers) | https://zadig.akeo.ie/ | (Esto es opcional en caso de que NO te detecte tu placa Arduino, no necesitas reiniciar el pc)
| **Arduino IDE** | https://www.arduino.cc/en/software |

---

## Cómo usar (paso a paso)

### 1. Poner el 16U2 en modo DFU

Conectá el Arduino **sin** los pines en corto.  
Después poné los dos pines (cerca del USB, los pads **RESET-EN**) en corto con un jumper o pinza, **mientras** conectás el USB.  
Una vez conectado, **soltá los pines**.

> Si no lo detecta, abrí **Zadig → Options → List All Devices** y revisá que el driver sea `libusb-win32` y si aparece "Unknown Device" dale reset.


### 2. Restaurar firmware USB-Serial y subir el sketch

Primero flasheá el firmware de serie para que el IDE pueda hablar con el 328P:

```
.\dfu-programmer atmega16u2 erase
.\dfu-programmer atmega16u2 flash --suppress-bootloader-mem Arduino-COMBINED-dfu-usbserial-atmega16u2-Uno-Rev3.hex
.\dfu-programmer atmega16u2 reset
```

Desconectá y conectá el Arduino.  
Ahora abrí el sketch (`.ino`) desde el **Arduino IDE**, seleccioná **Board: Arduino/UNO** compila y subilo.

### 3. Flashear firmware de teclado

Una vez subido el sketch, volvé a poner el 16U2 en modo DFU (pines en corto) y ejecutá:

```
.\dfu-programmer atmega16u2 erase
.\dfu-programmer atmega16u2 flash Arduino-keyboard-0.3.hex
.\dfu-programmer atmega16u2 reset
```

Desconectá y conectá. Al enchufarlo, el payload se ejecuta automáticamente.

---

## Comandos DFU (referencia rápida)

| Comando | Qué hace |
|---------|----------|
| `.\dfu-programmer atmega16u2 erase` | Borra el firmware actual |
| `.\dfu-programmer atmega16u2 flash --suppress-bootloader-mem archivo.hex` | Escribe un firmware nuevo |
| `.\dfu-programmer atmega16u2 reset` | Sale del modo DFU y reinicia |
| `.\dfu-programmer atmega16u2 erase` + `.\dfu-programmer atmega16u2 start` | Alternativa para resetear |

> Todos los comandos desde **PowerShell** o **CMD** en la misma carpeta donde está `dfu-programmer.exe`.

