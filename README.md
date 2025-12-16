# 🛰️ Sistema de Comunicación Satelital - Grupo 5

Sistema de comunicación satélite-tierra con tecnología LoRa que simula el envío y recepción de telemetría satelital en tiempo real. El sistema transmite datos de temperatura, humedad, distancia ultrasónica y posición orbital simulada, con validación por checksum y visualización gráfica mediante Python.

[![Version](https://img.shields.io/badge/version-3.0-blue.svg)](https://github.com/Kirbyaeroespacial/Sistema_Satelital_Grupo5/releases)
[![Arduino](https://img.shields.io/badge/Arduino-Compatible-00979D.svg)](https://www.arduino.cc/)
[![Python](https://img.shields.io/badge/Python-3.x-yellow.svg)](https://www.python.org/)

## Tabla de Contenidos

- [Características Principales](#-características-principales)
- [Video](https://drive.google.com/file/d/1oy3E7MwHHsBAGbqamduDBtTSIOgv2Od7/view?usp=drive_link)
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

### Versión 3.0

- 🌡️ **Telemetría en Tiempo Real**: Captura y transmisión de temperatura, humedad y distancia
- 📡 **Comunicación LoRa**: Enlace inalámbrico de largo alcance entre satélite y estación terrena
- 🛡️ **Validación de Datos**: Sistema de checksum para detección de errores en la transmisión
- 🗺️ **Tracking Orbital**: Simulación y visualización de la posición orbital del satélite
- 📊 **Dashboard Python**: Interfaz gráfica con visualización 2D de trayectoria y telemetría
- 🔧 **Control de Servo**: Sistema automático/manual para orientación de antenas
- 📝 **Sistema de Logs**: Registro de eventos y observaciones del usuario
- 📈 **Análisis Estadístico**: Cálculo de media de las últimas 10 temperaturas (procesable en satélite o tierra)

## Arquitectura del Sistema

```
┌─────────────────────────────────────────────────────────────┐
│                      SEGMENTO ESPACIAL                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Arduino (Satélite)                                  │   │
│  │  ├─ Sensor DHT22 (Temperatura/Humedad)              │   │
│  │  ├─ Sensor HC-SR04 (Distancia Ultrasónica)          │   │
│  │  ├─ Generador de Posición Orbital                   │   │
│  │  ├─ Módulo LoRa (Transmisor)                        │   │
│  │  └─ Sistema de Checksum                             │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ LoRa Link
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     SEGMENTO TERRESTRE                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Arduino (Estación Tierra)                           │   │
│  │  ├─ Módulo LoRa (Receptor)                           │   │
│  │  ├─ Validador de Checksum                            │   │
│  │  ├─ Servo Motor (Control Antena)                     │   │
│  │  └─ Interfaz Serial → Python                         │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Interfaz Python                                     │   │
│  │  ├─ Comunicación Serial                              │   │
│  │  ├─ Parser de Telemetría                             │   │
│  │  ├─ Gráficos en Tiempo Real                          │   │
│  │  │  ├─ Temperatura                                   │   │
│  │  │  ├─ Humedad                                       │   │
│  │  │  ├─ Distancia                                     │   │
│  │  │  └─ Trayectoria Orbital 2D                        │   │
│  │  ├─ Sistema de Logs                                  │   │
│  │  └─ Panel de Control                                 │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Requisitos de Hardware

### Segmento Espacial (Satélite)
- Arduino Uno/Nano o compatible
- Módulo LoRa (SX1276/SX1278)
- Sensor DHT11 (temperatura y humedad)
- Sensor HC-SR04 (distancia ultrasónica)
- Fuente de alimentación (baterías o USB)
- Cables de conexión

### Segmento Terrestre (Estación)
- Arduino Uno/Nano o compatible
- Módulo LoRa (SX1276/SX1278)
- Servo motor (SG90 o similar)
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

LoRa Module:
  - VCC → 3.3V
  - GND → GND
  - SCK → Pin 13
  - MISO → Pin 12
  - MOSI → Pin 11
  - NSS → Pin 10
  - RST → Pin 9
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
```

## Requisitos de Software

### Arduino
- Arduino IDE 1.8.x o superior
- Librerías requeridas:
  ```
  - LoRa by Sandeep Mistry
  - DHT sensor library by Adafruit
  - Adafruit Unified Sensor
  - Servo (incluida en Arduino IDE)
  ```

### Python
- Python 3.7 o superior
- Librerías requeridas (ver `requirements.txt`):
  ```
  pyserial>=3.5
  matplotlib>=3.5.0
  numpy>=1.21.0
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
pip install pyserial matplotlib numpy
```

## Configuración

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

| Tecla | Función |
|-------|---------|
| `ESC` | Cerrar aplicación |
| `S` | Guardar datos actuales |
| `R` | Resetear gráficas |
| `L` | Exportar logs |

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

### Formato de Trama

```
[TEMP][HUM][DIST][POS_X][POS_Y][CHECKSUM]
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

1. **Temperatura vs Tiempo**
   - Serie temporal con media móvil
   - Alertas de umbrales críticos

2. **Humedad vs Tiempo**
   - Indicador de humedad relativa
   - Zona de confort destacada

3. **Distancia vs Tiempo**
   - Medición de distancia ultrasónica
   - Útil para detección de objetos

4. **Trayectoria Orbital 2D**
   - Visualización en plano X-Y
   - Rastro de posiciones anteriores
   - Posición actual destacada

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

### Versión Actual: 3.0 ✅

- [x] Captura y envío de posición satelital
- [x] Gráficas dinámicas 2D
- [x] Sistema de logs y observaciones
- [x] Comunicación LoRa con checksum
- [x] Cálculo de media de temperaturas

## 👥 Autores

### Grupo 5

- **Kirbyaeroespacial** - [GitHub](https://github.com/Kirbyaeroespacial)
- **rpraena** - [GitHub](https://github.com/rpraena)
- **Michail2007** - [GitHub](https://github.com/Michail2007)


<div align="center">

**⭐ Si este proyecto te fue útil, considera darle una estrella en GitHub ⭐**

Hecho con ❤️ por el Grupo 5

[⬆ Volver arriba](#-sistema-de-comunicación-satelital---grupo-5)

</div>





