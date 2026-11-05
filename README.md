# ppp-2024-02-shm-pcb-altium

> **Red Sísmica del Austro (RSA) — Universidad de Cuenca**  
> Prácticas Preprofesionales (PPP) • Diseño de Hardware Electrónico, Sistemas Embebidos e Instrumentación Sísmica

---

## 📋 Ficha Técnica del Proyecto

| Campo | Detalle |
| :--- | :--- |
| **Código Institucional** | `RSA-PPP-2024-02` |
| **Nombre del Proyecto** | Sistema Distribuido de Monitorización de Salud Estructural (Serie Acelerógrafo SHM - Concentrador y Nodos Sensores) |
| **Pasante** | Jhonatan Paúl Cambisaca Sánchez (`jhonatan.cambisaca@ucuenca.edu.ec`) |
| **Carrera / Institución** | Ingeniería en Telecomunicaciones — Universidad de Cuenca |
| **Tutor Institucional** | Ing. Milton Muñoz (`milton.munozc@ucuenca.edu.ec`) — RSA |
| **Duración / Horas** | 96 horas (6 semanas) • Presencial |
| **Fecha de Ejecución** | 2024-02-08 al 2024-03-26 |
| **Estado** | `Culminado` |

---

## 🎯 Descripción General y Objetivos

La Red Sísmica del Austro (RSA) desarrolla estaciones acelerográficas y sistemas de monitorización de salud estructural (**SHM - Structural Health Monitoring**) destinados a la auscultación dinámica de obras civiles críticas (presas, puentes y edificaciones). La arquitectura del sistema se basa en un **Nodo Concentrador Principal** (que orquesta las comunicaciones y sincroniza la adquisición con un procesador central) y una red de **Nodos Sensores** periféricos gobernados por microcontroladores Microchip **dsPIC33EP** y acelerómetros triaxiales de ultra bajo ruido **ADXL355**.

En las versiones iniciales del hardware existían cuellos de botella severos: topología en estrella rudimentaria con mazos de cableado independientes, un error de inversión de pines en las líneas diferenciales A/B del transceptor MAX485 respecto a su datasheet, jumpers mecánicos propensos a desconexión por vibraciones y ausencia de un canal físico balanceado y permanente para la transmisión del pulso de sincronismo temporal.

Esta práctica preprofesional de 96 horas abordó el **rediseño completo y detallado a nivel de hardware y PCB en Altium Designer** para corregir integralmente estos circuitos, migrando la red a una topología en cascada (**Daisy Chain**) bajo norma de cableado estructurado **T-568B**, incorporando transceptores MAX485 dedicados en modo continuo para sincronismo de fase y generando dos variantes de PCB del nodo sensor (con plano de masa sólido y sin plano de tierra) para análisis comparativo de inmunidad ante ruido electromagnético.

### Objetivos Clave
1. **[Diagnóstico y Estandarización T-568B]:** Auditar esquemáticos previos, resolver el cruce de pines del MAX485 y normalizar el pinout RJ45 bajo estándar T-568B para datos, sincronismo y potencia.
2. **[Librerías Personalizadas EDA]:** Crear símbolos esquemáticos (`SchLib`) y huellas (`PcbLib`) con tolerancias mecánicas verificadas en Altium Designer.
3. **[Rediseño del Concentrador Principal]:** Eliminar jumpers manuales, incorporar transceptor MAX485 dedicado en transmisión permanente para sincronismo (`INT_SINC_1`) y rutar el inicio de la cadena Daisy Chain con terminación balanceada de 120 Ω.
4. **[Rediseño del Nodo Sensor con Enlace Pasante]:** Integrar conectores duales RJ45 (J2 entrada, J3 salida pasante), transceptor MAX485 en recepción continua para la interrupción externa del dsPIC y corregir la polarización del LED de diagnóstico `TEST1`.
5. **[Desarrollo de PCB Comparativo del Sensor]:** Diseñar el layout del PCB en dos variantes: **Versión A** con vertido de cobre (*Copper Pour*) en capas Top/Bottom y vías de costura (*via stitching*), y **Versión B** sin planos de tierra con trazas de retorno dedicadas.
6. **[Validación DRC y Expediente de Fabricación]:** Superar al 100% las reglas de diseño eléctrico (DRC), generar planos de manufactura y ensamble (`Job1.PDF`, `Job2.PDF`, archivos GerberX2) y compilar el Informe Técnico Final.

> 📄 Para consultar el cronograma detallado semana a semana y los checkpoints cuantificables, revisa el [Plan de Trabajo Oficial](docs/planificacion.md).

---

## 🏗️ Arquitectura del Sistema y Flujo de Señales

```text
+-----------------------------------------------------------------------------------------------------+
|                           SISTEMA DISTRIBUIDO SHM - TOPOLOGÍA DAISY CHAIN                           |
+-----------------------------------------------------------------------------------------------------+
|                                                                                                     |
|  +-----------------------------+                                                                    |
|  |   CONCENTRADOR PRINCIPAL    |                                                                    |
|  |  (Raspberry Pi + dsPIC33)   |                                                                    |
|  |                             |                                                                    |
|  |  [MAX485 Datos: A / B]      |                                                                    |
|  |  [MAX485 Sinc: SINC_A/B]    |                                                                    |
|  |  [Terminación 120 Ω (R2/R5)]|                                                                    |
|  +--------------+--------------+                                                                    |
|                 | (Puerto RJ45 T-568B)                                                              |
|                 v                                                                                   |
|  +-----------------------------+               +-----------------------------+                      |
|  |     NODO SENSOR 1 (SHM)     |               |     NODO SENSOR N (SHM)     |                      |
|  |                             |               |                             |                      |
|  | J2: Entrada RJ45            |               | J2: Entrada RJ45            |                      |
|  |  - Bus RS485 (ADXL355 Muest)| UTP T-568B    |  - Bus RS485 (ADXL355 Muest)|                      |
|  |  - RX Sinc (MAX485 -> INT)  | ------------> |  - RX Sinc (MAX485 -> INT)  |                      |
|  |  - dsPIC33EP Processing     |               |  - dsPIC33EP Processing     |                      |
|  | J3: Salida Pasante RJ45     |               | J3: Terminación de bus      |                      |
|  +-----------------------------+               +-----------------------------+                      |
+-----------------------------------------------------------------------------------------------------+
```

### 📌 Mapeo de Pines del Conector RJ45 (Norma T-568B)

| Par UTP | Pines RJ45 | Color T-568B | Señal | Función Técnica |
| :---: | :---: | :--- | :--- | :--- |
| **Par 2** | Pin 1 / Pin 2 | Blanco/Naranja – Naranja | `RS485_A` / `RS485_B` | Bus diferencial serie de datos bidireccional (Half Duplex). |
| **Par 3** | Pin 3 / Pin 6 | Blanco/Verde – Verde | `SINC_A` / `SINC_B` | Pulso diferencial de sincronismo temporal de alta velocidad (`INT_SINC`). |
| **Par 1** | Pin 4 / Pin 5 | Azul – Blanco/Azul | `+12V` / `VCC` | Línea de alimentación de continua distribuida hacia los sensores. |
| **Par 4** | Pin 7 / Pin 8 | Blanco/Marrón – Marrón | `GND` | Referencia común de masa y retorno de potencia. |

---

## 📂 Estructura del Repositorio

```text
ppp-2024-02-shm-pcb-altium/
├── altium/
│   ├── concentrador/                            # Proyecto Altium Concentrador Principal
│   │   ├── PCB_Project.PrjPcb                   # Archivo de proyecto Altium
│   │   ├── Sheet1.SchDoc                        # Esquemático depurado (sincronismo + MAX485)
│   │   ├── PCB1.PcbDoc                          # Layout PCB ruteado para conector T-568B
│   │   ├── imprimir.PcbDoc                      # Documento de impresión de pistas
│   │   ├── PcbLib1.PcbLib                       # Huellas de componentes (RJ45, SOIC, conectores)
│   │   ├── Schlib1.SchLib                       # Símbolos esquemáticos
│   │   ├── Job1.OutJob                          # Definición de salidas de fabricación
│   │   ├── PCB1.step                            # Modelo 3D consolidado del Concentrador
│   │   ├── Project Outputs for PCB_Project/     # Salidas de fabricación (GerberX2, reportes DRC)
│   │   └── Project Logs for PCB_Project/        # Historial de cambios y compilación ECO
│   │
│   └── nodo/                                    # Proyecto Altium Nodo Sensor SHM
│       ├── R-007 ACELEROGRAFO ADXL355 V5.PrjPcb # Archivo de proyecto Altium
│       ├── Esquema.SchDoc                       # Esquemático (puertos duales J2/J3, MAX485 sinc)
│       ├── PCB.PcbDoc                           # Versión A: Layout PCB con plano de tierra sólido
│       ├── fotolitoTop.PcbDoc                   # Versión B: Layout PCB sin plano de tierra / fotolito
│       ├── PcbLib1.PcbLib                       # Huellas verificadas del nodo sensor
│       ├── Schlib1.SchLib                       # Símbolos del dsPIC33, MAX485 y ADXL355
│       ├── Job2.OutJob                          # Configuración de salidas de fabricación
│       ├── Datasheet/                           # Hojas de datos técnicas de componentes críticos
│       ├── Librerias/                           # Bibliotecas complementarias (Libreria RSA, Propia)
│       └── Project Outputs for .../             # Reportes de DRC, archivos de taladrado y ensamble
│
├── docs/
│   ├── hardware/                                # Planos de ensamble, fotolitos y cortes CAD
│   │   ├── Job1_Concentrador.PDF                # Documento de ensamble oficial - Concentrador
│   │   ├── Job2_Nodo.PDF                        # Documento de ensamble oficial - Nodo Sensor
│   │   ├── Esquematico_PCB_Concentrador.pdf     # Planos esquemáticos y layout del Concentrador
│   │   ├── acelerometro_concetrador.pdf         # Diagramas auxiliares Concentrador
│   │   ├── Esquema nodo.pdf                     # Plano esquemático del Nodo Sensor
│   │   ├── acelerometro_nodo.pdf                # Diagramas auxiliares Nodo Sensor
│   │   ├── acelerografo(NoDo).pdf               # Resumen de conexiones del Nodo
│   │   ├── Fotolito.pdf                         # Fotolito consolidado
│   │   ├── fotolitoTOP.pdf                      # Fotolito de capa superior (Top Layer)
│   │   ├── fotolitoBOTTOM.pdf                   # Fotolito de capa inferior (Bottom Layer)
│   │   ├── cortes acrilico 5MM.dwg              # Planos mecánicos CAD para chasis de acrílico
│   │   └── PCB.DWG                              # Geometría CAD de placa
│   │
│   ├── informe_final/                           # Informe formal de culminación de prácticas
│   │   ├── InformeFinal_JhonatanCambisaca.pdf   # Versión aprobada en formato PDF
│   │   └── InformeFinal_JhonatanCambisaca.docx  # Documento editable original
│   │
│   └── planificacion.md                         # Plan de trabajo oficial institucional (96 h)
│
├── README.md                                    # Ficha técnica institucional del repositorio
└── .gitignore                                   # Reglas de exclusión para Altium, OS y temporales
```

---

## 🛠️ Herramientas de Desarrollo y Requisitos

* **Software EDA Principal:** Altium Designer (versión 20 o superior).
* **Visores de Salidas de Manufactura:**
  * Visor Gerber: Camtastic (integrado en Altium) o [Gerbv](https://gerbv.github.io/).
  * Visor Mecánico CAD: Autodesk DWG TrueView, AutoCAD o LibreCAD (para los archivos `.dwg` de corte de acrílico).
  * Visor 3D: FreeCAD o Altium 3D Viewer (para el modelo `PCB1.step`).
* **Instrumental de Laboratorio Asociado:**
  * Osciloscopio digital con decodificación RS485 para comprobación de tiempos de sincronismo.
  * Multímetro de precisión para verificación de caídas de tensión sobre líneas de alimentación UTP.

---

## 🚀 Guía de Uso de los Diseños Electrónicos

### 1. Apertura del Concentrador
1. Abrir Altium Designer.
2. Ir a `File > Open > Project` y seleccionar `altium/concentrador/PCB_Project.PrjPcb`.
3. Compilar el proyecto para verificar que el Electrical Rule Check (ERC) y Design Rule Check (DRC) reporten 0 violaciones.
4. Generar o visualizar la documentación de fabricación mediante `altium/concentrador/Job1.OutJob`.

### 2. Apertura del Nodo Sensor (Versión con planos y sin planos)
1. Abrir Altium Designer.
2. Seleccionar `altium/nodo/R-007 ACELEROGRAFO ADXL355 V5.PrjPcb`.
3. Para la versión con planos de tierra sólidos (**Versión A**), abrir `PCB.PcbDoc`.
4. Para la versión sin planos de tierra (**Versión B / Fotolito**), abrir `fotolitoTop.PcbDoc`.
5. Consultar los planos de ensamble y corte en `docs/hardware/Job2_Nodo.PDF` y `cortes acrilico 5MM.dwg`.

---

## 🛡️ Reglas de Trabajo y Control de Versiones

1. **Rama Principal:** La rama activa de trabajo es **`main`**.
2. **Formato de Commits:** Utilizar la convención estándar en minúsculas:
   - `feat: [nueva característica o módulo incorporado]`
   - `fix: [corrección de bug, pista o esquemático]`
   - `docs: [actualización de documentación o planos]`
   - `refactor: [reorganización de layout o componentes sin alterar netlist]`
3. **Control de Hitos por Git Tags:** Al culminar cada una de las 6 fases del cronograma, se utiliza una etiqueta fija (*tag*):
   ```bash
   git tag -a v0.1.0 -m "Fase 1 completada: Diagnostico previo, estandarizacion T-568B y librerias Altium"
   git push origin v0.1.0
   ```

   **Esquema de versionado para los tags:**
   * **Segundo dígito (`v0.x.0`):** Representa cada fase del plan de trabajo (`v0.1.0` para Fase 1, `v0.2.0` para Fase 2, hasta `v0.6.0` para Fase 6).
   * **Tercer dígito (`v0.x.y`):** Correcciones menores tras la revisión del tutor (ej. `v0.1.1`).
   * **Primer dígito (`v1.0.0`):** Estado final aprobado y acreditado de la pasantía.

---

## 📞 Contacto y Soporte Institucional

* **Pasante:** Jhonatan Paúl Cambisaca Sánchez (`jhonatan.cambisaca@ucuenca.edu.ec`)
* **Tutor Institucional:** Ing. Milton Muñoz (`milton.munozc@ucuenca.edu.ec`)
* **Institución:** [Red Sísmica del Austro (RSA)](https://redsismicaaustro.github.io/RSA-Metodologias) — Universidad de Cuenca
