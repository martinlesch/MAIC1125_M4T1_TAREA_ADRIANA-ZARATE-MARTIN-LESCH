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
1. Ambiente: Se recomienda el uso de GPU T4 en Colab.
2. Acceso: Clonar este repositorio y abrir el notebook en la carpeta /notebooks.
3. Configuración: Añade tu API Key de Roboflow donde dice "TU-API-KEY".

Ejecuta las celdas secuencialmente para instalar dependencias y realizar la inferencia.

**A continuación se van a desarrollar y explicar el código paso a paso:**

Verificar que 
```bash
!nvidia-smi
```
