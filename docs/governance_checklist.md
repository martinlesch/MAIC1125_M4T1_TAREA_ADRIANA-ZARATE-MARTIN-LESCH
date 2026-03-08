# Checklist de Gobernanza AECO
## 1. Procedencia de los Datos
* **Fuente:** Dataset público de Roboflow Universe - enlace: https://universe.roboflow.com/martins-workspace-7mzhv/maic_m4t3_tarea
* **Fecha de recolección:** Marzo 2026
* **Propietario:** Si bien las imágenes son libres de derechos, los propietarios de las mismas son múltiples y están detallados en el dataset de Roboflow Universe
* **Eficiencia de Almacenamiento:** El sistema solo almacena metadatos (clase, confianza, timestamp) y "recortes" (crops) de las detecciones confirmadas, evitando el almacenamiento masivo de video continuo innecesario.
* **Propósito Específico:** Los datos recolectados tienen como fin único la auditoría de costos de maquinaria; no se utilizan para el rastreo de productividad individual de los operarios.


## 2. Tratamiento de PII (Privacidad)
**Caras / Matrículas: Privacidad y Consentimiento** 
* Protección de Identidad: El dataset se enfoca exclusivamente en maquinaria pesada. Se ha aplicado un protocolo de exclusión de rostros y placas de vehículos particulares para cumplir con las normativas de privacidad en el entorno laboral.
* Consentimiento: El uso de imágenes capturadas en obra está sujeto a las cláusulas de los contratos de servicios, donde se especifica el monitoreo técnico para fines de auditoría y seguridad operativa.


## 3. Declaración de Riesgo
**Torregrúa (Activo de Costo Fijo)**
* **Falso Negativo de Alto Impacto (Omitir detección):** Si el modelo no detecta la permanencia de la grúa un día determinado, el reporte de auditoría indicará erróneamente que el equipo ha sido retirado. Esto conlleva la pérdida de trazabilidad sobre el costo de alquiler diario, permitiendo que el activo permanezca en sitio devengando gastos sin estar justificado por el cronograma de estructura.
* **Falso Positivo de Alto Impacto ("Alucinación"):** Si el modelo confunde un elemento estructural (como un andamio de gran escala o una antena) con una torregrúa, se generaría una falsa alarma de cobro. Esto obligaría a auditorías manuales innecesarias para desestimar un cobro de alquiler que nunca existió, restando credibilidad al sistema automatizado.

**Excavadora (Activo de Costo Variable)**
* **Falso Negativo de Alto Impacto (Omitir detección):** Si la excavadora está operando pero el modelo no la detecta (por oclusión o polvo), el registro de "horas-máquina" será menor al real. Esto resulta en una falsa sensación de ahorro o baja productividad, invalidando el sistema como herramienta de control de facturación frente al contratista.
* **Falso Positivo de Alto Impacto ("Alucinación"):** Identificar un camión o maquinaria estática como una excavadora activa inflaría artificialmente el reporte de horas trabajadas. El impacto directo es un sobrecosto financiero de $X/hora (costo variable) aceptado erróneamente por la gerencia de obra debido a una lectura de IA defectuosa.

**Notas de Mitigación:** *"Para mitigar estos riesgos, se recomienda establecer un umbral de confianza (conf \ge 0.5) y cruzar los datos de detección con una persistencia temporal (ej. la máquina debe aparecer en al menos 5 frames consecutivos) antes de registrar el evento en el log de auditoría."*

**Declaración de Limitaciones (Cuándo NO usar el modelo):**

Este modelo es una herramienta de soporte y **no debe ser la única fuente de verdad** en los siguientes escenarios:
* Condiciones Climáticas Extremas: En presencia de niebla densa, lluvia torrencial o ventiscas de polvo que reduzcan la visibilidad por debajo del 30%, la tasa de error aumenta significativamente.
* Oclusiones Severas: No debe utilizarse para certificar la ausencia de una máquina si el ángulo de visión está obstruido por estructuras permanentes de la obra.
* Entornos Nocturnos: Sin iluminación artificial adecuada, el modelo YOLOv8s pierde eficacia drásticamente, ya que no fue entrenado con imágenes térmicas o infrarrojas.


## 4. Humano en el Bucle
**Proceso de revisión:** Dado que las decisiones basadas en estas detecciones afectan directamente el presupuesto de obra, el sistema no opera de forma 100% autónoma. Se establece el siguiente protocolo de supervisión:
* Validación de Alertas: Cualquier detección de "Torregrúa" o "Excavadora" que genere una discrepancia superior al 15% respecto al parte de trabajo diario debe ser validada manualmente por el Jefe de Obra o el Auditor de Costos.
* Umbral de Auditoría: Las imágenes con un índice de confianza ($conf$) entre 0.25 y 0.50 se marcan automáticamente para "Revisión Humana". El operador debe confirmar si la detección es correcta o si se trata de un falso positivo (alucinación del modelo).
* Re-entrenamiento Continuo (Active Learning): Las capturas donde el modelo falló o tuvo dudas se etiquetan correctamente de forma manual y se reintegran al dataset original en Roboflow. Este proceso garantiza que el modelo aprenda de las condiciones específicas de esa obra (ej. nuevas oclusiones o ángulos de cámara).
* Auditoría Aleatoria: Se define un proceso de revisión manual sobre el 5% de las detecciones clasificadas como "seguras" (conf > 0.8) para asegurar que no existan sesgos sistemáticos en el modelo a largo plazo.


## 5. Licencia
* **Tipo:** Utiliza una licencia MIT y los derechos del dataset son públicos.
