# T2 PID440

# opciones de RQ:


## 1. Robustez Acústica / Ruido no estacionario

Vacío del paper: El modelo se probó en un entorno de laboratorio con audios limpios y solo un ruido sintético suave. ¿Qué pasa cuando el audio de entrada sufre degradación real de señal a ruido (SNR)?


«¿Cómo afecta la degradación de la relación señal a ruido (SNR) mediante ruido aditivo no estacionario al desempeño de clasificación de palabras clave del modelo BC-ResNet-1 sobre Google Speech Commands v2, en términos de exactitud (Top-1 Accuracy) y F1-score por clase?»


* Modelo: BC-ResNet-1.
* Tarea: Clasificación de comandos de voz (12 clases).
* Condición / Contexto: Niveles variables de ruido acústico (SNR a 20 dB, 10 dB, 5 dB y 0 dB).
* Métrica: Top-1 Accuracy y F1-score.


### Hipotesis


Al degradar la relación señal a ruido por debajo de 10 dB, el desempeño global de BC-ResNet-1 sufrirá una caída de exactitud superior a 15 puntos porcentuales respecto al baseline, concentrando la mayor tasa de degradación en el F1-score de los comandos fonéticamente homófonos (como 'no' frente a 'go'), debido a que el colapso espectral por average pooling pierde la energía de los transitorios consonánticos.


* Falsabilidad: Si al correr el experimento a 5 dB o 0 dB el modelo mantiene una exactitud alta (ej. > 85%) o si los errores no se concentran en los pares fonéticos mencionados, la hipótesis queda refutada empíricamente.


### Setup

* Dataset: Google Speech Commands v2 (conjunto oficial de evaluación/test de 12 clases: 10 comandos, silencio y unknown).
* Modelo Baseline: Arquitectura oficial BC-ResNet-1 ($\tau = 1$, 9.510 parámetros) evaluada sobre el conjunto de prueba limpio sin perturbaciones adicionales. 
* Variable a estudiar (Única variable): Relación Señal a Ruido (SNR) de las muestras de audio de prueba:

- Nivel 1 (Baseline): Audio original limpio.
- Nivel 2: SNR = 20 dB (ruido leve).
- Nivel 3: SNR = 10 dB (ruido moderado).
- Nivel 4: SNR = 0 dB (ruido severo / señal y ruido con igual energía).


### Métricas

* Top-1 Accuracy: Métrica estándar del benchmark para contrastar la caída global contra el paper original.
* Macro F1-Score y F1-Score por clase: Necesario para evaluar el impacto en clases específicas y detectar si los pares homófonos se degradan más rápido.
* Matriz de Confusión: Visualización para evidenciar las transiciones de falso reconocimiento.


### Plan de Experimentos

* Experimento Principal (Baseline): Cargar los pesos de BC-ResNet-1 (los de tu checkpoint o los oficiales) y evaluar la partición de prueba en condiciones estándar (debe reproducir el ~95%–96% que obtuviste en la T1).
* Variación Simple: Crear una función de perturbación en el DataLoader de prueba que sume ruido aditivo (por ejemplo, ruido blanco o ruido ambiental de fondo) a niveles controlados de SNR ($20, 10, 5, 0\text{ dB}$) y re-evaluar la red sin alterar ningún peso ni hiperparámetro.


### Viabilidad Funcional

* El código no requiere reentrenar épocas desde cero; se ejecuta un bucle de inferencia sobre el conjunto de test con la función de ruido.
* Tarda menos de 2 a 3 minutos por cada nivel de ruido en Google Colab.


### Analisis

* Coherencia: Analizar si el mecanismo de broadcasting amortigua el ruido o si el frequency average pooling lo hace más vulnerable al promediar el ruido en todo el espectro.
* Dificultades encontradas: Mapear la normalización correcta de las ondas al inyectar ruido para evitar recortes (clipping).


