# 🦾 Brazo Robótico de 4 GDL: Arquitectura Maestro-Esclavo y Cinemática

## 📖 Contexto e Introducción
Este proyecto implementa el control, simulación y modelado matemático de un brazo robótico de 4 Grados de Libertad (GDL). El objetivo principal es establecer un puente entre la teoría de la robótica fundamental y su implementación escalable en software y hardware.

El sistema utiliza una arquitectura distribuida **Maestro-Esclavo** comunicada en tiempo real vía **WebSockets**. La inteligencia central y el cálculo intensivo residen en una Raspberry Pi 5, mientras que la ejecución a bajo nivel (control PWM de los motores) se delega a un microcontrolador ESP32.

Mediante un enfoque de **desacoplo cinemático**, el sistema resuelve la posición espacial $(x, y, z)$ utilizando las tres primeras articulaciones (base, hombro y codo) y controla la orientación de la herramienta mediante un eje de torsión en la muñeca ($\theta_4$).

---

## 🏛️ Arquitectura del Sistema

El proyecto está dividido en responsabilidades claras para maximizar la velocidad y permitir futuras integraciones de Inteligencia Artificial (ML/RL):

- **Maestro (Raspberry Pi 5):** Ejecuta un servidor web en **Flask**. Contiene el motor de cálculo cinemático (procesando matrices con NumPy) y expone la Interfaz de Usuario (UI).
- **Esclavo (ESP32):** Actúa como un cliente WebSocket (*listener*) que recibe tramas JSON con ángulos ya calculados y genera las señales de control físico sin latencia.
- **Comunicación:** Enlace *full-duplex* por WebSockets, eliminando el coste de las cabeceras HTTP tradicionales para lograr un control fluido y telemetría en tiempo real.

### Estructura del Repositorio
```text
Robotic-Arm/
├── Codigo/
│   ├── ESP32/                # Firmware C++ (Cliente WebSocket y control PWM)
│   ├── RaspberryPi5/         # Servidor Flask (Cinemática, WebSockets y API)
│   └── Simulation/           # UI Web e Interfaz Gráfica (HTML/JS/CSS)
├── Ensamble/                 # Archivos de diseño CAD
├── ML/                       # Modelos de Machine Learning y datasets
└── STL Files/                # Archivos para impresión 3D
```

---

## ⚙️ Análisis Cinemático

El "cerebro" del robot en la Raspberry Pi procesa las matemáticas usando operaciones vectorizadas para evitar latencias.

### Cinemática Directa
Determina la posición y orientación del efector final a partir de los ángulos actuales de los motores ($\theta_1, \theta_2, \theta_3, \theta_4$). Se implementa mediante matrices de transformación homogénea para proyectar las longitudes de los eslabones ($L_1, L_2$) desde la base ($H$) hasta la punta de la herramienta.

### Cinemática Inversa
Para alcanzar un punto objetivo $P(x,y,z)$ con una orientación de torsión $\psi$, aplicamos un análisis geométrico optimizado:

1. **Base ($\theta_1$):** Calculada mediante la proyección del punto en el plano horizontal $XY$.
   $$ \theta_1 = \text{atan2}(y, x) $$
   
2. **Plano Vertical (Hombro $\theta_2$ y Codo $\theta_3$):** Reducción del problema a un plano 2D. Utilizando la Ley de Cosenos y la altura de la base ($H$), calculamos la hipotenusa $D$ y los ángulos internos:
   $$ \theta_3 = \pm \arccos\left( \frac{D^2 - L_1^2 - L_2^2}{2 \cdot L_1 \cdot L_2} \right) $$
   $$ \theta_2 = \alpha \pm \beta $$

3. **Orientación de Muñeca ($\theta_4$):** Al actuar como un eje de torsión, compensa la rotación de la base para alinear la pinza de forma precisa con el objetivo.
   $$ \theta_4 = \psi - \theta_1 $$

---

## 🛠️ Tecnologías y Software

- **Python (Raspberry Pi):**
  - **NumPy:** Motor matemático optimizado en C para resolver matrices y ecuaciones trigonométricas en microsegundos.
  - **Flask & Flask-SocketIO:** Servidor web, API REST y hub de WebSockets para comunicación asíncrona.
  - **Pandas / PyTorch:** (En la carpeta `ML/`) Para el registro de telemetría y futura implementación de algoritmos de Machine Learning.
- **C++ (ESP32):** Librerías de WebSockets y ArduinoJson para parsear los comandos del Maestro con máxima eficiencia.
- **Frontend (Web):** HTML/CSS y JavaScript nativo para la interfaz de control remoto.

---

## 🔩 Componentes Físicos y Archivos STL

### Hardware
- **Servidor / Maestro:** Raspberry Pi 5.
- **Controlador / Esclavo:** ESP32.
- **Actuadores:** 4 Servomotores o Motores Paso a Paso (Base, Hombro, Codo, Torsión).
- **Alimentación:** Fuente externa dimensionada para los motores (Tierras comunes con el ESP32).

### Diseño 3D (`/STL Files`)
Estructura optimizada para impresión 3D:
1. `base_pedestal.stl` (Incluye la altura $H$)
2. `eslabon_1.stl` (Longitud $L_1$)
3. `eslabon_2.stl` (Longitud $L_2$)
4. `muneca_torsion.stl` (Acople para el motor de $\theta_4$)
5. `gripper.stl` (Efector final)

---

## 🚀 Cómo empezar

### 1. Configurar el Esclavo (ESP32)
1. Abre el proyecto en PlatformIO o Arduino IDE (carpeta `Codigo/ESP32`).
2. Configura las credenciales de tu red Wi-Fi y la IP de la Raspberry Pi en `config.h`.
3. Compila y sube el firmware al ESP32.

### 2. Configurar el Maestro (Raspberry Pi 5)
1. Clona este repositorio en tu Raspberry Pi.
2. Instala las dependencias de Python:
   ```bash
   pip install numpy pandas flask flask-socketio
   ```
3. Inicia el servidor maestro:
   ```bash
   cd Codigo/RaspberryPi5
   python app.py
   ```
4. Abre tu navegador (desde cualquier dispositivo en la misma red) en `http://<IP_DE_LA_RASPBERRY>:5000` para iniciar el control.