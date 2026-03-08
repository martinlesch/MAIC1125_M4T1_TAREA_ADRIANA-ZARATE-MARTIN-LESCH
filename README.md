# AI-Driven Construction Management: Detección de Activos con YOLOv8
### Este repositorio presenta una solución de visión artificial orientada al sector AECO (Architecture, Engineering, Construction & Operations). El objetivo es optimizar el control de costos operativos mediante la auditoría automática de maquinaria en obra.

## Problema AECO y Criterios de Éxito
En la gestión de proyectos de construcción, los costos de alquiler de maquinaria representan una de las partidas más variables y difíciles de auditar.
* **El Problema:** Existe una discrepancia constante entre las horas de trabajo reportadas manualmente y el uso real de las excavadoras (costo variable), así como una permanencia excesiva de torregrúas (costo fijo) fuera de sus etapas críticas de estructura.
* **Criterio de Éxito:** Implementar un sistema de registro automático de presencia para contrastar:
1. Torregrúa: Días de permanencia real vs. Plazos de contrato.
2. Excavadora: Horas de actividad detectada vs. Partes de trabajo manuales.

## Especificaciones del Dataset y Modelo
**Clases:** 
1. Torregrúa: Activo crítico de coste fijo diario.
2. Excavadora: Activo de coste variable por hora de operación.

**Dataset:** Gestionado en Roboflow con un split de 80% Train / 20% Valid

Enlace del dataset: https://universe.roboflow.com/martins-workspace-7mzhv/maic_m4t3_tarea (v7)

## Cómo reproducir (pasos en Colab)
### Este repositorio está diseñado para ejecutarse en Google Colab sin instalaciones locales.
1. Acceso: Clonar o abrir el notebook en la "Notebook Colab". Se puede pulsar el botón para ir directamente a Colab o bien copiar y pegar el código. 
2. Ambiente: Se recomienda el uso de GPU T4 en Colab.
3. Configuración: Añade tu API Key de Roboflow donde dice "TU-API-KEY".

Ejecuta las celdas secuencialmente para instalar dependencias y realizar la inferencia.

**A continuación se van a desarrollar y explicar el código paso a paso:**

Verificar GPU disponible
```bash
!nvidia-smi
```

Instalar dependencias
```bash
!pip install ultralytics roboflow
```

Importar librerías
```bash
from ultralytics import YOLO
from roboflow import Roboflow
import os
from IPython.display import display, Image
from IPython import display
display.clear_output()
```

Descargar dataset desde Roboflow - en este paso debes añadir tu API KEY
```bash
rf = Roboflow(api_key="TU-API-KEY")  # Añade tu API KEY
project = rf.workspace("martins-workspace-7mzhv").project("maic_m4t3_tarea")
version = project.version(7)
dataset = version.download("yolov8")
```

Entrenar el modelo
```bash
!yolo task=detect mode=train model=yolov8s.pt data={dataset.location}/data.yaml epochs=30 imgsz=640 batch=16
```

Validar el modelo entrenado
```bash
!yolo task=detect mode=val model=/content/runs/detect/train/weights/best.pt data={dataset.location}/data.yaml
```

Hacer predicciones sobre imágenes de validación
```bash
!yolo task=detect mode=predict model=/content/runs/detect/train/weights/best.pt source={dataset.location}/valid/images save=True
```

Mostrar imágenes con predicciones
```bash
import glob
from IPython.display import Image, display

for image_path in glob.glob('/content/runs/detect/predict/*.jpg'):
    display(Image(filename=image_path, height=600))
    print("\n")
```

Mostrar métricas del modelo
```bash
model = YOLO("/content/runs/detect/train/weights/best.pt")

metrics = model.val()

print("Precisión (P):        ", metrics.box.mp)
print("Recall (R):           ", metrics.box.mr)
print("mAP50:                ", metrics.box.map50)
print("mAP50-95:             ", metrics.box.map)
```

Visualizar matriz de confusión y curvas de entrenamiento
```bash
Image(filename='/content/runs/detect/train/confusion_matrix.png', width=600)
Image(filename='/content/runs/detect/train/results.png', width=600)
```

Subida de imagen desde archivos locales
```bash
from google.colab import files
from IPython.display import display, Image as IPImage
import os

print('📁 Seleccioná la imagen desde tu computadora...')
uploaded = files.upload()

imagen_path = list(uploaded.keys())[0]
print(f'✅ Imagen cargada: {imagen_path}')

print('\n🖼️ Imagen original:')
display(IPImage(imagen_path, width=600))
```

Correr el modelo con la imagen cargada y mostrar resultados en texo y en imagen
```bash
import glob

MODEL_PATH = '/content/runs/detect/train/weights/best.pt'
model = YOLO(MODEL_PATH)

print(f'🔍 Detectando objetos en: {imagen_path}')
results = model.predict(
    source=imagen_path,
    save=True,          # Guarda la imagen con los bounding boxes
    conf=0.25,          # Umbral de confianza (ajustá según necesites)
    iou=0.45,           # Umbral IoU para NMS
    show_labels=True,
    show_conf=True
)

print('\n📊 Resultados de detección:')
for r in results:
    boxes = r.boxes
    if boxes is not None and len(boxes) > 0:
        print(f'  → Objetos detectados: {len(boxes)}')
        for box in boxes:
            cls_id = int(box.cls[0])
            conf   = float(box.conf[0])
            label  = model.names[cls_id]
            print(f'     • {label}: {conf:.1%} de confianza')
    else:
        print('  ⚠️ No se detectaron objetos con el umbral actual.')
        print('     Probá bajando conf= a 0.10 o 0.15')

predict_dirs = sorted(glob.glob('/content/runs/detect/predict*'), key=os.path.getmtime)
if predict_dirs:
    ultimo_dir = predict_dirs[-1]
    imagenes_resultado = glob.glob(os.path.join(ultimo_dir, '*.jpg')) + \
                         glob.glob(os.path.join(ultimo_dir, '*.png')) + \
                         glob.glob(os.path.join(ultimo_dir, '*.jpeg'))
    if imagenes_resultado:
        print(f'\n🖼️ Imagen con detecciones (guardada en: {ultimo_dir}):')
        display(IPImage(imagenes_resultado[0], width=600))
    else:
        print('No se encontró imagen de resultado en el directorio.')
else:
    print('No se encontró directorio de predicción.')
```

## Resumen de resultados (Análisis de métricas)

Basado en las curvas de entrenamiento ejecutadas:
* **Rendimiento General:** El modelo alcanzó un mAP50 cercano a 0.5 (50%) y un mAP50-95 de ~0.28. Aunque son valores iniciales para 30 épocas, la tendencia es claramente ascendente.
* **Precisión y Recall:** Se observa un Precision (B) de aproximadamente 0.8 (80%) al final del entrenamiento, lo que indica que cuando el modelo detecta una máquina, suele estar en lo correcto. El Recall es más bajo (~0.45), sugiriendo que aún omite algunas máquinas en entornos muy congestionados.
* 
Conclusiones Clave:
1. Convergencia Positiva: Las curvas de val/box_loss y val/cls_loss muestran un descenso sostenido, lo que indica que el modelo está aprendiendo y no hay un sobreajuste (overfitting) severo.
2. Estabilidad de Clase: La clase Torregrúa muestra mayor estabilidad por su morfología única, mientras que la Excavadora presenta retos por oclusiones con materiales de obra.
3. Potencial de Mejora: Dado que las métricas de precisión siguen subiendo en la época 30, un entrenamiento extendido a 100 épocas probablemente estabilizaría el Recall.

## Prueba de Reproducibilidad

* **Fecha y hora de la última ejecución exitosa:** 8:30 a.m. domingo, 8 de marzo de 2026 (GMT+1)
* **GPU usada:** GPU T4
* **Rango esperado de tiempo en ejecución:** Aproximadamente 4 minutos





