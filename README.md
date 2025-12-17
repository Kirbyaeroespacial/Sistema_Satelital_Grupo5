# 🛰️ Sistema de Comunicación Satelital - Grupo 5

Sistema de comunicación satélite-tierra con tecnología LoRa que simula el envío y recepción de telemetría satelital en tiempo real. El sistema transmite datos de temperatura, humedad, distancia ultrasónica y posición orbital simulada, con validación por checksum y visualización gráfica mediante Python.

[![Version](https://img.shields.io/badge/version-3.0-blue.svg)](https://github.com/Kirbyaeroespacial/Sistema_Satelital_Grupo5/releases)
[![Arduino](https://img.shields.io/badge/Arduino-Compatible-00979D.svg)](https://www.arduino.cc/)
[![Python](https://img.shields.io/badge/Python-3.x-yellow.svg)](https://www.python.org/)

## Tabla de Contenidos

- [Características Principales](#-características-principales)
- [Video](https://www.youtube.com/watch?v=6ODh15rp7SE)
- [Arquitectura del Sistema](#-arquitectura-del-sistema)
- [Requisitos de Hardware](#-requisitos-de-hardware)
- [Requisitos de Software](#-requisitos-de-software)
- [Instalación](#-instalación)
- [Configuración](#️-configuración)
- [Uso del Sistema](#-uso-del-sistema)
- [Estructura del Proyecto](#-estructura-del-proyecto)
- [Protocolo de Comunicación](#-protocolo-de-comunicación)
- [Visualización de Datos](#-visualización-de-datos)
- [Pruebas](#-pruebas)
- [Solución de Problemas](#-solución-de-problemas)
- [Roadmap](#-roadmap)
- [Autores](#-autores)
- [Licencia](#-licencia)

## Características Principales

### Versión 4.0
- Telemetría en Tiempo Real: Captura y transmisión de temperatura, humedad y distancia
- Comunicación LoRa: Enlace inalámbrico mediante SoftwareSerial entre satélite y estación terrena
- Validación de Datos: Sistema de checksum XOR para detección de errores en la transmisión
- Tracking Orbital: Simulación orbital elíptica con inclinación y rotación terrestre
- Ground Track: Traza terrestre del satélite sobre mapa mundial
- Dashboard Python: Interfaz gráfica con visualización 3D de órbita y telemetría
- Control de Servo: Sistema automático (barrido) y manual con control remoto de ángulo
- Sistema de Logs: Registro de eventos con filtrado por tipo y fecha
- Análisis Estadístico: Cálculo de media de las últimas 10 temperaturas (satélite o tierra)
- Panel Solar Automatizado: Despliegue/retracción mediante stepper controlado por fotorresistor
- Alarma de Timeout: Detección de pérdida de comunicación con LED parpadeante
- Telemetría Binaria: Frames de 27 bytes para transmisión eficiente
- Gestión de Turnos: Protocolo token passing para evitar colisiones

## Arquitectura del Sistema

```
┌─────────────────────────────────────────────────────────────┐
│                      SEGMENTO ESPACIAL                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Arduino (Satélite)                                  │   │
│  │  ├─ Sensor DHT11 (Temperatura/Humedad)              │   │
│  │  ├─ Sensor HC-SR04 (Distancia Ultrasónica)          │   │
│  │  ├─ Generador de Posición Orbital (Elíptica)        │   │
│  │  ├─ Motor Stepper 28BYJ-48 (Panel Solar)            │   │
│  │  ├─ Fotorresistor (Sensor de Luz)                   │   │
│  │  ├─ Servo Motor (Orientación)                       │   │
│  │  ├─ SoftwareSerial LoRa (TX=11, RX=10)              │   │
│  │  ├─ Sistema de Checksum XOR                         │   │
│  │  └─ Gestión de Turnos (Token Passing)               │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ LoRa Link
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     SEGMENTO TERRESTRE                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Arduino (Estación Tierra)                           │   │
│  │  ├─ SoftwareSerial LoRa (TX=11, RX=10)              │   │
│  │  ├─ Validador de Checksum                            │   │
│  │  ├─ Potenciómetro A0 (Control Manual)               │   │
│  │  ├─ Detección de Timeout (20s)                      │   │
│  │  └─ Interfaz Serial → Python                         │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Interfaz Python                                     │   │
│  │  ├─ Auto-detección de Puerto Serial                 │   │
│  │  ├─ Parser de Telemetría Binaria                     │   │
│  │  ├─ Gráficos en Tiempo Real                          │   │
│  │  │  ├─ Temperatura y Humedad                         │   │
│  │  │  ├─ Distancia (Radar Polar)                       │   │
│  │  │  └─ Órbita 3D                                     │   │
│  │  ├─ Ground Track sobre Mapa                          │   │
│  │  ├─ Sistema de Logs con Filtrado                     │   │
│  │  ├─ Control de Ángulo Manual (0-180°)               │   │
│  │  ├─ Monitor de Panel Solar                           │   │
│  │  └─ Panel de Control                                 │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Requisitos de Hardware

### Segmento Espacial (Satélite)
- Arduino Uno/Nano o compatible
- Sensor DHT11 (temperatura y humedad)
- Sensor HC-SR04 (distancia ultrasónica)
- Motor Stepper 28BYJ-48 con driver ULN2003
- Fotorresistor (LDR)
- Servo motor (SG90 o similar)
- Resistencia 10kΩ (pull-down para fotorresistor)
- LEDs indicadores (2x): Pin 12 (transmisión), Pin 13 (alarma)
- Fuente de alimentación (baterías o USB)
- Cables de conexión

### Segmento Terrestre (Estación)
- Arduino Uno/Nano o compatible
- Módulo LoRa (SX1276/SX1278)
- Servo motor (SG90 o similar)
- Potenciómetro 10kΩ (control manual)
- LED error (Pin 2)
- Cable USB para comunicación serial
- Ordenador con Python 3.x

### Conexiones Recomendadas

#### Arduino Satélite
```
DHT11:
  - VCC → 5V
  - GND → GND
  - DATA → Pin 2

HC-SR04:
  - VCC → 5V
  - GND → GND
  - TRIG → Pin 3
  - ECHO → Pin 4

Servo Motor:
  - VCC → 5V
  - GND → GND
  - SIGNAL → Pin 5

Stepper Motor (ULN2003):
  - IN1 → Pin 6
  - IN2 → Pin 7
  - IN3 → Pin 8
  - IN4 → Pin 9
  - VCC → 5V
  - GND → GND

Fotorresistor:
  - Un extremo → 5V
  - Otro extremo → A1 y resistencia 10kΩ
  - Resistencia 10kΩ → GND

SoftwareSerial:
  - RX → Pin 10
  - TX → Pin 11

LED Transmisión → Pin 12
LED Alarma → Pin 13
```

#### Arduino Estación Tierra
```
Servo Motor:
  - VCC → 5V
  - GND → GND
  - SIGNAL → Pin 9

LoRa Module:
  - VCC → 3.3V
  - GND → GND
  - SCK → Pin 13
  - MISO → Pin 12
  - MOSI → Pin 11
  - NSS → Pin 10
  - RST → Pin 9
Potenciómetro:
  - Terminal central → A0
  - Terminales laterales → 5V y GND

SoftwareSerial:
  - RX → Pin 10
  - TX → Pin 11

LED Error → Pin 2
```

## Requisitos de Software

### Arduino
- Arduino IDE 1.8.x o superior
- Librerías requeridas:
  ```
- DHT sensor library by Adafruit
  - Adafruit Unified Sensor
  - Servo (incluida en Arduino IDE)
  - Stepper (incluida en Arduino IDE)
  - SoftwareSerial (incluida en Arduino IDE)
  ```

### Python
- Python 3.7 o superior
- Librerías requeridas (ver `requirements.txt`):
  ```
  pyserial>=3.5
  matplotlib>=3.5.0
  numpy>=1.21.0
   pillow>=8.0.0
  ```

## Instalación

### 1. Clonar el Repositorio

```bash
git clone https://github.com/Kirbyaeroespacial/Sistema_Satelital_Grupo5.git
cd Sistema_Satelital_Grupo5
```

### 2. Configurar Arduino

#### Instalar Librerías
1. Abrir Arduino IDE
2. Ir a `Sketch → Include Library → Manage Libraries`
3. Buscar e instalar:
   - LoRa by Sandeep Mistry
   - DHT sensor library
   - Adafruit Unified Sensor

#### Cargar el Código del Satélite
```bash
1. Abrir satelite.ino en Arduino IDE
2. Seleccionar placa: Tools → Board → Arduino Uno
3. Seleccionar puerto: Tools → Port → [Tu Puerto]
4. Cargar: Sketch → Upload (Ctrl+U)
```

#### Cargar el Código de la Estación Tierra
```bash
1. Abrir estacion_tierra.ino en Arduino IDE
2. Repetir pasos de selección de placa y puerto
3. Cargar el sketch
```

### 3. Configurar Python

#### Crear Entorno Virtual (Recomendado)
```bash
python -m venv venv

# Windows
venv\Scripts\activate

# Linux/Mac
source venv/bin/activate
```

#### Instalar Dependencias
```bash
pip install -r requirements.txt
```

Si no existe `requirements.txt`, instalar manualmente:
```bash
pip install pyserial matplotlib numpy pillow
```

### Configuración del Puerto Serial

Editar en `interfaz.py`:
```python
# Buscar la línea de configuración del puerto
SERIAL_PORT = 'COM3'  # Windows: COM3, COM4, etc.
# SERIAL_PORT = '/dev/ttyUSB0'  # Linux
# SERIAL_PORT = '/dev/cu.usbserial-*'  # Mac

BAUD_RATE = 9600
```

### Configuración de LoRa

En ambos archivos `.ino`, ajustar si es necesario:
```cpp
// Frecuencia LoRa (debe ser igual en satélite y tierra)
#define LORA_FREQUENCY 433E6  // 433 MHz (Europa)
// #define LORA_FREQUENCY 915E6  // 915 MHz (América)

// Parámetros de transmisión
#define LORA_BANDWIDTH 125E3
#define LORA_SPREADING_FACTOR 7
#define LORA_CODING_RATE 5
```

## Uso del Sistema

### Inicio Rápido

1. **Conectar Hardware**
   - Conectar Arduino del satélite (sin conexión USB a PC)
   - Conectar Arduino de estación tierra vía USB a PC

2. **Iniciar Sistema**
   ```bash
   # Activar entorno virtual si se usa
   python interfaz.py
   ```

3. **Verificar Comunicación**
   - La interfaz mostrará datos recibidos en tiempo real
   - Las gráficas se actualizarán automáticamente
   - El log registrará eventos de comunicación

### Modos de Operación

#### Modo Automático (Por Defecto)
- El servo se orienta automáticamente según la posición del satélite
- Actualización continua de telemetría

#### Modo Manual
```
⚠️ NOTA: El modo manual está temporalmente deshabilitado en la versión 3.0
Próximamente se restaurará en futuras actualizaciones
```

### Comandos de la Interfaz
Botones Principales
Control de Transmisión:

Iniciar Transmisión (3:i): Comienza envío de telemetría
Parar Transmisión (3:p): Detiene envío
Reanudar Transmisión (3:r): Continúa tras pausa

Control de Modo Servo:

Modo Automático (4:a): Barrido automático 0-180°
Modo Manual (4:m): Control mediante entrada de ángulo

Configuración:

Intervalo (ms): Ajusta frecuencia de transmisión (200-10000 ms)
Ángulo Manual: Envía comando de ángulo específico (0-180°)
Sitio cálculo temp media (42:1): Alterna entre cálculo local (satélite) o remoto (tierra)

Visualización:

Ver Eventos: Abre registro con filtros por tipo y fecha
Ver Ground Track: Muestra traza terrestre sobre mapa

Sistema de Observaciones

Campo de texto para agregar notas personalizadas
Las observaciones se registran con timestamp en eventos.txt
Formato: YYYY-MM-DD HH:MM:SS|observacion|texto

## 📁 Estructura del Proyecto

```
Sistema_Satelital_Grupo5/
│
├── satelite.ino                    # Código Arduino del satélite
├── estacion_tierra.ino             # Código Arduino de estación tierra
├── interfaz.py                     # Interfaz gráfica Python principal
│
├── README.md                       # Este archivo
├── README_SAT.md                   # Documentación específica del satélite
├── README_GS.md                    # Documentación de estación tierra
│
├── test_checksum.ino               # Test de validación checksum (satélite)
├── test_checksum_GS.ino            # Test de validación checksum (tierra)
├── test_checksum_PY.py             # Test de checksum en Python
├── test_UI.py                      # Test de interfaz de usuario
│
├── requirements.txt                # Dependencias Python
└── docs/                           # Documentación adicional
    ├── diagrams/                   # Diagramas del sistema
    └── manuals/                    # Manuales de usuario
```

## 📡 Protocolo de Comunicación

### Formato de Trama Binaria

```
struct TelemetryFrame {
  uint8_t header;        // 0xAA (marcador de inicio)
  uint16_t humidity;     // Humedad × 100 (5023 = 50.23%)
  int16_t temperature;   // Temperatura × 100 (2156 = 21.56°C)
  uint16_t tempAvg;      // Temperatura media × 100
  uint16_t distance;     // Distancia en mm
  uint8_t servoAngle;    // Ángulo del servo (0-180)
  uint16_t time_s;       // Tiempo orbital en segundos
  // Coordenadas X, Y, Z (4 bytes cada una, little-endian)
  uint8_t x_b0, x_b1, x_b2, x_b3;
  uint8_t y_b0, y_b1, y_b2, y_b3;
  uint8_t z_b0, z_b1, z_b2, z_b3;
  uint8_t panelState;    // Estado panel solar (0-100%)
  uint8_t checksum;      // XOR de todos los bytes anteriores
};
```

#### Ejemplo de Trama
```
T:23.5|H:65.2|D:150|X:1250|Y:340|CS:A3F
```

### Campos de Datos

| Campo | Descripción | Unidad | Rango |
|-------|-------------|--------|-------|
| TEMP | Temperatura | °C | -40 a 80 |
| HUM | Humedad relativa | % | 0 a 100 |
| DIST | Distancia ultrasónica | cm | 2 a 400 |
| POS_X | Posición orbital X | km | -2000 a 2000 |
| POS_Y | Posición orbital Y | km | -2000 a 2000 |
| CS | Checksum | Hex | 00 a FF |

### Algoritmo de Checksum 

```cpp
uint8_t calculateChecksum(String data) {
  uint8_t checksum = 0;
  for (int i = 0; i < data.length(); i++) {
    checksum ^= data[i];  // XOR de todos los caracteres
  }
  return checksum;
}
```

### Validación de Datos

```python
def validate_checksum(data, received_checksum):
    calculated = 0
    for char in data:
        calculated ^= ord(char)
    return calculated == received_checksum
```

## 📊 Visualización de Datos

### Gráficas Disponibles
1. Órbita Satelital (3D)

Visualización tridimensional con matplotlib
Esfera verde representa la Tierra (R=6371 km)
Trayectoria en cian, posición actual en rojo
Actualización cada 500ms

2. Sonar de Distancia (Radar Polar)

Gráfica polar: ángulo servo vs distancia
Rango: 0-500 mm
Últimos 20 puntos visibles
Actualización cada 100ms

3. Temperatura y Humedad

Tres líneas:

Roja: Temperatura instantánea
Cian: Humedad relativa
Amarilla: Temperatura media (últimas 10)


Ventana deslizante: 100 puntos
Actualización cada 100ms

4. Ground Track (Ventana separada)

Traza terrestre sobre mapa mundial
Conversión XYZ → lat/lon
Marcador posición actual (rojo)
Trayectoria histórica (cian, hasta 600 puntos)
Líneas de referencia: Ecuador y Meridiano 0°

5. Indicador Panel Solar

Estados con colores:

0% RETRAÍDO: Rojo (#ff6b6b)
40% DESPLEGADO: Amarillo (#ffd93d)
60% DESPLEGADO: Verde claro (#6bcf7f)
100% DESPLEGADO: Verde (#51cf66)


Actualización cada 500ms

Sistema de Eventos
Tipos de eventos:

comando: Comandos enviados al satélite
alarma: Errores, timeouts, temperaturas críticas
observacion: Notas del usuario

Archivo: eventos.txt
Formato: YYYY-MM-DD HH:MM:SS|tipo|detalles
Filtros disponibles:

Por tipo: todos/comando/alarma/observacion
Por rango de fechas: desde/hasta (dd-mm-YYYY HH:MM:SS)
### Características de Visualización

- ✅ Actualización en tiempo real (~20 segundos por actualización)
- ✅ Ventana de datos configurable
- ✅ Exportación a imagen PNG
- ✅ Leyendas y etiquetas claras
- ✅ Colores diferenciados por tipo de dato

## 🧪 Pruebas

### Suite de Tests Incluida

#### Test de Checksum - Arduino Satélite
```bash
# Cargar test_checksum.ino en Arduino del satélite
# Verificar Serial Monitor para resultados
```

#### Test de Checksum - Arduino Tierra
```bash
# Cargar test_checksum_GS.ino en Arduino de tierra
# Comparar checksums recibidos vs calculados
```

#### Test de Checksum - Python
```bash
python test_checksum_PY.py
```

#### Test de Interfaz
```bash
python test_UI.py
```

### Pruebas Recomendadas

1. **Prueba de Alcance LoRa**
   - Separar gradualmente satélite y tierra
   - Medir RSSI (Received Signal Strength Indicator)
   - Documentar distancia máxima efectiva

2. **Prueba de Integridad de Datos**
   - Transmitir 1000 paquetes
   - Contar paquetes correctos vs corruptos
   - Calcular tasa de error

3. **Prueba de Latencia**
   - Medir tiempo desde captura hasta visualización
   - Optimizar buffer de serial

## 🔍 Solución de Problemas

### Problemas Comunes

#### No se reciben datos en la interfaz Python

```
Solución:
1. Verificar puerto serial correcto en interfaz.py
2. Comprobar que Arduino tierra está conectado
3. Revisar velocidad de baudios (debe ser 9600)
4. Verificar drivers CH340/FTDI instalados
```

#### Errores de checksum frecuentes

```
Solución:
1. Reducir distancia entre satélite y tierra
2. Verificar antenas conectadas correctamente
3. Ajustar spreading factor de LoRa (aumentar para mayor alcance)
4. Revisar interferencias electromagnéticas
```

#### Las gráficas no se actualizan

```
Solución:
1. Verificar que llegan datos al serial (ver consola)
2. Comprobar formato de tramas correcto
3. Revisar timeout de matplotlib
4. Reiniciar aplicación Python
```

#### Servo no responde

```
Solución:
1. Verificar conexión del servo (señal, VCC, GND)
2. Comprobar alimentación suficiente (5V estable)
3. Verificar pin configurado en código (Pin 9 por defecto)
4. Probar servo con sketch de ejemplo Arduino
```

#### Lecturas de sensores erróneas

```
DHT22:
- Verificar pull-up resistor de 10kΩ
- Esperar 2 segundos entre lecturas
- Verificar alimentación estable

HC-SR04:
- Verificar distancia al objeto (2-400 cm)
- Comprobar ángulo de medición (< 15°)
- Evitar superficies absorbentes de sonido
```

## 🗺️ Roadmap

### Versión Actual: 4.0 ✅
-  Panel solar automatizado con motor stepper 28BYJ-48
-  Control mediante fotorresistor con umbrales configurables
- Órbita elíptica con ecuación de Kepler
-  Inclinación orbital (51.6°) y rotación terrestre (ECEF)
-  Telemetría binaria de 27 bytes
-  Ground track con conversión XYZ→lat/lon
-  Control manual de ángulo desde interfaz (0-180°)
 - Alarma de timeout con LED parpadeante
-  Cálculo temperatura media configurable (local/remoto)
- Sistema de logs con filtrado avanzado
- Visualización 3D de órbita con matplotlib
- Auto-detección de puerto serial
- Movimiento no-bloqueante del stepper
- Gestión de turnos mediante token passing

## 👥 Autores

### Grupo 5

- **Kirbyaeroespacial** - [GitHub](https://github.com/Kirbyaeroespacial)
- **rpraena** - [GitHub](https://github.com/rpraena)
- **Michail2007** - [GitHub](https://github.com/Michail2007)


<div align="center">

Hecho con ❤️ por el Grupo 5

[⬆ Volver arriba](#-sistema-de-comunicación-satelital---grupo-5)

</div>



