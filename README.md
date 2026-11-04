# 🎛️ MIDI DJ - Controller and Visual Synthesizer Interface

> Controlador MIDI artesanal basado en **Arduino Leonardo (ATmega32u4)** con sensores táctiles capacitivos TTP223, botones mecánicos y potenciómetros analógicos, diseñado para la generación y control de visuales en tiempo real mediante **Hydra Synth**.

---

## ⚡ Características Principales

* **🔌 USB-MIDI Nativo (Plug & Play):** Gracias al microcontrolador ATmega32u4 y la librería oficial `MIDIUSB.h`, el dispositivo es reconocido de forma nativa por el sistema operativo sin requerir software puente adicional.
* **🖐️ Interfaz Híbrida (Táctil y Mecánica):** 
  * 3x Botones mecánicos configurados con resistencia Pull-up interna.
  * 2x Sensores táctiles capacitivos TTP223 para una respuesta de disparo instantánea.
* **🎛️ Control Analógico Continuo:** 2x Potenciómetros conectados a entradas analógicas para control de parámetros por Control Change (`CC 10` y `CC 11`).
* **🌐 Integración con Hydra Synth:** Uso directo de la librería oficial `hydra-midi` en la consola web para mapeo fluido de notas e impulsos de control.

---

## 🗺️ Mapeo de Pines y Señales (Pinout)

| Componente | Tipo de Entrada | Pin Arduino | Nota / Comando MIDI | Función Asociada |
| :--- | :---: | :---: | :---: | :--- |
| **Botón 1** | Digital (INPUT_PULLUP) | `Pin 2` | Nota 60 (C4) | Trigger Visual 1 |
| **Botón 2** | Digital (INPUT_PULLUP) | `Pin 3` | Nota 62 (D4) | Trigger Visual 2 |
| **Botón 3** | Digital (INPUT_PULLUP) | `Pin 4` | Nota 64 (E4) | Trigger Visual 3 |
| **Sensor Táctil 1** | Digital (TTP223) | `Pin 5` | Nota 65 (F4) | Modulación / Estrobo |
| **Sensor Táctil 2** | Digital (TTP223) | `Pin 6` | Nota 67 (G4) | Cambio de Estado / Paleta |
| **Potenciómetro 1** | Analógico | `A0` | CC 10 | Control de Zoom / Escala |
| **Potenciómetro 2** | Analógico | `A1` | CC 11 | Modulación de Capas / Velocidad |

---

## 💻 Código Firmware (Arduino IDE)

```cpp
#include "MIDIUSB.h"

const int pinPot1 = A0;
const int pinPot2 = A1;

const int pinBoton1 = 2;
const int pinBoton2 = 3;
const int pinBoton3 = 4;

const int pinTactil1 = 5;
const int pinTactil2 = 6;

const byte canalMIDI = 0; 

const byte notaB1  = 60; 
const byte notaB2  = 62; 
const byte notaB3  = 64; 
const byte notaT1  = 65; 
const byte notaT2  = 67; 

const byte ccPot1  = 10; 
const byte ccPot2  = 11; 

int ultimoCC1 = -1;
int ultimoCC2 = -1;

bool ultB1 = HIGH, ultB2 = HIGH, ultB3 = HIGH;
bool ultT1 = LOW,  ultT2 = LOW;

void setup() {
  pinMode(pinBoton1, INPUT_PULLUP);
  pinMode(pinBoton2, INPUT_PULLUP);
  pinMode(pinBoton3, INPUT_PULLUP);
  
  pinMode(pinTactil1, INPUT);
  pinMode(pinTactil2, INPUT);
}

void loop() {
  int actualCC1 = map(analogRead(pinPot1), 0, 1023, 0, 127);
  int actualCC2 = map(analogRead(pinPot2), 0, 1023, 0, 127);

  if (abs(actualCC1 - ultimoCC1) >= 1) {
    enviarControlChange(canalMIDI, ccPot1, actualCC1);
    ultimoCC1 = actualCC1;
  }
  if (abs(actualCC2 - ultimoCC2) >= 1) {
    enviarControlChange(canalMIDI, ccPot2, actualCC2);
    ultimoCC2 = actualCC2;
  }

  bool lecturaB1 = digitalRead(pinBoton1);
  bool lecturaB2 = digitalRead(pinBoton2);
  bool lecturaB3 = digitalRead(pinBoton3);
  bool lecturaT1 = digitalRead(pinTactil1);
  bool lecturaT2 = digitalRead(pinTactil2);

  if (lecturaB1 != ultB1) {
    if (lecturaB1 == LOW) enviarNotaOn(canalMIDI, notaB1, 127);
    else                  enviarNotaOff(canalMIDI, notaB1, 0);
    ultB1 = lecturaB1;
  }
  if (lecturaB2 != ultB2) {
    if (lecturaB2 == LOW) enviarNotaOn(canalMIDI, notaB2, 127);
    else                  enviarNotaOff(canalMIDI, notaB2, 0);
    ultB2 = lecturaB2;
  }
  if (lecturaB3 != ultB3) {
    if (lecturaB3 == LOW) enviarNotaOn(canalMIDI, notaB3, 127);
    else                  enviarNotaOff(canalMIDI, notaB3, 0);
    ultB3 = lecturaB3;
  }

  if (lecturaT1 != ultT1) {
    if (lecturaT1 == HIGH) enviarNotaOn(canalMIDI, notaT1, 127);
    else                   enviarNotaOff(canalMIDI, notaT1, 0);
    ultT1 = lecturaT1;
  }
  if (lecturaT2 != ultT2) {
    if (lecturaT2 == HIGH) enviarNotaOn(canalMIDI, notaT2, 127);
    else                   enviarNotaOff(canalMIDI, notaT2, 0);
    ultT2 = lecturaT2;
  }

  delay(10);
}

void enviarNotaOn(byte canal, byte nota, byte velocidad) {
  midiEventPacket_t notaOn = {0x09, (byte)(0x90 | canal), nota, velocidad};
  MidiUSB.sendMIDI(notaOn);
  MidiUSB.flush();
}

void enviarNotaOff(byte canal, byte nota, byte velocidad) {
  midiEventPacket_t notaOff = {0x08, (byte)(0x80 | canal), nota, velocidad};
  MidiUSB.sendMIDI(notaOff);
  MidiUSB.flush();
}
```
## 🚀 Guía de Instalación y Uso Rápido

### Fase 1: Configuración del Hardware
* **1. Instalación de la Librería:** En el IDE de Arduino, ir a `Herramientas -> Administrar Bibliotecas...`, buscar e instalar **`MIDIUSB`** (de Arduino).
* **2. Selección de Placa:** Asegurarse de seleccionar **Arduino Leonardo** en `Herramientas -> Placa`.
* **3. Carga del Firmware:** Conectar el Arduino Leonardo por USB y cargar el código al microcontrolador.

### Fase 2: Integración con Hydra Synth
* **1. Entrar a Hydra:** Ir a la plataforma web de live-coding [Hydra Synth](https://hydra.ojack.xyz/).
* **2. Ejecutar Script:** Copiar e inicializar la extensión **`hydra-midi`** ejecutando el siguiente bloque en la consola:
```JavaScript
await loadScript('[https://cdn.jsdelivr.net/npm/hydra-midi@latest/dist/index.js](https://cdn.jsdelivr.net/npm/hydra-midi@latest/dist/index.js)')
await midi.start().show()
```
---

## 📚 Referencias & Reconocimientos

* Autores: Vlinx y Zaiiv 
* 📹 **Programación MIDI USB:** Referencia de estructura y manejo en Arduino basada en el tutorial de [Nerd Musician](https://www.youtube.com/watch?v=Etb6uzAZwRA).
* 🎨 **Shaders Adicionales:** Repositorio de efectos [extra-shaders-for-hydra](https://gitlab.com/metagrowing/extra-shaders-for-hydra) desarrollado por **Thomas Jourdan** (*metagrowing*).
* 🕹️ **Inspiración de Estructura:** Guías de diseño abiertas inspiradas en proyectos como [Arduinoboy](https://github.com/trash80/arduinoboy).
---
## Conexiones y Materiales
<img width="581" height="666" alt="Captura de pantalla 2026-11-03 231222" src="https://github.com/user-attachments/assets/463943c8-3378-4505-a368-e9ee0d7afb5b" />
<img width="560" height="563" alt="Captura de pantalla 2026-11-03 231348" src="https://github.com/user-attachments/assets/32487959-4cc8-47ac-a109-bc93a69d1c23" />

---
## Prueba thinkercard
<img width="1098" height="763" alt="Captura de pantalla 2026-11-03 231311" src="https://github.com/user-attachments/assets/8e938855-4397-448e-84e6-73882b1b5083" />

[![Ver video demostrativo en YouTube](https://img.youtube.com/vi/lFxMT9V71v0/maxresdefault.jpg)](https://youtu.be/lFxMT9V71v0)
