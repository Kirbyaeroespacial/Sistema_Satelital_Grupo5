# Sistema_Satelital_Grupo5
Sistema de comunicación satélite-tierra con LoRa. Transmite datos de temperatura, humedad, distancia ultrasónica y posición orbital simulada. Incluye control de servo automático/manual, validación por checksum y visualización en tiempo real con Python (gráficos de sensores y trayectoria orbital).

# 🛸Código Satélite🛸
Este es el codigo actualizado del satélite en arduino hasta la fecha. En él consta todo el programa hasta la versión tres.

# Declaración de puertos series y demás variables:
El prorama inicia con el siguiente código y se identifica los puertos 10 y 11 como los que envían y reciben información. Además de los pines 2 (LED de funcionamiento ), 3, 4 (sensor de distancia) y 5 (para el servomotor). Por otro lado también se declaran las variables iniciales como los ángulos iniciales, los tiempos de espera, los estados del LED, el TOKEN para enviar y/o recibir información o el checksum.
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
# Sensor de distancia
En la siguiente fucnión se observa el fucnionamiento manual del servomotor a partir de un potenciómetro: 
  ``` bash
int pingSensor() {
    digitalWrite(trigPin, LOW);
    delayMicroseconds(4);
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);
    unsigned long dur = pulseIn(echoPin, HIGH, PULSE_TIMEOUT_US);
    if (dur == 0) return 0;
    return (int)(dur * 0.343 / 2.0);
}
```

# Protocolo de aplicación satélite
Este protocolo se lleva desarrollando desde la versión 2 y aun prosigue. En esta última versión dicho protocolo lo que hace es enviar un número al principio de tal manera que cuando se recibe identifica este número con un dato concreto. Por ejemplo al enviar _67_ es para el Token de enviar o recibir información. 
Este es el comando entero:
  _"1: "_: Recibe un número y es para enviar.
  _"2: "_: Para controlar manualmente el servo.
  _"3: "_: Para enviar o no.
  _"4: "_: Utilizado para el ángulo del sensor.
  _"5: "_: Para hacer el ángulo manualmente con el potenciómetro.
  
  ``` bash
void handleCommand(const String &cmd) {
    Serial.println("RX cmd: " + cmd);

    if (cmd == "67:1") {
        canTransmit = true;
        lastTokenTime = millis();
        return;
    } else if (cmd == "67:0") {
        canTransmit = false;
        return;
    }
    
    if (cmd.startsWith("1:")) {
        sendPeriod = max(200UL, cmd.substring(2).toInt());
        // confirm (opcional)
        sendPacketWithChecksum(67, "0"); // no alterar protocolo, solo ejemplo si quieres ack
    }
    else if (cmd.startsWith("2:")) {
        // comando para establecer ANGULO MANUAL (2:ang) — MODIF
        manualTargetAngle = constrain(cmd.substring(2).toInt(), 0, 180);
        Serial.println("Set manualTargetAngle via 2: -> " + String(manualTargetAngle));
        if (!autoDistance) {
            motor.write(manualTargetAngle);
            servoAngle = manualTargetAngle;
        }
        // Enviar confirmación/eco del nuevo ángulo para que ground/Python lo muestren. -- MODIF
        sendPacketWithChecksum(6, String(manualTargetAngle));
    }
    else if (cmd == "3:i" || cmd == "3:r") sending = true;
    else if (cmd == "3:p") sending = false;
    else if (cmd == "4:a") {
        autoDistance = true;
        Serial.println("Modo AUTO activado (4:a)");
        // confirmación del modo y del ángulo actual -- MODIF
        sendPacketWithChecksum(6, String(servoAngle));
    }
    else if (cmd == "4:m") { 
        autoDistance = false;
        motor.write(manualTargetAngle);
        servoAngle = manualTargetAngle;
        Serial.println("Modo MANUAL activado (4:m), ang -> " + String(manualTargetAngle));
        // confirmación del modeo manual y ángulo -- MODIF
        sendPacketWithChecksum(6, String(servoAngle));
    }
    else if (cmd.startsWith("5:")) {
        // comando para establecer ANGULO MANUAL (5:ang) — MODIF (acepta 5: también)
        int ang = constrain(cmd.substring(2).toInt(), 0, 180);
        manualTargetAngle = ang;
        Serial.println("Set manualTargetAngle via 5: -> " + String(manualTargetAngle));
        if (!autoDistance) servoAngle = manualTargetAngle;
        // confirmar ángulo recibido y aplicado
        sendPacketWithChecksum(6, String(manualTargetAngle));
    }
}
```

# Cálculo de la temperatura media
En esta función se hace el cálculo de la media de las últimas 10 temperaturas registradas. En el caso es que el cálculo de tres medias consecutivas sea mayor que 100ºC se enviarà una alerta.
  ```bash
void updateTempMedia(float nuevaTemp) {
  tempHistory[tempIndex] = nuevaTemp;
  tempIndex = (tempIndex + 1) % TEMP_HISTORY;
  if (tempIndex == 0) tempFilled = true;

  int n = tempFilled ? TEMP_HISTORY : tempIndex;
  float suma = 0;
  for (int i = 0; i < n; i++) suma += tempHistory[i];
  tempMedia = suma / n;

  medias[mediaIndex] = tempMedia;
  mediaIndex = (mediaIndex + 1) % 3;

  bool alerta = true;
  for (int i = 0; i < 3; i++) {
    if (medias[i] <= 100.0) alerta = false;
  }
  if (alerta) sendPacketWithChecksum(8, "e");
}
```

# Simulación orbital
Esta función se ha incorporado en la posición de una hipotética órbita satelital con unas funciones y valores ya asumidos.
  ```bash
void simulate_orbit() {
    unsigned long currentMillis = millis();
    double time = ((currentMillis - orbitStartTime) / 1000.0) * TIME_COMPRESSION;
    double angle = 2.0 * PI * (time / real_orbital_period);
    
    long x = (long)(r * cos(angle));
    long y = (long)(r * sin(angle));
    long z = 0;
    
    String payload = String((long)time) + ":" + String(x) + ":" + String(y) + ":" + String(z);
    sendPacketWithChecksum(9, payload);
}
```
# Inicia el programa
Todo lo de antes son funciones para que el programa pueda realizar lo que nosotros queramos. Con este _setup_ empieza todo.
  ```bash
void setup() {
    Serial.begin(9600);
    satSerial.begin(9600);
    pinMode(LEDPIN, OUTPUT);
    digitalWrite(LEDPIN, LOW);
    pinMode(trigPin, OUTPUT);
    pinMode(echoPin, INPUT);
    dht.begin();
    motor.attach(servoPin);
    motor.write(servoAngle);
    lastTokenTime = millis();
    
    // Inicializar órbita
    r = R_EARTH + ALTITUDE;
    real_orbital_period = 2.0 * PI * sqrt(pow(r, 3) / (G * M));
    orbitStartTime = millis();
    
    Serial.println("SAT listo con orbital");
}
```
# Bucle del satélite
Este bucle ya esta dentro del programa y aprovechando las funciones declaradas previamentes, varaiables y el protocolo de aplicación hacemos lo siguiente:
· Envío distancia y ángulo.
· Validadicón para saber si se puede enviar o no (Token).
· Envío de temperatura y humedad, además de la media.
· La posición orbital
  ```bash
void loop() {
    unsigned long now = millis();
    
    // Servo automático
    if (autoDistance && now - lastServoMove >= SERVO_MOVE_INTERVAL) {
        lastServoMove = now;
        servoAngle += servoDir * SERVO_STEP;
        if (servoAngle >= 180) { servoAngle = 180; servoDir = -1; }
        else if (servoAngle <= 0) { servoAngle = 0; servoDir = 1; }
        motor.write(servoAngle);
    }

    // Leer comandos con validación (prioridad)
    if (satSerial.available()) {
        String cmd = satSerial.readStringUntil('\n');
        cmd.trim();
        if (cmd.length()) validateAndHandle(cmd);
    }

    // Recuperación por timeout
    if (!canTransmit && now - lastTokenTime > TOKEN_TIMEOUT) {
        canTransmit = true;
        Serial.println("Timeout: recuperando transmisión");
    }

    // Envío de datos
    if (now - lastSend >= sendPeriod) {
        if (sending && canTransmit) {
            // Sensores
            float h = dht.readHumidity();
            float t = dht.readTemperature();
            if (isnan(h) || isnan(t)) {
                sendPacketWithChecksum(4, "e:1");
            } else {
                sendPacketWithChecksum(1, String((int)(h * 100)) + ":" + String((int)(t * 100)));
                updateTempMedia(t);
                sendPacketWithChecksum(7, String((int)(tempMedia * 100)));
            }

            int dist = pingSensor();
            if (dist == 0) sendPacketWithChecksum(5, "e:1");
            else sendPacketWithChecksum(2, String(dist));

            if (!motor.attached()) sendPacketWithChecksum(6, "e:1");
            else sendPacketWithChecksum(6, String(servoAngle));

            // === POSICIÓN ORBITAL ===
            simulate_orbit();

            // Libera turno
            sendPacketWithChecksum(67, "0");
            canTransmit = false;
            
        }

        digitalWrite(LEDPIN, HIGH);
        ledTimer = now;
        ledState = true;
        lastSend = now;
    }

    if (ledState && now - ledTimer > 80) {
        digitalWrite(LEDPIN, LOW);
        ledState = false;
    }
}
```
