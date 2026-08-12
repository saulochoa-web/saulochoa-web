# LACX: Liquidity Absorption Curvature Index

> **Estado del proyecto:** hipótesis cuantitativa experimental  
> **Frecuencia objetivo:** intradía, preferentemente velas de 5 minutos  
> **Implementación prevista:** Python y versión simplificada en Pine Script

## Aviso importante

Esta propuesta busca formular un indicador cuantitativo original, pero no es posible demostrar que sea completamente inédito sin revisar exhaustivamente la literatura académica, patentes, repositorios, código privado y modelos propietarios.

El **LACX** debe tratarse como una **hipótesis cuantitativa experimental**, no como una estrategia rentable validada, una recomendación de inversión ni una garantía de resultados futuros.

---

## Resumen

El **Liquidity Absorption Curvature Index (LACX)**, o **Índice de Curvatura de Absorción de Liquidez**, intenta medir si el mercado está absorbiendo presión compradora o vendedora sin desplazar significativamente el precio y si esa capacidad de absorción está empezando a deteriorarse.

El indicador no busca determinar si un activo está caro, barato, sobrecomprado o sobrevendido. Busca detectar la transición desde un mercado capaz de absorber órdenes hacia otro más frágil, en el cual una pequeña presión adicional puede provocar un desplazamiento desproporcionado del precio.

El LACX produce tres resultados principales:

1. **Dirección latente:** presión compradora o vendedora acumulada.
2. **Fragilidad:** estimación de la insuficiencia potencial de la liquidez disponible.
3. **Riesgo de ruptura:** intensidad potencial del movimiento si desaparece la absorción.

---

## 1. Nombre del indicador

### Liquidity Absorption Curvature Index, LACX

En español:

### Índice de Curvatura de Absorción de Liquidez

La contribución conceptual del LACX consiste en estudiar simultáneamente:

- la dirección y persistencia del flujo;
- la capacidad del mercado para absorberlo;
- el impacto marginal del flujo sobre el precio;
- el deterioro de la liquidez;
- la aceleración o curvatura de dicho impacto.

---

## 2. El problema actual

Muchos indicadores tradicionales procesan transformaciones de variables históricas como:

- retornos;
- máximos y mínimos;
- promedios;
- volatilidad;
- velocidad del movimiento;
- posición relativa dentro de una ventana.

El problema es que dos mercados pueden presentar precios y volatilidades similares, pero tener estructuras internas muy diferentes.

### Ejemplo conceptual

Supongamos que dos acciones suben un 1%:

- En la **acción A**, compradores agresivos ejecutan grandes cantidades, pero aparecen órdenes limitadas de venta capaces de absorber la demanda. El precio sube lentamente.
- En la **acción B**, un volumen comprador pequeño hace subir el precio rápidamente porque existe poca liquidez vendedora.

Un indicador basado solamente en el precio podría considerar ambos movimientos equivalentes. El LACX intentaría distinguirlos:

- **Acción A:** presión latente elevada, pero todavía existe absorción.
- **Acción B:** alta fragilidad y sensibilidad del precio, aunque el volumen sea menor.

### Punto ciego que intenta resolver

El LACX busca identificar la siguiente combinación:

$$
\text{Presión persistente}
+
\text{bajo desplazamiento inicial}
+
\text{deterioro de la absorción}
$$

Esta situación puede anticipar una transición de régimen:

1. Las órdenes agresivas presionan repetidamente el mercado.
2. La liquidez pasiva absorbe inicialmente esa presión.
3. La capacidad de absorción comienza a agotarse.
4. El impacto marginal de cada nueva orden aumenta.
5. El precio experimenta una expansión abrupta.

El indicador no intenta predecir cualquier movimiento. Intenta detectar **acumulación de energía microestructural** antes de una posible ruptura.

---

## 3. Frecuencia de análisis

El LACX está diseñado principalmente para análisis **intradía**.

### Configuración inicial recomendada

- **Frecuencia base:** velas de 5 minutos.
- **Ventana rápida:** 12 velas, equivalente a 1 hora.
- **Ventana principal:** 48 velas, equivalente a 4 horas.
- **Ventana de régimen:** 78 velas, aproximadamente una sesión regular de Estados Unidos.
- **Normalización histórica:** 20 sesiones.
- **Horizontes de evaluación:** 6, 12 y 24 velas, equivalentes a 30, 60 y 120 minutos.

La versión institucional puede calcularse a partir de ticks, eventos del libro de órdenes o barras de segundos. La versión accesible puede aproximarse con velas OHLCV.

---

## 4. Variables y datos de entrada

El indicador puede construirse en dos versiones.

### 4.1 Versión institucional

Utiliza datos tick-by-tick y del libro de órdenes:

- precio de cada operación;
- volumen de cada operación;
- clasificación de la orden agresora;
- bid y ask;
- profundidad disponible por nivel;
- altas, modificaciones y cancelaciones;
- spread;
- tiempo entre operaciones;
- volatilidad realizada intradía.

### 4.2 Versión accesible

Si no se dispone del libro de órdenes, puede aproximarse con:

- `open`;
- `high`;
- `low`;
- `close`;
- `volume`;
- número de transacciones, si está disponible;
- volumen comprador y vendedor, si el proveedor lo ofrece;
- VWAP;
- spread estimado o rango intrabar.

La versión basada en velas será menos precisa porque el sentido del flujo debe estimarse.

---

## 5. Variables fundamentales

### 5.1 Desequilibrio de flujo agresivo

Sea $Q_t^+$ el volumen comprador agresivo y $Q_t^-$ el volumen vendedor agresivo:

$$
OFI_t =
\frac{Q_t^+ - Q_t^-}
{Q_t^+ + Q_t^- + \varepsilon}
$$

El valor pertenece aproximadamente al intervalo $[-1,1]$:

- $OFI_t > 0$: predominio comprador.
- $OFI_t < 0$: predominio vendedor.
- $OFI_t \approx 0$: flujo equilibrado.

#### Aproximación con velas OHLCV

Cuando no existe clasificación real de operaciones, puede utilizarse:

$$
\widehat{OFI}_t =
\frac{2C_t-H_t-L_t}
{H_t-L_t+\varepsilon}
$$

La aproximación puede ponderarse por volumen relativo:

$$
VOFI_t =
\widehat{OFI}_t
\log\left(1+\frac{V_t}{\widetilde{V}_{t,k}+\varepsilon}\right)
$$

Aquí, $\widetilde{V}_{t,k}$ representa el volumen mediano histórico de la misma posición $k$ dentro de la sesión.

### 5.2 Desplazamiento efectivo del precio

$$
D_t =
\frac{\ln(M_t/M_{t-1})}
{\widehat{\sigma}_t\sqrt{\Delta t}+\varepsilon}
$$

Donde:

- $M_t$ es el precio medio entre bid y ask;
- $\widehat{\sigma}_t$ es la volatilidad local;
- $\Delta t$ es el intervalo temporal.

La normalización permite comparar activos y regímenes de volatilidad diferentes.

### 5.3 Impacto marginal del flujo

$$
I_t =
\frac{|D_t|}
{|OFI_t|+\varepsilon}
$$

- Un impacto alto indica que una presión pequeña mueve mucho el precio. Puede señalar poca profundidad o fragilidad.
- Un impacto bajo indica que una presión grande produce poco movimiento. Puede señalar absorción.

### 5.4 Persistencia direccional

$$
P_t =
\left|
\frac{
\sum_{j=0}^{n-1} w_j OFI_{t-j}
}{
\sum_{j=0}^{n-1} w_j |OFI_{t-j}|+\varepsilon
}
\right|
$$

Con pesos exponenciales:

$$
w_j=e^{-\lambda j}
$$

$P_t$ se aproxima a 1 cuando el flujo mantiene una dirección estable y se acerca a 0 cuando alterna entre compras y ventas.

### 5.5 Retención del precio

La retención mide cuánto flujo fue necesario para producir el desplazamiento observado:

$$
R_t =
\frac{
\sum_{j=0}^{n-1} w_j |OFI_{t-j}|
}{
\left|\sum_{j=0}^{n-1} w_jD_{t-j}\right|+\varepsilon
}
$$

Un valor elevado significa que existió presión significativa, pero el precio permaneció relativamente contenido. Esto puede representar:

- absorción real;
- órdenes ocultas;
- reposición de liquidez;
- resistencia institucional;
- ruido, por lo cual deben incorporarse filtros y pruebas estadísticas.

### 5.6 Tasa de desaparición de liquidez

Si se dispone del libro de órdenes:

$$
L_t^{\text{cancel}} =
\frac{
C_t^{bid}+C_t^{ask}
}{
A_t^{bid}+A_t^{ask}+C_t^{bid}+C_t^{ask}+\varepsilon
}
$$

Donde:

- $C_t$ representa volumen cancelado;
- $A_t$ representa volumen añadido.

Para medir fragilidad direccional conviene observar las cancelaciones del lado que contiene el movimiento. Ante presión compradora, interesa cuánto se retira la liquidez del ask.

### 5.7 Concentración temporal de las operaciones

$$
B_t =
\frac{
\operatorname{Var}(\Delta\tau_{t-n:t})
}{
\operatorname{E}(\Delta\tau_{t-n:t})^2+\varepsilon
}
$$

$\Delta\tau$ es el tiempo entre operaciones. Un incremento de $B_t$ puede representar ráfagas de actividad.

Una misma cantidad negociada puede tener efectos diferentes si llega uniformemente o se concentra dentro de pocos segundos o milisegundos.

---

## 6. Fórmula matemática

La innovación conceptual del LACX consiste en no utilizar solamente el nivel de impacto, sino su **curvatura con respecto al flujo acumulado**.

La pregunta principal es:

> ¿Cada unidad adicional de presión está moviendo el precio más que la unidad anterior?

### 6.1 Absorción firmada

Primero se construye una medida firmada de absorción:

$$
A_t =
\operatorname{sgn}(\overline{OFI}_t)
\cdot P_t
\cdot \log(1+R_t)
$$

Donde:

$$
\overline{OFI}_t=
\sum_{j=0}^{n-1}w_jOFI_{t-j}
$$

Interpretación:

- $A_t>0$: absorción frente a presión compradora.
- $A_t<0$: absorción frente a presión vendedora.
- $|A_t|$ alto: presión persistente con poco desplazamiento.

### 6.2 Curvatura de impacto

Definimos el flujo acumulado firmado:

$$
F_t=\sum_{j=0}^{n-1}w_jOFI_{t-j}
$$

Y el desplazamiento acumulado:

$$
X_t=\sum_{j=0}^{n-1}w_jD_{t-j}
$$

La pendiente local de impacto se aproxima mediante:

$$
\beta_t =
\frac{
\operatorname{Cov}_w(OFI,D)
}{
\operatorname{Var}_w(OFI)+\varepsilon
}
$$

La curvatura se define como el cambio estandarizado de esa pendiente:

$$
C_t =
\frac{
\beta_t-\beta_{t-h}
}{
\operatorname{MAD}(\beta_{t-m:t})+\varepsilon
}
$$

`MAD` es la desviación absoluta mediana.

- $C_t>0$: el mercado está respondiendo cada vez más al flujo.
- $C_t<0$: la capacidad de absorción está aumentando.
- $C_t\gg0$: posible agotamiento de la liquidez.

### 6.3 Fragilidad de liquidez

$$
G_t =
\operatorname{sigmoid}\left(
\alpha_1C_t
+\alpha_2Z(L_t^{\text{cancel}})
+\alpha_3Z(B_t)
+\alpha_4Z(Spread_t)
-\alpha_5Z(Depth_t)
\right)
$$

Con:

$$
\operatorname{sigmoid}(x)=\frac{1}{1+e^{-x}}
$$

Por construcción:

$$
0<G_t<1
$$

- Cerca de 0: mercado relativamente resistente.
- Cerca de 1: mercado frágil.

Si no existe información del libro, las cancelaciones y la profundidad pueden reemplazarse por aproximaciones basadas en rango, volumen e impacto. Esta sustitución debe documentarse porque modifica la interpretación del indicador.

### 6.4 Índice final LACX

$$
\boxed{
LACX_t=
100\cdot\tanh(A_t)\cdot G_t\cdot\operatorname{sigmoid}(C_t)
}
$$

El resultado queda aproximadamente dentro de:

$$
-100<LACX_t<100
$$

Para evitar una interpretación incompleta, el indicador también debe publicar un vector de estado:

$$
\boxed{
\mathcal{L}_t=
\left(LACX_t, G_t, A_t, C_t\right)
}
$$

El valor escalar indica dirección e intensidad, mientras que sus componentes explican por qué aparece la señal.

### 6.5 Incertidumbre estadística

Puede aplicarse un bootstrap por bloques sobre observaciones intradía para obtener:

$$
p_t^+=\Pr(LACX_t>0\mid\mathcal{D}_t)
$$

$$
p_t^-=\Pr(LACX_t<0\mid\mathcal{D}_t)
$$

También puede calcularse un intervalo de confianza o credibilidad empírico:

$$
CI_t^{90\%}=
\left[
q_{0.05}(LACX_t^*),
q_{0.95}(LACX_t^*)
\right]
$$

Una señal se considera débil si el intervalo contiene cero, aunque el valor puntual sea elevado.

---

## 7. Interpretación y señales

El LACX no debe utilizarse como sistema automático de compra o venta sin validación. Los siguientes estados son hipótesis de investigación.

### Estado 1: absorción compradora estable

$$
A_t>0,\qquad C_t\leq0,\qquad G_t<0.4
$$

Existe presión compradora, pero todavía está siendo absorbida. No constituye por sí sola una señal alcista porque el vendedor pasivo continúa conteniendo el precio.

**Lectura:** presión latente sin confirmación.

### Estado 2: ruptura alcista potencial

$$
A_t>0,\qquad C_t>z_c,\qquad G_t>g_c
$$

Además:

$$
LACX_t>\ell_c
$$

Umbrales iniciales únicamente para investigación:

- $C_t>1.5$;
- $G_t>0.70$;
- $LACX_t>60$;
- intervalo bootstrap completamente por encima de cero.

**Lectura:** la presión compradora persistente antes absorbida empieza a generar desplazamientos crecientes. La liquidez vendedora puede estar agotándose o retirándose.

### Estado 3: ruptura bajista potencial

$$
A_t<0,\qquad C_t>z_c,\qquad G_t>g_c
$$

Además:

$$
LACX_t<-\ell_c
$$

**Lectura:** las ventas agresivas antes absorbidas comienzan a producir un impacto marginal creciente. La liquidez compradora podría estar desapareciendo.

### Estado 4: falso rompimiento o agotamiento

Si el precio rompe un máximo, pero se observa:

$$
P_t\downarrow,\qquad C_t<0,\qquad G_t\downarrow
$$

Esto indicaría que:

- el flujo pierde persistencia;
- cada unidad adicional mueve menos el precio;
- la liquidez se está reconstruyendo.

**Lectura:** el rompimiento tiene menor respaldo microestructural.

Esto no implica necesariamente una reversión. Indica que la evidencia interna de continuación se está debilitando.

---

## 8. Medición del riesgo

Se propone un subíndice adicional:

$$
Risk_t=
G_t\cdot Z(\widehat{\sigma}_t)
\cdot\left(1+\max(C_t,0)\right)
\cdot\frac{1}{Depth_t+\varepsilon}
$$

El resultado puede transformarse a una escala de 0 a 100 mediante percentiles históricos.

| Percentil de riesgo | Interpretación |
|---:|---|
| 0–50 | Condición normal |
| 50–75 | Impacto creciente |
| 75–90 | Fragilidad elevada |
| 90–100 | Riesgo extremo de desplazamiento y slippage |

Una señal direccional alta con riesgo extremo no implica necesariamente una oportunidad más atractiva. Puede significar:

- peor precio de ejecución;
- mayor slippage;
- spreads inestables;
- saltos de precio;
- mayor sensibilidad a órdenes grandes;
- stops ejecutados lejos del nivel esperado.

### Matriz de lectura

| LACX | Fragilidad $G_t$ | Interpretación experimental |
|---|---:|---|
| Alto positivo | Alta | Ruptura alcista con liquidez frágil |
| Alto positivo | Baja | Presión compradora todavía absorbida |
| Cerca de cero | Alta | Fragilidad sin dirección clara |
| Bajo negativo | Alta | Ruptura bajista con liquidez frágil |
| Bajo negativo | Baja | Presión vendedora todavía absorbida |

La combinación **LACX cercano a cero y fragilidad alta** merece atención: el mercado podría estar preparado para un movimiento amplio, pero el indicador todavía no identifica una dirección fiable.

---

## 9. Diseño de la implementación

### Implementación principal en Python

Nombre del archivo:

```text
lacx_liquidity_absorption.py
```

Nombre de la clase:

```python
class LiquidityAbsorptionCurvature:
    pass
```

Método principal:

```python
def compute_lacx(self):
    pass
```

### Estructura sugerida del proyecto

```text
lacx/
├── __init__.py
├── data_validation.py
├── trade_classification.py
├── orderbook_features.py
├── absorption_model.py
├── curvature_estimator.py
├── uncertainty.py
├── signal_engine.py
└── backtest_event_study.py
```

### Salida mínima esperada

La implementación debería devolver, como mínimo:

```text
timestamp
lacx
direction
absorption
impact_curvature
fragility
risk_score
signal
forward_return_30m
forward_return_60m
forward_return_120m
```

### Versión simplificada para Pine Script

Nombre propuesto:

```text
LACX_Lite_AbsorptionFragility
```

La versión de Pine Script sería necesariamente una aproximación, ya que TradingView normalmente no proporciona todos los eventos del libro de órdenes, cancelaciones y datos microestructurales necesarios.

La versión simplificada podría utilizar:

- volumen;
- rango de la vela;
- posición del cierre;
- VWAP;
- velocidad del volumen;
- impacto normalizado;
- persistencia direccional.

---

## 10. Validación obligatoria

Para que el LACX pase de idea a indicador cuantitativo defendible, debería superar estas pruebas:

1. **Estudio de eventos** después de cruces de umbral.
2. **Walk-forward analysis** sin ajustar parámetros sobre datos futuros.
3. Inclusión realista de spread, comisiones, latencia y slippage.
4. Comparación contra un modelo nulo basado en volatilidad y volumen.
5. Control del sesgo de selección y del data snooping.
6. Pruebas en acciones, futuros y ETF con distintos niveles de liquidez.
7. Separación de resultados entre apertura, cierre, anuncios y sesiones normales.
8. Evaluación de la estabilidad de parámetros fuera de muestra.
9. Pruebas contra filtración accidental de información futura.
10. Evaluación económica, no solamente precisión estadística.
11. Comparación contra reglas simples de precio, volumen y volatilidad.
12. Pruebas de sensibilidad de ventanas, umbrales y costos de ejecución.

### Hipótesis estadística principal

Para un horizonte futuro $h$, debe evaluarse si:

$$
\operatorname{E}[r_{t+h}\mid LACX_t>q_{0.90}]
$$

es significativamente distinto y económicamente superior a:

$$
\operatorname{E}[r_{t+h}]
$$

De manera análoga, para señales bajistas debe analizarse:

$$
\operatorname{E}[r_{t+h}\mid LACX_t<q_{0.10}]
$$

Los cuantiles y umbrales deben estimarse únicamente con información disponible hasta el instante $t$.

---

## 11. Riesgos metodológicos

La investigación debe controlar, como mínimo, los siguientes riesgos:

- **Look-ahead bias:** uso accidental de datos futuros.
- **Survivorship bias:** selección de activos que sobrevivieron o fueron exitosos.
- **Data snooping:** prueba excesiva de parámetros hasta encontrar resultados favorables.
- **Overfitting:** ajuste demasiado específico a una muestra histórica.
- **No estacionariedad:** cambio de la relación entre flujo, liquidez e impacto.
- **Sesgo horario:** comparación incorrecta entre apertura, mediodía y cierre.
- **Sesgo del proveedor:** diferencias en la clasificación de operaciones y consolidación de volumen.
- **Subestimación de costos:** omisión del spread, impacto, comisiones y slippage.
- **Errores de sincronización:** desalineación entre trades, quotes y libro de órdenes.

---

## 12. Conclusión

La contribución conceptual del LACX consiste en medir **la curvatura del impacto mientras se deteriora la absorción**, en lugar de tratar el precio, volumen o volatilidad de manera aislada.

La hipótesis central es que una ruptura puede adquirir mayor probabilidad cuando coinciden:

1. presión direccional persistente;
2. bajo desplazamiento inicial;
3. absorción significativa;
4. deterioro de la capacidad de absorción;
5. impacto marginal creciente;
6. fragilidad elevada de la liquidez.

El valor real del indicador dependerá de demostrar, fuera de muestra y después de costos de transacción, que esta transición contiene información incremental sobre los retornos futuros o sobre el riesgo de desplazamientos extremos.

---

## Licencia y uso

Antes de publicar una implementación, se recomienda incorporar una licencia al repositorio, por ejemplo MIT, Apache-2.0 o una licencia privada, según el objetivo del proyecto.

Este documento describe una idea de investigación y no constituye asesoría financiera.
