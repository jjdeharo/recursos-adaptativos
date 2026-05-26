# Recursos educativos adaptativos bayesianos

Protocolo y ejemplos para crear recursos educativos que se adaptan al alumno usando inferencia bayesiana y entropía de Shannon.  
Acceso: **https://jjdeharo.github.io/recursos-adaptativos/**

---

## Qué es esto

Un recurso educativo adaptativo bayesiano es una aplicación que modifica su comportamiento en función de las respuestas del alumno. En lugar de seguir una secuencia fija, el sistema mantiene una estimación probabilística sobre el estado de aprendizaje y selecciona en cada momento la acción más informativa: la pregunta que más reduce la incertidumbre, la explicación más adecuada al nivel estimado, el ejercicio con mayor utilidad pedagógica.

El protocolo recogido en este sitio define cómo diseñar e implementar ese mecanismo de forma rigurosa. Cualquier modelo de IA puede generar un recurso concreto a partir de él.

---

## Crear un recurso con IA

1. Descarga el archivo [`documentacion_evaluacion_adaptativa_bayesiana.md`](documentacion_evaluacion_adaptativa_bayesiana.md)
2. Adjúntalo a cualquier modelo de IA (Claude, GPT-4, Gemini…)
3. Escribe el prompt:

   > Lee el documento adjunto e implementa el sistema siguiendo las instrucciones que contiene.

El modelo pedirá al docente el tema, el curso, el tipo de recurso y la finalidad antes de diseñar nada. El propio documento incluye una plantilla para anticipar esa información.

El resultado es siempre una página web estática autocontenida (HTML + CSS + JS en un único archivo), lista para usar sin servidor.

---

## Documentación matemática

Los fundamentos matemáticos detallados del sistema están disponibles en [`matematicas.md`](matematicas.md) e incluyen:

- Derivación completa de la regla de Bayes aplicada al estado del alumno
- Modelo IRT 3PL con análisis del parámetro de discriminación y la curva característica del ítem
- Entropía de Shannon: propiedades y ejemplos numéricos
- Ganancia esperada de información: desarrollo paso a paso y conexión con la información mutua
- Derivación del umbral de parada $H_{\text{stop}}$
- Justificación de la convención de escala θ/b (factor 2)
- Ejemplo numérico completo de una sesión adaptativa
- Hipótesis no jerárquicas y límites del modelo

---

## Protocolo completo

La documentación técnica y pedagógica está disponible en [`documentacion.html`](https://jjdeharo.github.io/recursos-adaptativos/documentacion.html) e incluye:

- Fundamentos de la inferencia bayesiana aplicada a la educación
- Modelo IRT de tres parámetros para calcular verosimilitudes dinámicas
- Selección adaptativa de preguntas por máxima ganancia de información (entropía de Shannon)
- Criterios de finalización basados en incertidumbre
- Mecanismos de recuperación automática
- Instrucciones completas de implementación para modelos de IA
- Plantilla para que el docente aporte el contexto educativo

---

## Ejemplos de aplicación

### Test adaptativo bayesiano
**https://jjdeharo.github.io/bayes-test/**  
Evaluación diagnóstica que selecciona preguntas según la ganancia esperada de información. Estima el nivel del alumno (básico, medio o avanzado) y ofrece una interpretación pedagógica al finalizar.  
Repositorio: [github.com/jjdeharo/bayes-test](https://github.com/jjdeharo/bayes-test)

### Itinerario adaptativo bayesiano
**https://jjdeharo.github.io/bayes-itinerario/**  
Recurso de aprendizaje que adapta el recorrido, las explicaciones y las actividades al estado estimado del alumno a lo largo de la sesión.  
Repositorio: [github.com/jjdeharo/bayes-itinerario](https://github.com/jjdeharo/bayes-itinerario)

---

## Estructura del proyecto

```
recursos-adaptativos/
├── index.html                                        Página principal del sitio
├── documentacion.html                                Protocolo completo (versión web)
├── documentacion_evaluacion_adaptativa_bayesiana.md  Protocolo en Markdown (para adjuntar a IA)
├── matematicas.html                                  Fundamentos matemáticos (versión web)
├── matematicas.md                                    Fundamentos matemáticos (Markdown)
└── README.md                                         Este documento
```

---

## Licencias

- Contenido: [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
- Código: [AGPL-3.0](https://www.gnu.org/licenses/agpl-3.0.html)

---

## Autor

Juan José de Haro · [bilateria.org](https://bilateria.org)
