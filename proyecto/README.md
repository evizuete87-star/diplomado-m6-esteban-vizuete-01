# Sistema de decisión para PagoYa — Grupo 4
**Integrantes:** Vizuete Esteban, Estrella Danilo
**Problema:** Crédito — Política de aprobación de microcréditos

## 1. Problema y datos
PagoYa ofrece microcréditos y necesita decidir a quién aprobar para maximizar su rentabilidad. Se analizaron **2 500 solicitudes históricas** con 12 variables (edad, ingreso, tipo de empleo, monto, plazo, ratio cuota/ingreso, mora previa, etc.). La tasa de default base es del **10,88 %**. La regla de negocio establece que un default pierde el 100 % del monto prestado, mientras que un buen pagador deja un margen de ganancia del 15 % del monto.

## 2. Modelos comparados
Se evaluaron tres familias de modelos utilizando una partición fija 70/30 estratificada (random_state=42):

| Modelo | AUC | Bal_Acc | Brier |
|---|---|---|---|
| Logística (class_weight='balanced') | 0,7460 | 0,6922 | 0,1778 |
| XGBoost (sin scale_pos_weight) | 0,7319 | 0,5343 | 0,0874 |
| **Bayes (PyMC) - Elegido** | **0,7412** | **0,5443** | **0,0882** |

Los tres modelos logran un AUC similar (0,732 - 0,746), lo que indica que la señal subyacente es fundamentalmente lineal. **Se eligió el modelo Bayesiano** por su alta calibración (Brier 0,0882) y su capacidad única de entregar **intervalos de incertidumbre (IC90)** por cada solicitud, algo vital para un comité de riesgos.

## 3. Qué explica el resultado
El análisis SHAP (explicabilidad) reveló que las variables más influyentes en el riesgo de impago son **es_informal** (empleo informal), **mora_previa**, **antiguedad_meses** y **ratio_cuota**. El EDA demostró que el ingreso absoluto no discrimina bien, por lo que creamos la variable `ratio_cuota` (nivel de sobreendeudamiento). El modelo Bayesiano permite además explicar cada caso individual mostrando el intervalo de confianza del 90 % de la probabilidad de default, identificando cuándo el modelo "no está seguro" (IC90 ancho) y requiere revisión humana.

## 4. Recomendación con cifras
- **Umbral de aprobación recomendado: `p < 0,14`**. 
  *El óptimo teórico matemático es 0,1304 (derivado de la regla 15% ganancia / 100% pérdida). El umbral empírico del modelo (0,14) coincide casi perfectamente, demostrando que las probabilidades bayesianas están perfectamente calibradas.*
- **Impacto Financiero:** Con el umbral óptimo, la ganancia neta esperada es de **$66 015** frente a los **$12 814** que se obtendrían aprobando a todos (**mejora del +415 %**).
- **Política Operativa:** 
  - Aprobación automática si `p < 0,10` (clientes seguros).
  - Revisión manual / ajuste de monto si `0,10 ≤ p < 0,15` (zona gris).
  - Rechazo automático si `p ≥ 0,15` (alto riesgo).
  - Tasa de aprobación esperada: **78,8 %** de las solicitudes. Tasa de default entre aprobados cae al **6,26 %**.

## 5. Limitaciones y próximos pasos
- **Datos simulados:** los resultados orientan la política, pero el umbral debe recalibrarse con datos reales de producción.
- **Variables faltantes:** no se incluyen variables macroeconómicas, historial crediticio en burós externos ni patrones de pago temporales.
- **Próximos pasos:** Incorporar variables de comportamiento transaccional en ventana temporal y realizar monitoreo de deriva (PSI) mensual para el reentrenamiento del modelo. La app "DeteKta" queda lista para integrarse vía API al core bancario.
