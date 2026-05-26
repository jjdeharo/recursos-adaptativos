# Fundamentos matemáticos de los sistemas educativos adaptativos bayesianos

**Juan José de Haro · [bilateria.org](https://bilateria.org)**

---

## Índice

1. [El problema que resuelve el sistema](#1-el-problema-que-resuelve-el-sistema)
2. [Representación probabilística del estado del alumno](#2-representación-probabilística-del-estado-del-alumno)
3. [Inferencia bayesiana](#3-inferencia-bayesiana)
4. [Modelo IRT de tres parámetros](#4-modelo-irt-de-tres-parámetros)
5. [Entropía de Shannon como medida de incertidumbre](#5-entropía-de-shannon-como-medida-de-incertidumbre)
6. [Ganancia esperada de información](#6-ganancia-esperada-de-información)
7. [Criterio de parada](#7-criterio-de-parada)
8. [Convención de escala entre θ y b](#8-convención-de-escala-entre-θ-y-b)
9. [Ejemplo numérico completo](#9-ejemplo-numérico-completo)
10. [Hipótesis no jerárquicas](#10-hipótesis-no-jerárquicas)
11. [Límites del modelo](#11-límites-del-modelo)

---

## 1. El problema que resuelve el sistema

Un sistema de evaluación tradicional asigna las mismas preguntas a todos los alumnos en el mismo orden. Esto genera dos ineficiencias:

- **Preguntas demasiado fáciles** para alumnos avanzados no aportan información útil sobre su nivel real.
- **Preguntas demasiado difíciles** para alumnos con dificultades generan frustración sin mejorar el diagnóstico.

La evaluación adaptativa resuelve esto seleccionando en cada momento la pregunta más informativa dado lo que ya se sabe del alumno. Para ello necesita tres ingredientes:

1. Una **representación del estado de conocimiento** del alumno (qué creemos saber de él).
2. Una **regla de actualización** que cambie esa representación tras cada respuesta.
3. Un **criterio de selección** que elija qué pregunta formular a continuación.

Este documento describe la matemática detrás de cada uno de esos tres ingredientes.

---

## 2. Representación probabilística del estado del alumno

### 2.1. El espacio de hipótesis

En lugar de asignar al alumno un valor fijo (una nota, una etiqueta), el sistema mantiene una **distribución de probabilidades** sobre un conjunto de hipótesis mutuamente excluyentes y exhaustivas.

Sea $\mathcal{H} = \{H_1, H_2, \ldots, H_n\}$ el conjunto de hipótesis posibles. Por ejemplo:

$$H_1 = \text{nivel básico}, \quad H_2 = \text{nivel medio}, \quad H_3 = \text{nivel avanzado}$$

En cada momento, el sistema mantiene un vector de probabilidades:

$$\mathbf{p} = \bigl(P(H_1),\, P(H_2),\, \ldots,\, P(H_n)\bigr)$$

con la restricción:

$$\sum_{i=1}^{n} P(H_i) = 1, \quad P(H_i) \geq 0 \;\; \forall i$$

Este vector expresa el **grado de creencia** del sistema sobre el estado real del alumno, no una certeza.

### 2.2. Distribución inicial

Si no existe información previa sobre el alumno, el sistema parte de una **distribución uniforme**:

$$P(H_i) = \frac{1}{n} \quad \forall i$$

Esta elección refleja ignorancia máxima: todas las hipótesis son igualmente plausibles antes de observar ninguna respuesta. Si existiera información previa fiable (resultados de cursos anteriores, diagnósticos previos), podría usarse como distribución inicial justificada.

### 2.3. Valor numérico de cada hipótesis

Para poder usar la función logística que genera las verosimilitudes (véase §4), cada hipótesis $H_i$ necesita un valor numérico $\theta_i$ que represente su posición en la escala de dominio. La convención recomendada es centrar los valores en cero con intervalos iguales:

| $n$ hipótesis | Valores $\theta_i$ |
|:---:|:---:|
| 2 | $-1,\; +1$ |
| 3 | $-2,\; 0,\; +2$ |
| 4 | $-3,\; -1,\; +1,\; +3$ |
| 5 | $-4,\; -2,\; 0,\; +2,\; +4$ |

Los valores concretos dependen de la escala de dificultad de las preguntas. La relación exacta entre θ y b se explica en §8.

---

## 3. Inferencia bayesiana

### 3.1. El teorema de Bayes

Cuando el alumno responde una pregunta, esa respuesta es una **evidencia** que debe modificar nuestra estimación de su estado. El mecanismo de actualización es el teorema de Bayes:

$$P(H_i \mid R) = \frac{P(R \mid H_i)\, P(H_i)}{P(R)}$$

donde:

- $P(H_i)$ es la probabilidad **prior**: lo que creíamos antes de ver la respuesta.
- $P(R \mid H_i)$ es la **verosimilitud**: la probabilidad de observar la respuesta $R$ si el alumno estuviera realmente en el estado $H_i$.
- $P(R)$ es la probabilidad marginal de la respuesta, calculada sumando sobre todas las hipótesis:

$$P(R) = \sum_{j=1}^{n} P(R \mid H_j)\, P(H_j)$$

- $P(H_i \mid R)$ es la probabilidad **posterior**: la creencia actualizada tras observar $R$.

### 3.2. Actualización tras un acierto o un fallo

En la práctica, la respuesta $R$ es binaria: acierto (A) o fallo (F). La actualización toma la forma:

$$P(H_i \mid A) = \frac{P(A \mid H_i)\, P(H_i)}{\displaystyle\sum_j P(A \mid H_j)\, P(H_j)}$$

$$P(H_i \mid F) = \frac{P(F \mid H_i)\, P(H_i)}{\displaystyle\sum_j P(F \mid H_j)\, P(H_j)}$$

Nótese que el denominador es simplemente una constante de normalización. En la implementación, basta con calcular los numeradores para todos los $i$ y dividir por su suma.

### 3.3. Actualización secuencial

Si el alumno responde varias preguntas, el proceso se aplica de forma **secuencial**: el posterior de una pregunta se convierte en el prior de la siguiente. Esto es matemáticamente equivalente a actualizar con todas las respuestas a la vez, siempre que las respuestas sean condicionalmente independientes dada la hipótesis verdadera.

$$\mathbf{p}^{(t+1)} = \text{Bayes}\!\left(\mathbf{p}^{(t)},\; R_{t+1}\right)$$

### 3.4. Por qué Bayes y no otros métodos

Bajo los supuestos habituales de coherencia probabilística y actualización por evidencia observada, la regla de Bayes es la forma natural de actualizar las probabilidades. Satisface simultáneamente:

- **Consistencia**: si se actualizan las probabilidades en dos pasos o en uno, el resultado es el mismo.
- **Proporcionalidad**: la actualización es proporcional a cuánto predice cada hipótesis la respuesta observada.
- **No dogmatismo**: ninguna hipótesis queda bloqueada en 0 o 1 siempre que su probabilidad inicial sea estrictamente positiva. Por eso conviene asignar probabilidades iniciales estrictamente positivas a todas las hipótesis plausibles.

Alternativas como las redes de reglas o los sistemas expertos clásicos no tienen estas propiedades y pueden quedar bloqueadas en diagnósticos incorrectos cuando el alumno responde de forma inesperada.

---

## 4. Modelo IRT de tres parámetros

### 4.1. El problema de las verosimilitudes

Para aplicar Bayes necesitamos $P(A \mid H_i, q)$: la probabilidad de que un alumno en el estado $H_i$ acierte la pregunta $q$. Esta probabilidad es la verosimilitud.

El sistema necesita generarlas automáticamente a partir de los parámetros de cada pregunta, sin que el docente rellene tablas de probabilidades.

### 4.2. La curva característica del ítem (ICC)

El modelo IRT (Item Response Theory) proporciona una familia de funciones para modelar $P(A \mid \theta, q)$. El modelo de tres parámetros (3PL) es:

$$P(A \mid \theta_i, q) = c_q + (1 - c_q) \cdot \frac{1}{1 + e^{-a(\theta_i - b_q)}}$$

Los tres parámetros son:

| Parámetro | Nombre | Significado |
|:---:|:---|:---|
| $a$ | Discriminación | Pendiente de la curva; controla cuánto separa la pregunta entre niveles distintos |
| $b_q$ | Dificultad | Valor de $\theta$ en el que la probabilidad de acierto (sin azar) llega al 50% |
| $c_q$ | Pseudo-azar | Probabilidad mínima de acierto; en ausencia de datos empíricos, se aproxima como $c_q \approx 1/m$ |

### 4.3. Comportamiento de la curva

Cuando $\theta_i \gg b_q$ (el alumno está muy por encima de la dificultad), el exponente $e^{-a(\theta_i - b_q)} \to 0$ y:

$$P(A \mid \theta_i, q) \to c_q + (1 - c_q) \cdot 1 = 1$$

Cuando $\theta_i \ll b_q$ (el alumno está muy por debajo de la dificultad), el exponente $\to +\infty$ y:

$$P(A \mid \theta_i, q) \to c_q + (1 - c_q) \cdot 0 = c_q$$

Cuando $\theta_i = b_q$, el argumento del exponente es cero y:

$$P(A \mid \theta_i, q) = c_q + (1 - c_q) \cdot \frac{1}{2} = \frac{1 + c_q}{2}$$

Este es el **punto de inflexión** de la curva: donde la pendiente es máxima y, por tanto, donde la pregunta es más discriminante.

### 4.4. El parámetro de discriminación $a$

El parámetro $a$ controla la pendiente de la curva logística. Su efecto puede verse derivando la ICC respecto a $\theta$:

$$\frac{d}{d\theta} P(A \mid \theta, q) = a \cdot (1 - c_q) \cdot \frac{e^{-a(\theta - b_q)}}{\left(1 + e^{-a(\theta - b_q)}\right)^2}$$

La pendiente máxima (en $\theta = b_q$) vale:

$$\left.\frac{d}{d\theta} P\right|_{\theta = b_q} = \frac{a\,(1 - c_q)}{4}$$

Un valor alto de $a$ produce una curva más escarpada: la pregunta discrimina mejor entre alumnos cerca de su dificultad, pero aporta poca información a alumnos claramente por encima o por debajo. Un valor bajo produce una curva más suave: la pregunta es útil en un rango más amplio de niveles, pero discrimina menos.

Los valores habituales en psicometría oscilan entre 0.5 y 2.5. Para sistemas educativos de propósito general, **$a = 1.5$ es un punto de partida razonable**.

### 4.5. La probabilidad de fallo

$$P(F \mid \theta_i, q) = 1 - P(A \mid \theta_i, q)$$

Esta es la verosimilitud que entra en la actualización bayesiana cuando el alumno falla.

### 4.6. Probabilidad mínima de acierto por azar

En preguntas de opción múltiple con $m_q$ opciones, puede usarse $c_q = 1/m_q$ como aproximación inicial en ausencia de datos empíricos:

$$c_q \approx \frac{1}{m_q}$$

En un modelo IRT calibrado, $c_q$ debería estimarse a partir de datos reales, porque el pseudoazar no siempre coincide con el azar puro: los distractores no son igualmente atractivos y algunos alumnos eliminan opciones antes de responder. Esta probabilidad pertenece a **cada pregunta**, no al test en conjunto. En respuestas numéricas o de texto exacto donde el azar es irrelevante, se usa $c_q = 0$.

---

## 5. Entropía de Shannon como medida de incertidumbre

### 5.1. Definición

La entropía de Shannon mide la incertidumbre de una distribución de probabilidades:

$$H(\mathbf{p}) = -\sum_{i=1}^{n} p_i \log_2 p_i$$

Se mide en **bits**. Por convenio, $0 \cdot \log_2 0 = 0$.

### 5.2. Propiedades relevantes

**Entropía mínima.** Si una hipótesis concentra toda la probabilidad ($p_k = 1$, $p_{i \neq k} = 0$), la entropía es cero: no hay incertidumbre.

**Entropía máxima.** Si todas las hipótesis son equiprobables ($p_i = 1/n$), la entropía es máxima:

$$H_{\max} = \log_2 n$$

Esto corresponde a la ignorancia total sobre el estado del alumno.

**Entropía esperada no creciente.** La entropía posterior puede aumentar o disminuir tras una respuesta concreta, según la evidencia observada (como ilustra el ejemplo de §9, donde el acierto en Q2 sube la entropía respecto al paso anterior). Lo que garantiza la selección por máxima ganancia de información es que la entropía posterior *esperada* —antes de conocer la respuesta— no supera la entropía actual.

### 5.3. Ejemplos numéricos

Con $n = 3$ hipótesis:

| Distribución $\mathbf{p}$ | Entropía $H$ (bits) | Interpretación |
|:---:|:---:|:---|
| $(0.33,\; 0.33,\; 0.33)$ | $1.58$ | Ignorancia total |
| $(0.60,\; 0.30,\; 0.10)$ | $1.30$ | Incertidumbre alta |
| $(0.80,\; 0.15,\; 0.05)$ | $0.88$ | Diagnóstico probable |
| $(0.95,\; 0.04,\; 0.01)$ | $0.32$ | Diagnóstico casi seguro |
| $(1.00,\; 0.00,\; 0.00)$ | $0.00$ | Certeza absoluta |

### 5.4. Por qué la entropía y no la probabilidad máxima

La probabilidad de la hipótesis más probable ($\max_i p_i$) es un indicador intuitivo, pero la entropía captura más información: distingue entre $(0.80, 0.15, 0.05)$ y $(0.80, 0.19, 0.01)$, que tienen el mismo máximo pero distinta distribución del resto. La entropía es además la cantidad natural que aparece en la ganancia de información (§6), lo que hace el sistema matemáticamente coherente.

---

## 6. Ganancia esperada de información

### 6.1. El criterio de selección

El sistema elige la siguiente pregunta buscando **maximizar la reducción esperada de entropía**. Para cada pregunta candidata $q$, se calcula la **ganancia esperada de información**:

$$IG(q) = H(\mathbf{p}) - \mathbb{E}\!\left[H(\mathbf{p}' \mid q)\right]$$

donde $\mathbf{p}'$ es la distribución posterior tras responder $q$, y la esperanza es sobre los dos posibles resultados (acierto o fallo).

### 6.2. Desarrollo completo

Definimos primero la probabilidad marginal de acierto en la pregunta $q$, usando la **ley de la probabilidad total**:

$$P(A \mid q) = \sum_{i=1}^{n} P(H_i) \cdot P(A \mid H_i, q)$$

Y la probabilidad marginal de fallo:

$$P(F \mid q) = 1 - P(A \mid q)$$

A continuación calculamos los **posteriores condicionales**, aplicando Bayes *antes de conocer la respuesta real*:

$$P(H_i \mid A, q) = \frac{P(A \mid H_i, q)\, P(H_i)}{P(A \mid q)}$$

$$P(H_i \mid F, q) = \frac{P(F \mid H_i, q)\, P(H_i)}{P(F \mid q)}$$

Con estos posteriores calculamos la entropía en cada escenario:

$$H_A(q) = -\sum_{i} P(H_i \mid A, q) \log_2 P(H_i \mid A, q)$$

$$H_F(q) = -\sum_{i} P(H_i \mid F, q) \log_2 P(H_i \mid F, q)$$

La entropía esperada tras formular la pregunta $q$ es:

$$\mathbb{E}\!\left[H(\mathbf{p}' \mid q)\right] = P(A \mid q) \cdot H_A(q) + P(F \mid q) \cdot H_F(q)$$

Y la ganancia de información es:

$$IG(q) = H(\mathbf{p}) - \left[P(A \mid q) \cdot H_A(q) + P(F \mid q) \cdot H_F(q)\right]$$

El sistema selecciona la pregunta $q^*$ con mayor $IG$:

$$q^* = \arg\max_{q \in \mathcal{Q}} IG(q)$$

donde $\mathcal{Q}$ es el conjunto de preguntas disponibles.

### 6.3. Conexión con la información mutua

La ganancia esperada de información de la pregunta $q$ es exactamente la **información mutua** entre la respuesta $R_q$ y el estado del alumno $H$:

$$IG(q) = I(H;\, R_q \mid \mathbf{p}) = \sum_{r \in \{A,F\}} P(r \mid q) \cdot KL\!\left(P(H \mid r, q) \;\|\; P(H)\right)$$

donde $KL$ es la divergencia de Kullback-Leibler. Maximizar $IG$ equivale a seleccionar la pregunta cuya respuesta, en promedio, más separa el posterior del prior.

### 6.4. Empates y diversidad de contenidos

En la práctica, varias preguntas pueden tener ganancias de información idénticas o muy próximas, especialmente si comparten los mismos parámetros $a$, $b_q$ y $c_q$. Una selección determinista entre empates produce tests sistemáticamente repetitivos entre sesiones distintas.

La solución recomendada es una **selección aleatoria ponderada**:

1. Calcular $IG(q)$ para todas las candidatas disponibles.
2. Reunir las candidatas con $IG \geq IG_{\max} - \varepsilon$ (por ejemplo $\varepsilon = 10^{-9}$).
3. Asignar a cada candidata un peso inversamente proporcional al número de veces que su categoría o concepto ha aparecido ya en la sesión.
4. Elegir con probabilidad proporcional a esos pesos.

Esto combina máxima utilidad informativa con diversidad temática.

---

## 7. Criterio de parada

### 7.1. Objetivo del umbral

El sistema debe detenerse cuando tenga suficiente confianza en el estado del alumno. El criterio natural es: detener cuando la hipótesis más probable supere un nivel de confianza $p_{\min}$ (por ejemplo, 0.80).

Sin embargo, comprobar directamente $\max_i P(H_i) \geq p_{\min}$ puede no ser suficiente, porque no tiene en cuenta cómo se reparte la probabilidad restante. La entropía es un indicador más completo.

### 7.2. Derivación del umbral $H_{\text{stop}}$

Buscamos la entropía de una distribución en la que la hipótesis más probable tiene probabilidad $p_{\min}$ y la probabilidad restante $1 - p_{\min}$ se reparte **uniformemente** entre las otras $n - 1$ hipótesis:

$$p_k = p_{\min}, \quad p_{i \neq k} = \frac{1 - p_{\min}}{n - 1}$$

La entropía de esta distribución es:

$$H_{\text{stop}} = -p_{\min} \log_2 p_{\min} - (n-1) \cdot \frac{1 - p_{\min}}{n-1} \log_2\!\left(\frac{1 - p_{\min}}{n-1}\right)$$

$$H_{\text{stop}} = -p_{\min} \log_2 p_{\min} - (1 - p_{\min}) \log_2\!\left(\frac{1 - p_{\min}}{n-1}\right)$$

### 7.3. Valores orientativos

Con $p_{\min} = 0.80$:

| $n$ hipótesis | $H_{\max} = \log_2 n$ (bits) | $H_{\text{stop}}$ (bits) | Fracción de incertidumbre restante |
|:---:|:---:|:---:|:---:|
| 2 | 1.00 | 0.72 | 72% |
| 3 | 1.58 | 0.92 | 58% |
| 4 | 2.00 | 1.04 | 52% |
| 5 | 2.32 | 1.12 | 48% |

### 7.4. Aproximación y criterio complementario

La fórmula de $H_{\text{stop}}$ asume que la probabilidad restante se distribuye uniformemente, lo cual es una aproximación. Dos distribuciones con el mismo máximo pueden tener entropías distintas:

- $(0.80,\; 0.10,\; 0.10)$: $H = 0.92$ bits
- $(0.80,\; 0.19,\; 0.01)$: $H = 0.78$ bits

La segunda tiene entropía menor aunque el máximo sea el mismo, porque la probabilidad está más concentrada. Por tanto, **conviene comprobar ambos criterios de forma complementaria**:

$$\text{parar si } H(\mathbf{p}) < H_{\text{stop}} \text{ Y } \max_i P(H_i) \geq p_{\min}$$

Ambos criterios no son completamente independientes: cuando $H_{\text{stop}}$ se deriva del mismo $p_{\min}$, el criterio de entropía no añade una exigencia distinta al de probabilidad máxima. El criterio de probabilidad máxima garantiza el nivel mínimo de confianza en la hipótesis ganadora; la entropía actúa como control complementario de la incertidumbre global. Como alternativa, puede añadirse un criterio de separación:

$$P(H_{\text{ganadora}}) - P(H_{\text{segunda}}) \geq \Delta_{\min}$$

que exige no solo que la hipótesis más probable supere $p_{\min}$, sino que esté suficientemente separada de la segunda candidata.

### 7.5. Criterios adicionales de parada

Además del criterio de entropía, el sistema debe contemplar:

- **Límite máximo de preguntas**: evita sesiones infinitas cuando la convergencia es lenta.
- **Ganancia mínima útil**: si la mejor pregunta disponible tiene $IG < \varepsilon_{\min}$, el banco de preguntas se ha agotado informativamente y continuar no mejora el diagnóstico.
- **Cobertura mínima de conceptos**: asegurar que ciertos conceptos obligatorios hayan sido evaluados.

---

## 8. Convención de escala entre θ y b

### 8.1. El problema del punto de inflexión

La curva logística del IRT 3PL tiene su pendiente máxima en $\theta_i = b_q$. Esto significa que la pregunta $q$ es **más discriminante** para alumnos cuyo nivel es cercano a la dificultad $b_q$.

Si el nivel extremo $\theta_{\max}$ coincide con la dificultad extrema $b_{\max}$, la pregunta más difícil sitúa al nivel avanzado justo en el punto de inflexión. En ese punto la probabilidad de acierto todavía no es alta: con 4 opciones y $c_q = 0.25$, vale $(1+c_q)/2 = 0.625$. Por tanto, acertar esa pregunta confirma débilmente el nivel avanzado.

Esto puede hacer que el sistema infrautilice las preguntas extremas: no porque sean inútiles, sino porque otras preguntas pueden producir una reducción esperada de incertidumbre mayor.

### 8.2. La regla del factor 2

Para evitar este problema, el rango de $\theta$ debe ser **estrictamente mayor** que el rango de $b$. En este sistema discreto, se adopta como convención práctica:

$$|\theta_{\max}| = 2 \cdot \max_q |b_q|$$

El factor 2 es una heurística útil, no una regla universal de IRT estándar. Puede ajustarse según el valor de $a$, de $c_q$ y de la probabilidad objetivo de acierto para los niveles extremos. Si se desea que un alumno de nivel extremo tenga una probabilidad objetivo $P^*$ de acertar una pregunta extrema, puede despejarse:

$$\theta_{\max} - b_{\max} = \frac{1}{a}\operatorname{logit}\!\left(\frac{P^* - c}{1 - c}\right), \qquad \operatorname{logit}(x) = \ln\frac{x}{1-x}$$

Así la separación entre escalas queda justificada por una probabilidad deseada, no por un factor fijo.

**Ejemplo.** Con 3 niveles de dificultad $b \in \{-1,\; 0,\; +1\}$:

- Escala incorrecta: $\theta \in \{-1,\; 0,\; +1\}$ → el nivel avanzado y la pregunta difícil tienen el mismo valor; la curva logística da $P(A) = (1 + c_q)/2 \approx 0.625$ para 4 opciones, un valor mediocre para confirmar el nivel avanzado.
- Escala correcta: $\theta \in \{-2,\; 0,\; +2\}$ → el nivel avanzado supera con holgura la dificultad extrema; $P(A) = c_q + (1-c_q) \cdot \sigma(1.5 \cdot (2-1)) \approx 0.82$, un valor suficientemente alto para que la pregunta sea informativa.

donde $\sigma(x) = 1/(1 + e^{-x})$ es la función sigmoide estándar.

### 8.3. Cálculo automático desde el banco de preguntas

La forma más robusta es calcular $\theta_{\max}$ directamente a partir de las dificultades del banco:

$$\theta_{\max} = 2 \cdot \max_{q} |b_q|$$

y distribuir los valores $\theta_i$ uniformemente en $[-\theta_{\max},\; +\theta_{\max}]$. Así la escala se adapta automáticamente al banco de preguntas, sin necesidad de ajuste manual.

---

## 9. Ejemplo numérico completo

### 9.1. Configuración

- **Hipótesis**: $H_1$ (básico), $H_2$ (medio), $H_3$ (avanzado)
- **Valores**: $\theta_1 = -2$, $\theta_2 = 0$, $\theta_3 = +2$
- **Prior inicial**: $\mathbf{p}^{(0)} = (0.333,\; 0.333,\; 0.333)$
- **Parámetro de discriminación**: $a = 1.5$
- **Criterio de parada**: $p_{\min} = 0.80$, con un mínimo de 2 preguntas antes de poder finalizar
- **Banco de preguntas** (simplificado):

| ID | Dificultad $b_q$ | Opciones $m_q$ | $c_q$ |
|:---:|:---:|:---:|:---:|
| Q1 | $-1$ (fácil) | 4 | 0.25 |
| Q2 | $0$ (media) | 4 | 0.25 |
| Q3 | $+1$ (difícil) | 4 | 0.25 |

### 9.2. Cálculo de verosimilitudes

Para cada par $(\theta_i, b_q)$ calculamos $P(A \mid \theta_i, q)$ con la fórmula IRT 3PL, $a = 1.5$, $c = 0.25$:

$$P(A \mid \theta_i, q) = 0.25 + 0.75 \cdot \frac{1}{1 + e^{-1.5(\theta_i - b_q)}}$$

**Para Q1** ($b_1 = -1$):

| Hipótesis | $\theta_i - b_q$ | $e^{-1.5 \cdot x}$ | $\sigma$ | $P(A)$ | $P(F)$ |
|:---:|:---:|:---:|:---:|:---:|:---:|
| $H_1$ | $-2 - (-1) = -1$ | $e^{1.5} = 4.48$ | $0.182$ | $0.387$ | $0.613$ |
| $H_2$ | $0 - (-1) = +1$ | $e^{-1.5} = 0.223$ | $0.818$ | $0.864$ | $0.136$ |
| $H_3$ | $2 - (-1) = +3$ | $e^{-4.5} = 0.011$ | $0.989$ | $0.992$ | $0.008$ |

**Para Q2** ($b_2 = 0$):

| Hipótesis | $\theta_i - b_q$ | $P(A)$ | $P(F)$ |
|:---:|:---:|:---:|:---:|
| $H_1$ | $-2$ | $0.286$ | $0.714$ |
| $H_2$ | $0$ | $0.625$ | $0.375$ |
| $H_3$ | $+2$ | $0.964$ | $0.036$ |

**Para Q3** ($b_3 = +1$):

| Hipótesis | $\theta_i - b_q$ | $P(A)$ | $P(F)$ |
|:---:|:---:|:---:|:---:|
| $H_1$ | $-3$ | $0.258$ | $0.742$ |
| $H_2$ | $-1$ | $0.387$ | $0.613$ |
| $H_3$ | $+1$ | $0.864$ | $0.136$ |

### 9.3. Selección de la primera pregunta

Con prior uniforme $\mathbf{p} = (0.333, 0.333, 0.333)$, calculamos la ganancia de información para cada pregunta.

**Entropía inicial:**

$$H^{(0)} = -3 \cdot 0.333 \cdot \log_2(0.333) = \log_2 3 = 1.585 \text{ bits}$$

**Para Q1** ($b = -1$):

$$P(A \mid Q1) = 0.333 \cdot 0.387 + 0.333 \cdot 0.864 + 0.333 \cdot 0.992 = 0.748$$

$$P(F \mid Q1) = 0.252$$

Posteriores tras acierto:

$$P(H_1 \mid A) = \frac{0.387 \cdot 0.333}{0.748} = 0.172, \quad P(H_2 \mid A) = \frac{0.864 \cdot 0.333}{0.748} = 0.385, \quad P(H_3 \mid A) = 0.443$$

Entropía posterior si acierta:

$$H_A(Q1) = -(0.172 \log_2 0.172 + 0.385 \log_2 0.385 + 0.443 \log_2 0.443) = 1.488 \text{ bits}$$

Posteriores tras fallo:

$$P(H_1 \mid F) = \frac{0.613 \cdot 0.333}{0.252} = 0.809, \quad P(H_2 \mid F) = 0.180, \quad P(H_3 \mid F) = 0.011$$

Entropía posterior si falla:

$$H_F(Q1) = -(0.809 \log_2 0.809 + 0.180 \log_2 0.180 + 0.011 \log_2 0.011) = 0.764 \text{ bits}$$

Ganancia de información:

$$IG(Q1) = 1.585 - (0.748 \cdot 1.488 + 0.252 \cdot 0.764) = 1.585 - 1.305 = 0.280 \text{ bits}$$

**Resumen de ganancias** (siguiendo el mismo procedimiento para Q2 y Q3):

| Pregunta | $IG$ (bits) |
|:---:|:---:|
| Q1 ($b = -1$) | **0.280** |
| Q2 ($b = 0$) | $0.275$ |
| Q3 ($b = +1$) | $0.212$ |

Con $c_q = 0.25$, la probabilidad de acierto por azar rompe la simetría entre preguntas fáciles y difíciles: un fallo en una pregunta fácil es muy diagnóstico, mientras que un acierto en una pregunta difícil todavía puede explicarse parcialmente por azar. En esta configuración, **el sistema selecciona Q1**, aunque Q2 queda prácticamente empatada.

### 9.4. Primera respuesta: fallo en Q1

El alumno falla Q1. Actualizamos:

$$P(H_1 \mid F, Q1) = \frac{0.613 \cdot 0.333}{0.613 \cdot 0.333 + 0.137 \cdot 0.333 + 0.008 \cdot 0.333} = \frac{0.204}{0.253} = 0.809$$

$$P(H_2 \mid F, Q1) = \frac{0.137 \cdot 0.333}{0.253} = 0.180, \quad P(H_3 \mid F, Q1) = \frac{0.008 \cdot 0.333}{0.253} = 0.011$$

$$\mathbf{p}^{(1)} = (0.809,\; 0.180,\; 0.011)$$

Entropía tras el fallo:

$$H^{(1)} = -(0.809 \log_2 0.809 + 0.180 \log_2 0.180 + 0.011 \log_2 0.011) = 0.764 \text{ bits}$$

La entropía ha bajado de 1.585 a 0.764 bits. El sistema sospecha con bastante fuerza que el alumno es de nivel básico.

### 9.5. Segunda pregunta y acierto en Q2

Con $\mathbf{p}^{(1)} = (0.809, 0.180, 0.011)$, el sistema vuelve a calcular las ganancias para Q2 y Q3 (Q1 ya se ha usado). En este estado asimétrico, Q2 resulta más informativa porque ayuda a revisar si el fallo anterior pudo ser accidental.

El alumno **acierta Q2** (media). Actualizamos con $P(A \mid H_i, Q2) = (0.286, 0.625, 0.964)$:

$$P(A \mid Q2, \mathbf{p}^{(1)}) = 0.809 \cdot 0.286 + 0.180 \cdot 0.625 + 0.011 \cdot 0.964 = 0.231 + 0.113 + 0.011 = 0.354$$

$$P(H_1 \mid A, Q2) = \frac{0.286 \cdot 0.809}{0.354} = 0.652, \quad P(H_2 \mid A, Q2) = \frac{0.625 \cdot 0.180}{0.354} = 0.318, \quad P(H_3 \mid A, Q2) = 0.030$$

$$\mathbf{p}^{(2)} = (0.652,\; 0.318,\; 0.030)$$

$$H^{(2)} = -(0.652 \log_2 0.652 + 0.318 \log_2 0.318 + 0.030 \log_2 0.030) = 1.080 \text{ bits}$$

El acierto en la pregunta media desplaza parte de la probabilidad hacia $H_2$, pero el fallo inicial en una pregunta fácil sigue pesando mucho. El diagnóstico queda entre básico y medio-bajo.

### 9.6. Evolución de la sesión

| Paso | Acción | $P(H_1)$ | $P(H_2)$ | $P(H_3)$ | $H$ (bits) |
|:---:|:---|:---:|:---:|:---:|:---:|
| 0 | Prior inicial | 0.333 | 0.333 | 0.333 | 1.585 |
| 1 | Falla Q1 (fácil) | 0.809 | 0.180 | 0.011 | 0.764 |
| 2 | Acierta Q2 (media) | 0.652 | 0.318 | 0.030 | 1.080 |

Nótese que el acierto en la pregunta media ha subido la entropía respecto al paso 1: la evidencia ha repartido más la probabilidad entre $H_1$ y $H_2$, aumentando la incertidumbre. Esto es correcto: el sistema sigue descartando casi por completo el nivel avanzado, pero ahora duda más entre básico y medio.

Con $p_{\min} = 0.80$ y $n = 3$, el umbral es $H_{\text{stop}} = 0.92$ bits. La entropía actual (1.080 bits) está por encima del umbral, así que el test continúa.

---

## 10. Hipótesis no jerárquicas

La función logística IRT 3PL asume que las hipótesis tienen un orden: más θ significa más nivel. Esto es adecuado para evaluar dominio, pero no cuando las hipótesis son errores conceptuales alternativos sin relación de orden.

**Ejemplo.** Supongamos tres hipótesis:

- $H_A$: confunde masa con peso
- $H_B$: confunde velocidad con aceleración  
- $H_C$: dominio correcto

En este caso no existe una escala única de «más o menos nivel». La función logística no es el modelo apropiado.

**Alternativa.** Definir las verosimilitudes directamente según el diagnóstico esperado de cada pregunta:

| Pregunta | $P(A \mid H_A)$ | $P(A \mid H_B)$ | $P(A \mid H_C)$ |
|:---|:---:|:---:|:---:|
| ¿La masa de un objeto cambia en la Luna? | 0.20 | 0.80 | 0.95 |
| ¿Un coche frenando tiene aceleración? | 0.75 | 0.15 | 0.95 |
| ¿La fuerza es proporcional a la masa? | 0.50 | 0.50 | 0.90 |

Estas verosimilitudes las define el docente o la IA a partir del conocimiento sobre qué errores produce cada confusión. La **actualización bayesiana es idéntica**; solo cambia la fuente de las verosimilitudes.

---

## 11. Límites del modelo

### 11.1. Calibración inicial

Las verosimilitudes generadas por la función logística con parámetros asignados por un experto son **estimaciones a priori**, no medidas empíricas. Un alumno real puede comportarse de forma diferente a lo que predice el modelo.

Si se acumulan datos reales (respuestas de muchos alumnos), los parámetros $a$, $b_q$ y $c_q$ pueden recalibrarse mediante métodos de estimación IRT (máxima verosimilitud marginal, estimación bayesiana de parámetros). Sin datos suficientes, el modelo es una aproximación razonable pero no una medida precisa.

### 11.2. Independencia condicional

La actualización bayesiana secuencial asume que las respuestas son **condicionalmente independientes** dada la hipótesis verdadera: conocer $H_i$ hace irrelevante la correlación entre respuestas. Esta asunción se viola cuando:

- Las preguntas tratan el mismo concepto muy específico y la respuesta a una da pistas sobre la siguiente.
- El alumno aprende durante el test (la hipótesis verdadera cambia).
- El alumno responde al azar en preguntas que no sabe, con patrones que dependen de la fatiga o del tiempo.

### 11.3. Estabilidad de las hipótesis

El modelo asume que el estado del alumno no cambia durante la sesión. En sesiones cortas de evaluación diagnóstica esto es razonable. En sesiones largas de aprendizaje adaptativo, el alumno puede mejorar durante la propia interacción, lo que haría que el posterior converja hacia una hipótesis que ya no refleja el estado actual.

### 11.4. Respuestas al azar

Si el alumno responde al azar sistemáticamente, el parámetro $c_q$ solo protege parcialmente: reduce el sesgo upward en las verosimilitudes de acierto, pero no elimina el ruido. Con suficientes respuestas aleatorias, el posterior puede converger hacia hipótesis incorrectas.

### 11.5. Número mínimo de preguntas

Con pocas preguntas, el posterior puede quedar sesgado por coincidencias (una racha de aciertos o fallos no representativa). Imponer un **número mínimo de preguntas** antes de activar el criterio de parada reduce este riesgo, a costa de alargar la sesión.

### 11.6. El modelo no sustituye al criterio docente

El sistema produce una **estimación probabilística**, no una verdad absoluta. Los resultados deben interpretarse como una ayuda a la decisión educativa, especialmente cuando:

- La sesión fue corta o el banco de preguntas era pequeño.
- La entropía final supera el umbral $H_{\text{stop}}$ (diagnóstico provisional).
- El alumno mostró un patrón de respuestas inusual.
- Las preguntas no habían sido calibradas con datos reales.

---

## Referencias y lecturas adicionales

- **Birnbaum, A.** (1968). Some latent trait models and their use in inferring an examinee's ability. En Lord & Novick, *Statistical Theories of Mental Test Scores*. Addison-Wesley.
- **Cover, T. M. & Thomas, J. A.** (2006). *Elements of Information Theory* (2.ª ed.). Wiley. [Entropía de Shannon e información mutua.]
- **van der Linden, W. J. & Hambleton, R. K.** (Eds.) (1997). *Handbook of Modern Item Response Theory*. Springer.
- **Wainer, H.** (Ed.) (2000). *Computerized Adaptive Testing: A Primer* (2.ª ed.). Lawrence Erlbaum. [Fundamentos de la evaluación adaptativa.]
- **Gelman, A. et al.** (2013). *Bayesian Data Analysis* (3.ª ed.). CRC Press. [Inferencia bayesiana general.]

---

*Licencia: [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · Juan José de Haro · [bilateria.org](https://bilateria.org)*
