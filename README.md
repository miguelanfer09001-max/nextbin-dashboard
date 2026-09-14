# ♻️ NextBin — Smart IoT Recycling System

> **Sistema automatizado de recolección, validación y gamificación de residuos PET en entornos educativos.**

![ESP32](https://img.shields.io/badge/Hardware-ESP32-blue?style=for-the-badge&logo=espressif)
![Supabase](https://img.shields.io/badge/Backend-Supabase-green?style=for-the-badge&logo=supabase)
![GitHub Pages](https://img.shields.io/badge/Frontend-GitHub%20Pages-black?style=for-the-badge&logo=github)
![C++](https://img.shields.io/badge/Language-C++-00599C?style=for-the-badge&logo=c%2B%2B)

---

## 📌 Descripción del Proyecto

**NextBin** es un punto ecológico inteligente desarrollado para el **Liceo Francés Jean Du Plessis** (Gachancipá) como parte del programa de articulación ambiental *Green School*. 

El sistema utiliza un microcontrolador **ESP32** que valida el ingreso físico de botellas plásticas mediante un mecanismo anti-trampa de doble sensor ultrasónico, registra los puntos del estudiante en una base de datos **Supabase (PostgreSQL)** en tiempo real y sincroniza los datos con una interfaz web de estilo *Cyberpunk* alojada en **GitHub Pages**.

---

## 🚀 Características Principales

* **Doble Validación Ultrasónica (Anti-Fraude):** Un sensor mide aproximación exterior y un segundo sensor en la rampa interna verifica la caída real del envase antes de otorgar puntos.
* **Control de Capacidad:** Bloquea automáticamente la recepción si la caneca supera el límite de llenado.
* **Backend Serverless / RPC:** Consultas directas a Supabase mediante funciones almacenadas (`sumar_puntos`) con soporte para inserción y actualización (*UPSERT*).
* **Gamificación en Tiempo Real:** Dashboard web dinámico con suscripción por WebSockets (Realtime) que muestra la tabla de clasificación (*Leaderboard*) y tienda de recompensas.
* **Interfaz de Hardware Local:** Pantalla LCD 16x2 vía I2C, indicadores LED y alertas sonoras programadas.

---

## 🛠️ Arquitectura y Tecnologías
─────────────────┐      HTTP REST / RPC      ┌───────────────────┐
│   ESP32 DevKit  │ ────────────────────────> │ Supabase Database │
│  (C++ Firmware) │ <──────────────────────── │   (PostgreSQL)    │
└────────┬────────┘                           └─────────┬─────────┘
│                                              │
Sensores & Servos                               WebSockets (Realtime)
│                                              │
▼                                              ▼
┌─────────────────┐                           ┌───────────────────┐
│ Contenedor PET  │                           │   Dashboard Web   │
│  (Físico/Wokwi) │                           │  (GitHub Pages)   │
└─────────────────┘                           └───────────────────┘
## 📋 Mapeo de Pines (Hardware)

| Componente | Pin ESP32 | Protocolo / Función |
| :--- | :--- | :--- |
| **Sensor Ultrasónico 1 (Entrada)** | Trig: `5` / Echo: `18` | Detección de presencia exterior |
| **Sensor Ultrasónico 2 (Interior)** | Trig: `19` / Echo: `23` | Control de caída y nivel de capacidad |
| **Servomotor 1 / Servomotor 2** | `GPIO 13` / `GPIO 27` | Control PWM de compuerta dual |
| **LCD 16x2 I2C** | SDA: `GPIO 21` / SCL: `GPIO 22` | Comunicación I2C (Dirección `0x27`) |
| **LED Verde / LED Rojo** | `GPIO 2` / `GPIO 4` | Indicadores de estado de operabilidad |
| **Buzzer Pasivo** | `GPIO 14` | Retroalimentación sonora |

---

## 🗄️ Esquema de Base de Datos (SQL)

La lógica de atribución de puntos se gestiona mediante la siguiente función PL/pgSQL en Supabase:

```sql
CREATE OR REPLACE FUNCTION sumar_puntos(user_id text, puntos_sumar int)
RETURNS void AS $$ BEGIN   INSERT INTO estudiantes (id, nombre, botellas, puntos)   VALUES (user_id, user_id, 1, puntos_sumar)   ON CONFLICT (id)    DO UPDATE SET      botellas = estudiantes.botellas + 1,     puntos = estudiantes.puntos + puntos_sumar; END; $$ LANGUAGE plpgsql;

Escalabilidad Futura (Fase 2)[ ] Visión Artificial (ESP32-CAM): Clasificación óptica mediante modelos de IA en Edge Impulse para validar polímeros PET transparentes.[ ] Discriminación Densimétrica (HX711): Integración de celda de carga de precisión para rechazar residuos de papel/cartón ($<10\text{g}$) o envases con líquidos ($>100\text{g}$).[ ] Alimentación Autónoma: Módulo de carga solar TP4056 con batería Li-Ion 18650.

Desarrollo Hardware / Firmware / IoT: Miguel Ángel Fernández Gutiérrez

Equipo Collaborador: Thomas Niño, Juan José Ordóñez, Ader Parra, Juan Esteban Rivera, Marlon Sacristán.

Director de Proyecto: Prof. Dairo Ferney Gómez Cortés

Institución: Liceo Francés Jean Du Plessis — Gachancipá, Cundinamarca.
