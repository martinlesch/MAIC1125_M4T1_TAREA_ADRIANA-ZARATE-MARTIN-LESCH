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
1.  Ambiente: Se recomienda el uso de GPU T4 en Colab.
2. Acceso: Clonar este repositorio y abrir el notebook en la carpeta /notebooks.
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








