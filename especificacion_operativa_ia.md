# Especificación operativa para IA

**Versión 2.0**

## Propósito

Este documento sirve para que una IA implemente recursos educativos adaptativos bayesianos de forma fiable.  
Es una especificación operativa breve.  
Si hace falta fundamento teórico, ejemplos o justificación matemática, consulta `documentacion.html` y `matematicas.html`.
Si hubiera conflicto entre ambos documentos, prevalece este.

## Instrucción de uso

Adjunta este documento a la IA y usa este prompt:

> Lee el documento adjunto e implementa el recurso siguiendo las reglas operativas aplicables al tipo de recurso solicitado.

## Flujo obligatorio

1. Antes de implementar, comprueba si tienes esta información:
   - tema;
   - curso o edad;
   - objetivo de aprendizaje;
   - tipo de recurso;
   - finalidad principal;
   - número de hipótesis o niveles;
   - tipo de interacción;
   - número aproximado de preguntas o pasos;
   - salida final esperada.
2. Si falta información esencial, pregunta solo por lo imprescindible.
3. Si la información ya está dada, no repitas preguntas y pasa a implementar.
4. El resultado debe ser un recurso web estático en `HTML + CSS + JavaScript`, preferentemente sin backend, salvo que se pida explícitamente otro formato. Puede organizarse en uno o varios archivos si eso mejora la claridad, el mantenimiento o la reutilización.

## Reglas de diseño

- El recurso no debe ser lineal si la finalidad exige adaptación.
- Cada respuesta del alumno debe modificar el estado estimado del sistema.
- Interpreta cada respuesta como evidencia parcial sobre una o varias hipótesis, no como prueba directa de conocimiento.
- La adaptación puede afectar a:
  - la siguiente pregunta;
  - la dificultad;
  - el tipo de actividad;
  - la explicación;
  - la ayuda o las pistas;
  - el itinerario;
  - el momento de finalizar.
- El resultado final debe ser pedagógico, no solo una puntuación.

## Estado del alumno

- Representa el estado del alumno como una distribución de probabilidad sobre `n` hipótesis.
- Si no hay información previa fiable, usa distribución uniforme.
- Si las hipótesis son jerárquicas, asígnales valores `theta` ordenados y centrados.
- Si el docente no fija valores, usa una escala simétrica centrada en 0.
- La escala `theta` es fija y depende solo del número de hipótesis, no del banco de preguntas. Con `n` hipótesis jerárquicas, usa valores centrados en `0` con intervalos de `2`: `theta_i = 2 * (i - 1) - (n - 1)`, es decir, `theta` recorre `{-(n-1), ..., +(n-1)}` y `theta_max = n - 1`. Ejemplos: `n = 2` → `{-1, +1}`; `n = 3` → `{-2, 0, +2}`; `n = 4` → `{-3, -1, +1, +3}`; `n = 5` → `{-4, -2, 0, +2, +4}`.
- Sitúa las dificultades dentro de la mitad central de esa escala: `b_q` debe quedar en `[-theta_max / 2, +theta_max / 2]`. Como `theta_max` depende de `n`, la conversión de dificultades también depende de `n`, no solo del número de categorías.
- Si el docente da `k` categorías cualitativas de dificultad, repártelas uniformemente en ese intervalo: `b_j = (theta_max / 2) * (2 * (j - 1) / (k - 1) - 1)` para `j = 1..k` (si `k = 1`, usa `b = 0`). Ejemplos: `n = 3` y `k = 3` → `{-1, 0, +1}`; `n = 3` y `k = 5` → `{-1, -0.5, 0, +0.5, +1}`; `n = 4` y `k = 3` → `{-1.5, 0, +1.5}`; `n = 2` y `k = 5` → `{-0.5, -0.25, 0, +0.25, +0.5}`.
- No uses una tabla de dificultades fija e independiente de `n`: con pocas hipótesis y muchas categorías produciría dificultades fuera del rango de niveles (por ejemplo, `n = 2` con `b = ±2`) y anularía la separación entre escalas.
- Si el docente da valores numéricos de `b_q` fuera del intervalo, recórtalos (clamp) al intervalo.
- No recalcules nunca `theta` a partir de los extremos del banco. Una sola pregunta atípicamente difícil o fácil no debe redefinir la escala: si estiras `theta` para acomodarla, saturas las verosimilitudes del resto del banco (todas las probabilidades quedan pegadas a `c_q` o a `1`) y el posterior da saltos sobreconfiados con una sola respuesta.
- Si la mayoría de las dificultades del banco quedara fuera del intervalo, no estires la escala: revisa con el docente la definición de niveles y dificultades, porque el diseño es incoherente.
- Invariante de comparabilidad: con `a_ef = 1.25` e intervalos de `2` entre hipótesis adyacentes, el producto `a_ef * Δtheta = 2.5` determina la fuerza máxima de una actualización, sea cual sea `n` (al crecer `n` crece `theta_max`, pero el espaciado entre hipótesis adyacentes se mantiene). Mantén este invariante entre recursos para que las confianzas y velocidades de convergencia sean comparables.

## Actualización bayesiana

Tras cada respuesta:

1. calcula la verosimilitud de esa respuesta bajo cada hipótesis;
2. multiplica prior por verosimilitud;
3. normaliza;
4. usa el resultado como nuevo estado.

Esto debe hacerse después de cada interacción relevante.

## Elección del modelo

- Si la finalidad principal es estimar un nivel ordenado de dominio, usa hipótesis ordinales e IRT 3PL.
- Si la finalidad principal es identificar una estrategia o error excluyente, usa hipótesis nominales con verosimilitudes explícitas por pregunta.
- Si varios errores, habilidades o necesidades pueden coexistir, usa dimensiones separadas o perfiles completos.
- Si el recurso debe estimar nivel y diagnosticar errores, combina una distribución ordinal global con distribuciones diagnósticas paralelas.
- No fuerces errores coexistentes dentro de una escala ordinal única.

## Verosimilitudes

- Si las hipótesis representan niveles ordenados de dominio, usa IRT 3PL.
- Fórmula recomendada:

`P(acierto | H_i, q) = c_q + (1 - c_q) / (1 + exp(-a * (theta_i - b_q)))`

- Usa por defecto:
  - `a_ef = 1.25` (discriminación efectiva objetivo)
  - `a = a_ef / (1 - c_q)`
  - `c_q = 1 / m_q` si hay azar
  - `c_q = 0` si no lo hay
- No fijes `a` directamente: fija la discriminación efectiva objetivo `a_ef = 1.25` y calcula `a` por pregunta con `a = a_ef / (1 - c_q) = 1.25 / (1 - c_q)`.
  - Hazlo **siempre**, aunque todas las preguntas tengan el mismo número de opciones. Esta regla iguala la pendiente máxima de la ICC, no garantiza que todos los formatos aporten la misma información esperada. Un `a` fijo con distintos `c_q` produce pendientes reales distintas y no comparables (por ejemplo, `a=1.5` fijo da `a_ef=1.0` con 3 opciones pero `a_ef=1.125` con 4). La selección por ganancia de información seguirá favoreciendo los ítems que aporten más evidencia.
  - `a_ef = 1.25` garantiza que `a` se mantenga dentro del rango habitual en psicometría (0.5–2.5) para cualquier formato de 2 o más opciones: el caso extremo, verdadero/falso (`c_q=0.5`), da exactamente `a = 2.5`.
  - Valores que debes obtener: abierta (`c_q=0`) → `a=1.25`; 5 opciones (`c_q=0.20`) → `a=1.5625`; 4 opciones (`c_q=0.25`) → `a≈1.667`; 3 opciones (`c_q=1/3`) → `a=1.875`; verdadero/falso (`c_q=0.5`) → `a=2.5`.
- Si el alumno falla:

`P(fallo | H_i, q) = 1 - P(acierto | H_i, q)`

- Si las hipótesis no son jerárquicas, distingue dos casos.
- Si son **alternativas excluyentes** (por ejemplo, estrategia A / estrategia B / dominio correcto), no uses IRT logística. Genera para cada pregunta un vector de `n` verosimilitudes `P(acierto | H_i, q)`, una por hipótesis.
- Si los errores o necesidades **pueden coexistir**, no los metas en una sola distribución nominal. Modela una dimensión por factor (`error largo`, `error cero`, etc.) o, de forma equivalente, una distribución sobre **perfiles completos** (`2^k` combinaciones posibles de `k` factores).
- En ese caso multifactorial, cada pregunta debe definir cómo responde cada perfil. Lo mínimo es una probabilidad de acierto por perfil; mejor aún, una distribución por opción `P(R = r | perfil, q)` para aprovechar qué distractor ha elegido el alumno, no solo si acertó o falló.
- Si partes de factores simples, asigna cada valor respondiendo: «si el alumno tuviera este error, ¿con qué probabilidad acertaría esta pregunta?». Bajo si la pregunta ataca el concepto que ese error distorsiona; alto si el error no interfiere.
- Acota cada valor: no por debajo del suelo de azar `1/m` (m opciones), salvo la excepción del punto siguiente; la hipótesis o perfil de dominio en torno a `0.9`–`0.95`, nunca `1`.
- Si un distractor concreto es la respuesta que produce un error, la probabilidad de **esa opción** debe subir bajo ese perfil, y la de acierto puede quedar cerca del suelo o por debajo: el alumno es atraído activamente hacia esa respuesta equivocada.
- No afines el decimal: usa tramos (`≈0.9` no afecta / `≈0.5` afectación parcial / `≈0.15–0.25` distractor que captura el error / `≈1/m` suelo si falla por azar). Lo importante es que los perfiles o factores correctos se separen claramente en las preguntas que discriminan.
- No uses tablas fijas globales si cada pregunta puede generar sus propias verosimilitudes.
- La calidad del diagnóstico no depende solo del algoritmo: depende también de cómo estén definidas las hipótesis, categorías, dificultades, conceptos y errores. Si esas clasificaciones están mal conformadas, si los errores relevantes no están bien identificados o si el banco cubre mal los casos importantes, las verosimilitudes generadas pueden representar mal la realidad y sesgar la adaptación.
- En el diagnóstico final no calcules una `theta` esperada (no tiene sentido sin orden). Si el modelo es nominal excluyente, puedes reportar la hipótesis MAP y su probabilidad. Si el modelo es multifactorial, reporta por cada factor si queda **presente**, **ausente** o **indeterminado**, con su probabilidad marginal o confianza asociada.

## Respuestas con crédito parcial

- Si una respuesta no es solo acierto o fallo, sino que admite grados (varios pasos, componentes ponderados, aciertos parciales), resúmela en una puntuación `s` entre `0` y `1`.
- Si solo tienes una puntuación parcial agregada `s`, construye una verosimilitud geométrica:

`L(H_i) = P(acierto | H_i, q)^s * P(fallo | H_i, q)^(1 - s)`

- Usa esta `L(H_i)` en la actualización bayesiana en lugar de elegir entre `P(acierto)` y `P(fallo)`. La normalización y el resto del proceso no cambian.
- Casos límite: `s = 1` equivale a acierto pleno; `s = 0` a fallo pleno. Un `s = 0.5` no es necesariamente neutro: favorece las hipótesis para las que el ítem predice una puntuación intermedia.
- Si el ítem tiene `J` componentes aproximadamente independientes y `s` es la fracción ponderada de componentes correctos, puedes conservar la fuerza de la evidencia usando `L(H_i) = P(acierto | H_i, q)^(sJ) * P(fallo | H_i, q)^((1 - s)J)`.
- Si tienes evidencia separada por componente, es preferible multiplicar las verosimilitudes de cada componente en lugar de reducir todo a una sola `s`.
- Define cómo se calcula `s` de forma explícita y autocorregible: suma ponderada de subcriterios, fracción de pasos correctos, cercanía a la solución numérica, etc. Los pesos deben sumar `1`.
- No trates como binaria una respuesta que admite grados: pierdes información diagnóstica.
- Para seleccionar la siguiente pregunta, calcula la ganancia de información con los resultados que realmente modele el ítem. Si trabajas con binario o crédito parcial, la aproximación con acierto y fallo plenos sigue siendo aceptable; si modelas distractores o perfiles completos, promedia sobre todas las respuestas relevantes del ítem.

## Suelo de azar en ítems compuestos

- Si un ítem se corrige por varios componentes con distinto número de opciones y se puntúa con crédito parcial, el suelo agregado no es la probabilidad de acertar el ítem completo, sino la puntuación parcial esperada por azar.
- Calcula ese suelo de puntuación esperada como media ponderada de los azares de cada componente:

`c_q = Σ_j w_j * c_j`  con  `c_j = 1 / m_j`  y  `Σ_j w_j = 1`

- Usa ese `c_q` agregado en la función logística solo cuando la ICC represente la puntuación esperada del ítem compuesto.
- Si el ítem compuesto se corrige como todo-o-nada, no uses la media ponderada: la probabilidad de acierto pleno por azar es el producto `Π_j c_j` si los componentes se aciertan independientemente.
- Los pesos `w_j` deben coincidir con los que uses para calcular la puntuación `s` del crédito parcial.

## Diagnóstico multidimensional

- Si necesitas saber no solo el nivel global sino qué habilidades o pasos fallan, mantén varias distribuciones bayesianas en paralelo: una por categoría o nivel y otra por cada dimensión diagnóstica (habilidad, paso, tipo de error).
- Actualiza cada distribución con la evidencia que le corresponde: el resultado global puede alimentar la creencia de nivel; cada subcriterio debe alimentar solo su dimensión. No uses el mismo acierto/fallo global para actualizar varias dimensiones independientes si la pregunta exige varias habilidades a la vez, porque duplicas evidencia y atribuyes mal la causa del error.
- Cada dimensión puede tener su propio suelo de azar `c` según su número de opciones, por lo que sus porcentajes no son directamente comparables entre sí: la referencia común es el valor latente `theta`.
- No metas en una sola distribución dimensiones que pueden coexistir: usa distribuciones separadas.
- Si varias dimensiones diagnósticas interactúan de manera fuerte, puedes sustituir las distribuciones independientes por una sola distribución sobre perfiles completos; lo importante es no forzar como excluyentes factores que en realidad pueden coexistir.
- El nivel por categoría guía qué practicar; el diagnóstico por dimensión guía qué explicar o reforzar.

## Preguntas y actividades

Cada pregunta o interacción autocorregible debe tener, cuando proceda:

- texto o enunciado;
- dificultad `b_q`;
- número de opciones `m_q`;
- categoría o concepto;
- criterio de corrección;
- ayuda o pista opcional;
- retroalimentación mínima tras la respuesta;
- explicación específica, especialmente si la finalidad principal es aprendizaje, práctica o refuerzo.

Si el recurso es procedural o tutorial, las interacciones pueden no ser preguntas clásicas, pero deben seguir siendo autocorregibles o evaluables de forma explícita.

## Selección adaptativa

Para cada candidata disponible:

1. calcula la probabilidad marginal de acierto;
2. simula posterior si hay acierto;
3. simula posterior si hay fallo;
4. calcula la entropía esperada posterior;
5. calcula la ganancia esperada de información.

Selecciona la candidata con mayor ganancia esperada de información cuando la finalidad principal sea diagnóstica y el objetivo sea estimar un nivel global.

En recursos de práctica adaptativa o refuerzo con varias categorías, tipos de problema o conceptos, usa una selección en dos fases:

1. **Fase diagnóstica inicial.** Hasta que cada categoría relevante tenga una muestra mínima de intentos, prioriza las categorías con menos evidencia. Dentro de ellas, usa la ganancia esperada de información para elegir la pregunta más diagnóstica. Un valor por defecto razonable es exigir al menos `2` intentos por categoría antes de salir de esta fase. Si hay muchas categorías, esta fase puede consumir la sesión (con `10` categorías y `2` intentos son ya `20` preguntas): agrupa categorías afines en bloques o reduce la muestra mínima para que el diagnóstico inicial no supere el máximo práctico.
2. **Fase de refuerzo.** Cuando todas las categorías relevantes tengan muestra mínima, prioriza la categoría con menor dominio estimado. Dentro de esa categoría, no elijas automáticamente la pregunta más difícil: selecciona una pregunta informativa y cercana al nivel estimado del alumno. Una regla razonable es combinar ganancia de información con adecuación de dificultad, penalizando preguntas demasiado alejadas de la zona de trabajo.

No uses la entropía de Shannon como único criterio permanente cuando la finalidad principal sea practicar o reforzar. Shannon indica dónde hay más incertidumbre diagnóstica; el refuerzo debe atender también, y preferentemente, a lo que el alumno domina menos.

En tests adaptativos de diagnóstico global con criterio de parada, no es obligatorio aplicar la fase de refuerzo. En ese caso basta con maximizar la información esperada, diversificando categorías en empates o cuando varias candidatas tengan utilidad equivalente.

Ten en cuenta que maximizar la información esperada tiende a proponer preguntas con probabilidad de acierto cercana al `50 %`. Con alumnado joven o con dificultades, puedes abrir con una pregunta asequible o intercalar alguna de éxito probable: pierdes algo de eficiencia informativa a cambio de sostener la motivación. Es una decisión pedagógica opcional.

Si varias son prácticamente equivalentes:

- rompe empates con aleatorización;
- favorece categorías o conceptos menos repetidos.

No uses selección determinista simple en empates.

Evita repetir la misma pregunta de forma mecánica:

- no repitas exactamente el mismo ítem de manera consecutiva, salvo que el diseño pida de forma explícita un reintento inmediato;
- aplica penalización por frecuencia o una ventana de exclusión reciente para que un ítem ya usado pierda prioridad durante varias selecciones;
- si varias candidatas tienen utilidad parecida, prefiere la menos reciente y la menos repetida;
- solo permite reusar un ítem cuando el banco sea pequeño, no existan alternativas comparables o el objetivo pedagógico sea revisar de forma deliberada ese mismo caso.

No confundas este problema con un simple criterio de desempate:

- la aleatorización solo ayuda si existen varias candidatas razonables;
- para evitar repeticiones hace falta redundancia local en el banco: varias preguntas alternativas por categoría, nivel de dificultad, tipo de ítem o error relevante;
- si en una zona diagnóstica solo hay un ítem útil, la IA debe reconocer que el banco es insuficiente para una adaptación variada en esa zona;
- en ese caso no simules variedad con una falsa aleatorización: reutiliza el ítem solo cuando sea necesario, cambia temporalmente de objetivo pedagógico o deja el resultado como limitado por el tamaño del banco.

## Criterio de parada

Detén la sesión cuando se cumplan criterios razonables de cierre, por ejemplo:

- mínimo de preguntas ya cumplido;
- entropía por debajo de `H_stop`;
- hipótesis más probable por encima de `p_min`;
- no quedan preguntas útiles;
- la mejor pregunta restante aporta muy poca información;
- se alcanza el máximo práctico.

Si usas `p_min`, calcula el umbral orientativo:

`H_stop = -p_min * log2(p_min) - (1 - p_min) * log2((1 - p_min) / (n - 1))`

Tras cumplir el mínimo de preguntas, un cierre diagnóstico firme debe comprobar preferentemente ambas condiciones: `H <= H_stop` y `max(p_i) >= p_min`. Si se cierra por máximo, banco agotado o utilidad marginal baja sin cumplirlas, presenta el resultado como provisional.

Reglas mínimas:

- no cierres demasiado pronto;
- no alargues artificialmente la sesión cuando la utilidad marginal ya es baja;
- si la incertidumbre sigue siendo alta, el resultado debe indicarse como provisional.

## Refuerzo continuo sin parada

- En recursos de práctica o refuerzo abierto puede no haber criterio de parada: la sesión continúa mientras el alumno practique.
- En ese modo, el estado estimado no es un diagnóstico cerrado, sino una estimación viva que se actualiza con cada respuesta.
- No apliques `H_stop` ni `p_min` para cerrar la sesión; úsalos, si acaso, solo para informar del grado de confianza alcanzado.
- Mantén la selección en dos fases durante toda la sesión: diagnóstico mínimo por categoría y, después, refuerzo de lo menos dominado.
- El estado del alumno puede cambiar mientras practica: aplica olvido exponencial para que la estimación siga su estado actual. Antes de cada actualización, atenúa el posterior elevándolo a `lambda` y renormalizando: `p_i <- p_i^lambda / Σ_j p_j^lambda`.
- Usa `lambda ≈ 0.9`–`0.98` en práctica o refuerzo continuo (regla práctica: `lambda = 1 - 1/W`, con `W` el número de respuestas recientes que deben dominar la estimación). En recursos diagnósticos de sesión corta usa `lambda = 1` (sin olvido): ahí solo añadiría ruido.
- Con olvido activo, presenta la confianza como referida al estado reciente del alumno.
- Si necesitas modelar explícitamente el aprendizaje (por ejemplo, mayor probabilidad de subir de nivel justo tras una explicación), usa un modelo de transición (Bayesian Knowledge Tracing); consulta `matematicas.html §3.5`.

## Itinerarios por etapas

Si el recurso tiene fases, técnicas o etapas sucesivas:

- distingue entre estimación global y estimación local por etapa;
- no promociones una etapa usando solo la creencia global acumulada;
- decide la superación de la etapa con evidencia generada dentro de esa etapa.

Para dar una etapa por superada, conviene exigir al menos:

- confianza local suficiente;
- entropía local suficientemente baja;
- y un mínimo explícito de rendimiento observado en esa etapa.

Ejemplo razonable:

- `p_min = 0.80`
- mínimo de `60 %` de aciertos en la etapa

Si el alumno repite una etapa:

- reinicia la estimación local de esa etapa, salvo justificación pedagógica explícita en contra.

Terminar una etapa no implica necesariamente haberla superado.

## Recuperación y refuerzo

- Si el alumno muestra dificultades, el sistema debe poder:
  - ofrecer pistas;
  - mostrar explicación;
  - proponer refuerzo;
  - cambiar el tipo de actividad;
  - reducir temporalmente la dificultad;
  - permitir reintento o repaso.
- Si muestra dominio, puede:
  - avanzar;
  - ampliar;
  - aumentar complejidad;
  - reducir ayuda.

## Resultado final

La salida final debe incluir, según el tipo de recurso:

- diagnóstico o nivel estimado;
- grado de confianza;
- dificultades detectadas;
- fortalezas observadas;
- recomendación pedagógica;
- siguiente paso.

Si es un itinerario o actividad de aprendizaje, añade además:

- recorrido seguido;
- etapas superadas o no;
- ayudas usadas;
- áreas a reforzar.

No declares un dominio alto basándote en muy pocos intentos. Si el resultado hace afirmaciones por categoría o dimensión, exige una muestra mínima en esas categorías o dimensiones antes de presentarlas como firmes. Si la estimación es global, la muestra mínima puede referirse al conjunto de la sesión; limita o marca como provisional la estimación cuando la evidencia sea escasa.

No devuelvas solo una nota o etiqueta.

Si dos hipótesis terminan con probabilidades próximas, muestra la distribución posterior completa (por ejemplo, un diagrama de barras), no solo la etiqueta ganadora.

## Restricciones de implementación

- La interfaz debe ser comprensible para alumnado y profesorado.
- Debe mostrar con claridad el progreso y la retroalimentación.
- Si usas fórmulas visibles, acompáñalas de interpretación legible.
- Evita depender de backend si no se ha pedido.
- Evita preguntas abiertas largas sin corrección automática fiable.

## Valores por defecto recomendados

Si el docente no especifica parámetros:

- `n = 3` hipótesis o niveles;
- discriminación efectiva objetivo `a_ef = 1.25`; calcula siempre `a = 1.25 / (1 - c_q)` por pregunta (con `c_q = 0` da `a = 1.25`; máximo `a = 2.5` en verdadero/falso);
- `p_min = 0.80`;
- mínimo de preguntas: entre `4` y `6`;
- mínimo de diagnóstico por categoría en práctica adaptativa: `2` intentos;
- máximo práctico: entre `10` y `20`, según el tipo de recurso;
- peso de la ganancia de información en la fase de refuerzo: `α = 0.65` (rango razonable `0.6`–`0.7`);
- olvido exponencial: `lambda = 0.95` en práctica o refuerzo continuo (memoria efectiva ≈ `20` respuestas); `lambda = 1` en diagnóstico de sesión corta;
- crédito parcial: pesos de subcriterios definidos por el diseño, sumando `1`;
- muestra mínima por categoría o dimensión antes de mostrar dominio alto: `2`–`4` intentos.

## Qué no debe hacer la IA

- No confundir una explicación teórica con una regla obligatoria de implementación.
- No usar la creencia global para promocionar etapas locales en itinerarios.
- No cerrar el recurso sin justificar la certeza alcanzada.
- No presentar como firme un resultado con incertidumbre alta.
- No limitar la salida a una puntuación desnuda.
- No tratar como binaria una respuesta que admite grados: usa crédito parcial.
- No declarar dominio alto con muestra insuficiente.

## Validación y fiabilidad (opcional)

Estas comprobaciones no forman parte del flujo obligatorio: son una capa opcional que aumenta la honestidad del diagnóstico sin requerir datos empíricos. Aplícalas cuando la finalidad sea diagnóstica y el resultado vaya a usarse para decidir.

- **Ajuste del patrón individual (person-fit).** Al cerrar una sesión, evalúa si el patrón de respuestas es coherente con el nivel estimado. Calcula el índice estandarizado `l_z` a partir de las probabilidades de acierto que el modelo asigna a las preguntas respondidas bajo la hipótesis más probable. Si `l_z` es muy negativo (orientativamente `< -2`), marca el diagnóstico como poco fiable aunque el posterior sea alto: suele indicar respuestas incoherentes con la dificultad, descuidos o azar. Es señal de cautela, no prueba formal; con pocas preguntas es solo orientativo.
- **Separabilidad del diseño (Monte Carlo).** Como propiedad del test (no del alumno), estima con qué fiabilidad el banco distingue los niveles: genera respondentes sintéticos situados en el `theta` de cada hipótesis, hazles el test reutilizando la misma selección adaptativa y el mismo criterio de parada, y construye la matriz de confusión (nivel real frente a diagnosticado). Preséntala como fiabilidad bajo el modelo, nunca como validez empírica: los respondentes salen del propio modelo, así que mide si el diseño discrimina los niveles, no si los parámetros reflejan la realidad. **Esta validación es una herramienta del creador del recurso, no del alumnado: no debe mostrarse en la interfaz del alumno.** Impleméntala en un archivo o utilidad aparte que el autor pueda ejecutar al diseñar o revisar el test, no en el material que recibe el alumno.

Consulta `matematicas.html §11.7–§11.8` para las fórmulas y el encuadre completo.

## Nota sobre el modelo

La función IRT 3PL descrita aquí se usa como generador inicial de verosimilitudes cuando no hay datos empíricos. No equivale a una calibración psicométrica validada. Si se acumulan suficientes respuestas reales, conviene recalibrar dificultades, discriminación y pseudoazar.
