# Analisis de fallos 
Las imagenes están guardadas en las carpetas "falso_negativo" y "falso_positivo"

## Falsos Negativos
### Caso 1: Torregrúas adicionales en el sector central de la obra
**Detección Lograda:** 1 Torregrúa (Confianza: 95%)

1. Solapamiento Geométrico (Occlusion/Crowding): Las grúas omitidas se encuentran en una línea de visión muy cercana a la grúa detectada. En Computer Vision, cuando dos objetos de la misma clase están muy juntos, el algoritmo de Non-Maximum Suppression (NMS) puede interpretar erróneamente que los cuadros delimitadores pertenecen al mismo objeto y "suprimir" los de menor confianza.
2. Escala y Contraste: Las grúas del fondo tienen un grosor de estructura menor en píxeles debido a la distancia. Al estar contra un cielo nublado con tonos similares (gris/blanco), el modelo no logra extraer suficientes características (features) para confirmar la clase por encima del umbral de confianza establecido ($conf=0.25$).
3. Sesgo de Entrenamiento: Es probable que el dataset contenga más imágenes de grúas aisladas que de "bosques de grúas", lo que dificulta la detección de instancias múltiples en un espacio reducido.
   
*"Para este tipo de escenas, el auditor debe realizar una validación manual. Un fallo aquí resultaría en una subestimación del 66% de la capacidad instalada en obra, afectando gravemente el control de costos fijos por alquiler de maquinaria."*

### Caso 2: Detección múltiple de activos en condiciones de luz de atardecer/contraluz
**Falso Negativo (Omitido)** 

El modelo ignoró por completo la Torregrúa central (la que está más cerca de la estructura del edificio en construcción).
1. Contraluz y Pérdida de Textura: Al estar situada frente a una zona de nubes iluminadas, la estructura de celosía de la grúa pierde contraste. El modelo de CV depende de los gradientes de borde; aquí, la silueta se "funde" con el fondo.
2. Falta de Contexto Geométrico: A diferencia de las otras grúas que muestran el brazo largo (flecha) de forma horizontal y clara, esta grúa está en un ángulo donde su geometría se confunde con los pilares verticales del edificio en construcción.

**Falso Positivo / Error de Localización**

En la zona izquierda, el modelo genera dos cuadros delimitadores para la misma Torregrúa.
1. Fallo del NMS (Non-Maximum Suppression): El algoritmo encargado de limpiar cuadros duplicados falló porque las detecciones no se solapan lo suficiente o tienen una variación de clase/confianza que el modelo interpretó como dos objetos distintos (quizás confundiendo el mástil con la flecha).
2. Baja Confianza: Nota que los valores son bajos (0.43). Cuando la confianza es fronteriza, el modelo es inestable y tiende a "fragmentar" el objeto.

*"Este caso demuestra que el modelo YOLOv8s es sensible a las condiciones de iluminación extrema (Golden Hour). Para mitigar esto, se recomienda aplicar Aumentos de Datos (Augmentations) de tipo 'Blur' o 'Brightness contrast' durante el entrenamiento, y ajustar el parámetro IOU (Intersection over Union) en la inferencia para forzar la eliminación de cuadros duplicados en un mismo activo."*

### Caso 3: Falso Negativo por Escala y Falta de Contraste
**Situación:** Escena de obra con maquinaria operando bajo un puente

1. Ausencia de Color (Desafío Cromático): Al ser una imagen con una paleta de colores muy limitada (casi monocromática o desaturada), el modelo pierde una de sus características de detección más fuertes: el amarillo/naranja vibrante típico de la maquinaria CAT. Sin el contraste de color, la excavadora se "mimetiza" con las rocas y el terreno grisáceo.
2. Complejidad del Fondo (Clutter): La excavadora de la izquierda está situada directamente detrás de un acopio de rocas y junto a un pilar del puente. Las líneas verticales del pilar y la textura irregular de las piedras confunden al extractor de características de YOLO, impidiendo que reconozca la silueta de la máquina como un objeto independiente.
3. Puntos de Vista No Convencionales: La excavadora de primer plano muestra un ángulo lateral/trasero muy pegado al suelo. Si el dataset de entrenamiento tiene una predominancia de fotos de perfil completo o ángulos elevados, el modelo falla al procesar esta perspectiva específica donde el brazo hidráulico se cruza visualmente con la estructura del puente.

   
*"Este error subraya la importancia de incluir imágenes en escala de grises o con variaciones de saturación durante el entrenamiento (Data Augmentation). Un FN en este escenario implicaría que el sistema no registraría ninguna actividad en una zona de trabajo crítica, generando un vacío en la auditoría de horas-máquina."*

## Falsos positivos
### Caso 1: Falso Positivo por Fragmentación (Duplicidad)
**Detección de Torregrúa en entorno de obra congestionado con acopios**

1. Fragmentación del Objeto (Object Splitting): El modelo YOLOv8s logró identificar correctamente la parte superior de la grúa (flecha y contrapeso) con una confianza del 60%. Sin embargo, falló al consolidar el mástil vertical como parte del mismo objeto.
2. Confusión de Fondo (Context Clutter): El segundo cuadro delimitador (el de abajo, vacío) fue generado porque el modelo confundió las líneas verticales del mástil de la grúa con la estructura de andamios y encofrados que está justo detrás. Al no tener una textura sólida, el modelo interpretó erróneamente que la estructura de la grúa terminaba donde empezaba el desorden de la obra, creando una detección "fantasma" en el espacio inferior.
3. Baja Confianza y Estabilidad: Una confianza del 0.60 indica que el modelo no está totalmente seguro. En este rango de incertidumbre, es común que el extractor de características de YOLO genere detecciones inestables o fragmentadas si el objeto no está perfectamente aislado del fondo.

   
*"Este FP demuestra la necesidad de ajustar el parámetro de IOU (Intersection over Union) durante la inferencia para forzar al algoritmo de NMS a fusionar cuadros delimitadores que estén muy cercanos verticalmente sobre la misma clase. Un FP aquí duplicaría erróneamente el cobro diario por el mismo activo fijo."*

### Caso 2: Falso Positivo por Confusión de Patrón Estructural

1. Mimetismo Geométrico (Pattern Mimicry): El modelo YOLOv8s logró identificar correctamente la grúa real con una confianza aceptable del 72.8%. Sin embargo, falló al interpretar la fachada del edificio de la izquierda. Esta fachada está compuesta por líneas horizontales y verticales metálicas repetitivas que crean un patrón de celosía muy similar a la estructura del mástil de una torregrúa.
2. Ángulo de Visión No Convencional: La toma en contrapicado distorsiona las perspectivas. El modelo asocia las líneas convergentes del edificio con la geometría vertical de una grúa vista desde abajo, especialmente cuando hay una grúa real justo al lado para "confirmar" el contexto de "obra".
3. Baja Confianza y Frontera de Decisión: La detección falsa tiene una confianza muy baja (0.31). Esto indica que el modelo está en el límite de su umbral de decisión (seteadas en 0.25 en tu script). En este rango de incertidumbre, cualquier patrón visual vagamente familiar (como la celosía metálica contra el cielo) puede disparar una detección errónea si el modelo está sesgado a encontrar grúas en entornos de construcción.
   
*"Este FP subraya la necesidad de un Humano en el Bucle para validar detecciones con confianza inferior a 0.50. Una alucinación aquí activaría un protocolo de cobro de alquiler por un activo inexistente, resultando en un sobrecosto directo del 100% en la partida de grúas para esa jornada."*

### Caso 3: Falso Positivo por Confusión de Clase Semántica
**Escena de movimiento de tierras con múltiples equipos y alta presencia de polvo en suspensión**

1. Confusión de Activos Similares (Inter-class Similarity): El modelo identifica correctamente dos excavadoras reales (0.71 y 0.51), pero genera un Falso Positivo en el extremo izquierdo (0.26) sobre un camión grúa o pluma hidráulica.
Causa: El brazo extendido de la pluma comparte una firma visual (geometría lineal y mecánica) casi idéntica al brazo de una excavadora, especialmente cuando la resolución es baja debido a la distancia.
2. Degradación por Condiciones Ambientales (Polvo): El polvo en suspensión actúa como un filtro de ruido que suaviza los bordes de los objetos. Esto reduce la capacidad del modelo para distinguir texturas específicas (como las orugas de una excavadora frente a las ruedas de un camión), llevando al modelo a generalizar cualquier "brazo mecánico amarillo" como una EXCAVADORA.
3. Inestabilidad en el Umbral Crítico: La confianza de 0.26 está apenas por encima de tu límite de detección (0.25). Este es un "falso positivo débil" que demuestra que el modelo está intentando encontrar patrones donde la información visual es ambigua.
   
*"Este caso resalta el riesgo de contaminación de métricas de costo variable. Confundir un camión pluma con una excavadora inflaría artificialmente las horas de operación de movimiento de tierras. Se recomienda entrenar una clase adicional de 'Otros Equipos' para ayudar al modelo a aprender por contraste qué elementos NO pertenecen a las categorías críticas de Torregrúa y Excavadora."*
