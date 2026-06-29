# Especificación operativa para IA

**Versión 1.0**

## Propósito

Este documento sirve para que una IA implemente recursos educativos adaptativos bayesianos de forma fiable.  
Es una especificación operativa breve.  
Si hace falta fundamento teórico, ejemplos o justificación matemática, consulta `documentacion.html` y `matematicas.html`.
Si hubiera conflicto entre ambos documentos, prevalece este.

## Instrucción de uso

Adjunta este documento a la IA y usa este prompt:

> Lee el documento adjunto e implementa el recurso siguiendo exactamente sus reglas operativas.

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
4. El resultado debe ser una página web estática autocontenida en un único archivo `HTML + CSS + JavaScript`, salvo que se pida explícitamente otro formato.

## Reglas de diseño

- El recurso no debe ser lineal si la finalidad exige adaptación.
- Cada respuesta del alumno debe modificar el estado estimado del sistema.
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
- Cuando uses dificultades `b_q` centradas, haz que la escala `theta` abarque un rango mayor que la escala de dificultad. Regla práctica: `theta_max = 2 * max(abs(b_q))`.

## Actualización bayesiana

Tras cada respuesta:

1. calcula la verosimilitud de esa respuesta bajo cada hipótesis;
2. multiplica prior por verosimilitud;
3. normaliza;
4. usa el resultado como nuevo estado.

Esto debe hacerse después de cada interacción relevante.

## Verosimilitudes

- Si las hipótesis representan niveles ordenados de dominio, usa IRT 3PL.
- Fórmula recomendada:

`P(acierto | H_i, q) = c_q + (1 - c_q) / (1 + exp(-a * (theta_i - b_q)))`

- Usa por defecto:
  - `a = 1.5`
  - `c_q = 1 / m_q` si hay azar
  - `c_q = 0` si no lo hay
- Si el alumno falla:

`P(fallo | H_i, q) = 1 - P(acierto | H_i, q)`

- Si las hipótesis no son jerárquicas, no uses IRT logística. Define verosimilitudes diagnósticas específicas.
- No uses tablas fijas globales si cada pregunta puede generar sus propias verosimilitudes.

## Respuestas con crédito parcial

- Si una respuesta no es solo acierto o fallo, sino que admite grados (varios pasos, componentes ponderados, aciertos parciales), resúmela en una puntuación `s` entre `0` y `1`.
- Construye la verosimilitud por interpolación entre acierto y fallo:

`L(H_i) = s * P(acierto | H_i, q) + (1 - s) * P(fallo | H_i, q)`

- Usa esta `L(H_i)` en la actualización bayesiana en lugar de elegir entre `P(acierto)` y `P(fallo)`. La normalización y el resto del proceso no cambian.
- Casos límite: `s = 1` equivale a acierto pleno; `s = 0` a fallo pleno; `s = 0.5` no aporta información y deja el posterior casi intacto.
- Define cómo se calcula `s` de forma explícita y autocorregible: suma ponderada de subcriterios, fracción de pasos correctos, cercanía a la solución numérica, etc. Los pesos deben sumar `1`.
- No trates como binaria una respuesta que admite grados: pierdes información diagnóstica.

## Suelo de azar en ítems compuestos

- Si un ítem se corrige por varios componentes con distinto número de opciones, su probabilidad de acierto por azar no es `1/m`.
- Calcula el suelo agregado como media ponderada de los azares de cada componente:

`c_q = Σ_j w_j * c_j`  con  `c_j = 1 / m_j`  y  `Σ_j w_j = 1`

- Usa ese `c_q` agregado en la función logística al generar las verosimilitudes del ítem completo.
- Los pesos `w_j` deben coincidir con los que uses para calcular la puntuación `s` del crédito parcial.

## Diagnóstico multidimensional

- Si necesitas saber no solo el nivel global sino qué habilidades o pasos fallan, mantén varias distribuciones bayesianas en paralelo: una por categoría o nivel y otra por cada dimensión diagnóstica (habilidad, paso, tipo de error).
- Actualiza con la misma respuesta todas las distribuciones relevantes: el resultado global alimenta la creencia de nivel; cada subcriterio alimenta la creencia de su dimensión.
- Cada dimensión puede tener su propio suelo de azar `c` según su número de opciones, por lo que sus porcentajes no son directamente comparables entre sí: la referencia común es el valor latente `theta`.
- No metas en una sola distribución dimensiones que pueden coexistir: usa distribuciones separadas.
- El nivel por categoría guía qué practicar; el diagnóstico por dimensión guía qué explicar o reforzar.

## Preguntas y actividades

Cada pregunta o interacción autocorregible debe tener, cuando proceda:

- texto o enunciado;
- dificultad `b_q`;
- número de opciones `m_q`;
- categoría o concepto;
- criterio de corrección;
- ayuda o pista opcional;
- explicación o retroalimentación.

Si el recurso es procedural o tutorial, las interacciones pueden no ser preguntas clásicas, pero deben seguir siendo autocorregibles o evaluables de forma explícita.

## Selección adaptativa

Para cada candidata disponible:

1. calcula la probabilidad marginal de acierto;
2. simula posterior si hay acierto;
3. simula posterior si hay fallo;
4. calcula la entropía esperada posterior;
5. calcula la ganancia esperada de información.

Selecciona la candidata con mayor ganancia esperada de información.

En recursos de práctica adaptativa o refuerzo con varias categorías, tipos de problema o conceptos, usa una selección en dos fases:

1. **Fase diagnóstica inicial.** Hasta que cada categoría relevante tenga una muestra mínima de intentos, prioriza las categorías con menos evidencia. Dentro de ellas, usa la ganancia esperada de información para elegir la pregunta más diagnóstica. Un valor por defecto razonable es exigir al menos `2` intentos por categoría antes de salir de esta fase.
2. **Fase de refuerzo.** Cuando todas las categorías relevantes tengan muestra mínima, prioriza la categoría con menor dominio estimado. Dentro de esa categoría, no elijas automáticamente la pregunta más difícil: selecciona una pregunta informativa y cercana al nivel estimado del alumno. Una regla razonable es combinar ganancia de información con adecuación de dificultad, penalizando preguntas demasiado alejadas de la zona de trabajo.

No uses la entropía de Shannon como único criterio permanente cuando la finalidad principal sea practicar o reforzar. Shannon indica dónde hay más incertidumbre diagnóstica; el refuerzo debe atender también, y preferentemente, a lo que el alumno domina menos.

Si varias son prácticamente equivalentes:

- rompe empates con aleatorización;
- favorece categorías o conceptos menos repetidos.

No uses selección determinista simple en empates.

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

No declares un dominio alto basándote en muy pocos intentos. Exige una muestra mínima por categoría o dimensión antes de mostrar niveles altos de dominio, y limita o marca como provisional la estimación cuando la evidencia sea escasa.

No devuelvas solo una nota o etiqueta.

## Restricciones de implementación

- La interfaz debe ser comprensible para alumnado y profesorado.
- Debe mostrar con claridad el progreso y la retroalimentación.
- Si usas fórmulas visibles, acompáñalas de interpretación legible.
- Evita depender de backend si no se ha pedido.
- Evita preguntas abiertas largas sin corrección automática fiable.

## Valores por defecto recomendados

Si el docente no especifica parámetros:

- `n = 3` hipótesis o niveles;
- `a = 1.5`;
- `p_min = 0.80`;
- mínimo de preguntas: entre `4` y `6`;
- mínimo de diagnóstico por categoría en práctica adaptativa: `2` intentos;
- máximo práctico: entre `10` y `20`, según el tipo de recurso;
- peso de la ganancia de información en la fase de refuerzo: `α = 0.65` (rango razonable `0.6`–`0.7`);
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

## Nota sobre el modelo

La función IRT 3PL descrita aquí se usa como generador inicial de verosimilitudes cuando no hay datos empíricos. No equivale a una calibración psicométrica validada. Si se acumulan suficientes respuestas reales, conviene recalibrar dificultades, discriminación y pseudoazar.
