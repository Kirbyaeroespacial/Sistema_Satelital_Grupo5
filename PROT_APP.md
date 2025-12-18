# 📡 Protocolo de Aplicación
| ID | Protocolo | Datos                 | Descripción                                     |
| -- | --------- | --------------------- | ----------------------------------------------- |
| 1  | `prot1`   | `humedad:temperatura` | Sensor DHT (ej: `5023:2156` = 50.23%, 21.56 °C) |
| 2  | `prot2`   | `distancia_mm`        | Sensor ultrasónico (ej: `350` = 35 cm)          |
| 3  | `prot3`   | `estado`              | Estado de transmisión (`ok` / `error`)          |
| 4  | `prot4`   | `a/m`                 | Modo **AUTO** o **MANUAL**                      |
| 5  | `prot5`   | `ángulo`              | Posición actual del servo (0–180°)              |
| 6  | `prot6`   | `ángulo`              | Confirmación de movimiento del servo            |
| 7  | `prot7`   | `temp_media`          | Temperatura media acumulada                     |
| 8  | `prot8`   | `e`                   | Error o alarma del sistema                      |
| 9  | `prot9`   | `time:x:y:z`          | Datos orbitales (posición en metros)            |
| 10 | `prot10`  | `0/40/60/100`         | Estado del panel solar (% de apertura)          |
| 67 | `Token`   | `0/1`                 | Control de turnos (quién puede transmitir)      |
| 99 | `Stats`   | `num_errores`         | Reporte de mensajes corruptos                   |


# 🎮 Comandos de Control
| Comando     | Formato    | Acción                                  |
| ----------- | ---------- | --------------------------------------- |
| Mover servo | `5:ángulo` | Posicionar servo (0–180°)               |
| Modo AUTO   | `4:a`      | El satélite controla el servo           |
| Modo MANUAL | `4:m`      | El PC / Potenciómetro controla el servo |
