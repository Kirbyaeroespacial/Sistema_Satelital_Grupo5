# Sistema_Satelital_Grupo5
Sistema de comunicación satélite-tierra con LoRa. Transmite datos de temperatura, humedad, distancia ultrasónica y posición orbital simulada. Incluye control de servo automático/manual, validación por checksum y visualización en tiempo real con Python (gráficos de sensores y trayectoria orbital).

# 🛸Codigo Satélite🛸
Este es el codigo actualizado del satélite en arduino hasta la fecha. En él consta todo el programa hasta la versión tres.

# Declaracion de puertos series y demás variables:
El prorama inicia con el siguiente código y se identifica los puertos 10 y 11 como los que envian y reciben información. Además de los pines 2 (LED de funcionamiento ), 3, 4 (sensor de distancia) y 5 (para el servomotor). Por otro lado también se declaran las variables iniciales como los ángulos iniciales, los tiempos de espera, los estados del LED, el TOKEN para enviar y/o recibir información o el checksum.
  ```bash
   #include <DHT.h> 
#include <SoftwareSerial.h>
#include <Servo.h>

#define DHTPIN 2
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

SoftwareSerial satSerial(10, 11); // RX=10, TX=11
const uint8_t LEDPIN = 12;
bool sending = false;
unsigned long lastSend = 0;
unsigned long sendPeriod = 20000UL;

const uint8_t servoPin = 5;
Servo motor;

const uint8_t trigPin = 3;
const uint8_t echoPin = 4;
const unsigned long PULSE_TIMEOUT_US = 30000UL;

bool autoDistance = true;
int servoAngle = 90;
int servoDir = 1;
int manualTargetAngle = 90;

const int SERVO_STEP = 2;
const unsigned long SERVO_MOVE_INTERVAL = 40;
unsigned long lastServoMove = 0;

bool ledState = false;
unsigned long ledTimer = 0;

// Sistema de turnos - CORREGIDO: inicia con permiso
bool canTransmit = true;
unsigned long lastTokenTime = 0;
const unsigned long TOKEN_TIMEOUT = 6000; // CORREGIDO: de 8000 a 6000

// Checksum
int corruptedCommands = 0;

// Media móvil temperatura
#define TEMP_HISTORY 10
float tempHistory[TEMP_HISTORY];
int tempIndex = 0;
bool tempFilled = false;
float tempMedia = 0.0;
float medias[3] = {0, 0, 0};
int mediaIndex = 0;

// === SIMULACIÓN ORBITAL ===
const double G = 6.67430e-11;
const double M = 5.97219e24;
const double R_EARTH = 6371000;
const double ALTITUDE = 400000;
const double TIME_COMPRESSION = 90.0;
double real_orbital_period;
double r;
unsigned long orbitStartTime = 0;
   ```

# Función checksum:
Esta función se ha incorporado recientemente en la versión 3. Lo que realiza es el cálculo del checksum del mensaje que se envía a la estación de tierra con la función _calcChecksum_ y para tal de poder enviarla la función _sendPacketWithChecksum_
  ```bash
String calcChecksum(const String &msg) {
  uint8_t xorSum = 0;
  for (unsigned int i = 0; i < msg.length(); i++) {
    xorSum ^= msg[i];
  }
  String hex = String(xorSum, HEX);
  hex.toUpperCase();
  if (hex.length() == 1) hex = "0" + hex;
  return hex;
}

void sendPacketWithChecksum(uint8_t type, const String &payload) {
  String msg = String(type) + ":" + payload;
  String chk = calcChecksum(msg);
  String fullMsg = msg + "*" + chk;
  satSerial.println(fullMsg);
  Serial.println("-> " + fullMsg);
}
  ```
Más adelante en el código (no al siguiente), se encuentra la función _validateAndHandle_ de manera que si el checksum enviado con corresponde con el que devería el mensaje se descara ya que es considerado un mensaje corrupto.
  ``` bash
void validateAndHandle(const String &data) {
    int asterisco = data.indexOf('*');
    if (asterisco == -1) {
        Serial.println("CMD sin checksum, descartado");
        corruptedCommands++;
        return;
    }
    
    String msg = data.substring(0, asterisco);
    String chkRecv = data.substring(asterisco + 1);
    String chkCalc = calcChecksum(msg);
    
    if (chkRecv == chkCalc) {
        handleCommand(msg);
    } else {
        Serial.println("CMD corrupto, descartado");
        corruptedCommands++;
    }
}
```
# Protocolo de aplicación satélite
