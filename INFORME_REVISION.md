# Informe de revisión metodológica, conceptual y pedagógica

> ## Estado del informe (actualizado 2026-07-09)
>
> **Este documento es un histórico acumulado, no una lista de fallos vigentes.** Casi todo lo que describe ya está corregido en la metodología y en el código. Antes de citar un hallazgo como problema abierto, compruébese aquí:
>
> | Ronda | Contenido | Estado |
> |-------|-----------|--------|
> | **Primera** (2026-07-08) | Hallazgos 1.1–1.4, 2.1–2.6, 3.2 | **Históricos: todos aplicados** a los nueve documentos y propagados al código de los seis programas. |
> | **Segunda** (2026-07-09) | Hallazgos N1–N9 (+ N1b, surgido al propagar N1 al código) | **Históricos: todos aplicados** y propagados. |
> | **Tercera** (2026-07-09, autor: Codex) | Puntos T1–T9, propuestos para estudio | **Cerrada.** Aplicados: T1 (parcheado), T2, T3, T4, T5 (rectificando su enunciado), T7, T8, T9. Desestimado: T6. |
>
> **No queda ningún punto abierto.** Todo está aplicado, o desestimado con razones medidas (T6), o aplicado tras rectificar lo que el propio informe proponía (T5: el remedio que pedía degradaba el diagnóstico). Cada hallazgo lleva su estado en la sección correspondiente y en la tabla resumen de su ronda; el registro cronológico de los cambios está en `CAMBIOS_METODOLOGIA.md`.
>
> Queda además fuera del alcance por decisión del autor: la auditoría de accesibilidad profunda (punto 2 del hallazgo 3.2).

**Documentos revisados:** `especificacion_operativa_ia.md` (v2.0), `documentacion.html` (v2.0), `matematicas.html` (v2.0), `guia_docente.html` (v1.1), `README.md`, `CAMBIOS_METODOLOGIA.md`.
**Fecha:** 2026-07-08.
**Método:** lectura completa de los cuatro documentos, verificación numérica independiente del ejemplo de la sección 9 de los fundamentos matemáticos, y comprobación por cálculo de las afirmaciones cuantitativas sobre escala, discriminación efectiva y criterios de parada.

---

# Primera ronda (2026-07-08) — histórica, aplicada

## Valoración general

La metodología es sólida, internamente coherente en su mayor parte y notablemente honesta en la declaración de sus límites (calibración a priori, circularidad de la validación Monte Carlo, aproximación de `l_z`, redundancia entre `H_stop` y `p_min`). Comprobaciones que se hicieron y salieron bien:

- **El ejemplo numérico de §9 de los fundamentos es correcto en su totalidad**: verosimilitudes IRT, posteriores, entropías y ganancias de información (IG(Q1)=0.280, IG(Q2)=0.275, IG(Q3)=0.212 bits) coinciden con el recálculo independiente. También es correcta la afirmación no calculada en el texto de que, tras el fallo en Q1, Q2 es más informativa que Q3 (0.065 frente a 0.020 bits).
- **La tabla de `H_stop` es correcta** (0.72 / 0.92 / 1.04 / 1.12 bits para n = 2..5 con p_min = 0.80), y la observación de que `max(p_i) ≥ p_min` implica `H ≤ H_stop` cuando el umbral se deriva del mismo `p_min` es matemáticamente cierta.
- **La sustitución de la regla lineal de crédito parcial por la verosimilitud geométrica** está bien fundamentada: la geométrica se maximiza en `p_i = s`, la lineal es monótona en `p_i` y empuja siempre a un extremo. El ejemplo numérico de §4.8 es correcto.
- **La escala θ fija con recorte de dificultades** (cambio de la v2.0) es una decisión de diseño defendible y bien argumentada: evita que un ítem atípico redefina la escala.
- **La distinción excluyente / coexistente** (nominal unifactorial frente a multifactorial por perfiles) es conceptualmente correcta y es raro verla tan bien delimitada en material dirigido a docentes.

Los hallazgos siguientes se ordenan por relevancia. Ninguno invalida la metodología; los tres primeros sí pueden producir comportamientos indeseados en recursos generados a partir de la especificación tal como está redactada.

---

## 1. Hallazgos metodológicos relevantes

### 1.1. El umbral fijo del 60 % de aciertos por etapa choca con la selección adaptativa

**Dónde:** especificación («Itinerarios por etapas»: `p_min = 0.80` + «mínimo de 60 % de aciertos en la etapa»), protocolo §13.3.

**Problema:** bajo selección adaptativa, el porcentaje bruto de aciertos no es comparable entre alumnos ni informativo sobre el nivel: es una propiedad conocida de los CAT que la selección por máxima información empuja la tasa de acierto de *todos* los alumnos hacia un valor similar. Con la propia metodología:

- la selección por máxima IG apunta a ítems con probabilidad de acierto cercana al 50 % (el protocolo §14 lo dice explícitamente);
- un ítem situado en el nivel del alumno (`b = θ`) da una probabilidad de acierto de `(1+c)/2`: 50 % sin azar, 62.5 % con 4 opciones, 75 % en verdadero/falso.

Es decir, un alumno que **realmente domina la etapa** y recibe preguntas seleccionadas por máxima información rondará por diseño el 50–62 % de aciertos, justo sobre el umbral del 60 %. El criterio puede bloquear sistemáticamente la promoción de alumnos competentes, y el resultado depende del formato de los ítems (con preguntas abiertas el bloqueo es casi seguro; con V/F casi nunca ocurre). La intención del control (protegerse de una calibración mala) es legítima, pero la métrica elegida es la única que la selección adaptativa vuelve inservible.

**Sugerencia:** definir el mínimo de rendimiento en relación con lo que el modelo predice, no en términos absolutos. Opciones: (a) exigir que la tasa observada no quede muy por debajo de la tasa esperada bajo la hipótesis de dominio local (esto es esencialmente el person-fit de §11.7, que ya está en la metodología y sería el instrumento natural aquí); (b) exigir el acierto de 1–2 ítems «de salida» de dificultad representativa seleccionados sin criterio informativo; o (c) si se mantiene el umbral fijo, advertir que solo es válido cuando la selección dentro de la etapa no es por máxima IG y documentar la dependencia del formato.

### 1.2. El prior uniforme en modelos multifactoriales implica asumir que cada error está presente con probabilidad 0.5

**Dónde:** especificación («Estado del alumno»: «Si no hay información previa fiable, usa distribución uniforme»), aplicada al caso multifactorial de perfiles (`2^k` combinaciones).

**Problema:** la regla del prior uniforme está pensada para hipótesis de nivel, donde es razonable. Pero un prior uniforme sobre los `2^k` perfiles (o sobre `{presente, ausente}` por factor) codifica que **cada error tiene una probabilidad a priori del 50 % de estar presente**, lo cual rara vez refleja la prevalencia real de un error conceptual en un aula. Consecuencias: con poca evidencia, las probabilidades marginales de error parten de 0.5 y la interfaz puede mostrar errores «indeterminados» o incluso «probables» que el alumno no tiene (sesgo hacia el falso positivo en las primeras preguntas), y el diagnóstico «presente/ausente/indeterminado» hereda ese sesgo.

**Sugerencia:** para factores de error, recomendar un prior informativo moderado (por ejemplo, `P(error) ≈ 0.2–0.3`, o el valor que el docente estime como prevalencia en su grupo) en lugar del uniforme, o al menos advertir explícitamente en la especificación que el uniforme sobre factores binarios no es «neutral»: es una afirmación fuerte sobre prevalencia. Nótese que la propia metodología ya pide priors justificados cuando hay información previa; aquí la información previa (los errores conceptuales son minoritarios) casi siempre existe.

### 1.3. El «invariante de comparabilidad» no controla lo que dice controlar: la evidencia máxima de un fallo escala con la `a` nominal, no con `a_ef`

**Dónde:** especificación («Estado del alumno», último punto: «el producto `a_ef · Δtheta = 2.5` determina la fuerza máxima de una actualización»).

**Problema:** fijar `a_ef` iguala la **pendiente máxima** de la ICC entre formatos, pero la fuerza de una actualización bayesiana la determina la **razón de verosimilitudes** entre hipótesis, y esa razón no queda igualada. En el lado del fallo, `P(F|θ) = (1-c)(1-σ(a(θ-b)))`: el factor `(1-c)` se cancela en el cociente y la razón máxima entre hipótesis adyacentes tiende a `e^(a·Δθ)` con la `a` **nominal** `= a_ef/(1-c)`. Verificado numéricamente con `a_ef = 1.25` y `Δθ = 2`, para un ítem fácil fallado:

| Formato | `c_q` | `a` | Razón de verosimilitud de fallo entre niveles adyacentes |
|---|---|---|---|
| Abierta | 0 | 1.25 | ≈ 12 |
| 4 opciones | 0.25 | 1.667 | ≈ 28 |
| Verdadero/falso | 0.5 | 2.5 | ≈ 148 |

Un solo fallo en un ítem V/F fácil aporta una evidencia un orden de magnitud mayor que el mismo fallo en una pregunta abierta fácil: exactamente el tipo de salto de posterior «casi determinista» que la propia especificación quiere evitar cuando prohíbe estirar la escala. La especificación ya reconoce que igualar `a_ef` «no garantiza que todos los formatos aporten la misma información esperada», pero la frase «determina la fuerza máxima de una actualización» es técnicamente incorrecta y da una garantía que el diseño no ofrece.

**Sugerencia:** (a) corregir la redacción del invariante (afirma comparabilidad de pendiente, no de fuerza de evidencia); (b) considerar un suelo para la verosimilitud de fallo bajo la hipótesis superior — el techo de dominio 0.90–0.95 que ya se usa en el caso nominal cumpliría esa función también aquí: con `P(acierto) ≤ 0.95` el fallo nunca aporta razones de verosimilitud extremas y modela además el descuido (*slip*), que el 3PL puro no contempla. De hecho hay una asimetría conceptual en la metodología actual: el caso nominal impone «techo de dominio, nunca 1» y el caso ordinal permite `P(acierto) → 0.99+` para el nivel alto en ítems fáciles, que es la fuente de estas actualizaciones violentas.

### 1.4. La garantía del «factor 2» se diluye al disminuir `n` y no es homogénea entre recursos

**Dónde:** fundamentos §8.2–8.3, especificación («Estado del alumno»).

**Problema:** con la escala fija, el margen entre el nivel extremo y el ítem más difícil es `θ_max − b_max = (n−1)/2`, que **depende de `n`**. El argumento de §8.2 a favor del factor 2 usa `n = 3` (P ≈ 0.86–0.88 de que el nivel alto acierte el ítem más difícil). Pero con `n = 2` (domina / no domina, un caso que la propia documentación ofrece), ese margen es 0.5 y la probabilidad cae a **0.65** (abierta) o **0.77** (4 opciones): el valor «mediocre» que §8.2 describe como el problema que el factor 2 evita. Verificado numéricamente:

| `n` | P(nivel máximo acierta el ítem más difícil), `c=0` | `c=0.25` |
|---|---|---|
| 2 | 0.651 | 0.773 |
| 3 | 0.777 | 0.881 |
| 4 | 0.867 | 0.943 |
| 5 | 0.924 | 0.974 |

Es decir, el factor 2 garantiza la *forma* de la separación pero no la probabilidad objetivo `P*`, y las «confianzas y velocidades de convergencia comparables entre recursos» que promete el invariante solo se sostienen entre recursos con el mismo `n`. No es un error grave (la fórmula del logit de §8.2 ya permite derivar la separación desde una `P*` deseada), pero la especificación presenta la comparabilidad entre recursos como general.

**Sugerencia:** o bien acotar la afirmación de comparabilidad a «entre recursos con el mismo número de hipótesis», o bien definir la escala desde la `P*` objetivo (la fórmula ya existe en §8.2) en lugar de fijar el espaciado en 2 para todo `n`. Como mínimo, advertir que con `n = 2` los ítems extremos confirman débilmente y que conviene compensar con más preguntas.

---

## 2. Incoherencias internas y huecos conceptuales

### 2.1. §11.1 de los fundamentos sigue presentando `a = 1.5` como valor por defecto

§11.1 dice: «el uso de valores por defecto contrastados (`a = 1,5`, `c_q ≈ 1/m_q`) está respaldado por la literatura». Esto contradice la regla v2.0 (fijar `a_ef = 1.25` y derivar `a` por ítem, «hazlo siempre») y contradice el propio registro de cambios, que declara: «se evita presentar `a=1.5` como valor directo por defecto». §4.4 sí está actualizado (aclara que los ejemplos usan `a = 1.5` con fines ilustrativos); §11.1 quedó sin tocar. Es la incoherencia editorial más visible del conjunto, porque un lector (o una IA) que llegue a §11.1 puede volver al `a` fijo.

### 2.2. El único control de parada que «añade exigencia» no está en la especificación operativa

Los fundamentos (§7.4) y el protocolo (§9, §17) explican que comprobar `H ≤ H_stop` junto a `max(p_i) ≥ p_min` es redundante, y que el control adicional real es la **separación mínima respecto a la segunda hipótesis** (`P(ganadora) − P(segunda) ≥ Δ_min`). Sin embargo, la especificación operativa —el documento que prevalece y el único que la IA suele recibir— recomienda precisamente la pareja redundante («un cierre diagnóstico firme debe comprobar preferentemente ambas condiciones») y **no menciona el criterio de separación**. El resultado práctico es que los recursos generados implementarán el control redundante y no el útil. Convendría añadir `Δ_min` a la especificación (con un valor por defecto, p. ej. 0.3–0.4) o, al menos, dejar de presentar la doble comprobación como el cierre «firme».

### 2.3. El marco epistemológico promete evidencias que la parte operativa nunca modela

El protocolo §2.1 afirma que el sistema «observa respuestas, **tiempos**, elecciones, **ayudas usadas** u otras interacciones evaluables y las interpreta como evidencias». Ningún documento define verosimilitudes para tiempos de respuesta ni para el uso de ayudas; la maquinaria operativa solo procesa acierto/fallo, crédito parcial y opción elegida. No es un error de las fórmulas, pero sí una promesa conceptual sin desarrollo: o se acota la frase («en esta versión, las evidencias modeladas son…»), o se indica explícitamente que tiempos y ayudas son extensiones fuera del alcance actual. Relacionado: el uso de pistas *dentro* de un ítem cambia la probabilidad de acierto de ese ítem, y la metodología no dice cómo tratar la respuesta acertada-con-pista en la actualización (¿acierto pleno?, ¿`s` reducido?). Los mecanismos de recuperación (§15) recomiendan dar pistas sin decir cómo afectan a la evidencia.

### 2.4. El exponente `J` del crédito parcial asume componentes intercambiables

La regla `L ∝ p^{sJ}(1−p)^{(1−s)J}` equivale a tratar los `J` componentes como ensayos Bernoulli independientes **con la misma probabilidad `p_i` del ítem completo**. Si los componentes difieren claramente en dificultad (lo habitual en una tarea por pasos), esta forma sobrecuenta la evidencia respecto al modelo correcto por componentes. El texto ya recomienda multiplicar verosimilitudes por componente cuando se conocen, pero convendría añadir la advertencia simétrica: el atajo con `J` solo es razonable si los componentes son de dificultad parecida; en caso de duda, usar `J` menor que el número real de componentes (evidencia más conservadora).

### 2.5. Falta situar la propuesta frente a los modelos estándar que resuelven los mismos problemas

Dos ausencias bibliográficas llaman la atención en un documento que por lo demás referencia bien:

- **Crédito parcial:** la psicometría tiene modelos politómicos canónicos para esto (Samejima, *Graded Response Model*, 1969; Masters, *Partial Credit Model*, 1982). La verosimilitud geométrica es una aproximación razonable y mucho más simple, pero el lector debería saber que existe la solución estándar y por qué no se adopta (complejidad de parametrización sin datos).
- **Diagnóstico multifactorial:** la distribución sobre perfiles completos de `2^k` factores **es** un modelo de clases latentes de diagnóstico cognitivo; la variante «el error atrae hacia un distractor» es la lógica de los modelos DINA/DINO (Junker & Sijtsma, 2001; de la Torre, 2009). Citarlos no es cortesía académica: esa literatura contiene exactamente las técnicas de calibración que §10.4 declara «fuera de esta metodología», y daría al lector la puerta de entrada correcta.

Ninguna de las dos ausencias es un error; ambas son huecos que debilitan la revisabilidad pública que se busca.

### 2.6. Menores

- El prompt recomendado difiere entre documentos: README («…siguiendo exactamente sus reglas operativas»), especificación («…siguiendo las reglas operativas aplicables al tipo de recurso solicitado») y guía docente (una tercera variante). Trivial, pero al ser el punto de entrada del método conviene unificar.
- Protocolo §9 clasifica «refuerzo previo / práctica guiada / consolidación / ampliación» como hipótesis no jerárquicas; son razonablemente ordinales (grados de necesidad de apoyo). Como ejemplo de «alternativas en la decisión» funciona, pero es un ejemplo fronterizo que puede confundir justo en la distinción que más cuesta enseñar.
- La fase diagnóstica de 2 intentos por categoría convive con la regla de «2–4 intentos antes de mostrar dominio alto»: coherente, pero en el límite inferior; con 2 intentos y olvido activo la marginal por categoría es muy volátil. Ya está advertido en el texto; solo señalo que 2 es el mínimo defendible, no un valor cómodo.

---

## 3. Observaciones pedagógicas

### 3.1. Bien resuelto

- El coste motivacional de la selección informativamente óptima (~50 % de fallo) está identificado y tratado como decisión pedagógica explícita y opcional. Es el tratamiento correcto.
- La insistencia en resultado pedagógico (no nota), provisionalidad, y mostrar la distribución completa cuando hay hipótesis próximas es coherente con la evaluación formativa que se cita (Black & Wiliam).
- La guía docente es honesta («desconfía de los diagnósticos con pocas preguntas», «no sustituye tu criterio») y el ejemplo de la planta mustia para excluyente/coexistente es muy bueno.

### 3.2. Mejorables

- **Efecto etiquetado.** El resultado prototípico comunica al alumno un nivel («básico/medio/avanzado») con su probabilidad. Aun con las cautelas de provisionalidad, la literatura sobre mentalidades y expectativas sugiere que la etiqueta de rasgo («eres nivel básico») tiene más riesgo que la formulación de tarea («te conviene practicar X antes de Y»). La metodología ya contiene la solución (la «recomendación pedagógica» y el «siguiente paso» del resultado final); bastaría con indicar que, en la vista del alumno, la recomendación debe tener más peso visual que la etiqueta de nivel, y que la etiqueta con probabilidades puede reservarse para la vista docente. Ahora mismo esa decisión de presentación queda sin guía.
- **El diagnóstico de errores comunicado al alumno.** En los modelos de error («confundes masa con peso», probabilidad 0.85) el efecto etiquetado es más delicado que en niveles, porque atribuye una creencia errónea concreta. Convendría una línea sobre cómo comunicar errores probables al alumno (como hipótesis a comprobar juntos, no como sentencia), especialmente en primaria.
- **Alumnado que responde al azar o con ansiedad.** §11.4 trata el azar como límite del modelo y §11.7 lo detecta a posteriori con `l_z`. Pedagógicamente faltaría la recomendación operativa mínima: si el person-fit o una racha inverosímil sugieren respuesta al azar *durante* la sesión (no solo al cierre), el recurso debería poder pausar y reconducir («parece que vas muy rápido…») en lugar de seguir consumiendo banco. Ahora mismo `l_z` solo se calcula al cerrar.
- **Accesibilidad y NEAE.** La guía docente menciona «interfaz muy simple, letra grande» como algo que el docente puede pedir. Dado el público objetivo (aulas reales, alumnado diverso), unas restricciones mínimas de accesibilidad en la especificación (contraste, navegación por teclado, no depender del color, tiempo sin límite por defecto) costarían tres líneas y evitarían que cada docente tenga que acordarse de pedirlas.
- **Privacidad.** El recurso estático sin backend es, de hecho, una decisión excelente de privacidad (los datos del alumno no salen del navegador), pero ningún documento la reivindica ni advierte de lo contrario: si un docente pide «guardar los resultados de mis alumnos», la IA añadirá almacenamiento sin que la metodología diga nada sobre datos de menores. Una nota breve en la especificación («si se pide persistencia de resultados, advertir sobre protección de datos y preferir exportación local») cerraría el hueco.

---

## 4. Resumen priorizado

**Todos los hallazgos de esta tabla están aplicados** (documentos y código); se conservan como histórico.

| # | Hallazgo | Tipo | Gravedad |
|---|----------|------|----------|
| 1.1 | Umbral fijo 60 % de aciertos por etapa incompatible con selección por máxima IG | Metodológico | Alta |
| 1.2 | Prior uniforme multifactorial = prevalencia de error del 50 % a priori | Metodológico | Alta |
| 1.3 | «Invariante de comparabilidad» no iguala la fuerza de la evidencia (fallo en V/F fácil: LR ≈ 148 vs 12) | Metodológico | Media-alta |
| 2.2 | El criterio de separación `Δ_min` (el único no redundante) falta en la especificación operativa | Coherencia | Media |
| 2.1 | §11.1 de fundamentos mantiene `a = 1.5` como defecto, contra la regla v2.0 | Coherencia | Media |
| 1.4 | El factor 2 no garantiza la misma `P*` para todo `n` (con `n=2`, P=0.65) | Metodológico | Media |
| 2.3 | Tiempos, ayudas y pistas se anuncian como evidencia pero no se modelan (ni la respuesta con pista) | Conceptual | Media |
| 3.2 | Presentación de etiquetas de nivel/error al alumno sin guía; person-fit solo al cierre; accesibilidad y privacidad sin mención | Pedagógico | Media |
| 2.4 | Exponente `J` asume componentes de igual dificultad | Metodológico | Baja |
| 2.5 | Faltan referencias a modelos politómicos (GRM/PCM) y de diagnóstico cognitivo (DINA) | Académico | Baja |
| 2.6 | Prompts no unificados; ejemplo fronterizo de hipótesis «no jerárquicas» | Editorial | Baja |

Lo verificado y correcto (ejemplo numérico completo, tablas de `H_stop`, la implicación `p_min → H_stop`, la corrección del crédito parcial lineal→geométrico, la argumentación de la escala fija) da confianza en que el núcleo del método está bien construido; los hallazgos 1.1–1.3 son los que convendría abordar antes de que se generen muchos recursos con la especificación actual, porque afectan al comportamiento de los recursos generados y no solo a la exposición.

---

# Segunda ronda (2026-07-09) — histórica, aplicada

**Método:** relectura completa de los cuatro documentos ya en v2.0 (con los 11 hallazgos de la primera ronda aplicados), buscando fallos nuevos, con verificación numérica de los hallazgos N1–N3.

## N1. El olvido exponencial deriva los priors informativos hacia la uniforme — Alta

La regla `p_i ← p_i^λ` renormalizada tiene la **uniforme** como punto fijo: sin evidencia nueva, cualquier distribución deriva hacia ella. Verificado: un factor de error con prior `P(error) = 0.25` sin evidencia sube a 0.34 en 10 pasos, 0.40 en 20 y ≈ 0.47 en 40 con `λ = 0.95`, reapareciendo como «indeterminado» o «probable» sin que el alumno haya hecho nada: en práctica continua, el olvido **reintroducía el falso positivo que el hallazgo 1.2 corrigió**. La variante «mezclar con la uniforme» tenía el mismo defecto de forma explícita. Además, la redacción («antes de cada actualización») era ambigua sobre si se atenúan las distribuciones que no reciben evidencia; las dos lecturas fallaban (deriva a 0.5 o ausencia total de olvido). **Corrección aplicada:** olvido anclado al prior, `p_i ∝ p_i^λ · π_i^(1−λ)` (punto fijo = prior; el prior conserva peso exactamente 1 al desenrollar la recursión; con prior uniforme coincide con la regla anterior), atenuación de todas las distribuciones en cada paso, y variante de mezcla hacia el prior. Consecuencia asociada corregida: la muestra mínima por categoría debe contarse en ventana reciente (`W ≈ 1/(1−λ)`), porque la evidencia caduca con el olvido pero un contador acumulado no.

## N1b. La calibración de `lambda` estaba subespecificada para recursos con varias distribuciones — Alta

**Detectado al propagar N1 al código de `labcom` (2026-07-09), no en la revisión documental.**

La corrección N1 pide atenuar **todas** las distribuciones en cada respuesta, y mantenía la regla `lambda = 1 - 1/W` con `W` = «respuestas recientes que deben dominar la estimación». Ambas cosas juntas son incoherentes en cuanto hay más de una distribución: cada una **envejece** una vez por respuesta, pero solo **recibe evidencia** cuando la pregunta le corresponde. Si la distribución `d` se actualiza en 1 de cada `K_d` respuestas, su memoria en intentos propios es `1/(K_d·(1-lambda))`, es decir, `K_d` veces menor que la memoria en respuestas.

Verificado en `labcom` (6 tipos de problema, 6 dimensiones): cada respuesta actualiza las 6 dimensiones pero solo 1 de los 6 tipos. Con `lambda = 0.95` para todo, cada tipo tendría una memoria de `20/6 ≈ 3.3` intentos propios — bastante para **borrar el diagnóstico inicial**, que son 2 intentos por tipo. Aplicar N1 literalmente habría degradado el recurso.

**Corrección aplicada (2026-07-09):** fijar la memoria objetivo `M` en **intentos de la propia distribución** (por defecto 20) y derivar un `lambda` por distribución:

`lambda_d = (1 - 1/M)^(1/K_d)`

Con `K_d = 1` se recupera `lambda = 1 - 1/M`. Con `M = 20`: `lambda = 0.95` para una dimensión evaluada en cada respuesta y `lambda = 0.95^(1/6) ≈ 0.9915` para cada una de 6 categorías (ambas ≈ 20 intentos propios de memoria; comprobado numéricamente para `K = 1, 3, 6, 10`).

Dos consecuencias más, también aplicadas:

- **La ventana de la muestra mínima se cuenta sin ponderar.** Un recuento ponderado por `lambda^k` haría que dos intentos consecutivos sumaran `1 + 0.95 = 1.95`, incumpliendo un umbral entero de `2` y endureciendo la puerta en silencio. La ventana es deslizante: intentos dentro de las últimas `1/(1-lambda_d)` respuestas.
- **El contador con caducidad gobierna las puertas de dominio, no la interfaz.** Si el recurso muestra al alumno cuántos ejercicios ha resuelto, ese número es el total real y no caduca. Y una categoría que se queda sin evidencia dentro de su ventana ha vuelto a su prior: debe presentarse como *sin datos recientes*, no en rojo. Marcar como débil lo que el modelo ya no sostiene es acusar al alumno de algo que no ha mostrado.

Editado: especificación «Refuerzo continuo sin parada» y «Valores por defecto», protocolo §13.2, fundamentos §3.5 y §7.6 (ES/CA/EN).

## N2. Reusar un ítem tras el feedback es evidencia contaminada sin descuento — Media-alta

La metodología exige retroalimentación con explicación tras cada respuesta, permite reusar ítems con banco pequeño y contempla el reintento inmediato, pero un acierto sobre un ítem cuya solución acaba de mostrarse mide memoria del feedback, no dominio — y se tomaba como acierto pleno. Incoherente con el tratamiento de las pistas (`s < 1`). **Corrección aplicada:** el acierto en ítem reutilizado tras corrección se trata como las pistas (crédito parcial con `s` reducido, o exclusión de la actualización si la explicación dio la respuesta), y se recomienda la redundancia local mediante variantes parametrizadas en lugar de repetición literal.

## N3. Los «ítems de salida» heredan la dependencia del formato — Media

Verificado (n = 3, ítem de salida en `b = 0.5`, `a_ef = 1.25`, techo 0.95): en V/F un no-dominante pasa 1 ítem el 61 % de las veces (37 % con dos); en abierta, exigir 2 de 2 bloquea al 25 % de quienes sí dominan. La especificación no decía ni de qué formato deben ser los ítems de salida ni cómo elegir entre 1 y 2. **Corrección aplicada:** evitar V/F en ítems de salida; con 4-5 opciones exigir 2 de 2; advertencia sobre el bloqueo en abierta; el filtro complementa a `p_min`, no decide en solitario.

## N4. El ejemplo canónico §9 no aplicaba el techo de dominio — Media (coherencia)

Fundamentos §9 usa `P(A|H₃,Q1) = 0.992` y `P(A|H₃,Q2) = 0.964`, por encima del techo 0.95 que la primera ronda convirtió en regla; §4.3 describía `P → 1` sin mencionar el techo y el protocolo §12 citaba el suelo de azar como condición pero no el techo. Misma clase de incoherencia que el antiguo 2.1: la excepción sin señalizar justo donde el lector implementa. **Corrección aplicada:** nota de simplificación en §9.1 (con los dos valores afectados), mención del techo en §4.3 y en la declaración de ejemplos ilustrativos de §4.4, y el techo añadido junto al suelo en el protocolo §12.

## N5. Selección y parada indefinidas para el caso multifactorial factorizado — Media

El procedimiento de selección y los criterios de parada estaban definidos sobre una sola distribución; §10.2 resolvía perfiles completos pero no las dimensiones paralelas. **Corrección aplicada:** ganancia total del ítem = suma de ganancias por distribución (la entropía conjunta factorizada es la suma de entropías), promediando sobre los resultados que el ítem modele; parada por factor (decidido al salir de la zona indeterminada con muestra mínima; los no decididos se reportan como indeterminados).

## N6. «Ausente por defecto»: el prior informativo roza el umbral de confianza — Baja-media

Con `P(error) = 0.2–0.3`, «ausente» arranca en 0.7–0.8 sin evidencia: un factor apenas preguntado podía reportarse «ausente» con aparente seguridad. **Corrección aplicada:** exigir muestra mínima por factor y distinguir en el reporte «ausente confirmado» de «sin evidencia suficiente».

## N7. Modelo combinado A+D sin regla de reconciliación — Baja-media

Nivel ordinal y factores de error se actualizan con la misma evidencia y se reportaban como hallazgos independientes, sin nada que impida un resultado incoherente («nivel avanzado» + errores «presentes»). **Corrección aplicada:** presentarlos como estimaciones marginales paralelas y, en caso de tensión, el error como matiz del nivel («domina X, aunque persiste el error Y»).

## N8. Fórmula de utilidad subespecificada — Baja

`utilidad = α·IG_normalizada + (1−α)·ajuste_de_dificultad` no definía la normalización ni el ajuste; `α = 0.65` no significa nada sin las escalas. **Corrección aplicada:** `IG_norm = IG(q)/max IG` entre candidatas y `ajuste = max(0, 1 − |b_q − E[θ]|/2)`, ambos en `[0, 1]`.

## N9. `l_z` como criterio de etapa con N minúsculo — Baja

La aproximación normal de `l_z` es débil con los 4-8 ítems de una etapa. **Corrección aplicada:** comparar aciertos observados frente a esperados con margen de ~1 desviación típica (o test binomial exacto) y tratarlo como señal orientativa, no como bloqueo duro.

## Resumen de la segunda ronda

| # | Hallazgo | Gravedad | Estado |
|---|----------|----------|--------|
| N1 | Olvido exponencial deriva priors informativos hacia la uniforme (falso positivo de 1.2 reintroducido) + contador de muestra mínima sin caducidad | Alta | Corregido |
| N1b | `lambda` subespecificada con varias distribuciones: un `lambda` común da memoria `M/K_d` intentos propios y borra el diagnóstico (detectado al propagar N1 al código) | Alta | Corregido |
| N2 | Reuso de ítems tras feedback sin descuento de evidencia | Media-alta | Corregido |
| N3 | Ítems de salida dependientes del formato (V/F deja pasar 61 % de no-dominantes) | Media | Corregido |
| N4 | Ejemplo §9, §4.3 y protocolo §12 sin el techo de dominio | Media | Corregido |
| N5 | Selección y parada indefinidas con dimensiones paralelas | Media | Corregido |
| N6 | «Ausente» declarable sin evidencia por el prior informativo | Baja-media | Corregido |
| N7 | Combinado A+D sin reconciliación nivel↔factores | Baja-media | Corregido |
| N8 | Utilidad de refuerzo sin escalas definidas | Baja | Corregido |
| N9 | `l_z` de etapa con aproximación normal débil | Baja | Corregido |

---

# Tercera ronda (2026-07-09) — Puntos para estudiar y rectificar si procede

**Autor de esta tercera revisión:** Codex.  
**Estado:** T1, T2 y T3 aplicados a especificación, protocolo y fundamentos (ES/CA/EN) el 2026-07-09. Tras la evaluación independiente de la ronda (Claude, 2026-07-09): T1 parcheado (defaults de `w_b`, definición de «bloque decidido», parada sobre IG cruda), T4 y T8 aplicados. Después: T7 y T9 aplicados, **T6 desestimado** y **T5 aplicado rectificando su enunciado** (ver sus secciones). **Ronda cerrada.**

## Criterio rector

La metodología debe mantener una separación clara de responsabilidades:

- El **docente** expresa la intención educativa en lenguaje natural: qué quiere trabajar, con qué curso, qué errores observa, qué resultado espera y qué desea modificar.
- La **IA** toma las decisiones metodológicas y técnicas: modelo, parámetros, priors, ponderaciones, criterio de parada, validación, tipo de banco necesario y forma de presentar la incertidumbre.
- La especificación no debe trasladar al docente decisiones que no puede entender sin conocer la metodología. Cuando falte una decisión técnica, la solución no debe ser “preguntar al docente”, sino definir una regla operativa para que la IA la tome y, como mucho, explique el supuesto en lenguaje comprensible.

Los hallazgos siguientes se formulan desde ese criterio: el problema no es que el docente no aporte más datos técnicos, sino que algunas decisiones todavía quedan insuficientemente automatizadas para la IA.

## T1. Modelo combinado nivel + errores: falta ponderación automática de objetivos — Media-alta

La especificación permite combinar una distribución ordinal global con distribuciones diagnósticas paralelas de errores, y para varias distribuciones recomienda usar la suma de ganancias esperadas. Eso es correcto si todas las dimensiones tienen el mismo peso pedagógico, pero no siempre ocurre.

Riesgo: si hay muchos factores de error, la suma de sus ganancias puede dominar la selección y desplazar el diagnóstico de nivel; si hay pocas dimensiones diagnósticas, el nivel global puede absorber casi toda la decisión. En recursos mixtos (“nivel y además errores”), el comportamiento dependerá del número de factores más que de la finalidad educativa.

No debe pedirse al docente que fije pesos. La IA debería decidirlos a partir de la finalidad expresada:

- si el objetivo principal es nivel, priorizar la distribución ordinal y usar errores como diagnóstico secundario;
- si el objetivo principal es detectar errores, priorizar factores;
- si el recurso es mixto, usar ponderación normalizada por bloques, no por número bruto de dimensiones;
- si un bloque ya está decidido, desplazar peso hacia los bloques aún inciertos.

Una regla posible: calcular ganancia por bloques pedagógicos (`nivel`, `errores`, `categorías`) y no por dimensión individual, normalizando cada bloque antes de combinarlos.

**Corrección aplicada (2026-07-09):** la especificación, el protocolo y los fundamentos pasan a usar bloques pedagógicos con normalización por bloque y pesos automáticos inferidos de la finalidad educativa. Los pesos no se piden al docente.

**Parche posterior (2026-07-09, tras evaluación independiente):** la corrección dejaba tres huecos, ahora cerrados en los tres documentos (ES/CA/EN): (a) valores por defecto de los pesos (`w = 0.7` para el bloque central de la finalidad y `0.3` repartido; iguales por bloque si es mixta); (b) definición de «bloque decidido» (nivel: `max(p_i) ≥ p_min` con mínimo de preguntas; errores: todos los factores fuera de la zona indeterminada con muestra mínima); (c) la utilidad normalizada no está en bits —reescala el mejor candidato de cada bloque a 1 aunque sus ganancias sean minúsculas—, así que el criterio de parada por ganancia mínima debe evaluarse sobre la IG cruda total, nunca sobre la utilidad, y para el sobrepeso de bloques casi agotados se documentan alternativas (media por dimensión sobre ganancias crudas, o normalización por entropía restante).

## T2. Falta criterio operativo para elegir entre factores independientes y perfiles completos — Media

La metodología distingue bien entre factores paralelos y perfiles completos, pero deja poco operacional cuándo la IA debe pasar de una representación factorizada a una distribución sobre perfiles. El texto reconoce que la factorización ignora correlaciones, pero no da una regla de decisión.

Riesgo: en errores que interactúan, el recurso puede devolver combinaciones marginales poco plausibles o seleccionar preguntas que parecen óptimas por factor pero no discriminan perfiles reales de alumnado.

No debe preguntarse al docente por “independencia condicional” ni por “perfiles completos”. La IA debería decidir así:

- usar factores independientes cuando cada error afecta a preguntas separables y la interpretación final es por error;
- usar perfiles completos cuando la respuesta esperada cambia por combinaciones de errores, cuando un error enmascara otro o cuando la intervención pedagógica depende de la combinación;
- si `2^k` perfiles es inviable, agrupar errores relacionados o diagnosticar por fases.

La pregunta al docente, si hace falta, debe ser pedagógica: “¿estos errores pueden aparecer juntos o uno suele impedir/ver tapar al otro?”, no técnica.

**Corrección aplicada (2026-07-09):** la especificación, el protocolo y los fundamentos incorporan el criterio automático: factores independientes si los errores se evidencian e interpretan por separado; perfiles completos si la combinación cambia la respuesta esperada, enmascara errores o determina la intervención; agrupación o diagnóstico por fases si `2^k` perfiles es inviable.

## T3. Validación Monte Carlo generada siempre que proceda, ejecutada si el entorno lo permite — Media-alta

La validación por simulación se describe correctamente como separabilidad bajo el modelo, no como validez empírica. Pero queda como capa opcional y no fija criterios mínimos de aceptación.

Riesgo: una IA puede generar un recurso diagnóstico con banco pobre, ejecutar o no ejecutar la simulación, y no tener una regla clara para decidir si el recurso es utilizable, provisional o necesita más ítems.

Para recursos diagnósticos o recursos que deciden promoción de etapa, la validación debería quedar preparada durante la generación, no como carga metodológica del docente. En entornos con ejecución (CLI, notebook, entorno local), la IA debería ejecutarla; en chats sin ejecución, debería generar una utilidad para ejecutarla en navegador y marcarla como pendiente. La especificación debería indicar:

- número mínimo de simulaciones por hipótesis o perfil;
- métricas mínimas: diagonal de matriz de confusión por nivel, exactitud equilibrada, tasa de indeterminados y longitud media;
- criterio de fallo: si un nivel/perfil no se distingue suficientemente, ampliar banco, revisar dificultades/verosimilitudes o marcar explícitamente la limitación;
- salida para docente: “este recurso distingue bien A/B, pero no separa C/D con fiabilidad”, sin tecnicismos innecesarios.

**Corrección aplicada (2026-07-09):** la especificación, el protocolo y los fundamentos ya distinguen entre entornos ejecutables y chats sin ejecución. En diagnóstico o promoción de etapa, la IA debe ejecutar Monte Carlo si puede; si no puede, debe generar una utilidad de validación separada o una vista docente/autora, marcar la validación como pendiente y no afirmar que el banco está comprobado. Se añaden valores por defecto (`500`–`1000` simulaciones por hipótesis/perfil), métricas mínimas y umbral orientativo de alerta (`< 0.70` de clasificación correcta bajo el propio modelo o confusión sistemática).

## T4. La revisión docente debe ser de contenido, no metodológica — Media

El README y la guía docente insisten correctamente en que el docente no necesita conocer la metodología. Sin embargo, la calidad depende mucho de hipótesis, dificultad, errores, cobertura y redundancia local. Falta convertir esa dependencia en una revisión docente sencilla y no técnica.

Riesgo: el docente puede aceptar un recurso generado por IA sin revisar lo único que sí puede validar bien: adecuación curricular, corrección de enunciados, respuestas, distractores, lenguaje y errores típicos reales.

No conviene pedir al docente que revise parámetros bayesianos. Sí conviene que la IA genere automáticamente una lista de comprobación en lenguaje docente:

- “¿Estos errores son errores reales de tu alumnado?”
- “¿Hay alguna pregunta demasiado fácil/difícil para el curso?”
- “¿Las respuestas correctas y explicaciones son correctas?”
- “¿Falta algún caso importante del tema?”
- “¿El lenguaje es adecuado para la edad?”

Esto preserva el principio de mínima carga metodológica y mejora la calidad del banco.

**Corrección aplicada (2026-07-09):** nueva sección «Revisión docente del contenido» en la especificación (la IA genera la lista de comprobación al entregar el recurso, en conversación o vista docente, nunca en la del alumno; si el docente señala un problema con sus palabras, la IA ajusta el banco o el modelo), párrafo en el protocolo §21 y aviso en la guía docente §6 («Revisa el contenido, no la técnica»), en ES/CA/EN.

## T5. Crédito parcial: la selección sigue siendo demasiado binaria en tareas por pasos — Media

La actualización con crédito parcial está bien corregida, pero la selección de la siguiente actividad permite seguir usando la aproximación binaria de acierto/fallo pleno. En tareas procedimentales, lo más diagnóstico suele ser qué paso falla, no si el ejercicio completo se acierta o falla.

Riesgo: en recursos de matemáticas, programación, resolución de problemas o procedimientos largos, la IA puede elegir ítems por una ganancia estimada que no refleja la evidencia parcial que realmente observará.

No debe pedirse al docente que modele la distribución de puntuaciones parciales. La IA debería aplicar una regla automática:

- si el ejercicio tiene subcriterios definidos, calcular la ganancia esperada por componentes cuando sea posible;
- si no se conoce la distribución de `s`, usar escenarios discretos plausibles (`0`, parcial bajo, parcial alto, `1`) en lugar de solo acierto/fallo;
- reservar la aproximación binaria para ítems realmente binarios o cuando el crédito parcial sea marginal.

**Corrección aplicada (2026-07-09), rectificando el enunciado.** El diagnóstico de T5 es correcto —la selección y la actualización no hablaban del mismo espacio de resultados— pero **el remedio propuesto en el segundo punto es dañino y se ha descartado tras medirlo**. La primera viñeta, en cambio, resultó ser la corrección de verdad, y su alcance es mayor de lo que T5 supone: no es un ajuste de la selección, es un cambio de la actualización.

**Lo que se midió.** Recurso real (`labcom`: 7 subcriterios ponderados con suelos de azar `1/2` y `1/6`, 3 hipótesis en `θ ∈ {−2,0,2}`, banco de 39 ítems por tipo con `b ∈ {−1,0,+1}`, `a_ef=1.25`, techo 0.95). Sesiones de 4 preguntas, 1200 respondentes por hipótesis, 3 semillas. Exactitud equilibrada y probabilidad media asignada a la hipótesis verdadera:

| Selección | Actualización | Exactitud | `P(θ real)` |
|---|---|---|---|
| dos extremos `s ∈ {0,1}` (regla vigente) | geométrica | 98.5 % | 0.75 |
| distribución exacta de `s` | geométrica | 95.9 % | 0.71 |
| **cuatro escenarios de `s` (lo que pide T5)** | geométrica | **93.7 %** | 0.68 |
| por componentes | por componentes | **99.3 %** | **0.99** |

**Por qué el remedio de T5 empeora las cosas.** La verosimilitud geométrica se maximiza en `p_i = s`: manda toda puntuación intermedia hacia las hipótesis intermedias. Un ítem de dificultad media produce puntuaciones intermedias con *cualquier* alumno, así que concentra el posterior geométrico en la hipótesis del medio con independencia del estado real. Promediar la ganancia sobre la distribución verdadera de `s` mientras se actualiza con la geométrica hace que el selector detecte esa concentración, la lea como información y prefiera justo los ítems que no discriminan. La huella es visible: el uso de ítems de dificultad media sube del `9 %` al `27 %`. La expresión `IG = H(p) − Σ_s P(s)·H(p'|s)` solo es una información mutua si `p'` es el posterior bajo el modelo que **genera** `s`; con un posterior geométrico deja de serlo y pasa a medir la concentración de una creencia mal especificada.

**La regla que se adopta** es de coherencia: *la ganancia se calcula con la misma verosimilitud con la que se actualiza*. De ahí:

1. Si los componentes se observan por separado, no se resumen en `s`: se multiplican sus verosimilitudes (lo que la especificación **ya prefería**) y la ganancia se promedia sobre sus combinaciones. Selección y actualización comparten modelo.
2. Si solo se dispone de la `s` agregada, se actualiza con la geométrica y la ganancia se promedia sobre los dos extremos: es exactamente la fórmula binaria de fundamentos §6.2, y es la coherente con esa verosimilitud.
3. No se cruzan los dos niveles.

**Efecto colateral encontrado.** El resumen geométrico no era «conservador», era **infra-confiado**: asignaba `0.75` de probabilidad a la hipótesis verdadera mientras acertaba el `98 %` de las veces, es decir, descartaba evidencia que ya tenía. El modelo por componentes la recupera (`0.99`, en línea con su exactitud).

**Riesgo de sobreconteo (hallazgo 2.4) descartado empíricamente.** El producto de verosimilitudes supone independencia condicional. Se simuló un mundo con dependencia fuerte (si el alumno no identifica el tipo, la fórmula y los parámetros caen a nivel de azar): el modelo por componentes sigue ganando (`97.3 %` frente a `94.4 %`) y su sobreconfianza medida es de `−0.3` puntos porcentuales. El aviso de 2.4 sigue vigente para el atajo del exponente `J`, que multiplica la evidencia de *un* ítem por `J`; no para el producto de componentes efectivamente observados por separado.

**Dónde se aplicó.** Especificación «Respuestas con crédito parcial», protocolo §5.8 y fundamentos §6.6 (nueva subsección), en ES/CA/EN. En el código: `labcom` es el único programa con crédito parcial por componentes; se cambió `updateBayesType` al producto de verosimilitudes, `bayesIG` al promedio sobre las `2^7` combinaciones, y la validación Monte Carlo, que simulaba una respuesta binaria por ítem y por tanto no reproducía el modelo del recurso. Documentación de `labcom` actualizada en sus cinco lenguas.

## T6. El techo de dominio como recorte duro puede introducir una discontinuidad artificial — Baja-media — **DESESTIMADO (2026-07-09)**

*Enunciado original:* el techo `P(acierto) <= 0.95`, formulado como recorte duro, aplana la curva por arriba y puede perder discriminación entre niveles altos en ítems fáciles; dos niveles altos pueden recibir la misma verosimilitud, «reduciendo información útil y creando una discontinuidad no psicométrica». Se proponía sustituirlo por un 4PL con asíntota superior.

**Se desestima.** Tres razones, la tercera cuantificada.

**1. No hay discontinuidad.** `P(θ) = min(c + (1−c)·σ(a(θ−b)), 0.95)` es continua en todo su dominio: lo que tiene en el punto de corte es un codo, un salto de la *derivada*, no de la función. Ninguna verosimilitud da un salto. La objeción nombra un defecto que el recorte no tiene.

**2. La discriminación ya se deriva del techo, no a pesar de él.** La regla del método es `a = a_ef/(d − c_q)` con `d = 0.95` el propio techo: la pendiente se calibra *contando con* la asíntota. El techo no es un parche aplicado después de ajustar la curva. Un 4PL con asíntota superior obligaría a rederivar esa relación y a recalcular los números documentados en los seis programas hermanos.

**3. La información que el techo elimina es precisamente la que no debía existir.** Verificado numéricamente (`n = 3` niveles en `θ = −1, 0, +1`, `a_ef = 1.25`, banco de 18 ítems combinando formato y dificultad):

- La pérdida de ganancia de información por el recorte **nunca supera 0.0497 bits**, sobre un máximo alcanzable de `log₂ 3 = 1.585`. En porcentaje la pérdida sí llega a ser grande (86 % en un V/F con `b = −1.5`), pero se concentra **exactamente en los ítems que ya eran los menos informativos del banco**: los que pierden mucho porcentaje partían de 0.04–0.11 bits, frente a los 0.17 del mejor ítem. El recorte degrada lo que el selector ya descartaba.
- Con prior uniforme, el ítem elegido es el mismo con techo y sin él (`abierta`, `b = 0.0`).
- Con un alumno de nivel alto (posterior 0.02 / 0.33 / 0.65) el techo **sí cambia la elección**: sin techo el selector pide un ítem de `b = 0.0`; con techo pide uno de `b = +0.5`. **Ese cambio es la conducta correcta**, no un defecto. La supuesta «información útil» del ítem fácil provenía solo de que el modelo sin techo asume `P(acierto) = 0.97` para un alumno fuerte, es decir, trata un despiste en un ítem fácil como prueba casi concluyente de no-dominio. Eliminar esa inferencia es la razón por la que existe el techo (hallazgo 1.3).

Queda un efecto real y aceptado: si un ítem recortado llega a administrarse igualmente (banco pequeño, etapa forzada), no distingue entre los dos niveles altos. Contribuir cero es la contribución correcta cuando el modelo no puede separarlos sin asumir una certeza implausible.

Coste alto (rederivar la discriminación, recalcular la documentación numérica de seis repos) frente a un beneficio acotado por ~0.05 bits en ítems que el selector evita. **No se hará.** Si en el futuro se adoptara un banco con muchos ítems fáciles obligatorios, convendría reabrirlo.

## T7. Prior de errores: falta traducción automática desde lenguaje natural del docente — Baja-media

El prior informativo `P(error) ≈ 0.2`–`0.3` corrige el falso positivo del prior uniforme. Pero también puede sesgar hacia falso negativo cuando el docente describe un error muy frecuente en su grupo.

No debe pedirse al docente una prevalencia numérica. La IA puede inferir ajustes suaves desde lenguaje natural:

- “muchos”, “la mayoría”, “muy frecuente” → prior más alto, por ejemplo `0.4`;
- “algunos”, “a veces” → mantener `0.2`–`0.3`;
- “casos aislados”, “pocos” → prior más bajo, por ejemplo `0.1`–`0.2`.

La especificación debería definir esta traducción como heurística automática y permitir que el recurso explique: “parto de que este error puede ser frecuente porque así se ha descrito”.

**Corrección aplicada (2026-07-09):** heurística añadida a la especificación («Verosimilitudes» + «Valores por defecto») y al protocolo §5.1, en ES/CA/EN. Tabla `muchos → 0.40` / `algunos o nada dicho → 0.20–0.30` / `pocos → 0.10–0.15`, con tres cotas verificadas numéricamente:

- **Nunca `≥ 0.5` ni `< 0.10`.** Por encima de `0.5` el prior afirma que el error es más probable que su ausencia (el sesgo que el prior informativo corrige). Por debajo de `0.10` se necesitan `3` fallos discriminantes para confirmar un error real, que un banco pequeño puede no dar.
- **La muestra mínima no se relaja.** Es lo único que protege en el extremo alto: desde `0.40`, un solo fallo discriminante (`LR ≈ 4`) sube la marginal a `0.727` y cruzaría el umbral de «presente». Sin la muestra mínima, la palabra del docente diagnosticaría al alumno.
- **El prior es el ancla del olvido (N1).** Un factor descrito como frecuente vuelve a `0.40` sin evidencia reciente, no a `0.25`. Es la conducta coherente, y se documenta para que no se lea como una regresión de N1.

El ajuste desplaza la decisión **un ítem** (`2` fallos discriminantes desde el defecto, `1` desde `0.40`, `3` desde `0.10`) y en ningún punto del rango permitido el prior por sí solo declara un error «presente». No se toca fundamentos: es una heurística operativa, sin derivación matemática nueva.

## T8. Ambigüedad terminológica: `theta` como referencia común en dimensiones nominales — Baja

La especificación dice que cada dimensión puede tener su propio suelo de azar y que sus porcentajes no son directamente comparables; añade que la referencia común es `theta`. Esto es válido para dimensiones ordinales, pero no para factores nominales de error, donde no hay `theta` con significado.

Riesgo: una IA puede intentar calcular o comparar `theta` en diagnósticos nominales, justo lo que otros apartados prohíben.

Conviene separar:

- en dimensiones ordinales, la referencia latente puede ser `theta`;
- en factores nominales, solo se reportan probabilidades marginales, estado (`presente`, `ausente`, `indeterminado`) y evidencia mínima;
- no debe existir `theta` esperada para errores sin orden.

**Corrección aplicada (2026-07-09):** la distinción ordinal/nominal ya estaba incorporada en la especificación y el protocolo (sin registrar); se completa el punto que faltaba, fundamentos §2.4, en ES/CA/EN (con remisión a §10.3).

## T9. El informe histórico puede confundirse con estado vigente — Baja (documental)

El propio `INFORME_REVISION.md` contiene hallazgos antiguos ya corregidos, seguidos de rondas posteriores. Eso es útil como histórico, pero puede inducir a pensar que fallos ya aplicados siguen vigentes si se lee parcialmente.

Sugerencia documental: añadir al inicio un índice de estado o una nota visible:

- primera ronda: hallazgos históricos, aplicados;
- segunda ronda: hallazgos históricos, aplicados;
- tercera ronda: puntos propuestos para estudio, no aplicados.

Esto evitaría duplicar en futuros informes problemas que ya fueron corregidos.

**Corrección aplicada (2026-07-09):** añadido al inicio del informe el bloque «Estado del informe», con tabla por rondas y mención explícita de que solo T5 y T7 siguen abiertos; las cabeceras de la primera y la segunda ronda las marcan como históricas y aplicadas; cada punto de la tercera ronda lleva su estado en su propia sección.

## Resumen priorizado de la tercera ronda

| # | Punto a estudiar | Gravedad | Estado |
|---|------------------|----------|--------|
| T1 | Ponderación automática en recursos mixtos nivel + errores | Media-alta | Aplicado y parcheado |
| T3 | Validación Monte Carlo generada siempre que proceda, ejecutada si el entorno lo permite | Media-alta | Aplicado |
| T2 | Criterio IA para elegir factores independientes vs perfiles completos | Media | Aplicado |
| T4 | Revisión docente de contenido, no metodológica | Media | Aplicado |
| T5 | Selección demasiado binaria en tareas con crédito parcial | Media | Aplicado **rectificando el enunciado** (el remedio propuesto degradaba el diagnóstico; el fallo estaba en la actualización, no en la selección) |
| T6 | Techo de dominio como recorte duro en vez de modelo de slip/asíntota | Baja-media | **Desestimado** (no hay discontinuidad; pérdida ≤ 0.05 bits en ítems que el selector evita) |
| T7 | Priors de errores ajustables desde lenguaje natural del docente | Baja-media | Aplicado |
| T8 | Uso ambiguo de `theta` en dimensiones nominales | Baja | Aplicado |
| T9 | Separar mejor histórico corregido y estado vigente del informe | Baja | Aplicado |
