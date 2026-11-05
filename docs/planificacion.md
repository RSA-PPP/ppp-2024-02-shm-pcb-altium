---
titulo: "Plan de Trabajo de Prácticas Laborales: Diseño Detallado de PCBs para Sistema de Monitorización de Salud Estructural (Concentrador y Nodos Sensores Daisy Chain)"
proyecto: "Sistema Distribuido de Monitorización de Salud Estructural (Serie Acelerógrafo SHM - Concentrador y Nodos Sensores)"
codigo_proyecto: "RSA-PPP-2024-02"
area_tematica: "Diseño de Hardware Electrónico, Sistemas Embebidos e Instrumentación Sísmica"
estado: "Culminado"
version: "1.0"
fecha_creacion: "2024-02-08"
fecha_actualizacion: "2024-03-26"

pasante:
  nombre: "Jhonatan Paúl Cambisaca Sánchez"
  cedula: "N/D"
  correo: "jhonatan.cambisaca@ucuenca.edu.ec"
  carrera: "Ingeniería en Telecomunicaciones"
  institucion: "Universidad de Cuenca"

tutoria:
  tutor_institucional: "Ing. Milton Muñoz. (Red Sísmica del Austro - RSA)"
  correo_tutor_institucional: "milton.munozc@ucuenca.edu.ec"

cronograma:
  duracion_total_horas: 96
  dedicacion_semanal_horas: 16
  duracion_semanas: 6
  fecha_inicio: "2024-02-08"
  fecha_fin_estimada: "2024-03-22"
  modalidad: "Presencial"

tecnologias:
  - "Software de Diseño Electrónico EDA: Altium Designer (Esquemáticos, PCB Layout, Editores de Librerías SchLib/PcbLib, DRC)"
  - "Transceptores y Comunicaciones Diferenciales: MAX485 (Transmisión y Recepción dedicada de pulso de sincronismo y datos RS485)"
  - "Arquitectura de Procesamiento: Microchip dsPIC (Serie dsPIC33EP) y Raspberry Pi"
  - "Sensores de Auscultación: Acelerómetro Triaxial de Bajo Ruido ADXL355"
  - "Interconexión y Cableado Estructurado: Conectores RJ45 hembra (8P8C), Cable UTP Categoría 5e/6 normalizado bajo T-568B"
  - "Topologías de Red Física: Topología en Cadena Daisy Chain (enlace serie punto a punto con nodo concentrador de inicio)"
  - "Técnicas de Ruteo PCB: Planos de tierra (Copper Pour Top/Bottom) vs. Ruteo sin plano de masa, terminación balanceada de línea (120 Ω)"

repositorio:
  url: "https://github.com/RSA-PPP/ppp-2024-02-shm-pcb-altium"
  rama_base: "main"
---

---

## 1. Antecedentes y Justificación Técnica

1. **Contexto del Sistema y Estado Inicial del Artefacto:** 
   La Red Sísmica del Austro (RSA) desarrolla estaciones acelerográficas y sistemas de monitorización de salud estructural (SHM) destinados a puentes, presas y edificaciones críticas. El sistema se basa en una arquitectura distribuida conformada por un **Nodo Concentrador Principal** (encargado de orquestar la red, sincronizar temporalmente y enlazar con un procesador central Raspberry Pi) y múltiples **Nodos Sensores** periféricos basados en el microcontrolador dsPIC33EP y acelerómetros triaxiales ADXL355. En las versiones preliminares del hardware, la interconexión física entre el concentrador y los sensores utilizaba una topología en estrella rudimentaria con jumpers mecánicos y conectores RJ45 individuales por cada sensor, lo cual aumentaba exponencialmente el volumen de cableado, la complejidad de instalación en obra civil y la susceptibilidad a ruido electromagnético.

2. **Limitaciones Críticas o Cuellos de Botella Identificados:** 
   * **Incompatibilidad con Topología Daisy Chain:** Las placas originales no permitían enlazar los sensores en serie (Daisy Chain). Cada nodo requería un tendido independiente de cable UTP hasta el concentrador, limitando la escalabilidad de la red a distancias mayores.
   * **Error de Mapeo de Pines en el Bus RS485:** En los diseños previos se detectó un intercambio erróneo entre las líneas diferenciales **A** y **B** en el transceptor MAX485 respecto a su *datasheet*, lo que causaba inversión de polaridad lógica e impedía la comunicación UART diferencial entre placas.
   * **Inexistencia de Canal Físico Dedicado de Sincronismo:** El pulso de sincronización compartía líneas o no contaba con un controlador diferencial de línea dedicado y permanentemente habilitado, originando jitter, latencia indeterminada y posibles reflexiones de señal al carecer de terminaciones balanceadas.
   * **Conectividad Manual Propensa a Fallas:** La presencia de jumpers de selección manual en el concentrador comprometía la confiabilidad del sistema ante vibraciones y requería manipulación física en campo. Asimismo, el LED de prueba `TEST1` en el nodo sensor presentaba problemas de polarización debido a la posición de su resistencia limitadora.
   * **Falta de Estandarización en Pines RJ45:** El conexionado físico no seguía la norma internacional de cableado estructurado T-568B, dificultando el uso de cables comerciales estandarizados.

3. **Desacoplamiento del Alcance y Propuesta de Solución:** 
   Esta pasantía de 96 horas se focaliza exclusivamente en la **actualización y diseño detallado a nivel de hardware (PCB en Altium Designer)** para corregir y normalizar de forma definitiva los circuitos impresos del sistema:
   * Rediseñar el **Nodo Concentrador** para operar como cabecera de la red Daisy Chain bajo estándar T-568B, eliminando jumpers, corrigiendo los pines del MAX485 e integrando un segundo transceptor MAX485 dedicado exclusivamente al envío continuo de pulsos de sincronismo con resistencias de terminación de 120 Ω.
   * Rediseñar el **Nodo Sensor** incorporando puertos duales RJ45 (J2 de entrada y J3 de salida pasante) para continuidad de cadena Daisy Chain, añadiendo el transceptor MAX485 receptor de sincronismo (modo escucha continua) y corrigiendo la polarización del LED de test.
   * Generar dos versiones comparativas del PCB para el Nodo Sensor: una versión con planos de tierra completos (Top y Bottom) y una versión sin planos, permitiendo evaluar el comportamiento ante ruido electromagnético en fases experimentales posteriores.

---

## 2. Objetivos del Plan de Trabajo

### 2.1. Objetivo General
Diseñar y actualizar detalladamente las placas de circuito impreso (PCB) para el Nodo Concentrador Principal y el Nodo Sensor del sistema de monitorización de salud estructural (SHM) en Altium Designer, migrando la comunicación física a una topología en cadena Daisy Chain normalizada bajo el estándar T-568B, incorporando transceptores MAX485 dedicados para sincronismo temporal y generando versiones con y sin planos de tierra.

### 2.2. Objetivos Específicos
1. Auditar los esquemáticos preexistentes y crear las librerías personalizadas de símbolos esquemáticos (`SchLib`) y huellas (`PcbLib`) en Altium Designer para los componentes corregidos.
2. Actualizar el diseño esquemático del Nodo Concentrador eliminando jumpers y conectores obsoletos, corrigiendo el cruce de pines A y B del MAX485 de datos e integrando un transceptor MAX485 dedicado para el pulso de sincronización (`INT_SINC_1`) en modo transmisión permanente.
3. Rutar y generar el layout final del PCB para el Nodo Concentrador, configurando el conector RJ45 de inicio de cadena Daisy Chain bajo la norma T-568B con resistencias de terminación de línea balanceadas (R2 y R5).
4. Actualizar el diseño esquemático del Nodo Sensor, incorporando un segundo puerto RJ45 (J2 entrada, J3 salida) para enlace Daisy Chain, añadiendo el transceptor MAX485 de sincronismo en modo escucha permanente hacia el dsPIC y corrigiendo la posición de la resistencia del LED `TEST1`.
5. Diseñar dos versiones completas de layout PCB para el Nodo Sensor: Versión A con plano de tierra sólido (Top y Bottom) y Versión B sin planos de masa, optimizando anchos de pista para alimentación y señales críticas.
6. Validar las reglas de diseño (DRC) de ambas placas y compilar el expediente técnico de fabricación que incluye planos de ensamble (`Job1.PDF`), empaquetado de archivos fuente (`ActualizacionAcelerografoADXL355.rar`) y el Informe Técnico Final de prácticas laborales.

---

## 3. Plan de Trabajo Detallado y Checkpoints Cuantificables

### Fase 1: Diagnóstico de Diseños Previos, Estandarización y Creación de Librerías
* **Duración:** 16 horas (Semana 1)
* **Objetivo:** Analizar los fallos del hardware previo, establecer el mapeo de pines para la norma T-568B y construir las librerías de componentes en Altium Designer.
* **Actividades técnicas:**
  1. Revisión exhaustiva del esquemático original del concentrador y nodos sensores; identificación de pines invertidos en transceptores MAX485 y circuitos de señalización.
  2. Definir el esquema de asignación de pines del conector RJ45 según la norma de cableado estructurado T-568B para los pares diferenciales:
     - Par 1: Datos RS485 (Líneas A / B).
     - Par 2: Señal de Sincronismo Diferencial (Líneas SINC_A / SINC_B).
     - Pares restantes: Alimentación continua (VCC / 12V) y Referencia común (GND).
  3. Crear y validar en Altium Designer los símbolos esquemáticos y huellas (footprints) con tolerancias mecánicas adecuadas para conectores RJ45 con apantallamiento, transceptores MAX485 SOIC-8/DIP y componentes pasivos.
  4. Configurar las reglas de diseño (Design Rules) preliminares en Altium (clearance mínimo de 0.254 mm, anchos de pista para potencia de 0.8–1.0 mm y para señales de 0.3–0.4 mm).
* **Checkpoints medibles para revisión:**
  - [ ] **CP 1.1:** Tabla de pines estandarizada T-568B aprobada para señales de datos, sincronismo y alimentación en topología Daisy Chain.
  - [ ] **CP 1.2:** Librería personalizada `SchLib` y `PcbLib` creada y verificada sin discrepancias entre pines lógicos y pads físicos.
  - [ ] **CP 1.3:** Reglas de diseño (DRC rules) configuradas y guardadas en el entorno Altium Designer.

---

### Fase 2: Rediseño Esquemático del Nodo Concentrador
* **Duración:** 16 horas (Semana 2)
* **Objetivo:** Modificar el circuito esquemático del Concentrador Principal eliminando conexiones obsoletas e incorporando la etapa de sincronización dedicada y corrección del bus RS485.
* **Actividades técnicas:**
  1. Eliminar los jumpers manuales y los conectores RJ45 individuales que correspondían al esquema en estrella previo.
  2. Implementar un indicador LED de estado como reemplazo visual de las funciones anteriormente configuradas por jumpers.
  3. Corregir el conexionado de los pines A (no inversor) y B (inversor) del transceptor MAX485 de datos para alinearlo rigurosamente con el datasheet del fabricante.
  4. Agregar un segundo circuito integrado MAX485 destinado al canal de sincronismo:
     - Conectar pines 2 ($\overline{\text{RE}}$) y 3 ($\text{DE}$) a $\text{VCC}$ para forzar el estado de transmisión permanente (*driver enabled*).
     - Conectar el pin 4 ($\text{DI}$) a la línea `INT_SINC_1` proveniente del microcontrolador dsPIC.
     - Conectar las salidas diferenciales (pines 6 y 7) a las líneas `SINC_A` y `SINC_B`.
  5. Mantener las resistencias de terminación de línea balanceada (R2 y R5 de 120 Ω) en paralelo entre líneas diferenciales.
  6. Conectar las señales resultantes al conector RJ45 de inicio de la cadena Daisy Chain según el mapeo T-568B.
* **Checkpoints medibles para revisión:**
  - [ ] **CP 2.1:** Esquemático del Concentrador actualizado sin advertencias de compilación en Altium (*Electrical Rule Check* aprobado).
  - [ ] **CP 2.2:** Etapa de sincronismo implementada con MAX485 en modo transmisión forzada y acoplada a `INT_SINC_1`.
  - [ ] **CP 2.3:** Circuito de LEDs de alerta y estado corregido sin presencia de jumpers mecánicos.

---

### Fase 3: Ruteo y Diseño Detallado de PCB del Nodo Concentrador
* **Duración:** 16 horas (Semana 3)
* **Objetivo:** Diseñar la placa de circuito impreso (PCB Layout) del Nodo Concentrador cumpliendo con directrices de compatibilidad electromagnética y aislamiento de pistas.
* **Actividades técnicas:**
  1. Transferir los cambios del esquemático al layout mediante Engineering Change Order (ECO) en Altium.
  2. Distribuir espacialmente los componentes (floorplanning): ubicar el conector RJ45 de salida en el borde de placa, agrupar los dos transceptores MAX485 cerca de sus resistencias de terminación y ubicar la interfaz del dsPIC/Raspberry Pi.
  3. Rutar las pistas diferenciales de datos (`RS485_A`/`RS485_B`) y de sincronismo (`SINC_A`/`SINC_B`) garantizando simetría y acoplamiento de trazas.
  4. Dimensionar pistas de alimentación para minimizar caídas de tensión a lo largo de la tarjeta.
  5. Verificación de reglas de diseño (DRC) del layout del Concentrador.
* **Checkpoints medibles para revisión:**
  - [ ] **CP 3.1:** Layout del Concentrador finalizado con todas las conexiones al 100% ruteadas (0 unrouted nets).
  - [ ] **CP 3.2:** Reporte de DRC en Altium con 0 violaciones de espaciado, cortocircuitos o agujeros no conectados.
  - [ ] **CP 3.3:** Modelo 3D y vista preliminar de ensamble visualizada sin colisiones mecánicas de conectores RJ45 o peinetas.

---

### Fase 4: Rediseño Esquemático del Nodo Sensor
* **Duración:** 16 horas (Semana 4)
* **Objetivo:** Actualizar el esquemático del Nodo Sensor para admitir topología Daisy Chain pasante, recepción de sincronismo diferencial y corrección del LED de prueba.
* **Actividades técnicas:**
  1. Incorporar dos conectores RJ45 (J2 y J3) configurados bajo T-568B:
     - J2: Conector de entrada procedente del concentrador o del sensor predecesor.
     - J3: Conector de salida pasante hacia el siguiente nodo de la cadena en serie.
     - Enlazar directamente las líneas de datos, sincronismo y alimentación entre J2 y J3.
  2. Incorporar el transceptor MAX485 de recepción de sincronismo:
     - Conectar los pines 2 ($\overline{\text{RE}}$) y 3 ($\text{DE}$) a $\text{GND}$ para mantener al receptor en estado activo continuo (*receiver enabled*).
     - Conectar el pin 1 ($\text{RO}$) a la línea `INT_SINC` conectada al pin de interrupción externa del microcontrolador dsPIC.
     - Conectar los pines 6 y 7 a las líneas diferenciales de sincronismo del bus Daisy Chain.
  3. Subsanar el circuito del LED de diagnóstico `TEST1`: reubicar la resistencia limitadora de corriente antes del ánodo del LED para asegurar una conmutación lógica fiable desde el microcontrolador.
  4. Verificar el desacoplo de alimentación y las conexiones de la interfaz SPI con el acelerómetro triaxial ADXL355.
* **Checkpoints medibles para revisión:**
  - [ ] **CP 4.1:** Esquemático del Nodo Sensor actualizado con doble conector RJ45 (J2 y J3) en conexión pasante Daisy Chain.
  - [ ] **CP 4.2:** Circuito receptor de sincronismo MAX485 acoplado correctamente a la interrupción del dsPIC.
  - [ ] **CP 4.3:** Circuito del LED `TEST1` corregido y compilación eléctrica (ERC) superada sin advertencias.

---

### Fase 5: Ruteo y Diseño de PCB del Nodo Sensor (Versión con y sin Plano de Tierra)
* **Duración:** 18 horas (Semana 5)
* **Objetivo:** Desarrollar dos versiones de placa de circuito impreso para el Nodo Sensor (con y sin plano de tierra) para análisis comparativo de integridad de señal y ruido.
* **Actividades técnicas:**
  1. Transferir el esquemático del sensor al entorno de diseño PCB en Altium Designer.
  2. Posicionar estratégicamente los conectores RJ45 J2 y J3 en caras opuestas o accesibles para facilitar el cableado lineal en campo.
  3. **Diseño Versión A (Con plano de tierra):**
     - Rutar pistas de señal y alimentación en doble cara.
     - Generar polígonos de cobre sólido (Copper Pour) conectados a GND en las capas Top y Bottom.
     - Colocar vías de costura (*via stitching*) para enlazar los planos de masa superior e inferior y reducir la impedancia de retorno.
  4. **Diseño Versión B (Sin plano de tierra):**
     - Rutar las pistas de retorno de GND como trazas individuales dedicadas sin vertido de polígonos de cobre, manteniendo idéntica ubicación de componentes y trazas de señal para un posterior contraste experimental.
  5. Ejecución de DRC exhaustivo en ambas versiones.
* **Checkpoints medibles para revisión:**
  - [ ] **CP 5.1:** PCB Layout Versión A (con planos de tierra Top/Bottom) 100% ruteado y con DRC aprobado.
  - [ ] **CP 5.2:** PCB Layout Versión B (sin planos de tierra) 100% ruteado y con DRC aprobado.
  - [ ] **CP 5.3:** Comparativa geométrica y de ruteo documentada para ambas versiones del nodo sensor.

---

### Fase 6: Validación de Salidas de Fabricación, Empaquetado e Informe Final
* **Duración:** 14 horas (Semana 6)
* **Objetivo:** Compilar los entregables técnicos de fabricación, estructurar los archivos fuente y redactar el Informe Técnico Final de prácticas laborales.
* **Actividades técnicas:**
  1. Generar la documentación de manufactura y planos de ensamble (`Job1.PDF`) desde Altium Designer.
  2. Compilar el archivo comprimido consolidado `ActualizacionAcelerografoADXL355.rar` con todos los proyectos `.PrjPcb`, esquemáticos `.SchDoc`, PCBs `.PcbDoc` y librerías `.SchLib`/`.PcbLib`.
  3. Redactar el documento final formal: `InformeFinal_JhonatanCambisaca.docx` y exportar a versión PDF, detallando la justificación de cambios, esquemas comparativos antes/después, asignación T-568B y conclusiones técnicas.
  4. Revisión técnica con el tutor institucional y suscripción de actas de acreditación de 96 horas de prácticas.
* **Checkpoints medibles para revisión:**
  - [ ] **CP 6.1:** Documentación de planos de ensamble (`Job1.PDF`) y paquete comprimido de diseño consolidado.
  - [ ] **CP 6.2:** Informe Técnico Final (`InformeFinal_JhonatanCambisaca.pdf/.docx`) revisado y aprobado por el tutor institucional de la RSA.

---

## 4. Resumen de Fases y Cronograma Semanal (96 Horas)

El trabajo se ejecutó en un lapso de **6 semanas** con una dedicación promedio de **16 horas semanales** (hasta 20 horas por semana según necesidad técnica):

| Fase | Semanas | Horas | Enfoque Principal |
| :---: | :---: | :---: | :--- |
| **Fase 1** | Semana 1 | **16 h** | Diagnóstico del hardware previo, definición de estándar T-568B y creación de librerías Altium. |
| **Fase 2** | Semana 2 | **16 h** | Rediseño esquemático del Concentrador (eliminación de jumpers, corrección MAX485 y sincronismo). |
| **Fase 3** | Semana 3 | **16 h** | Ruteo de pistas, layout PCB y verificación DRC del Nodo Concentrador. |
| **Fase 4** | Semana 4 | **16 h** | Rediseño esquemático del Nodo Sensor (puertos duales RJ45 Daisy Chain, receptor MAX485 y LED TEST1). |
| **Fase 5** | Semana 5 | **18 h** | Layout PCB del Nodo Sensor en dos variantes: Versión con planos GND y Versión sin planos. |
| **Fase 6** | Semana 6 | **14 h** | Planos de ensamble (`Job1.PDF`), empaquetado de archivos fuente y redacción del Informe Final. |
| **Total** | **6 Semanas** | **96 h** | **Planificación Global de Prácticas Laborales II** |

---

## 5. Entregables y Criterios de Éxito de la Pasantía

1. **Librerías de Componentes Personalizadas en Altium Designer:**
   * Archivo de librería esquemática (`.SchLib`) y de huellas (`.PcbLib`) con conectores RJ45 apantallados, MAX485, componentes discretos y microcontrolador dsPIC.
2. **Diseño PCB del Nodo Concentrador Principal:**
   * Esquemático depurado sin jumpers mecánicos y con etapa de sincronismo permanente (`INT_SINC_1`).
   * Placa de circuito impreso ruteada para conector RJ45 de inicio de cadena Daisy Chain bajo T-568B con resistencias de 120 Ω y 0 violaciones de DRC.
3. **Diseño PCB del Nodo Sensor (2 Versiones):**
   * Esquemático con dos conectores RJ45 pasantes (J2 entrada, J3 salida) y receptor MAX485 para `INT_SINC`.
   * **Versión 1:** PCB con planos de tierra continuos en capas Top y Bottom (*Copper Pour* con *via stitching*).
   * **Versión 2:** PCB sin planos de tierra, basada en trazas de retorno dedicadas para evaluación de ruido e inmunidad electromagnética.
4. **Planos y Documentación de Fabricación:**
   * Documento de fabricación y ensamble de componentes en formato PDF (`Job1.PDF`).
   * Archivo comprimido `ActualizacionAcelerografoADXL355.rar` conteniendo la totalidad de los proyectos de Altium Designer actualizados.
5. **Informe Técnico Final:**
   * Documento formal institucional (`InformeFinal_JhonatanCambisaca.pdf` / `.docx`) conteniendo la descripción detallada, esquemas comparativos y conclusiones de las prácticas preprofesionales.
