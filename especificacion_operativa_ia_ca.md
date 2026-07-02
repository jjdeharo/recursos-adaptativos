# Especificació operativa per a IA

**Versió 1.5**

## Propòsit

Aquest document serveix perquè una IA implementi recursos educatius adaptatius bayesians de manera fiable.  
És una especificació operativa breu.  
Si cal fonament teòric, exemples o justificació matemàtica, consulta `documentacion.html` i `matematicas.html`.  
Si hi hagués conflicte entre tots dos documents, preval aquest.

## Instrucció d'ús

Adjunta aquest document a la IA i utilitza aquest prompt:

> Llegeix el document adjunt i implementa el recurs seguint les regles operatives aplicables al tipus de recurs sol·licitat.

## Flux obligatori

1. Abans d'implementar, comprova si tens aquesta informació:
   - tema;
   - curs o edat;
   - objectiu d'aprenentatge;
   - tipus de recurs;
   - finalitat principal;
   - nombre d'hipòtesis o nivells;
   - tipus d'interacció;
   - nombre aproximat de preguntes o passos;
   - sortida final esperada.
2. Si falta informació essencial, pregunta només per allò imprescindible.
3. Si la informació ja ha estat donada, no repeteixis preguntes i passa a implementar.
4. El resultat ha de ser un recurs web estàtic en `HTML + CSS + JavaScript`, preferentment sense backend, llevat que s'hagi demanat explícitament un altre format. Es pot organitzar en un o diversos fitxers si això millora la claredat, el manteniment o la reutilització.

## Regles de disseny

- El recurs no ha de ser lineal si la finalitat exigeix adaptació.
- Cada resposta de l'alumne ha de modificar l'estat estimat del sistema.
- L'adaptació pot afectar:
  - la pregunta següent;
  - la dificultat;
  - el tipus d'activitat;
  - l'explicació;
  - l'ajuda o les pistes;
  - l'itinerari;
  - el moment de finalitzar.
- El resultat final ha de ser pedagògic, no només una puntuació.

## Estat de l'alumne

- Representa l'estat de l'alumne com una distribució de probabilitat sobre `n` hipòtesis.
- Si no hi ha informació prèvia fiable, utilitza una distribució uniforme.
- Si les hipòtesis són jeràrquiques, assigna-les valors `theta` ordenats i centrats.
- Si el docent no fixa valors, utilitza una escala simètrica centrada en 0.
- Quan facis servir dificultats `b_q` centrades, fes que l'escala `theta` abasti un rang més gran que l'escala de dificultat. Regla pràctica: `theta_max = 2 * max(abs(b_q))`.
- Cas límit: si `max(abs(b_q)) = 0` (totes les preguntes amb la mateixa dificultat central), aquesta regla donaria `theta_max = 0` i les hipòtesis col·lapsarien en el mateix valor. Fes servir llavors un mínim `theta_max = 1`.

## Actualització bayesiana

Després de cada resposta:

1. calcula la versemblança d'aquesta resposta sota cada hipòtesi;
2. multiplica el prior per la versemblança;
3. normalitza;
4. utilitza el resultat com a nou estat.

Això s'ha de fer després de cada interacció rellevant.

## Versemblances

- Si les hipòtesis representen nivells ordenats de domini, utilitza IRT 3PL.
- Fórmula recomanada:

`P(encert | H_i, q) = c_q + (1 - c_q) / (1 + exp(-a * (theta_i - b_q)))`

- Utilitza per defecte:
  - `a = 1.5`
  - `c_q = 1 / m_q` si hi ha atzar
  - `c_q = 0` si no n'hi ha
- Si l'alumne falla:

`P(error | H_i, q) = 1 - P(encert | H_i, q)`

- Si les hipòtesis no són jeràrquiques (errors conceptuals sense ordre), no facis servir IRT logística. Genera per a cada pregunta un vector de `n` versemblances `P(encert | H_i, q)`, una per hipòtesi.
- Assigna cada valor responent: «si l'alumne tingués H_i, amb quina probabilitat encertaria aquesta pregunta?». Baix si la pregunta ataca el concepte que aquest error distorsiona; alt si l'error no interfereix.
- Acota cada valor: no per sota del terra d'atzar `1/m` (m opcions), llevat de l'excepció del punt següent; la hipòtesi de domini al voltant de `0.9`–`0.95`, mai `1`.
- Si un distractor concret és la resposta que produeix H_i, posa aquesta cel·la a prop del terra o per sota: l'alumne és atret activament cap a aquesta opció equivocada.
- No afinis el decimal: fes servir trams (`≈0.9` no afecta / `≈0.5` afectació parcial / `≈0.15–0.25` distractor que captura l'error / `≈1/m` terra si falla per atzar). El que importa és que la hipòtesi correcta superi clarament les altres en les preguntes que discriminen.
- L'error és el complementari `1 - P(encert | H_i, q)`. El vector no suma 1: són `n` probabilitats d'encert independents, no una distribució.
- No facis servir taules fixes globals si cada pregunta pot generar les seves pròpies versemblances.
- En el diagnòstic final no calculis una `theta` esperada (no té sentit sense ordre): reporta la hipòtesi de màxima probabilitat posterior (MAP) i la seva probabilitat com a confiança.

## Respostes amb crèdit parcial

- Si una resposta no és només encert o error, sinó que admet graus (diversos passos, components ponderats, encerts parcials), resumeix-la en una puntuació `s` entre `0` i `1`.
- Construeix la versemblança per interpolació entre encert i error:

`L(H_i) = s * P(encert | H_i, q) + (1 - s) * P(error | H_i, q)`

- Utilitza aquesta `L(H_i)` en l'actualització bayesiana en lloc de triar entre `P(encert)` i `P(error)`. La normalització i la resta del procés no canvien.
- Casos límit: `s = 1` equival a encert ple; `s = 0` a error ple; `s = 0.5` no aporta informació i deixa el posterior gairebé intacte.
- Defineix com es calcula `s` de manera explícita i autocorregible: suma ponderada de subcriteris, fracció de passos correctes, proximitat a la solució numèrica, etc. Els pesos han de sumar `1`.
- No tractis com a binària una resposta que admet graus: perds informació diagnòstica.
- Per seleccionar la pregunta següent, calcula el guany d'informació amb els dos escenaris binaris (encert i error plens) com a aproximació; la versemblança interpolada es fa servir només en l'actualització, un cop observada la resposta.

## Sòl d'atzar en ítems compostos

- Si un ítem es corregeix per diversos components amb un nombre diferent d'opcions, la probabilitat d'encert per atzar no és `1/m`.
- Calcula el sòl agregat com a mitjana ponderada dels atzars de cada component:

`c_q = Σ_j w_j * c_j`  amb  `c_j = 1 / m_j`  i  `Σ_j w_j = 1`

- Utilitza aquest `c_q` agregat en la funció logística quan generis les versemblances de l'ítem complet.
- Els pesos `w_j` han de coincidir amb els que facis servir per calcular la puntuació `s` del crèdit parcial.

## Diagnòstic multidimensional

- Si necessites saber no només el nivell global sinó quines habilitats o passos fallen, mantén diverses distribucions bayesianes en paral·lel: una per categoria o nivell i una per cada dimensió diagnòstica (habilitat, pas, tipus d'error).
- Actualitza amb la mateixa resposta totes les distribucions rellevants: el resultat global alimenta la creença de nivell; cada subcriteteri alimenta la creença de la seva dimensió.
- Cada dimensió pot tenir el seu propi sòl d'atzar `c` segons el seu nombre d'opcions, de manera que els seus percentatges no són directament comparables entre si: la referència comuna és el valor latent `theta`.
- No posis en una sola distribució dimensions que poden coexistir: utilitza distribucions separades.
- El nivell per categoria guia què practicar; el diagnòstic per dimensió guia què explicar o reforçar.

## Preguntes i activitats

Cada pregunta o interacció autocorregible ha de tenir, quan escaigui:

- text o enunciat;
- dificultat `b_q`;
- nombre d'opcions `m_q`;
- categoria o concepte;
- criteri de correcció;
- ajuda o pista opcional;
- retroalimentació mínima després de la resposta;
- explicació específica, especialment si la finalitat principal és aprenentatge, pràctica o reforç.

Si el recurs és procedural o tutorial, les interaccions poden no ser preguntes clàssiques, però han de continuar sent autocorregibles o avaluables de manera explícita.

## Selecció adaptativa

Per a cada candidata disponible:

1. calcula la probabilitat marginal d'encert;
2. simula el posterior si hi ha encert;
3. simula el posterior si hi ha error;
4. calcula l'entropia esperada posterior;
5. calcula el guany esperat d'informació.

Selecciona la candidata amb un guany esperat d'informació més gran quan la finalitat principal sigui diagnòstica i l'objectiu sigui estimar un nivell global.

En recursos de pràctica adaptativa o reforç amb diverses categories, tipus de problema o conceptes, utilitza una selecció en dues fases:

1. **Fase diagnòstica inicial.** Fins que cada categoria rellevant tingui una mostra mínima d'intents, prioritza les categories amb menys evidència. Dins d'elles, utilitza el guany esperat d'informació per triar la pregunta més diagnòstica. Un valor per defecte raonable és exigir almenys `2` intents per categoria abans de sortir d'aquesta fase. Si hi ha moltes categories, aquesta fase pot consumir la sessió (amb `10` categories i `2` intents ja són `20` preguntes): agrupa categories afins en blocs o redueix la mostra mínima perquè el diagnòstic inicial no superi el màxim pràctic.
2. **Fase de reforç.** Quan totes les categories rellevants tinguin mostra mínima, prioritza la categoria amb menys domini estimat. Dins d'aquesta categoria, no triïs automàticament la pregunta més difícil: selecciona una pregunta informativa i propera al nivell estimat de l'alumne. Una regla raonable és combinar guany d'informació amb adequació de dificultat, penalitzant preguntes massa allunyades de la zona de treball.

No facis servir l'entropia de Shannon com a únic criteri permanent quan la finalitat principal sigui practicar o reforçar. Shannon indica on hi ha més incertesa diagnòstica; el reforç ha d'atendre també, i preferentment, allò que l'alumne domina menys.

En tests adaptatius de diagnòstic global amb criteri d'aturada, no és obligatori aplicar la fase de reforç. En aquest cas n'hi ha prou amb maximitzar la informació esperada, diversificant categories en empats o quan diverses candidates tenen una utilitat equivalent.

Tingues en compte que maximitzar la informació esperada tendeix a proposar preguntes amb una probabilitat d'encert propera al `50 %`. Amb alumnat jove o amb dificultats, pots obrir amb una pregunta assequible o intercalar-ne alguna d'èxit probable: perds una mica d'eficiència informativa a canvi de sostenir la motivació. És una decisió pedagògica opcional.

Si diverses candidates són pràcticament equivalents:

- trenca empats amb aleatorització;
- afavoreix categories o conceptes menys repetits.

No facis servir selecció determinista simple en empats.

## Criteri d'aturada

Atura la sessió quan es compleixin criteris raonables de tancament, per exemple:

- mínim de preguntes ja complert;
- entropia per sota de `H_stop`;
- hipòtesi més probable per sobre de `p_min`;
- no queden preguntes útils;
- la millor pregunta restant aporta molt poca informació;
- s'assoleix el màxim pràctic.

Si fas servir `p_min`, calcula el llindar orientatiu:

`H_stop = -p_min * log2(p_min) - (1 - p_min) * log2((1 - p_min) / (n - 1))`

Després de complir el mínim de preguntes, un tancament diagnòstic ferm ha de comprovar preferentment totes dues condicions: `H <= H_stop` i `max(p_i) >= p_min`. Si es tanca per màxim, banc exhaurit o utilitat marginal baixa sense complir-les, presenta el resultat com a provisional.

Regles mínimes:

- no tanquis massa aviat;
- no allarguis artificialment la sessió quan la utilitat marginal ja és baixa;
- si la incertesa continua sent alta, el resultat s'ha d'indicar com a provisional.

## Reforç continu sense aturada

- En recursos de pràctica o reforç obert pot no haver-hi criteri d'aturada: la sessió continua mentre l'alumne practiqui.
- En aquest mode, l'estat estimat no és un diagnòstic tancat, sinó una estimació viva que s'actualitza amb cada resposta.
- No apliquis `H_stop` ni `p_min` per tancar la sessió; utilitza'ls, si de cas, només per informar del grau de confiança assolit.
- Mantén la selecció en dues fases durant tota la sessió: diagnòstic mínim per categoria i, després, reforç del que menys es domina.
- L'estat de l'alumne pot canviar mentre practica: aplica oblit exponencial perquè l'estimació segueixi el seu estat actual. Abans de cada actualització, atenua el posterior elevant-lo a `lambda` i renormalitzant: `p_i <- p_i^lambda / Σ_j p_j^lambda`.
- Fes servir `lambda ≈ 0.9`–`0.98` en pràctica o reforç continu (regla pràctica: `lambda = 1 - 1/W`, amb `W` el nombre de respostes recents que han de dominar l'estimació). En recursos diagnòstics de sessió curta fes servir `lambda = 1` (sense oblit): allà només afegiria soroll.
- Amb oblit actiu, presenta la confiança com a referida a l'estat recent de l'alumne.
- Si necessites modelar explícitament l'aprenentatge (per exemple, més probabilitat de pujar de nivell just després d'una explicació), fes servir un model de transició (Bayesian Knowledge Tracing); consulta `matematicas.html §3.5`.

## Itineraris per etapes

Si el recurs té fases, tècniques o etapes successives:

- distingeix entre estimació global i estimació local per etapa;
- no promociones una etapa utilitzant només la creença global acumulada;
- decideix la superació de l'etapa amb evidència generada dins d'aquesta etapa.

Per donar una etapa per superada, convé exigir almenys:

- confiança local suficient;
- entropia local suficientment baixa;
- i un mínim explícit de rendiment observat en aquesta etapa.

Exemple raonable:

- `p_min = 0.80`
- mínim del `60 %` d'encerts a l'etapa

Si l'alumne repeteix una etapa:

- reinicia l'estimació local d'aquella etapa, llevat de justificació pedagògica explícita en contra.

Acabar una etapa no implica necessàriament haver-la superada.

## Recuperació i reforç

- Si l'alumne mostra dificultats, el sistema ha de poder:
  - oferir pistes;
  - mostrar explicació;
  - proposar reforç;
  - canviar el tipus d'activitat;
  - reduir temporalment la dificultat;
  - permetre reintent o repàs.
- Si mostra domini, pot:
  - avançar;
  - ampliar;
  - augmentar la complexitat;
  - reduir l'ajuda.

## Resultat final

La sortida final ha d'incloure, segons el tipus de recurs:

- diagnòstic o nivell estimat;
- grau de confiança;
- dificultats detectades;
- fortaleses observades;
- recomanació pedagògica;
- pas següent.

Si és un itinerari o activitat d'aprenentatge, afegeix-hi a més:

- el recorregut seguit;
- etapes superades o no;
- ajudes utilitzades;
- àrees a reforçar.

No declaris un domini alt basant-te en molt pocs intents. Si el resultat fa afirmacions per categoria o dimensió, exigeix una mostra mínima en aquestes categories o dimensions abans de presentar-les com a fermes. Si l'estimació és global, la mostra mínima pot referir-se al conjunt de la sessió; limita o marca com a provisional l'estimació quan l'evidència sigui escassa.

No retornis només una nota o etiqueta.

Si dues hipòtesis acaben amb probabilitats properes, mostra la distribució posterior completa (per exemple, un diagrama de barres), no només l'etiqueta guanyadora.

## Restriccions d'implementació

- La interfície ha de ser comprensible per a l'alumnat i el professorat.
- Ha de mostrar amb claredat el progrés i la retroalimentació.
- Si fas servir fórmules visibles, acompanya-les d'una interpretació llegible.
- Evita dependre d'un backend si no s'ha demanat.
- Evita preguntes obertes llargues sense correcció automàtica fiable.

## Valors per defecte recomanats

Si el docent no especifica paràmetres:

- `n = 3` hipòtesis o nivells;
- `a = 1.5`;
- `p_min = 0.80`;
- mínim de preguntes: entre `4` i `6`;
- mínim de diagnòstic per categoria en pràctica adaptativa: `2` intents;
- màxim pràctic: entre `10` i `20`, segons el tipus de recurs;
- pes del guany d'informació en la fase de reforç: `α = 0.65` (rang raonable `0.6`–`0.7`);
- oblit exponencial: `lambda = 0.95` en pràctica o reforç continu (memòria efectiva ≈ `20` respostes); `lambda = 1` en diagnòstic de sessió curta;
- crèdit parcial: pesos de subcriteris definits pel disseny, sumant `1`;
- mostra mínima per categoria o dimensió abans de mostrar domini alt: `2`–`4` intents.

## Què no ha de fer la IA

- No confondre una explicació teòrica amb una regla obligatòria d'implementació.
- No usar la creença global per promocionar etapes locals en itineraris.
- No tancar el recurs sense justificar la certesa assolida.
- No presentar com a ferm un resultat amb incertesa alta.
- No limitar la sortida a una puntuació nua.
- No tractar com a binària una resposta que admet graus: utilitza crèdit parcial.
- No declarar domini alt amb una mostra insuficient.

## Validació i fiabilitat (opcional)

Aquestes comprovacions no formen part del flux obligatori: són una capa opcional que augmenta l'honestedat del diagnòstic sense requerir dades empíriques. Aplica-les quan la finalitat sigui diagnòstica i el resultat s'hagi d'usar per decidir.

- **Ajust del patró individual (person-fit).** En tancar una sessió, avalua si el patró de respostes és coherent amb el nivell estimat. Calcula l'índex estandarditzat `l_z` a partir de les probabilitats d'encert que el model assigna a les preguntes respostes sota la hipòtesi més probable. Si `l_z` és molt negatiu (orientativament `< -2`), marca el diagnòstic com a poc fiable encara que el posterior sigui alt: sol indicar respostes incoherents amb la dificultat, descuits o atzar. És un senyal de cautela, no una prova formal; amb poques preguntes és només orientatiu.
- **Separabilitat del disseny (Monte Carlo).** Com a propietat del test (no de l'alumne), estima amb quina fiabilitat el banc distingeix els nivells: genera respondents sintètics situats en el `theta` de cada hipòtesi, fes-los el test reutilitzant la mateixa selecció adaptativa i el mateix criteri d'aturada, i construeix la matriu de confusió (nivell real enfront de diagnosticat). Presenta-la com a fiabilitat sota el model, mai com a validesa empírica: els respondents surten del propi model, així que mesura si el disseny discrimina els nivells, no si els paràmetres reflecteixen la realitat. **Aquesta validació és una eina del creador del recurs, no de l'alumnat: no s'ha de mostrar en la interfície de l'alumne.** Implementa-la en un fitxer o utilitat a part que l'autor pugui executar en dissenyar o revisar el test, no en el material que rep l'alumne.

Consulta `matematicas.html §11.7–§11.8` per a les fórmules i l'enquadrament complet.

## Nota sobre el model

La funció IRT 3PL descrita aquí s'utilitza com a generador inicial de versemblances quan no hi ha dades empíriques. No equival a una calibració psicomètrica validada. Si s'acumulen suficients respostes reals, convé recalibrar dificultats, discriminació i pseudoatzar.
