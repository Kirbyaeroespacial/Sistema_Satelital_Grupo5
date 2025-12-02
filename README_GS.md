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
