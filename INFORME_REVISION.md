# Informe de revisión metodológica, conceptual y pedagógica

**Documentos revisados:** `especificacion_operativa_ia.md` (v2.0), `documentacion.html` (v2.0), `matematicas.html` (v2.0), `guia_docente.html` (v1.1), `README.md`, `CAMBIOS_METODOLOGIA.md`.
**Fecha:** 2026-07-08.
**Método:** lectura completa de los cuatro documentos, verificación numérica independiente del ejemplo de la sección 9 de los fundamentos matemáticos, y comprobación por cálculo de las afirmaciones cuantitativas sobre escala, discriminación efectiva y criterios de parada.

---

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

# Segunda ronda (2026-07-09)

**Método:** relectura completa de los cuatro documentos ya en v2.0 (con los 11 hallazgos de la primera ronda aplicados), buscando fallos nuevos, con verificación numérica de los hallazgos N1–N3.

## N1. El olvido exponencial deriva los priors informativos hacia la uniforme — Alta

La regla `p_i ← p_i^λ` renormalizada tiene la **uniforme** como punto fijo: sin evidencia nueva, cualquier distribución deriva hacia ella. Verificado: un factor de error con prior `P(error) = 0.25` sin evidencia sube a 0.34 en 10 pasos, 0.40 en 20 y ≈ 0.47 en 40 con `λ = 0.95`, reapareciendo como «indeterminado» o «probable» sin que el alumno haya hecho nada: en práctica continua, el olvido **reintroducía el falso positivo que el hallazgo 1.2 corrigió**. La variante «mezclar con la uniforme» tenía el mismo defecto de forma explícita. Además, la redacción («antes de cada actualización») era ambigua sobre si se atenúan las distribuciones que no reciben evidencia; las dos lecturas fallaban (deriva a 0.5 o ausencia total de olvido). **Corrección aplicada:** olvido anclado al prior, `p_i ∝ p_i^λ · π_i^(1−λ)` (punto fijo = prior; el prior conserva peso exactamente 1 al desenrollar la recursión; con prior uniforme coincide con la regla anterior), atenuación de todas las distribuciones en cada paso, y variante de mezcla hacia el prior. Consecuencia asociada corregida: la muestra mínima por categoría debe contarse en ventana reciente (`W ≈ 1/(1−λ)`), porque la evidencia caduca con el olvido pero un contador acumulado no.

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
| N2 | Reuso de ítems tras feedback sin descuento de evidencia | Media-alta | Corregido |
| N3 | Ítems de salida dependientes del formato (V/F deja pasar 61 % de no-dominantes) | Media | Corregido |
| N4 | Ejemplo §9, §4.3 y protocolo §12 sin el techo de dominio | Media | Corregido |
| N5 | Selección y parada indefinidas con dimensiones paralelas | Media | Corregido |
| N6 | «Ausente» declarable sin evidencia por el prior informativo | Baja-media | Corregido |
| N7 | Combinado A+D sin reconciliación nivel↔factores | Baja-media | Corregido |
| N8 | Utilidad de refuerzo sin escalas definidas | Baja | Corregido |
| N9 | `l_z` de etapa con aproximación normal débil | Baja | Corregido |
