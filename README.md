# Proyecto: Detección de Anomalías en Radiografías de Tórax

## Objetivo del trabajo

Desarrollar un análisis y un prototipo de clasificación multi-etiqueta para la detección automática de anomalías en radiografías de tórax. El objetivo incluye preprocesamiento, entrenamiento de modelos de visión por computador sobre una versión muestreada del dataset NIH Chest X-ray, y evaluación de métricas relevantes para clasificación multi-etiqueta.

---

## Alumnos participantes

- Coaguila Fuentes, Edison Jean Franco (U202213102)
- Aragón Ovalle, Alfredo Mauricio (U202210494)
- Turpo Queque, Joe Maicol (U202124254)

---
## Breve descripción del dataset

Se utiliza una versión muestreada del dataset "NIH Chest X-ray" (Random Sample of NIH Chest X-ray Dataset). El conjunto muestro contiene aproximadamente 5,606 imágenes en formato 1024x1024 junto con un archivo CSV de metadatos (`sample_labels.csv`) que incluye atributos como:

- Image Index: nombre del archivo de imagen
- Finding Labels: etiquetas de diagnóstico (14 patologías más "No Finding")
- Follow-up, Patient ID, Patient Age, Patient Gender, View Position, OriginalImageWidth/Height, OriginalImagePixelSpacing_x/_y

Las 14 patologías incluyen: Atelectasis, Cardiomegaly, Consolidation, Edema, Effusion, Emphysema, Fibrosis, Hernia, Infiltration, Mass, Nodule, Pleural_Thickening, Pneumonia y Pneumothorax. El dataset completo original comprende 112,120 radiografías provenientes de 30,805 pacientes; para este proyecto se usó la versión muestreada.

---

# Propuesta de modelización

El proyecto plantea un enfoque basado en:

🔹 **1. Transfer Learning**  
Se propone DenseNet121 debido a su éxito comprobado en tareas similares como CheXNet.  
También se evalúan ResNet50 y EfficientNetB0.

🔹 **2. Modelo multitarea multimodal**  
Para mejorar la sensibilidad del sistema, se plantea un modelo que combine:

- Características visuales de la radiografía (CNN)  
- Metadatos clínicos procesados (edad, sexo, vista)

Con **dos salidas simultáneas**:

- Clasificación binaria: Normal vs. Anormal  
- Clasificación multietiqueta: 14 patologías

🔹 **3. Métricas centradas en valor clínico**  
Las métricas priorizadas son:

- PR-AUC por clase
- Recall (minimizar falsos negativos)  
- F2-score (recall > precision)  
- AUC-ROC

---

## Modelización e implementación

Se implementaron dos modelos principales:

 **Modelo CNN 1**  
- DenseNet121 / ResNet50
- Salida Multietiqueta

 **Modelo Multitarea (imagen + metadatos)**  
- ResNet50 + red densa para metadatos 
- Salida binaria + multietiqueta

### Fases de entrenamiento

1. **Entrenamiento inicial**: Durante esta fase, el backbone del modelo se mantuvo congelado para preservar las representaciones generales aprendidas en tareas previas. Se realizaron 7 épocas de entrenamiento inicial, tras las cuales se evaluaron los distintos enfoques y se seleccionó el mejor modelo preliminar según el desempeño.

2. **Fine-tuning progresivo**: Una vez identificado el modelo base más prometedor, se procedió a descongelar entre el 93 % y el 95 % de sus capas para permitir una adaptación fina a los datos del dominio médico. Durante esta etapa, se utilizó un learning rate reducido y se incrementó gradualmente la complejidad del entrenamiento para evitar sobreajuste y garantizar una convergencia estable.

3. **Calibración de umbrales por clase**: Para maximizar la utilidad clínica del modelo, se realizó una calibración específica por patología. Esta optimización se basó en el F2-score y se apoyó en el análisis detallado de las curvas Precision–Recall. Cada umbral de decisión se ajustó de forma individual para balancear sensibilidad y especificidad.

### Artefactos generados

- `model.keras`  
- `multilabel_binarizer.joblib`  
- `age_scaler.joblib`  
- `optimal_thresholds.json`  
- `model_config.json`

---

## Optimización de hiperparámetros

Se empleó **Keras Tuner – Bayesian Optimization**, ajustando:

- Learning rate  
- Neuronas densas  
- Weight decay  
- Dropout  
- Parámetros de fusión en el modelo multitarea

Los mejores modelos preliminares se reentrenaron completamente.

---

## Publicación y demo en HuggingFace

Se desarrolló una demo interactiva con Gradio. 

Puede visualizarse a través del siguiente enlace:

https://huggingface.co/spaces/u202210494/DMT_TF

---

## Resultados principales

**Mejor desempeño global: Modelo multitarea**

- Mayor recall por clase 
- Mejor interpretación de metadatos 
- Clasificador binario con recall = 0.9981

**Mejoras respecto al modelo inicial**

El primer prototipo tenía recalls muy bajos, varios en 0.00.  
Con el modelo final:

- Se mejoró  precisión y recall  
- Varias patologías críticas pasaron a ser detectables
- Se estabilizó el rendimiento en clases desbalanceadas

El uso de metadatos fue decisivo para mejorar la detección de:

- Atelectasis  
- Consolidation  
- Effusion  
- Infiltration  
- Mass

---

## Conclusiones

- Se construyó un pipeline completo de visión por computador aplicable a radiografías médicas.  
- El enfoque multimodal (imagen + metadatos) superó ampliamente al modelo basado solo en imágenes.  
- El fine-tuning profundo y la calibración de umbrales por clase fueron esenciales para aumentar el recall, métrica crítica en aplicaciones médicas.  
- El modelo final constituye un prototipo funcional, con una arquitectura adaptable y código modular.


## Licencia

Este trabajo se distribuye bajo la licencia MIT.
