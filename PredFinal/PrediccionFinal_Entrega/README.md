# Modificaciones del Pipeline (`v8` vs. `v0`)

Este documento detalla los cambios implementados en la versión final (`z739_final_senior_v8`) respecto al script base (`z739_final_senior`) para la competencia **Analista Sr. 2026**.

---

## 1. Configuración y Reproducibilidad
* **Identificador de experimento (`PARAM$experimento`):** Es el **`9557`**.
* **Semilla primigenia (`PARAM$semilla_primigenia`):** Actualizada a **`109843`**.
* **Paralelización (`num_threads`):** Se configuró explícitamente **`num_threads = 10`** en `PARAM$lgbm$param_fijos` para aprovechar los núcleos disponibles de la VM durante el entrenamiento.

---

## 2. Selección de Variables (`CanaritosAsesinos`)
Se endureció el criterio de filtrado de variables para reducir ruido y dimensionalidad antes de la Optimización Bayesiana, segun lo observado en los Experimentos Colaborativos:
* **`PARAM$CN$ratio`:** Incrementado de `0.2` a **`0.75`** (75% de variables canarito respecto al total de atributos), logrando una estimación empírica más estable de la distribución de ruido.
* **`PARAM$CN$desvios`:** Reducido de `2` a **`0`**, fijando el umbral de corte directamente en la mediana de la posición de los canaritos (`median(pos)`).

---

## 3. Optimización Bayesiana (`mlrMBO`) con Semillerío Integrado
Se rediseñó la búsqueda de hiperparámetros bajo un enfoque tipo **"Random Forest de LightGBMs"**, alineando la función objetivo con el comportamiento de un ensamble:

* **Semillerío dentro de la función objetivo (`PARAM$BO$semillerio = 50`):**
  * En lugar de evaluar cada combinación de hiperparámetros con un único modelo, `EstimarGanancia_AUC_lightgbm()` entrena **50 modelos con semillas distintas**, promedia sus probabilidades sobre el conjunto de validación (`202107`) y calcula el `AUC` sobre el ensamble resultante.
* **Presupuesto de iteraciones (`PARAM$hipeparametertuning$num_interations`):** Aumentado de `30` a **`100` iteraciones**.
* **Nuevo espacio de búsqueda (`PARAM$hipeparametertuning$hs`):** Ajustado para modelos más compactos, con mayor tasa de aprendizaje y baja fracción de columnas para maximizar la diversidad entre semillas:

| Hiperparámetro | `v0` (Base) | `v8` (Final) | Objetivo del ajuste |
| :--- | :--- | :--- | :--- |
| **`num_iterations`** | $2^2$ a $2^{10}$ ($4$ a $1024$) | **$2^4$ a $2^8$ ($16$ a $256$)** | Menor cantidad de árboles por modelo individual para evitar sobreajuste y acelerar el semillerío. |
| **`num_leaves`** | $2^2$ a $2^{11}$ ($4$ a $2048$) | **$2^4$ a $2^{9.5}$ ($16$ a $\approx 724$)** | Acota la complejidad estructural de cada árbol. |
| **`learning_rate`** | $2^{-7}$ a $2^{-0.1}$ ($\approx 0.0078$ a $0.93$) | **$2^{-3.5}$ a $2^{-0.5}$ ($\approx 0.088$ a $0.707$)** | Tasas de aprendizaje altas cuya varianza se compensa al promediar las semillas. |
| **`feature_fraction`** | $0.05$ a $0.90$ | **$0.08$ a $0.45$** | Mayor submuestreo de columnas para descorrelacionar los modelos del ensamble. |
| **Regularización de hoja** | Optimiza `min_sum_hessian_in_leaf` ($2^{-10}$ a $2^{10}$) con `min_data_in_leaf = 0` | Optimiza **`min_data_in_leaf`** ($2^6$ a $2^{11.5}$, i.e., **$64$ a $\approx 2896$**) | Asegura soporte mínimo de observaciones por hoja frente a tasas de aprendizaje elevadas. |

---

## 4. Entrenamiento Final (`Final Train`) y Predicción
* **Escalado proporcional de `min_data_in_leaf`:**
  * Dado que la Optimización Bayesiana se ejecuta con *undersampling* del 20% sobre la clase `CONTINUA` (`PARAM$trainingstrategy$training_pct = 0.2`) y el entrenamiento final (`dfinal_train`) utiliza el **100% de los datos**, se escaló el hiperparámetro óptimo por el inverso de dicha proporción ($\times 5$) para preservar la granularidad relativa de las hojas:
    $$\text{min\_data\_in\_leaf}_{\text{final}} = \text{round}\left(\frac{2^{x\$\text{min\_data\_in\_leaf}}}{\text{training\_pct}}\right)$$
* **Semillerío final (`PARAM$FT$semillerio`):** Ampliado de `10` a **`100` modelos** para promediar las probabilidades sobre `202109` y estabilizar el ranking en los envíos a Kaggle (`8000` a `12000`).