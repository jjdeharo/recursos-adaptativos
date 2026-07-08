# Cambios metodológicos

Registro breve de cambios técnicos relevantes en la metodología pública.

## 2026-07-08 (tarde) — Escala θ fija

- Versiones unificadas a `2.0` en los tres documentos (especificación operativa, protocolo y fundamentos matemáticos), en ES/CA/EN.
- **Escala `theta` fija por convención en lugar de acoplada al banco.** Se sustituye la regla `theta_max = 2·max|b_q|` (que recalculaba la escala a partir de los extremos del banco) por una escala fija que depende solo del número de hipótesis `n`: `theta_i = 2·(i-1) - (n-1)`, con `theta_max = n - 1` y espaciado 2 entre hipótesis adyacentes. Esto generaliza limpiamente a cualquier `n` (2, 3, 4, …) y coincide con la tabla ya presente en §2.3 de los fundamentos.
- **Las dificultades se sitúan dentro de la escala, no al revés.** `b_q ∈ [-theta_max/2, +theta_max/2]`; los valores atípicos se recortan (clamp) a ese intervalo. Motivo: una sola pregunta atípica ya no estira la escala ni satura las verosimilitudes del resto del banco, y los diagnósticos quedan comparables entre recursos con bancos distintos.
- **Conversión de dificultades cualitativas dependiente de `n`.** Se sustituye la tabla fija (2→±0,5; 3→±1; 4→±1,5; 5→±2), que dependía solo del número de categorías, por `b_j = (theta_max/2)·(2(j-1)/(k-1) - 1)`. Así la conversión depende del número de niveles además del de categorías y se evita la incoherencia con pocas hipótesis y muchas categorías (p. ej. `n=2` asignaba dificultades hasta ±2, invirtiendo la separación entre escalas).
- Se elimina el caso límite `max|b_q| = 0 → theta_max = 1`, innecesario con la escala fija.
- Invariante de comparabilidad `a_ef · Δtheta = 2.5` (con `a_ef = 1.25` e intervalos de 2), estable entre recursos y para cualquier `n`.
- Ejemplos alineados con la 2.0:
  - **labcom**: sustituida la verosimilitud lineal de crédito parcial por la geométrica `P(A)^s·P(F)^(1-s)` en el motor (`index.html`: `updateBayesType`, `updateDim`), en el script de validación (`validacion-banco.html`) y en toda la documentación (`apendice` y `funcionamiento` en ES/CA/EN/EU/GA), con el ejemplo numérico recalculado (con `s=0.75` el posterior se desplaza al nivel intermedio, no siempre al superior). Su escala ya era fija `theta {-2,0,2}`.
  - **bayes-test**: sustituida la escala acoplada al banco (`theta = 2·max|b_q|`) por la escala fija `theta_max = n-1` con recorte de dificultades a `[-1,+1]`, en `index.html`, `scripts/simulate.js`, `validacion.html` y `ayuda.html`, y actualizado el `README.md`. La simulación Monte Carlo mantiene la separación de niveles (≈91/88/94 %), ya que las dificultades del banco (`{-1,0,1}`) están dentro de la escala.
  - **bayes-acentuacion** ya usaba la escala fija `{-2,0,2}`.

## 2026-07-08

- Versiones actualizadas: especificación operativa `1.10`, protocolo `1.13` y fundamentos matemáticos `1.8`, en ES/CA/EN.
- Sustituida la regla lineal de crédito parcial por una verosimilitud geométrica `p^s(1-p)^(1-s)` en la especificación operativa, el protocolo y los fundamentos. Motivo: la regla lineal favorecía siempre la hipótesis con mayor `p` cuando `s > 0.5`, aunque la puntuación parcial fuese más compatible con un nivel intermedio.
- Aclarado que, si hay componentes observables, es preferible multiplicar verosimilitudes por componente; la puntuación agregada `s` es una aproximación.
- Reformulado el suelo de azar de ítems compuestos: la media ponderada es puntuación parcial esperada por azar, no probabilidad de acertar el ítem completo. Para corrección todo-o-nada, el suelo correcto es el producto de los azares de los componentes independientes.
- Rebajada la interpretación de `a_ef = a(1-c_q)`: iguala la pendiente máxima de la ICC, no la información esperada de todos los formatos.
- Unificada la recomendación de discriminación: se mantiene `a_ef=1.25` como defecto operativo y se evita presentar `a=1.5` como valor directo por defecto.
- Matizada la escala `theta_max = 2*max(abs(b_q))`: queda como heurística rápida, no como regla más robusta; se recomienda fijar la escala por constructo o probabilidad objetivo cuando sea posible.
- Añadida cautela en diagnóstico multidimensional: cada dimensión debe actualizarse con evidencia específica, no con el mismo acierto/fallo global si varias habilidades intervienen simultáneamente.
- Ajustado el criterio de empates por ganancia de información: `ε = 10^-9` queda como empate numérico exacto; para ganancias próximas se recomienda margen relativo práctico.
- Matizado el uso de FII frente a ganancia de información: la diferencia relevante es criterio puntual frente a criterio sobre toda la distribución posterior, no una oposición simple entre frecuentismo y Bayes.
- Reforzada la cautela sobre `l_z` en tests adaptativos y añadidas referencias a Snijders (2001) y van Krimpen-Stoop & Meijer (2002).
