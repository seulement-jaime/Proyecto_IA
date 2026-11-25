# Guion Maestro de Sustentación y Explicación de Código: Proyecto Final LEGO IA

**Curso:** Inteligencia Artificial — Universidad del Magdalena  
**Docente:** Ing. Gian Carlo Daraviña Peña, M.Sc.  
**Integrantes:** Julio Alfonso Curiel Guerra · Andy Amaya · Sebastian Ruiz · Jaime Ruiz · Luis Villalba  
**Archivos de apoyo:**
- Presentación PowerPoint: [Presentacion_Sustentacion_LEGO_IA_Mejorada.pptx](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Presentacion_Sustentacion_LEGO_IA_Mejorada.pptx)
- Presentación PDF: [Presentacion_Sustentacion_LEGO_IA_Mejorada.pdf](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Presentacion_Sustentacion_LEGO_IA_Mejorada.pdf)
- Script de Spyder: [Proyecto_Final_Spyder.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Proyecto_Final_Spyder.py)

---

> [!TIP]
> **Ficha Rápida de Cifras Clave (Memorizar en 2 minutos antes de exponer)**
> - **Total de imágenes del dataset:** `8.044` (`3.464` en *Base Images* de $640 \times 480\text{ px}$ + `4.580` en *Cropped Images* de tamaño estandarizado a $64 \times 64\text{ px}$).
> - **Total de clases:** `20` categorías (`9` familia **Brick** [bloques altos] + `11` familia **Plate** [placas planas]).
> - **Partición sin fuga de datos (*Data Leakage*):** Agrupada por `timestamp_id` (12 dígitos) sobre los `4.580` recortes $\rightarrow$ **70% Train (`3.206`)**, **15% Val (`687`)**, **15% Test (`687`)**.
> - **Calidad de Segmentación OpenCV (HSV Dual Otsu):** **Recall de píxel = `95,16%`**, **Pixel Accuracy = `81,92%`**, **Dice = `0,7011`**, **IoU = `0,6457`**.
> - **Vectores de características:** **Escenario 1 (Solo Forma) = `301` dim** (`6` geométricas + `7` Momentos de Hu + `288` HOG) vs. **Escenario 2 (Forma + Color) = `325` dim** (`301` + `24` histogramas HSV). La forma aporta **~90%** del poder predictivo; el color solo suma entre **`+1,02` y `+3,34` puntos**.
> - **Ranking de Exactitud (*Accuracy*) en Test (`687` muestras):**
>   1. **CNN (PyTorch):** **`95,63%`** (`657` aciertos / `30` errores, F1-Macro `95,55%`, entrenamiento `~40,5 min`, inferencia `11,94 ms`).
>   2. **SVM (Kernel RBF, $C=5,0$):** **`90,83%`** (F1-Macro `90,77%`, entrenamiento **`12,01 s`**, inferencia `1,27 ms`).
>   3. **Red Neuronal Densa (ANN / MLP):** **`89,23%`** (F1-Macro `88,99%`, entrenamiento `10,51 s`, inferencia **`0,009 ms`**).
>   4. **Perceptrón (One-vs-Rest):** **`76,27%`** (`1,89 s`).
>   5. **SGAdaline (SGD con pérdida MSE):** **`69,72%`** (`45,46 s`).
>   6. **Árbol de Decisión (CART Gini, profundidad 14):** **`67,83%`** (`1,48 s`, inferencia `0,001 ms`).

---

## PARTE 1: Guion Hablado Diapositiva por Diapositiva (1 a 18)

Si el profesor llama a cualquiera de los 5 a exponer desde el principio o desde cualquier diapositiva intermedia, este es el discurso exacto:

### Diapositiva 01 — Portada
> *"Buenos días, profesor Gian Carlo y compañeros. A continuación sustentaremos nuestro proyecto final de Inteligencia Artificial, titulado **Segmentación y Clasificación de Piezas LEGO mediante Machine Learning y Deep Learning**. El trabajo integra una etapa de Procesamiento Digital de Imágenes con OpenCV para localizar y aislar cada pieza, seguida de una evaluación experimental comparativa de los **6 algoritmos** estudiados en el curso sobre un catálogo de **20 categorías LEGO** y **8.044 imágenes**."*

### Diapositiva 02 — Resumen Ejecutivo y Síntesis de Hallazgos Clave
> *"En esta diapositiva resumimos los cuatro resultados principales de nuestra investigación:*
> 1. *Primero, auditamos las **8.044 imágenes** garantizando **0% de fuga de datos (Data Leakage)** gracias a una partición agrupada por el identificador de captura `timestamp_id` (70% entrenamiento, 15% validación y 15% prueba).*
> 2. *Segundo, diseñamos un algoritmo de segmentación en OpenCV denominado **HSV Dual Otsu**, que alcanza un **Recall de píxel del 95,16%**, reteniendo prácticamente toda la geometría física de cada pieza.*
> 3. *Tercero, demostramos experimentalmente que la **geometría y los tetones explican cerca del 90% del poder predictivo**, y que la **Red Neuronal Convolucional (CNN)** es el modelo ganador absoluto con **95,63% de exactitud** (`657` aciertos de `687` pruebas).*
> 4. *Cuarto, encontramos que el modelo **SVM con Kernel RBF** es la alternativa de mayor eficiencia computacional: alcanza **90,83% de exactitud** entrenándose en apenas **12,01 segundos**, reduciendo el tiempo de entrenamiento en un 99,5% respecto a la CNN."*

### Diapositiva 03 — Planteamiento del Problema, Justificación y Objetivos
> *"¿Cuál es el problema técnico que resolvemos? Cuando recibimos una fotografía de una pieza LEGO, clasificarla en una de las 20 categorías enfrenta cuatro dificultades visuales:*
> - ***Variación cromática severa:*** *Un mismo bloque, como un `Brick 2x4`, viene en azul, rojo, amarillo, verde, marrón o negro. El modelo no puede aprender que 'azul significa Brick 2x4'; debe reconocer la forma topológica y los tetones sin sesgo de color.*
> - ***Similitud geométrica entre clases vecinas:*** *Por ejemplo, diferenciar un `Plate 1x2` de un `Plate 1x3`, o un `Brick 2x4` (alto) de un `Plate 2x4` (plano) cuando la cámara toma la foto desde arriba.*
> - ***Perspectiva, rotación y brillo del plástico:*** *Las piezas están rotadas en cualquier ángulo y el plástico ABS genera reflejos especulares y sombras entre los tetones.*
> - ***Piezas en contacto:*** *Si dos piezas se tocan, un contorno normal las une como si fueran una sola.*
>
> *Por eso justificamos una **arquitectura desacoplada**: primero OpenCV segmenta y aísla la región de interés (ROI) quitando el fondo, y luego comparamos los 6 modelos supervisados a través de nuestros 7 objetivos específicos."*

### Diapositiva 04 — Estructura del Dataset `LEGO_BRICKS` y Auditoría de Calidad
> *"Nuestro dataset se divide en dos colecciones complementarias:*
> - ***Base Images (`3.464` fotografías de escena completa a $640 \times 480\text{ px}$):*** *Muestran la pieza sobre una mesa retroiluminada de tono cian claro y se utilizan para evaluar la segmentación con OpenCV y el pipeline completo.*
> - ***Cropped Images (`4.580` recortes centrados en las `20` categorías):*** *9 categorías de la familia **Brick** (ladrillos altos) y 11 categorías de la familia **Plate** (placas delgadas).*
> - *Como vemos en la **Figura 1** (arriba a la derecha), las 20 clases están bien balanceadas, con entre 188 y 329 muestras por categoría. Notamos en nuestra auditoría que 5 clases Brick (`1x1` a `2x2`) no tenían foto en `Base Images`, pero sí cuentan con `1.116` recortes en `Cropped Images`, por lo que los 6 clasificadores se entrenan sobre las 20 clases completas.*
> - *En la **Figura 2** (abajo a la derecha) vemos que las resoluciones originales de los recortes varían según el tamaño físico de la pieza (desde $64 \times 51\text{ px}$ en `Plate_1x1` hasta $232 \times 125\text{ px}$ en `Brick_2x4`), lo que justifica estandarizar todas las entradas a **$64 \times 64$ píxeles**. Además, verificamos con hashes MD5 que hay **0 imágenes corruptas y 0 duplicados exactos**."*

### Diapositiva 05 — Control de Fuga de Datos (*Data Leakage*) y Particionado
> *"Para que la evaluación sea científicamente honesta, evitamos el error de Data Leakage. En este dataset, cada foto de escena (`2_Brick_2x2_L_180708213430.jpg`) y su recorte (`1_Brick_2x2_L_180708213430.jpg`) comparten un código temporal de 12 dígitos llamado `timestamp_id` (`180708213430`).*
> *En `data_loader.py` agrupamos por `timestamp_id` de forma estratificada y dividimos los 4.580 recortes en:*
> - ***70% Entrenamiento (`3.206` muestras)***
> - ***15% Validación (`687` muestras)*** *para elegir hiperparámetros, kernels y aplicar Early Stopping*
> - ***15% Prueba (`687` muestras)*** *para la evaluación final.*
>
> *Además, el escalador **`StandardScaler`** ($z = (x - \mu) / \sigma$) se ajusta con `fit_transform` **únicamente sobre las 3.206 muestras de entrenamiento**, y luego solo aplica `transform` sobre validación y prueba."*

### Diapositiva 06 — Pipeline de Segmentación en OpenCV: HSV Dual Otsu y Morfología
> *"¿Cómo funciona nuestro segmentador en `hsv_segmenter.py`? Si usáramos un umbral simple en escala de grises o en BGR, fallaríamos porque las piezas amarillas o claras se confunden con el brillo de la mesa y las piezas negras tienen saturación baja. Por eso convertimos la imagen de **BGR a HSV** y aplicamos una estrategia **Dual Otsu**:*
> 1. ***Canal de Saturación ($S$):*** *Aplicamos el método automático de **Otsu**, que maximiza la varianza entre clases $\sigma_B^2(T)$. Como el fondo retroiluminado tiene baja saturación y las piezas de colores (rojo, azul, amarillo, verde) tienen alta saturación, $M_S$ captura todas las piezas cromáticas.*
> 2. ***Canal de Valor/Brillo ($V$):*** *Las piezas negras o grises oscuras absorben la luz ($V$ bajo). Aplicamos un umbral inverso ($V < 130$ y adaptativo respecto a la mediana del fondo) para capturar piezas oscuras y las paredes laterales 3D sombreadas de los Bricks ($M_V$).*
> 3. ***Fusión y Morfología:*** *Unimos ambas máscaras con un **OR lógico** ($M_{\text{raw}} = M_S \cup M_V$). Luego aplicamos una **Apertura morfológica $5 \times 5$** (erosión seguida de dilatación) para borrar puntos de ruido del fondo, y un **Cierre morfológico $9 \times 9$** (dilatación seguida de erosión, 2 iteraciones) para rellenar los huecos oscuros que se forman en los tetones superiores.*
> 4. ***Contornos y Recorte:*** *Con `cv2.findContours` filtramos contornos menores a $400\text{ px}^2$ y extraemos la caja delimitadora agregando un margen de seguridad (`margin = 15 a 22 px`) para no cortar las esquinas de la pieza."*

### Diapositiva 07 — Validación de Segmentación y Separación con Watershed
> *"Para medir con números qué tan buena es nuestra máscara, en `segmentation_evaluator.py` comparamos las máscaras sobre 35 pares de imágenes frente a Ground Truth (Tabla 2).*
> *Nuestro método **HSV Dual Otsu** logra un **Recall de píxel de 95,16%** y **Pixel Accuracy de 81,92%** (con Dice de `0,7011` e IoU de `0,6457`). En visión artificial para clasificación, un Recall del 95,16% es lo más importante porque garantiza que no estamos mutilando tetones ni bordes del ladrillo.*
> *A la derecha (**Figura 3**) evaluamos el algoritmo **Watershed** para cuando dos piezas se tocan: calculamos la **Transformada de Distancia Euclidiana** $D(x,y)$ y tomamos como semillas seguras los píxeles donde $D(x,y) > 0,45 \cdot D_{\max}$. Como muestra la figura, mientras el contorno estándar une las dos piezas en 1 solo bloque, Watershed las separa en 2 cajas independientes. Sin embargo, comprobamos que si se aplica Watershed a una pieza individual, los relieves de los tetones pueden fragmentarse; por eso se deja como opción activable solo cuando hay piezas en contacto."*

### Diapositiva 08 — Ingeniería de Características: Geometría, Hu, HOG y Color HSV
> *"Una vez tenemos el recorte de $64 \times 64$ píxeles, ¿cómo alimentamos a los 5 modelos clásicos y a la red neuronal densa? En `feature_extractor.py` construimos cuatro familias de descriptores:*
> 1. ***6 Características Geométricas Normalizadas:*** *Área normalizada, perímetro normalizado, relación de aspecto ($w/h$), extensión ($A / (w \cdot h)$), solidez ($A / A_{\text{convex\_hull}}$, clave para detectar piezas en forma de 'L' que tienen concavidad) y circularidad ($4\pi A / P^2$, clave para `Plate_1x1_Round`).*
> 2. ***7 Momentos Invariantes de Hu:*** *Invariantes a traslación, escala y rotación 2D, estabilizados con logaritmo con signo: $h_i = -\text{sign}(H_i)\log_{10}(|H_i| + 10^{-10})$.*
> 3. ***288 Descriptores HOG (Histograma de Gradientes Orientados):*** *Dividimos la imagen $64 \times 64$ en celdas de $16 \times 16\text{ px}$ ($4 \times 4$ celdas), agrupadas en bloques de $2 \times 2$ ($3 \times 3 = 9$ bloques) con 4 celdas por bloque y 8 orientaciones angulares ($9 \times 4 \times 8 = 288$ valores). El HOG es fundamental porque captura la cuadrícula repetitiva de los tetones cilíndricos y las rejillas (`Grill`).*
> 4. ***24 Histogramas de Color HSV:*** *8 bins para Matiz ($H$), 8 para Saturación ($S$) y 8 para Valor ($V$).*
>
> *Con esto definimos dos escenarios: **Escenario 1 (Solo Forma = 301 características)** y **Escenario 2 (Forma + Color = 325 características)**."*

### Diapositiva 09 — Experimento Forma vs. Color: La Geometría Define a la Pieza
> *"En la Tabla 3 y la **Figura 5** entrenamos los 5 modelos basados en descriptores bajo ambos escenarios. El resultado es contundente:*
> - *Usando **únicamente las 301 características de forma (sin nada de color)**, el SVM ya alcanza **89,81%** de exactitud y la Red Neuronal MLP **88,06%**.*
> - *Cuando añadimos las 24 variables de color (325 características), la ganancia es de apenas **`+1,02` puntos en SVM** (`90,83%`) y **`+1,17` puntos en MLP** (`89,23%`).*
> - *Esto demuestra científicamente que **cerca del 90% de la capacidad discriminativa proviene de la forma y los gradientes HOG de los tetones**, evitando que el sistema sufra de sesgo cromático."*

### Diapositiva 10 — Arquitectura y Sintonización de los 6 Modelos Evaluados
> *"En esta tabla resumimos la formulación matemática y los hiperparámetros óptimos de los **6 modelos del curso**:*
> 1. ***Perceptrón de Rosenblatt (`perceptron_model.py`):*** *Clasificador lineal por umbral en esquema One-vs-Rest (20 clasificadores binarios), con tasa $\eta_0 = 0,01$, regularización $L_2$ y calibración sigmoide para obtener probabilidades.*
> 2. ***SGAdaline (`sgadaline_model.py`):*** *Implementación propia desde cero en Python (`BinarySGAdaline` y `MulticlassSGAdaline`). A diferencia del Perceptrón que actualiza sobre el escalón discontinuo, Adaline actualiza sus pesos sobre la salida lineal continua $\mathbf{w}^\top \mathbf{x} + b$ minimizando el **Error Cuadrático Medio (MSE)** mediante Descenso de Gradiente Estocástico (SGD) muestra a muestra, con decaimiento de tasa $\eta_t = \eta_0 / (1 + 0,001 \cdot t)$ durante 60 épocas.*
> 3. ***Support Vector Machine — SVM (`svm_model.py`):*** *Maximiza el margen geométrico $2/\|\mathbf{w}\|$. Hicimos búsqueda en validación probando Kernel Lineal (`83,70%`), Polinómico grado 3 (`88,21%`) y **Kernel Gaussiano RBF con $C=5,0$**, que resultó ganador con **`90,83%`** en prueba.*
> 4. ***Árbol de Decisión CART (`decision_tree_model.py`):*** *Divide el espacio minimizando la **Impureza de Gini** ($I_G = 1 - \sum p_k^2$). Sintonizamos la profundidad máxima en validación (`max_depth = 14`, `min_samples_split = 4`) para evitar que memorice ruido.*
> 5. ***Red Neuronal Artificial Densa — ANN / MLP (`neural_net_model.py`):*** *Perceptrón multicapa en PyTorch con arquitectura `325 → 256 → 128 → 20`. Cada capa oculta incluye `BatchNorm1d`, activación `ReLU` y `Dropout(0,3)`, entrenada 35 épocas con `Adam` (`lr = 0,001`).*
> 6. ***Red Neuronal Convolucional — CNN (`cnn_model.py`):*** *Opera directamente sobre los píxeles RGB $64 \times 64 \times 3$ sin extraer HOG manual. Tiene 3 bloques convolucionales dobles (`32 → 64 → 128` filtros $3 \times 3$) con `BatchNorm2d`, `MaxPool2d`, `AdaptiveAvgPool2d(4×4)` y un clasificador denso de `256` neuronas con `Dropout(0,4)`."*

### Diapositiva 11 — Benchmark Comparativo General de los 6 Modelos
> *"En la Tabla 4 y la **Figura 6** comparamos los 6 modelos sobre los mismos **687 recortes de prueba**:*
> - ***1.º lugar — CNN:*** *`95,63%` de Accuracy y `95,55%` de F1-Macro. Es el modelo más preciso (+4,80 puntos sobre SVM), con un costo de entrenamiento en CPU de 40,5 minutos y latencia de 11,94 ms por imagen.*
> - ***2.º lugar — SVM Kernel RBF:*** *`90,83%` de Accuracy y `91,46%` de Precisión Macro, entrenándose en tan solo **12,01 segundos** (202 veces más rápido que la CNN) y con 1,27 ms de latencia.*
> - ***3.º lugar — Red Neuronal Densa (ANN/MLP):*** *`89,23%` de Accuracy en **10,51 segundos** de entrenamiento y la latencia más rápida entre los modelos de alta precisión: **9 microsegundos (`0,009 ms`) por muestra**.*
> - ***4.º, 5.º y 6.º lugar:*** *Perceptrón (`76,27%`), SGAdaline (`69,72%`) y Árbol de Decisión (`67,83%`)."*

### Diapositiva 12 — Análisis de Modelos Clásicos: Perceptrón, SGAdaline y Árbol CART
> *"¿Por qué los modelos lineales y el árbol se quedan entre 67% y 76%?*
> - *En **Perceptrón (`76,27%`)** y **SGAdaline (`69,72%`)**, trazar 20 hiperplanos planos en un problema donde las piezas rotan y cambian de perspectiva es insuficiente porque las clases forman variedades curvas no linealmente separables. Además, la pérdida cuadrática MSE de SGAdaline penaliza al cuadrado las variaciones de iluminación en los histogramas, mientras que el umbral del Perceptrón con regularización $L_2$ tolera mejor esos valores extremos.*
> - *En el **Árbol de Decisión CART (`67,83%`)**, los cortes son ortogonales (una sola variable por nodo, $x_j \le \theta$), lo que produce fronteras en forma de escalera. Sin embargo, el árbol nos da un resultado muy valioso en la **Figura 11** (abajo a la derecha): el gráfico de **Importancia de Características**, donde comprobamos que las métricas geométricas (extensión, aspecto, solidez) y las celdas centrales del HOG concentran más del **65% de la ganancia de información Gini**."*

### Diapositiva 13 — Modelos No Lineales de Alto Desempeño: SVM RBF y Red Densa (MLP)
> *"Cuando permitimos fronteras curvas no lineales sobre las 325 características, la exactitud sube cerca del 90%:*
> - *Abajo a la izquierda (**Figura 9**) vemos la Matriz de Confusión del **SVM con Kernel RBF (`90,83%`)**, donde el truco del kernel proyecta los 325 descriptores a un espacio de dimensión superior logrando una diagonal muy limpia.*
> - *Abajo a la derecha (**Figura 10**) vemos las curvas de aprendizaje de la **Red Neuronal Densa MLP (`89,23%`)**: a diferencia del sobreajuste que veíamos en clase cuando una red no se regulariza, aquí gracias a `BatchNorm1d`, `Dropout(0,3)` y `ReduceLROnPlateau`, la curva roja de pérdida de validación baja de `1,75` a `0,34` junto con la azul de entrenamiento sin divergir."*

### Diapositiva 14 — Red Neuronal Convolucional (CNN): Arquitectura y Convergencia (Clase 7)
> *"En la diapositiva 14 vemos en detalle nuestra **CNN ganadora (`95,63%`)**, donde aplicamos directamente lo aprendido en la **Clase 7 del curso**:*
> 1. ***Data Augmentation en línea:*** *Rotaciones aleatorias de $\pm 15^\circ$, reflejo horizontal (`RandomHorizontalFlip`) y variaciones de brillo y contraste (`ColorJitter`).*
> 2. ***3 Bloques Convolucionales Dobles:*** *El Bloque 1 aplica dos convoluciones de `32` filtros $3\times 3$ + `BatchNorm2d` + `ReLU` + `MaxPool2d` (reduciendo de $64\times 64$ a $32\times 32$); el Bloque 2 sube a `64` filtros ($16\times 16$); el Bloque 3 sube a `128` filtros ($8\times 8$) seguido de `AdaptiveAvgPool2d(4×4)`, generando `2.048` características espaciales.*
> 3. ***Regularización y Early Stopping por mejor validación:*** *Cabezal denso de `256` neuronas con `BatchNorm1d`, `Dropout(0,40)` y optimizador `AdamW` (`lr = 0,001` — recordando de la Clase 4 y 5 que un `lr = 0,1` haría divergir el gradiente dejando a la red adivinando al azar). Guardamos automáticamente los pesos de la época con mayor exactitud de validación.*
> *En la **Figura 7** vemos que `Train Loss` y `Val Loss` bajan juntas hasta `~0,15` sin sobreajuste."*

### Diapositiva 15 — Matriz de Confusión de la CNN y Diagnóstico de Errores (Clase 8)
> *"Aplicando el análisis de **Matriz de Confusión de la Clase 8**, en la **Figura 8** evaluamos las `687` imágenes de prueba en la CNN:*
> - *Logramos **`657` aciertos en la diagonal principal y solo `30` errores en total** en las 20 clases.*
> - *Varias clases alcanzan **100% de exactitud**, como `Brick_1x3`, `Plate_1x1_Round` y `Plate_1x2_Grill`.*
> - *¿Por qué ocurren los 30 errores restantes? Identificamos dos causas físicas:*
>   1. ***Longitudes contiguas en perspectiva oblicua (caso `Plate_1x4`):*** *De 33 piezas `Plate_1x4`, acierta 28; confunde 4 con `Plate_1x3` y 1 con `Plate_2x4` cuando la pieza está inclinada respecto a la cámara y sufre acortamiento visual.*
>   2. ***Vista cenital pura entre Brick y Plate:*** *Cuando la foto es tomada totalmente desde arriba a $90^\circ$, la pared lateral del `Brick` no se ve y su cara superior de tetones luce idéntica a un `Plate` del mismo tamaño."*

### Diapositiva 16 — Sistema Integrado End-to-End, Filtro de Piel YCrCb y Spyder IDE
> *"En el Capítulo 6 unimos todo en un pipeline End-to-End de 8 pasos (`pipeline.py` y `Proyecto_Final_Spyder.py`):*
> - *Arriba a la izquierda (**Figura 12**) vemos los 4 paneles generados automáticamente al ingresar una fotografía base: **(1) Fotografía Original**, **(2) Máscara Binaria OpenCV**, **(3) Pieza Aislada** y **(4) Detección con contorno verde, caja delimitadora y clasificación `Brick_2x2_L` con `98,0%` de confianza**.*
> - *Arriba a la derecha vemos una mejora clave para fotos tomadas fuera del laboratorio sosteniendo la pieza con los dedos: como la piel humana tiene saturación cromática que engañaría al umbral HSV, agregamos un **filtro de supresión de piel en el espacio YCrCb** ($Cr \in [133, 173], Cb \in [77, 127]$) que borra la mano de la máscara y aísla únicamente el ladrillo LEGO.*
> - *Todo este sistema está estructurado en **7 bloques `# %%` dentro de Spyder IDE** (`Proyecto_Final_Spyder.py`), permitiendo inspeccionar todas las matrices en el **Variable Explorer** y probar cualquier imagen en vivo con los **6 modelos al mismo tiempo**."*

### Diapositiva 17 — Limitaciones Identificadas y Hoja de Ruta (Trabajo Futuro)
> *"Con criterio ingenieril identificamos 5 limitaciones actuales y su solución en trabajo futuro:*
> 1. *Para fondos domésticos muy texturados (madera o alfombras), reemplazar la umbralización HSV por **segmentación de instancias con Deep Learning (YOLOv8-seg o Mask R-CNN)**.*
> 2. *Activar **Watershed** de forma condicional analizando puntos de concavidad para no fragmentar tetones en piezas sueltas.*
> 3. *Resolver la ambigüedad cenital entre Brick y Plate integrando **cámaras de profundidad RGB-D o estimación de pose 6D (DenseFusion)** para brazos robóticos.*
> 4. *Reducir la latencia de la CNN de 11,94 ms a menos de 2 ms aplicando **Transfer Learning con MobileNetV3 y cuantización INT8 en ONNX**.*
> 5. *Ampliar de 20 clases a un catálogo abierto mediante **Redes Siamesas (Open-Set Recognition)**."*

### Diapositiva 18 — Conclusiones Generales y Referencias Bibliográficas
> *"En conclusión:*
> 1. *Construimos y validamos un sistema completo de visión por computador sobre **8.044 imágenes y 20 clases LEGO** con **0% de Data Leakage**.*
> 2. *El segmentador **HSV Dual Otsu** alcanzó **95,16% de Recall**, conservando intacta la geometría de los bloques.*
> 3. *La **CNN** es el modelo de mayor exactitud (**`95,63%`**, F1-Macro `95,55%`), demostrando que aprender filtros convolucionales 2D supera a aplanar la imagen en descriptores manuales.*
> 4. *El **SVM con Kernel RBF** es el modelo de mejor relación costo-beneficio (**`90,83%` en `12,01 segundos`**).*
> 5. *Confirmamos que **la forma geométrica y los gradientes HOG aportan ~90% de la precisión**, haciendo al sistema robusto frente a cambios de color.*
> *Muchas gracias, profesor. Ahora podemos pasar a Spyder para mostrarle la ejecución del código en vivo."*

---

## PARTE 2: Cómo Explicar y Demostrar el Código en Spyder (`Proyecto_Final_Spyder.py`)

Si el profesor dice: **"Ábranme el código en Spyder y explíquenme cómo funciona"**, abres [Proyecto_Final_Spyder.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Proyecto_Final_Spyder.py) en Spyder 6. El archivo tiene **7 celdas `# %%`** (igual que los scripts que él hizo en clase). Aquí tienes qué hace cada celda y qué mostrarle en el **Variable Explorer** y en el panel **Plots**:

### Mapa de las 7 Celdas `# %%` en `Proyecto_Final_Spyder.py`

| Celda en Spyder | Líneas | Qué hace al presionar `Ctrl + Enter` | Qué mostrarle al Profesor en Spyder |
| :--- | :---: | :--- | :--- |
| **`# %% 1. Librerias y Configuracion del Dataset LEGO_BRICKS`** | [L18–L99](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Proyecto_Final_Spyder.py#L18-L99) | Conecta automáticamente la carpeta `LEGO_BRICKS` desde [config.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/config.py), carga `dataset_splits.json` (`3.206` train, `687` val, `687` test) y grafica una cuadrícula $4 \times 5$ con 1 imagen real de cada una de las 20 categorías. | En **Consola**: conteo de `3.206 / 687 / 687`.<br>En **Plots**: catálogo visual de las 20 clases LEGO (`class_names`). |
| **`# %% 2. Cargar Caracteristicas Estandarizadas`** | [L102–L124](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Proyecto_Final_Spyder.py#L102-L124) | Carga las matrices precalculadas y estandarizadas con `StandardScaler` de ambos escenarios (`features_shape_only.npz` de 301 col. y `features_shape_and_color.npz` de 325 col.). | En **Variable Explorer**: hacer doble clic en `X_train` `(3206, 325)`, `X_test` `(687, 325)`, `X_train_shape` `(3206, 301)` y `y_test` `(687,)`. |
| **`# %% 3. Cargar los 6 Modelos Entrenados y Evaluar sobre Test`** | [L127–L180](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Proyecto_Final_Spyder.py#L127-L180) | Carga desde `models/checkpoints/` los 6 modelos ya entrenados (`Perceptrón`, `SGAdaline`, `SVM`, `Árbol`, `ANN/MLP` y `CNN`), predice sobre `X_test` (`687` muestras) y calcula las 6 matrices de confusión $20 \times 20$. *(Nota: si `REENTRENAR_DESDE_CERO = True`, reentrena todo desde cero).* | En **Variable Explorer**: hacer doble clic en `df_comparacion_porcentaje` (tabla con las métricas de los 6 modelos) y en `cm_cnn` o `cm_svm` (matrices $20 \times 20$). |
| **`# %% 4. Graficas de Desempeño y Curvas de Aprendizaje`** | [L183–L203](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Proyecto_Final_Spyder.py#L183-L203) | Muestra en el panel **Plots** las curvas de pérdida y exactitud (`Train` vs `Val`) de la ANN y la CNN, la comparativa Forma vs Color y la importancia de variables del Árbol. | En **Plots**: señalar cómo `Train Loss` y `Val Loss` bajan juntas sin el *overfitting* que ocurría en la Clase 5. |
| **`# %% 5. Visualizar las Matrices de Confusion de los 6 Modelos`** | [L206–L227](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Proyecto_Final_Spyder.py#L206-L227) | Grafica en una sola figura $2 \times 3$ las **6 matrices de confusión** juntas para comparar visualmente cómo la diagonal se vuelve más nítida desde Árbol (`67,83%`) hasta CNN (`95,63%`). | En **Plots**: comparar la dispersión fuera de la diagonal del Árbol/Adaline frente a la diagonal limpia de SVM y CNN. |
| **`# %% 6. Prediccion sobre Lote de Imagenes de Prueba`** | [L230–L277](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Proyecto_Final_Spyder.py#L230-L277) | Toma 20 imágenes de prueba (1 de cada clase LEGO), calcula `probs = mod_cnn.predict_proba_single_image(...)`, extrae la clase con `np.argmax(probs)` y la confianza con `probs.max()`, imprime `Real vs Predicción vs Confianza` y grafica las 20 piezas (título verde = acierto, rojo = error). | En **Consola**: `19 de 20` aciertos con confianzas de `0.70` a `1.00`.<br>En **Plots**: cuadrícula con las 20 predicciones. |
| **`# %% 7. Subir / Probar Cualquier Imagen desde Spyder`** | [L280–L435](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/Proyecto_Final_Spyder.py#L280-L435) | Define `probar_imagen_en_spyder(...)`: toma cualquier foto (`Base Images`, `Cropped Images` o una foto elegida con ventana de Windows), la segmenta con OpenCV y la clasifica con **los 6 modelos al mismo tiempo**, mostrando 2 figuras de diagnóstico y guardando `df_prediccion_6_modelos`. | En **Plots**: Figura de 4 paneles de segmentación OpenCV + Figura con Recorte, Referencia del Catálogo, Top-5 de probabilidades y barras de los 6 modelos. |

---

### ¿En qué archivo está cada parte técnica si el profesor pide abrir un módulo específico?

Si el profesor pregunta *"¿Dónde programaron el Adaline / la CNN / la segmentación / el HOG?"*, abres el archivo correspondiente dentro de `Proyecto_IA/`:

1. **[segmentation/hsv_segmenter.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/segmentation/hsv_segmenter.py) (Segmentación OpenCV):**
   - `segment_hsv_otsu(img)` ([L34–L56](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/segmentation/hsv_segmenter.py#L34-L56)): Convierte `cv2.cvtColor(img, cv2.COLOR_BGR2HSV)`, aplica `cv2.THRESH_OTSU` al canal $S$, aplica `cv2.THRESH_BINARY_INV` al canal $V$ y los une con `cv2.bitwise_or(mask_s, mask_v)`.
   - `clean_mask(mask)` ([L134–L145](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/segmentation/hsv_segmenter.py#L134-L145)): Aplica `cv2.MORPH_OPEN` ($5 \times 5$) y `cv2.MORPH_CLOSE` ($9 \times 9$, 2 iteraciones).
   - `apply_watershed_separation(img, binary_mask)` ([L147–L177](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/segmentation/hsv_segmenter.py#L147-L177)): Calcula `cv2.distanceTransform(binary_mask, cv2.DIST_L2, 5)`, umbraliza a `0.45 * dist_transform.max()`, etiqueta con `cv2.connectedComponents` y ejecuta `cv2.watershed`.
   - `remove_skin_tones(img, mask)` ([L179–L189](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/segmentation/hsv_segmenter.py#L179-L189)): Convierte a `cv2.COLOR_BGR2YCrCb` y filtra piel humana en `Cr ∈ [133, 173], Cb ∈ [77, 127]`.

2. **[features/feature_extractor.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/features/feature_extractor.py) (Extracción de las 301 y 325 características + `StandardScaler`):**
   - `extract_shape_features(img_bgr)` ([L109–L173](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/features/feature_extractor.py#L109-L173)): Redimensiona a $64 \times 64$, calcula las 6 métricas geométricas (`area`, `perimeter`, `aspect_ratio`, `extent`, `solidity`, `circularity`), los 7 Momentos de Hu (`cv2.HuMoments(cv2.moments(c))`) y los 288 descriptores HOG (`skimage.feature.hog` con `orientations=8, pixels_per_cell=(16,16), cells_per_block=(2,2)`).
   - `extract_color_features(img_bgr)` ([L175–L194](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/features/feature_extractor.py#L175-L194)): Calcula `cv2.calcHist` con 8 bins para $H$, $S$ y $V$ ($8+8+8 = 24$).
   - `extract_and_scale_single(img_bgr)` ([L96–L107](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/features/feature_extractor.py#L96-L107)): Extrae el vector de 325 características y le aplica `scaler_all.transform(...)` para estandarizarlo con la media y desviación de entrenamiento.

3. **Carpeta `models/` (Los 6 Modelos):**
   - [models/perceptron_model.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/models/perceptron_model.py): `Perceptron(max_iter=2000, eta0=0.01, penalty='l2')` envuelto en `CalibratedClassifierCV`.
   - [models/sgadaline_model.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/models/sgadaline_model.py): Clase `BinarySGAdaline` programada paso a paso (`net_input = np.dot(xi, self.w_) + self.b_`, `error = target - net_input`, `self.w_ += eta * error * xi`, `epoch_loss += 0.5 * (error ** 2)`) y `MulticlassSGAdaline` que entrena 20 neuronas Adaline en esquema One-vs-Rest (+1 para la clase activa, -1 para el resto).
   - [models/svm_model.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/models/svm_model.py): `SVC(kernel='rbf', C=5.0, gamma='scale', probability=True)` y la función `tune_svm_kernels` que compara Lineal, Polinómico y RBF.
   - [models/decision_tree_model.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/models/decision_tree_model.py): `DecisionTreeClassifier(criterion='gini', max_depth=14, min_samples_split=4)` y gráfico de `feature_importances_`.
   - [models/neural_net_model.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/models/neural_net_model.py): Red `MLPNet` en PyTorch (`325 → 256 → 128 → 20`) con `nn.Linear`, `nn.BatchNorm1d`, `nn.ReLU`, `nn.Dropout(0.3)` y `CrossEntropyLoss`.
   - [models/cnn_model.py](file:///C:/Users/jaime/Documents/curso%20de%20IA/Agentes%20de%20IA/Proyecto_IA/models/cnn_model.py): Red `LEGOConvNet` con 3 bloques `ConvBlock` (`Conv2d(3×3) → BatchNorm2d → ReLU → Conv2d(3×3) → BatchNorm2d → ReLU → MaxPool2d(2×2)`), canales `3 → 32 → 64 → 128`, `AdaptiveAvgPool2d((4,4))`, `Flatten` (`128 × 4 × 4 = 2048`), `Linear(2048, 256)`, `Dropout(0.4)` y `Linear(256, 20)`.

---

## PARTE 3: Banco de Preguntas del Profesor Gian Carlo Daraviña (con Respuestas Exactas)

Estas son las preguntas que el profesor suele hacer en sustentación para verificar que todos entendieron tanto la teoría de clase como el código:

### 1. *"¿Cuál es la diferencia entre el Perceptrón y el Adaline (SGAdaline) que vieron en la Clase 1?"*
- **Respuesta:** En el **Perceptrón**, la actualización de los pesos $\Delta \mathbf{w} = \eta (y - \hat{y})\mathbf{x}$ usa la salida **después** de la función escalón ($\hat{y} \in \{-1, +1\}$), por lo que solo aprende cuando se equivoca de clase y los saltos son discretos. En cambio, en **Adaline / SGAdaline**, la actualización $\Delta \mathbf{w} = \eta (y - z)\mathbf{x}$ usa la salida **lineal continua** $z = \mathbf{w}^\top \mathbf{x} + b$ **antes** del cuantizador, minimizando una función de costo derivable que es el **Error Cuadrático Medio ($MSE = \frac{1}{2}\sum (y_i - z_i)^2$)** mediante Descenso de Gradiente Estocástico (SGD) muestra por muestra.

### 2. *"¿Por qué es obligatorio usar `StandardScaler` antes de entrenar Perceptrón, SGAdaline, SVM y la Red Neuronal?"*
- **Respuesta:** Porque nuestras 325 características tienen escalas numéricas muy distintas (por ejemplo, la relación de aspecto puede valer `1.8`, un momento de Hu logarítmico puede valer `7.5` y un bin de histograma `0.02`). Si no estandarizamos a media $0$ y varianza $1$ ($z = (x - \mu)/\sigma$), las variables con números más grandes dominan el producto punto $\mathbf{w}^\top \mathbf{x}$ y la distancia Euclidiana $\|\mathbf{x} - \mathbf{x}'\|^2$ del kernel RBF de SVM, haciendo que el gradiente oscile y no converja. Y solo hacemos `fit_transform` en `X_train` para no filtrar información estadística del conjunto de prueba (*Data Leakage*).

### 3. *"En SVM, ¿qué es el margen, qué hace el parámetro $C$ y por qué ganó el Kernel RBF?"*
- **Respuesta:** El **margen** es la distancia entre el hiperplano separador y las muestras más cercanas de cada clase (llamadas **vectores de soporte**). El parámetro **$C$** controla la penalización por errores (variables de holgura $\xi_i$): un $C$ muy grande obliga a un margen duro propenso a sobreajuste, mientras que un $C$ moderado ($C = 5,0$) permite un margen blando robusto al ruido. Ganó el **Kernel RBF** ($K(\mathbf{x}, \mathbf{x}') = \exp(-\gamma \|\mathbf{x} - \mathbf{x}'\|^2)$) con `90,83%` frente al Lineal (`83,70%`) porque proyecta las 325 características a un espacio de dimensión infinita donde las rotaciones y perspectivas de las piezas sí son separables.

### 4. *"En el Árbol de Decisión, ¿qué mide la Impureza de Gini y por qué limitaron `max_depth = 14`?"*
- **Respuesta:** La **Impureza de Gini** ($I_G(t) = 1 - \sum_{k=1}^{20} p(k \mid t)^2$) mide la probabilidad de clasificar erróneamente una muestra tomada al azar en el nodo $t$; vale $0$ cuando el nodo es 100% puro (todas las piezas del nodo son de la misma clase). Limitamos la profundidad máxima a `max_depth = 14` porque un árbol sin límite (`max_depth = None`) sigue creando ramas hasta dejar 1 sola muestra por hoja, memorizando el conjunto de entrenamiento (*overfitting*) y perdiendo capacidad de generalización en prueba.

### 5. *"En la CNN, ¿de dónde sale el número `2048` al hacer el `Flatten` antes de la capa densa?"*
- **Respuesta:** *(¡Pregunta clásica del profesor con la calculadora!)* Nuestro tercer bloque convolucional entrega **`128` mapas de características** (porque tiene `128` filtros), y luego la capa `AdaptiveAvgPool2d((4, 4))` deja cada mapa con un alto de `4` y ancho de `4`. Al aplanar con `Flatten`, multiplicamos $\text{Alto} \times \text{Ancho} \times \text{Canales} = 4 \times 4 \times 128 = \mathbf{2.048}$ entradas que van hacia la capa densa de `256` neuronas.

### 6. *"¿Qué pasaría en la CNN o en la MLP si pusiéramos `learning_rate = 0.1` en el optimizador Adam y cómo evitaron el sobreajuste de la Clase 5?"*
- **Respuesta:** Como vimos en la Clase 4 y 5 con CIFAR-10, un `learning_rate = 0.1` en Adam es demasiado alto: los pasos del gradiente se saltan el mínimo del valle de pérdida y la red se queda estancada adivinando al azar (para 20 clases sería $1/20 = 5\%$ de exactitud). Por eso usamos `learning_rate = 0.001` con decaimiento `CosineAnnealingLR`. Y para evitar el sobreajuste (*overfitting*) donde `val_loss` subía después de la época 10, aplicamos las tres técnicas de la Clase 7: **Data Augmentation** (rotaciones $\pm 15^\circ$, flips y brillo), **Batch Normalization** en cada bloque, **Dropout de `0,40`** y **Early Stopping** guardando los pesos de la mejor época de validación.

### 7. *"¿Por qué la CNN (`95,63%`) le gana a la Red Neuronal Densa MLP (`89,23%`) y al SVM (`90,83%`)?"*
- **Respuesta:** Porque la MLP y el SVM reciben un vector 1D de 325 números donde se pierde la posición exacta 2D de cada tetón respecto a las paredes del ladrillo. En cambio, la **CNN** aplica filtros convolucionales $3 \times 3$ directamente sobre la matriz 2D de la imagen ($64 \times 64 \times 3$), conservando la vecindad espacial entre píxeles y aprendiendo representaciones jerárquicas: en el Bloque 1 detecta bordes y sombras, en el Bloque 2 detecta tetones circulares y ángulos, y en el Bloque 3 reconoce la forma tridimensional completa del bloque LEGO.
