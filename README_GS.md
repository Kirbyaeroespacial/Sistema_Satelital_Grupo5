# 🌍Código Ground Station🌍
Este es el codigo actualizado del satélite en arduino hasta la fecha. En él consta todo el programa hasta la versión tres.

# Declaración de variables
Al principio se declaran los puertos seriales 10 y 11 como sucede en el satélite, el pin de error, el potenciómetro, el checksum y también el Token para poder enviar/recibir.
  ```bash
#include <SoftwareSerial.h>
SoftwareSerial mySerial(10, 11); // RX, TX

int errpin = 2;
char potent = A0;
unsigned long lastReceived = 0;
unsigned long last = 0;
const unsigned long timeout = 8000;
const unsigned long delay_ang = 1500;

bool satHasToken = false;
unsigned long lastTokenSent = 0;
const unsigned long TOKEN_CYCLE = 2500;

int corruptedFromSat = 0;
unsigned long lastStatsReport = 0;
const unsigned long STATS_INTERVAL = 10000;
```

# Checksum
Al igual que en el código de satélite, en el GroundStation también se calcula y se envía el cheskum con las funciones _calcChecksum_ y _sendWithChecksum_ además del _validateMessage_ para ver si el mensaje esta corrupto o no, es lo mismo que en el satélite.
``` bash
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

void sendWithChecksum(const String &msg) {
  String chk = calcChecksum(msg);
  String fullMsg = msg + "*" + chk;
  mySerial.println(fullMsg);
  Serial.println("GS-> " + fullMsg); // AÑADIDO: debug
}

bool validateMessage(const String &data, String &cleanMsg) {
  int asterisco = data.indexOf('*');
  if (asterisco == -1) return false;
  
  cleanMsg = data.substring(0, asterisco);
  String chkRecv = data.substring(asterisco + 1);
  String chkCalc = calcChecksum(cleanMsg);
  
  return (chkRecv == chkCalc);
}
```

# Protocolo de aplicación
El protocolo de aplicación también lo hemos aplicado al Ground Station para que el envío y procesamiento de datos sea una tarea mas sencilla.

  ```bash
void prot1(String valor) { Serial.println("1:" + valor); }
void prot2(String valor) { Serial.println("2:" + valor); }
void prot3(String valor) { Serial.println("3:" + valor); }
void prot4(String valor) { Serial.println("4:" + valor); }
void prot5(String valor) { Serial.println("5:" + valor); }
void prot6(String valor) { Serial.println("6:" + valor); }
void prot7(String valor) { Serial.println("7:" + valor); }
void prot8(String valor) { Serial.println("8:e"); }

// === PROTOCOLO ORBITAL (reformatea para Python) ===
void prot9(String valor) {
  int sep1 = valor.indexOf(':');
  int sep2 = valor.indexOf(':', sep1 + 1);
  int sep3 = valor.indexOf(':', sep2 + 1);
  
  if (sep1 > 0 && sep2 > 0 && sep3 > 0) {
    String x = valor.substring(sep1 + 1, sep2);
    String y = valor.substring(sep2 + 1, sep3);
    String z = valor.substring(sep3 + 1);
    
    Serial.print("Position: (X: ");
    Serial.print(x);
    Serial.print(" m, Y: ");
    Serial.print(y);
    Serial.print(" m, Z: ");
    Serial.print(z);
    Serial.println(" m)");
  }
}
```

# Inicio del programa
A partir de aquí inicia todas las funciones del GS, todo lo previo son funciones para poder llevar a cabo el funcionamiento.
  ```bash
void setup() {
  Serial.begin(9600);
  mySerial.begin(9600);
  Serial.println("COMM LISTO orbital");
  pinMode(errpin, OUTPUT);
  lastTokenSent = millis();
  lastStatsReport = millis();
  lastReceived = millis(); // AÑADIDO: evitar timeout inicial
}
```
# Bucle del Ground Station
A partir de ahora todo esto se hace constantemente para en el arduino de la estación de tierra. Lo primero que hace es la verificación del Token para poder enviar o no datos, luego hace la verifiacion del checksum y finalmente la opción de determinar el ángulo a partir de un potenciómetro.
Por otro lado también esta el protocolo de aplicacion y el encendido y apagado del pin en caso de error.
```bash
void loop() {
  unsigned long now = millis();

  // Gestión de turnos
  if (!satHasToken && now - lastTokenSent > TOKEN_CYCLE) {
    sendWithChecksum("67:1");
    satHasToken = true;
    lastTokenSent = now;
  }

  // Estadísticas cada 10s
  if (now - lastStatsReport > STATS_INTERVAL) {
    if (corruptedFromSat > 0) {
      Serial.println("99:" + String(corruptedFromSat));
      corruptedFromSat = 0;
    }
    lastStatsReport = now;
  }

  // CORREGIDO: Comandos de usuario con checksum
  if (Serial.available()) {
    String command = Serial.readStringUntil('\n');
    command.trim();
    if (command.length() > 0) {
      // Verificar si el comando YA tiene checksum (viene de Python corregido)
      if (command.indexOf('*') != -1) {
        // Ya tiene checksum, enviar directamente
        mySerial.println(command);
        Serial.println("GS-> " + command); // AÑADIDO: debug
      } else {
        // No tiene checksum, añadirlo (compatibilidad con comandos manuales)
        sendWithChecksum(command);
      }
    }
  }

  // Ángulo del potenciómetro con checksum (CORREGIDO: delay aumentado)
  if (now - last > delay_ang) {
    int potval = analogRead(potent);
    int angle = map(potval, 0, 1023, 180, 0);
    sendWithChecksum("5:" + String(angle));
    last = now;
  }
  
  // Recepción con validación
  if (mySerial.available()) {
    String data = mySerial.readStringUntil('\n');
    data.trim();

    if (data.length() > 0) {
      String cleanMsg;
      
      if (!validateMessage(data, cleanMsg)) {
        Serial.println("SAT-> CORRUPTO: " + data); // AÑADIDO: debug
        corruptedFromSat++;
        digitalWrite(errpin, HIGH);
        delay(100);
        digitalWrite(errpin, LOW);
        return;
      }
      
      Serial.println("SAT-> OK: " + cleanMsg); // AÑADIDO: debug
      
      int sepr = cleanMsg.indexOf(':');
      if (sepr > 0) {
        int id = cleanMsg.substring(0, sepr).toInt();
        String valor = cleanMsg.substring(sepr + 1);

        if (id == 67 && valor == "0") {
          satHasToken = false;
          lastTokenSent = now;
          return;
        }

        // Protocolo normal + orbital
        if (id == 1) prot1(valor);
        else if (id == 2) prot2(valor);
        else if (id == 3) prot3(valor);
        else if (id == 4) prot4(valor);
        else if (id == 5) prot5(valor);
        else if (id == 6) prot6(valor);
        else if (id == 7) prot7(valor);
        else if (id == 8) prot8(valor);
        else if (id == 9) prot9(valor);  // ORBITAL

        if (valor.startsWith("e")) {
          digitalWrite(errpin, HIGH);
          delay(500);
          digitalWrite(errpin, LOW);
        }
      }
      lastReceived = now;
    }
  }

  // Timeout
  if (now - lastReceived > timeout) {
    Serial.println("TIMEOUT: sin datos del satélite");
    digitalWrite(errpin, HIGH);
    delay(100);
    digitalWrite(errpin, LOW);
    delay(50);
    lastReceived = now; // AÑADIDO: evitar spam de timeout
  }
}
```

