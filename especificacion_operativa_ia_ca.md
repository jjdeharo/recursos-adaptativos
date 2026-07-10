# Especificació operativa per a IA

**Versió 2.3**

## Propòsit

Aquest document serveix perquè una IA implementi recursos educatius adaptatius bayesians de manera fiable.  
És una especificació operativa breu.  
Si cal fonament teòric, exemples o justificació matemàtica, consulta el protocol (https://jjdeharo.github.io/recursos-adaptativos/documentacion_ca.html) i els fonaments matemàtics (https://jjdeharo.github.io/recursos-adaptativos/matematicas_ca.html). Si només tens adjunt aquest fitxer i no pots navegar, actua amb el que aquí s'especifica; no inventis contingut d'aquests documents.  
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

## Perfil i mode: quines seccions apliquen

Amb la informació anterior, classifica el recurs **abans d'implementar**. Aquesta classificació decideix quines seccions d'aquest document has d'aplicar; no barregis regles de perfils o modes diferents.

**Perfil** (què s'estima; els criteris de decisió són a «Elecció del model»):

- `A` · **Ordinal**: nivells ordenats de domini, globals o per categoria.
- `B` · **Nominal excloent**: hipòtesis alternatives; només una pot ser certa en cada alumne.
- `C` · **Multifactorial**: errors o factors que poden coexistir.
- `A+C` · **Combinat**: nivell ordinal i factors d'error alhora.

**Mode** (com acaba la sessió):

- `Diagnòstic`: la sessió convergeix i es tanca amb un resultat.
- `Pràctica`: sense final previst; estimació viva amb oblit.
- `Itinerari`: etapes successives amb promoció entre elles.

| Secció | Aplica només si |
|---|---|
| Estat de l'alumne (escala `theta`) | perfil `A` o `A+C` (en `B` i `C` no existeix `theta`) |
| Versemblances · part IRT 3PL | perfil `A` o la part de nivell de `A+C` |
| Versemblances · part nominal i per perfils | perfil `B`, `C` o `A+C` |
| Respostes amb crèdit parcial | hi ha ítems amb graus o per passos |
| Sòl d'atzar en ítems compostos | hi ha ítems de diversos components |
| Diagnòstic multidimensional | perfil `C`, `A+C`, o diverses distribucions en paral·lel |
| Selecció adaptativa · blocs pedagògics | perfil `A+C` o diverses distribucions en paral·lel |
| Selecció adaptativa · dues fases i utilitat `α` | mode `Pràctica` |
| Criteri d'aturada | mode `Diagnòstic` o `Itinerari` |
| Reforç continu sense aturada | mode `Pràctica` |
| Itineraris per etapes | mode `Itinerari` |
| Validació i fiabilitat | finalitat diagnòstica o promoció d'etapes |

Les seccions no llistades apliquen sempre. Exemples de barreja indeguda que aquesta taula evita: activar oblit (`lambda < 1`) en un diagnòstic de sessió curta; calcular una `theta` esperada en perfils `B` o `C`; exigir ítems de sortida fora d'un itinerari; muntar blocs pedagògics amb una sola distribució.

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
- Prior d'hipòtesis de **nivell**: si no hi ha informació prèvia fiable, utilitza una distribució uniforme. Prior de **factors d'error** binaris: mai uniforme; fes servir el prior informatiu moderat de «Versemblances» (`P(error) ≈ 0.2`–`0.3`). *Per què:* un uniforme sobre els `2^k` perfils (o sobre `{present, absent}` de cada factor) equival a afirmar que cada error té un 50 % de probabilitat a priori d'estar present; no és neutre i esbiaixa les primeres preguntes cap al fals positiu (errors que l'alumne no té mostrats com a «indeterminats» o «probables»).
- Si les hipòtesis són jeràrquiques, assigna-les valors `theta` ordenats i centrats.
- Si el docent no fixa valors, utilitza una escala simètrica centrada en 0.
- L'escala `theta` és fixa i depèn només del nombre d'hipòtesis, no del banc de preguntes. Amb `n` hipòtesis jeràrquiques, fes servir valors centrats en `0` amb intervals de `2`: `theta_i = 2 * (i - 1) - (n - 1)`, és a dir, `theta` recorre `{-(n-1), ..., +(n-1)}` i `theta_max = n - 1`. Exemples: `n = 2` → `{-1, +1}`; `n = 3` → `{-2, 0, +2}`; `n = 4` → `{-3, -1, +1, +3}`; `n = 5` → `{-4, -2, 0, +2, +4}`.
- Situa les dificultats dins de la meitat central d'aquesta escala: `b_q` ha de quedar a `[-theta_max / 2, +theta_max / 2]`. Com que `theta_max` depèn de `n`, la conversió de dificultats també depèn de `n`, no només del nombre de categories.
- Si el docent dona `k` categories qualitatives de dificultat, reparteix-les uniformement en aquest interval: `b_j = (theta_max / 2) * (2 * (j - 1) / (k - 1) - 1)` per a `j = 1..k` (si `k = 1`, fes servir `b = 0`). Exemples: `n = 3` i `k = 3` → `{-1, 0, +1}`; `n = 3` i `k = 5` → `{-1, -0.5, 0, +0.5, +1}`; `n = 4` i `k = 3` → `{-1.5, 0, +1.5}`; `n = 2` i `k = 5` → `{-0.5, -0.25, 0, +0.25, +0.5}`.
- No facis servir una taula de dificultats fixa i independent de `n`: amb poques hipòtesis i moltes categories produiria dificultats fora del rang de nivells (per exemple, `n = 2` amb `b = ±2`) i anul·laria la separació entre escales.
- Si el docent dona valors numèrics de `b_q` fora de l'interval, retalla'ls (clamp) a l'interval.
- No recalculis mai `theta` a partir dels extrems del banc. Una sola pregunta atípicament difícil o fàcil no ha de redefinir l'escala: si estires `theta` per acomodar-la, satures les versemblances de la resta del banc (totes les probabilitats queden enganxades a `c_q` o a `1`) i el posterior fa salts sobreconfiats amb una sola resposta.
- Si la majoria de les dificultats del banc quedés fora de l'interval, no estiris l'escala: revisa amb el docent la definició de nivells i dificultats, perquè el disseny és incoherent.
- Mantén entre recursos l'invariant de comparabilitat `a_ef * Δtheta = 2.5` (amb `a_ef = 1.25` i intervals de `2` entre hipòtesis adjacents); amb `n = 2` compensa a més amb més preguntes o defineix l'escala des d'una probabilitat objectiu `P*`. *Què garanteix i què no:* l'invariant iguala la **pendent màxima** de la ICC entre formats per a qualsevol `n`, però la comparabilitat de confiances i velocitats és estricta només entre recursos amb el mateix `n` — el marge entre el nivell extrem i l'ítem més difícil és `(n - 1) / 2`, i amb `n = 2` els ítems extrems confirmen feblement (P ≈ 0.65 en oberta, ≈ 0.77 en 4 opcions) —. Tampoc no iguala la força màxima de l'evidència d'una fallada: en el quocient de versemblances de fallada el factor `(1 - c)` es cancel·la i la raó tendeix a `e^(a * Δtheta)` amb la `a` **nominal** (≈ 12 en oberta, ≈ 28 en 4 opcions, ≈ 148 en vertader/fals amb `Δtheta = 2`); aquests salts els acota el sostre de domini de «Versemblances».

## Actualització bayesiana

Després de cada resposta:

1. calcula la versemblança d'aquesta resposta sota cada hipòtesi;
2. multiplica el prior per la versemblança;
3. normalitza;
4. utilitza el resultat com a nou estat.

Això s'ha de fer després de cada interacció rellevant.

## Elecció del model

- Si la finalitat principal és estimar un nivell ordenat de domini, utilitza hipòtesis ordinals i IRT 3PL.
- Si la finalitat principal és identificar una estratègia o error excloent, utilitza hipòtesis nominals amb versemblances explícites per pregunta.
- Si diversos errors, habilitats o necessitats poden coexistir, utilitza dimensions separades o perfils complets. La IA decideix l'arquitectura: utilitza dimensions separades quan cada factor s'interpreta i es corregeix per separat; utilitza perfils complets quan la resposta esperada canvia per combinacions de factors, un error n'emmascara un altre o la intervenció pedagògica depèn de la combinació. Si `2^k` perfils és inviable, agrupa factors relacionats o diagnostica per fases.
- Si el recurs ha d'estimar nivell i diagnosticar errors, combina una distribució ordinal global amb distribucions diagnòstiques paral·leles i organitza-les en **blocs pedagògics** (`nivell`, `errors`, `categories`, etc.). No preguntis al docent per pesos tècnics: infereix-los de la finalitat expressada.
- No forcis errors coexistents dins d'una escala ordinal única.

## Versemblances

- Si les hipòtesis representen nivells ordenats de domini, utilitza IRT 3PL.
- Fórmula recomanada:

`P(encert | H_i, q) = c_q + (1 - c_q) / (1 + exp(-a * (theta_i - b_q)))`

- Utilitza per defecte:
  - `a_ef = 1.25` (discriminació efectiva objectiu)
  - `a = a_ef / (1 - c_q)`
  - `c_q = 1 / m_q` si hi ha atzar
  - `c_q = 0` si no n'hi ha
- No fixis `a` directament: fixa la discriminació efectiva objectiu `a_ef = 1.25` i calcula `a` per pregunta amb `a = a_ef / (1 - c_q) = 1.25 / (1 - c_q)`.
  - Fes-ho **sempre**, encara que totes les preguntes tinguin el mateix nombre d'opcions. Aquesta regla iguala el pendent màxim de la ICC, no garanteix que tots els formats aportin la mateixa informació esperada. Un `a` fix amb `c_q` diferents produeix pendents reals diferents i no comparables (per exemple, `a=1.5` fix dona `a_ef=1.0` amb 3 opcions però `a_ef=1.125` amb 4). La selecció per guany d'informació continuarà afavorint els ítems que aportin més evidència.
  - `a_ef = 1.25` garanteix que `a` es mantingui dins del rang habitual en psicometria (0.5–2.5) per a qualsevol format de 2 o més opcions: el cas extrem, vertader/fals (`c_q=0.5`), dona exactament `a = 2.5`.
  - Valors que has d'obtenir: oberta (`c_q=0`) → `a=1.25`; 5 opcions (`c_q=0.20`) → `a=1.5625`; 4 opcions (`c_q=0.25`) → `a≈1.667`; 3 opcions (`c_q=1/3`) → `a=1.875`; vertader/fals (`c_q=0.5`) → `a=2.5`.
- Aplica un **sostre de domini** també en el cas ordinal: acota `P(encert | H_i, q) <= 0.95` (equivalentment, `P(fallada) >= 0.05`) per a tota hipòtesi, inclòs el nivell més alt en ítems fàcils. El 3PL pur deixa que `P(encert) → 1` en ítems fàcils amb més pseudoatzar, de manera que una sola fallada dispara el posterior gairebé de manera determinista (amb `c = 0.5` la raó de versemblança d'una fallada entre nivells adjacents arriba a ≈ 148, davant de ≈ 12 en oberta). El sostre modela el descuit (*slip*), que el 3PL no preveu, i alinea el cas ordinal amb el nominal, que ja imposa `0.9`–`0.95`, mai `1`. Fes servir `0.90` si vols ser més conservador.
- Si l'alumne falla:

`P(error | H_i, q) = 1 - P(encert | H_i, q)`

- Si les hipòtesis no són jeràrquiques, distingeix dos casos.
- Si són **alternatives excloents** (per exemple, estratègia A / estratègia B / domini correcte), no facis servir IRT logística. Genera per a cada pregunta un vector de `n` versemblances `P(encert | H_i, q)`, una per hipòtesi.
- Si els errors o necessitats **poden coexistir**, no els forcis dins d'una sola distribució nominal. Modela una dimensió per factor (`error llarg`, `error del zero`, etc.) o, de manera equivalent, una distribució sobre **perfils complets** (`2^k` combinacions possibles de `k` factors).
- **Prior dels factors d'error: no uniforme.** Assigna a cada factor una probabilitat a priori baixa d'estar present (per defecte `P(error) ≈ 0.2`–`0.3`), mai l'uniforme `0.5`. El recurs aplica aquest valor de manera automàtica; el docent pot ajustar la prevalença del seu grup si ho vol, però no hi està obligat. *Per què:* els errors conceptuals solen ser minoritaris; l'uniforme no és neutre (afirma que cada alumne té cada error amb probabilitat `0.5`) i produeix falsos positius en les primeres preguntes.
- **Ajusta aquest prior amb el que el docent ja t'ha dit, sense preguntar-li xifres.** Si en descriure el recurs el docent qualifica la freqüència d'un error, tradueix-ho tu a un prior; no li demanis una prevalença numèrica ni mostris el valor triat a la interfície de l'alumne.

  | Com ho descriu el docent | `P(error)` inicial |
  |---|---|
  | «molts», «la majoria», «els passa sempre», «molt freqüent» | `0.40` |
  | «alguns», «de vegades», «bastants», o no diu res | `0.20`–`0.30` (defecte) |
  | «pocs», «casos aïllats», «algun de solt», «rar» | `0.10`–`0.15` |

  Regles que acoten l'heurística:
  - **Mai `≥ 0.5`, mai `< 0.10`.** Per damunt de `0.5` el prior afirma que l'error és més probable que la seva absència, que és el biaix que aquest apartat corregeix. Per sota de `0.10` caldrien `3` errades discriminants per confirmar un error real, i en un banc petit pot no confirmar-se mai.
  - **La mostra mínima no es relaxa mai, i amb `0.40` és l'única cosa que protegeix.** Des de `0.40`, una sola errada discriminant (`LR ≈ 4`) porta la marginal a `0.727` i creua el llindar de «present»: sense exigir la mostra mínima d'evidència, la paraula del docent bastaria per diagnosticar l'alumne. Mantén els `2` intents per factor.
  - **El prior és també l'àncora de l'oblit.** Un factor amb `P(error) = 0.40` que es quedi sense evidència recent no torna a `0.25`, torna a `0.40`, i reapareixerà com a «indeterminat». És coherent: el docent va dir que aquest error és freqüent en el seu grup.

  *Per què:* l'ajust és deliberadament suau. Mou la decisió **un ítem**, no la predetermina: confirmar un error exigeix `2` errades discriminants des del prior per defecte, `1` des de `0.40` i `3` des de `0.10`. El prior tot sol no pot declarar un error **present** en cap punt del rang permès. El cas que corregeix és el fals negatiu simètric del que va motivar el prior informatiu: quan el docent ja sap que un error és freqüent en el seu grup, partir de `0.25` gasta preguntes a redescobrir-lo.
- En aquest cas multifactorial, cada pregunta ha de definir com respon cada perfil. El mínim és una probabilitat d'encert per perfil; encara millor és una distribució per opció `P(R = r | perfil, q)` per aprofitar quin distractor ha triat l'alumne, i no només encert/error.
- Si parteixes de factors simples, assigna cada valor responent: «si l'alumne tingués aquest error, amb quina probabilitat encertaria aquesta pregunta?». Baix si la pregunta ataca el concepte que l'error distorsiona; alt si l'error no interfereix.
- Acota cada valor: no per sota del terra d'atzar `1/m` (m opcions), llevat de l'excepció del punt següent; la hipòtesi o perfil de domini al voltant de `0.9`–`0.95`, mai `1`.
- Si un distractor concret és la resposta que produeix un error, la probabilitat d'**aquella opció** ha d'augmentar sota aquell perfil, i la probabilitat d'encert pot quedar a prop del terra o per sota: l'alumne és atret activament cap a aquella resposta equivocada.
- No afinis el decimal: fes servir trams (`≈0.9` no afecta / `≈0.5` afectació parcial / `≈0.15–0.25` distractor que captura l'error / `≈1/m` terra si falla per atzar). El que importa és que els factors o perfils correctes se separin clarament en les preguntes que discriminen.
- No facis servir taules fixes globals si cada pregunta pot generar les seves pròpies versemblances.
- La qualitat del diagnòstic no depèn només de l'algorisme: depèn també de com estiguin definides les hipòtesis, categories, dificultats, conceptes i errors. Si aquestes classificacions estan mal conformades, si els errors rellevants no estan ben identificats o si el banc cobreix malament els casos importants, les versemblances generades poden representar malament la realitat i esbiaixar l'adaptació.
- En el diagnòstic final no calculis una `theta` esperada (no té sentit sense ordre). Si el model és nominal excloent, pots reportar la hipòtesi MAP i la seva probabilitat. Si el model és multifactorial, reporta per a cada factor si queda **present**, **absent** o **indeterminat**, amb la seva probabilitat marginal o confiança associada.
- Amb prior informatiu (`P(error) ≈ 0.2`–`0.3`), l'absència arrenca ja a `0.7`–`0.8` sense cap evidència: no declaris un factor **absent** només perquè la seva marginal superi el llindar de confiança. Exigeix a més la seva mostra mínima d'evidència i distingeix en el report «absent confirmat» (amb evidència) de «sense evidència suficient» (el valor per defecte del prior).

## Respostes amb crèdit parcial

- Si una resposta no és només encert o error, sinó que admet graus (diversos passos, components ponderats, encerts parcials), resumeix-la en una puntuació `s` entre `0` i `1`.
- Si només tens una puntuació parcial agregada `s`, construeix una versemblança geomètrica:

`L(H_i) = P(encert | H_i, q)^s * P(error | H_i, q)^(1 - s)`

- Utilitza aquesta `L(H_i)` en l'actualització bayesiana en lloc de triar entre `P(encert)` i `P(error)`. La normalització i la resta del procés no canvien.
- Casos límit: `s = 1` equival a encert ple; `s = 0` a error ple. Un `s = 0.5` no és necessàriament neutre: afavoreix les hipòtesis per a les quals l'ítem prediu una puntuació intermèdia.
- Si l'ítem té `J` components aproximadament independents i `s` és la fracció ponderada de components correctes, pots conservar la força de l'evidència fent servir `L(H_i) = P(encert | H_i, q)^(sJ) * P(error | H_i, q)^((1 - s)J)`.
- Compte: l'exponent `J` tracta la resposta com a `J` assajos independents amb la **mateixa** probabilitat `p_i` de l'ítem complet, així que només és raonable si els components són de **dificultat semblant**. Si difereixen clarament en dificultat (l'habitual en tasques per passos), aquesta forma **sobrecompta** l'evidència davant del model per components; en cas de dubte, fes servir un `J` menor que el nombre real de components (evidència més conservadora).
- Si tens evidència separada per component, és preferible multiplicar les versemblances de cada component en comptes de reduir-ho tot a una sola `s`.
- Defineix com es calcula `s` de manera explícita i autocorregible: suma ponderada de subcriteris, fracció de passos correctes, proximitat a la solució numèrica, etc. Els pesos han de sumar `1`.
- No tractis com a binària una resposta que admet graus: perds informació diagnòstica.
- **Calcula el guany d'informació amb la mateixa versemblança amb què actualitzaràs.** Aquesta és la regla; la resta se'n dedueix.
- Regla operativa, per ordre de preferència:
  1. **Si coneixes els components de la resposta, no la resumeixis en `s`.** Actualitza multiplicant les versemblances de cada component (com ja es diu més amunt) i calcula el guany fent la mitjana sobre les combinacions de resultats dels components. És el cas millor: selecció i actualització comparteixen model.
  2. **Si només disposes de la `s` agregada**, actualitza amb la versemblança geomètrica i calcula el guany fent la mitjana sobre els **dos extrems** (`s = 0` i `s = 1`) amb `P(encert | H_i, q)`. Això és exactament el guany del model binari que la geomètrica estén, i és l'elecció coherent.
  3. **No facis la mitjana del guany sobre la distribució real de `s` mentre actualitzes amb la geomètrica.** Sembla més fi i és pitjor: vegeu-ne el perquè.
- *Per què:* la versemblança geomètrica envia tota puntuació intermèdia cap a les hipòtesis intermèdies (per disseny: es maximitza en `p_i = s`). Un ítem de dificultat mitjana produeix puntuacions intermèdies amb **qualsevol** alumne, de manera que concentra el posterior en la hipòtesi del mig sigui quin sigui l'estat real. Si calcules el guany sobre la distribució vertadera de `s` però actualitzes amb la geomètrica, el selector detecta aquesta concentració, la interpreta com a informació i prefereix justament aquests ítems: la confiança puja i l'encert baixa. Mesurat sobre un recurs real de crèdit parcial (7 subcriteris ponderats, 3 hipòtesis, banc de 39 ítems per tipus, 4 preguntes per sessió, 3 llavors): la regla actual encerta el **98.5 %**; fer la mitjana sobre la distribució exacta de `s` mantenint la geomètrica baixa al **95.9 %**, i fer-ho sobre quatre escenaris discrets (`0`, parcial baix, parcial alt, `1`) baixa al **93.7 %**. L'ús d'ítems de dificultat mitjana passa del `9 %` al `27 %`. En canvi, passar al model per components del punt 1 **puja** al `99.3 %` i corregeix de passada una infraconfiança crònica de la geomètrica (assigna `0.75` de probabilitat a la hipòtesi vertadera quan encerta el `98 %` de les vegades; per components, `0.99`). Comprovat també amb components fortament dependents entre si: el model per components continua guanyant (`97.3 %` enfront de `94.4 %`) i no arriba a sobrecomptar evidència.

## Sòl d'atzar en ítems compostos

- Si un ítem es corregeix per diversos components amb diferent nombre d'opcions i es puntua amb crèdit parcial, el sòl agregat no és la probabilitat d'encertar l'ítem complet, sinó la puntuació parcial esperada per atzar.
- Calcula aquest sòl de puntuació esperada com a mitjana ponderada dels atzars de cada component:

`c_q = Σ_j w_j * c_j`  amb  `c_j = 1 / m_j`  i  `Σ_j w_j = 1`

- Utilitza aquest `c_q` agregat en la funció logística només quan la ICC representi la puntuació esperada de l'ítem compost.
- Si l'ítem compost es corregeix com a tot-o-res, no facis servir la mitjana ponderada: la probabilitat d'encert ple per atzar és el producte `Π_j c_j` si els components s'encerten independentment.
- Els pesos `w_j` han de coincidir amb els que facis servir per calcular la puntuació `s` del crèdit parcial.

## Diagnòstic multidimensional

- Si necessites saber no només el nivell global sinó quines habilitats o passos fallen, mantén diverses distribucions bayesianes en paral·lel: una per categoria o nivell i una per cada dimensió diagnòstica (habilitat, pas, tipus d'error).
- Actualitza cada distribució amb l'evidència que li correspon: el resultat global pot alimentar la creença de nivell; cada subcriteri ha d'alimentar només la seva dimensió. No facis servir el mateix encert/error global per actualitzar diverses dimensions independents si la pregunta exigeix diverses habilitats alhora, perquè dupliques evidència i atribueixes malament la causa de l'error.
- Cada dimensió pot tenir el seu propi sòl d'atzar `c` segons el seu nombre d'opcions, de manera que els percentatges no són directament comparables entre dimensions. En dimensions ordinals, la referència comuna és el valor latent `theta`; en factors nominals d'error, no calculis `theta`: reporta probabilitats marginals, estat (`present`, `absent`, `indeterminat`) i mostra mínima d'evidència.
- No posis en una sola distribució dimensions que poden coexistir: utilitza distribucions separades.
- Si diverses dimensions diagnòstiques interactuen de manera forta, substitueix les distribucions independents per una sola distribució sobre perfils complets. Utilitza perfils complets quan la resposta esperada canvia per combinacions de factors, quan un error n'emmascara un altre o quan la intervenció depèn de la combinació; utilitza dimensions separades quan els factors s'evidencien i s'interpreten de manera raonablement independent. No preguntis al docent per aquesta decisió en termes tècnics: dedueix-la de la descripció pedagògica dels errors.
- El nivell per categoria guia què practicar; el diagnòstic per dimensió guia què explicar o reforçar.
- Si combines una distribució ordinal de nivell amb factors d'error (model combinat), tots dos són estimacions marginals paral·leles alimentades per la mateixa evidència, no troballes independents. Agrupa'ls per blocs pedagògics per a la selecció (`nivell`, `errors`, `categories`), comprova la coherència del resultat final i, si el nivell estimat és alt i algun error queda «present», presenta-ho com un matís del nivell («domina X, tot i que persisteix l'error Y»), no com a conclusions contradictòries juxtaposades.

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

### Bancs generats: variants parametritzades

Si el banc genera els ítems a partir de plantilles (mateix concepte, dificultat i format amb dades diferents), aplica aquestes regles. Sense elles, un banc generat no és més fiable que un d'escrit a mà: és menys fiable, perquè ningú no ha llegit la majoria dels seus ítems.

- **Deriva la resposta correcta, no la transcriguis.** La plantilla ha de calcular la solució aplicant la mateixa regla que l'ítem ensenya sobre els paràmetres sortejats (per a `b^m · b^n`, calcular `b^(m+n)`). Així només hi ha una regla per revisar per plantilla, en comptes d'una resposta per variant que pugui estar mal copiada, i la correcció de totes les variants se segueix de la correcció d'aquella regla.
- **Mantén fixos per plantilla els metadades que alimenten el model**: dificultat `b_q`, nombre d'opcions `m_q`, categoria, factor d'error i posició de l'opció distractora. No els sortegis. Així totes les variants d'una plantilla són equivalents sota el model, i la validació Monte Carlo del banc (que s'executa sobre les plantilles) continua sent aplicable als ítems que veu l'alumne.
- **Verifica cada variant de manera automàtica**, abans de lliurar i sobre moltes variants per plantilla. Com a mínim: (1) la resposta marcada com a correcta reprodueix el valor de l'expressió de l'enunciat, avaluada numèricament en valors de prova **lluny de `0` i d'`1`**, on moltes igualtats falses es compleixen per casualitat; (2) les opcions són diferents entre si, i cap opció incorrecta no té el mateix valor que la correcta, perquè això deixa la pregunta sense resposta única; (3) enunciat, pista, explicació i opcions es representen sense error. Quan una variant incompleixi alguna condició, torna a sortejar els paràmetres en lloc de lliurar-la.
- **La verificació numèrica prova la matemàtica, no la redacció.** Un enunciat pot ser cert i tot i així haver perdut la pregunta: si la plantilla simplifica l'expressió abans de mostrar-la, pot acabar ensenyant la resposta (`a^0` presentat ja com a `1`). Cap comprovació numèrica no detecta això. Llegeix una mostra de variants de cada plantilla abans de donar el banc per bo.
- Comprova que cada plantilla admeti **prou variants** per a la durada prevista d'una sessió. Una plantilla amb dues o tres variants torna a produir repeticions i, amb elles, la contaminació del feedback que les variants venien a evitar.

## Selecció adaptativa

Per a cada candidata disponible:

1. calcula la probabilitat marginal d'encert;
2. simula el posterior si hi ha encert;
3. simula el posterior si hi ha error;
4. calcula l'entropia esperada posterior;
5. calcula el guany esperat d'informació.

Selecciona la candidata amb un guany esperat d'informació més gran quan la finalitat principal sigui diagnòstica i l'objectiu sigui estimar un nivell global.

Si l'estat són diverses distribucions paral·leles (factors d'error o dimensions diagnòstiques), calcula el guany esperat de cada distribució i agrupa'l per **blocs pedagògics**. Dins de cada bloc pots sumar els guanys de les seves distribucions (`IG_bloc = Σ IG_d`), perquè l'entropia conjunta factoritzada és la suma d'entropies. Per combinar blocs, normalitza cadascun entre les candidates disponibles (`IG_bloc_norm(q) = IG_bloc(q) / max IG_bloc`) i utilitza pesos automàtics segons la finalitat. Valors per defecte: si la finalitat se centra en un bloc, `w = 0.7` per a aquell bloc i `0.3` repartit entre els altres; si és mixta, pesos iguals per bloc (`w_b = 1 / nombre de blocs`), no pel nombre brut de dimensions. Un bloc està **decidit** quan assoleix el seu criteri de confiança: el de nivell, quan `max(p_i) >= p_min` amb el mínim de preguntes complert; el d'errors, quan tots els seus factors estan decidits (marginal fora de la zona indeterminada, amb la seva mostra mínima). Quan un bloc queda decidit, reparteix el seu pes proporcionalment entre els blocs encara incerts. No demanis aquests pesos al docent. Fes la mitjana sobre els resultats que l'ítem modeli (per opció, si hi ha distractors diagnòstics).

Tres cauteles amb aquesta utilitat combinada:

- no està en bits: la normalització reescala el millor candidat de cada bloc a `1` encara que els seus guanys siguin minúsculs, així que el criteri d'aturada «la millor pregunta aporta molt poca informació» s'ha d'avaluar sobre la **IG crua total** (en bits), mai sobre la utilitat normalitzada;
- pel mateix motiu, un bloc gairebé exhaurit pot quedar sobrerepresentat just abans de creuar el seu criteri de «decidit»; si això resulta visible, pondera els guanys crus del bloc (fent servir la mitjana per dimensió en lloc de la suma) o normalitza per l'entropia restant del bloc en comptes del màxim;
- la suma entre blocs és exacta per a la creença factoritzada que manté el sistema, no per al posterior conjunt veritable: quan diverses distribucions s'actualitzen amb evidència compartida, les seves marginals estan correlacionades i la suma ignora aquesta redundància. Tracta-la com un indicador d'esgotament del banc, no com a informació conjunta, i no composis confiances marginals en una probabilitat conjunta. Consulta `matematicas.html §10.6` (https://jjdeharo.github.io/recursos-adaptativos/matematicas_ca.html#s10-6).

En recursos de pràctica adaptativa o reforç amb diverses categories, tipus de problema o conceptes, utilitza una selecció en dues fases:

1. **Fase diagnòstica inicial.** Fins que cada categoria rellevant tingui una mostra mínima d'intents, prioritza les categories amb menys evidència. Dins d'elles, utilitza el guany esperat d'informació per triar la pregunta més diagnòstica. Un valor per defecte raonable és exigir almenys `2` intents per categoria abans de sortir d'aquesta fase. Tingues present que `2` és el mínim defensable, no un valor còmode: amb oblit actiu la marginal per categoria és volàtil, així que apuja la mostra mínima si el banc ho permet. Si hi ha moltes categories, aquesta fase pot consumir la sessió (amb `10` categories i `2` intents ja són `20` preguntes): agrupa categories afins en blocs o redueix la mostra mínima perquè el diagnòstic inicial no superi el màxim pràctic.
2. **Fase de reforç.** Quan totes les categories rellevants tinguin mostra mínima, prioritza la categoria amb menys domini estimat. Dins d'aquesta categoria, no triïs automàticament la pregunta més difícil: selecciona una pregunta informativa i propera al nivell estimat de l'alumne. Una regla raonable és `utilitat = α * IG_norm + (1 - α) * ajust`, amb les dues escales definides a `[0, 1]`: `IG_norm = IG(q) / max IG` entre les candidates del moment, i `ajust = max(0, 1 - |b_q - E[theta]| / 2)`, que val `1` quan la dificultat coincideix amb el nivell estimat (`E[theta] = Σ p_i * theta_i`) i `0` quan s'allunya un interval complet de nivell. Sense definir les dues escales en el mateix rang, el pes `α` no significa res.

Perquè aquest `ajust` signifiqui alguna cosa, **cada categoria ha de cobrir el rang de dificultats per si sola**, no només el banc en conjunt. Si una categoria no té cap ítem a prop de l'extrem superior del rang, `ajust` val `0` en totes les seves candidates per a un alumne situat en el nivell més alt, la utilitat es redueix al guany d'informació i el terme d'adequació deixa d'existir justament per als alumnes que havia de protegir; el mateix passa a l'extrem inferior amb les categories sense ítems fàcils. Tingues present que, amb les dificultats confinades a la meitat central de l'escala, el millor `ajust` assolible en un nivell extrem ja és només `1 - theta_max / 4` (amb `n = 4`, `0.25`): això és l'esperable; un `0` no ho és. Abans de lliurar, tabula la cobertura de categories per dificultat i comprova que cap casella extrema no queda buida.

No facis servir l'entropia de Shannon com a únic criteri permanent quan la finalitat principal sigui practicar o reforçar. Shannon indica on hi ha més incertesa diagnòstica; el reforç ha d'atendre també, i preferentment, allò que l'alumne domina menys.

En tests adaptatius de diagnòstic global amb criteri d'aturada, no és obligatori aplicar la fase de reforç. En aquest cas n'hi ha prou amb maximitzar la informació esperada, diversificant categories en empats o quan diverses candidates tenen una utilitat equivalent.

Tingues en compte que maximitzar la informació esperada tendeix a proposar preguntes amb una probabilitat d'encert propera al `50 %`. Amb alumnat jove o amb dificultats, pots obrir amb una pregunta assequible o intercalar-ne alguna d'èxit probable: perds una mica d'eficiència informativa a canvi de sostenir la motivació. És una decisió pedagògica opcional.

Si diverses candidates són pràcticament equivalents:

- trenca empats amb aleatorització;
- afavoreix categories o conceptes menys repetits.

No facis servir selecció determinista simple en empats.

Evita repetir la mateixa pregunta de manera mecànica:

- no repeteixis exactament el mateix ítem de manera consecutiva, llevat que el disseny demani de manera explícita un reintent immediat;
- aplica una penalització per freqüència o una finestra d'exclusió recent perquè un ítem ja utilitzat perdi prioritat durant diverses seleccions;
- si diverses candidates tenen una utilitat semblant, prioritza la menys recent i la menys repetida;
- només permet reutilitzar un ítem quan el banc sigui petit, no hi hagi alternatives comparables o l'objectiu pedagògic sigui revisar deliberadament aquell mateix cas.

No confonguis aquest problema amb un simple criteri de desempat:

- l'aleatorització només ajuda si existeixen diverses candidates raonables;
- per evitar repeticions cal redundància local al banc: diverses preguntes alternatives per categoria, nivell de dificultat, tipus d'ítem o error rellevant;
- si en una zona diagnòstica només hi ha un ítem útil, la IA ha de reconèixer que el banc és insuficient per a una adaptació variada en aquella zona;
- en aquest cas no simulis varietat amb una falsa aleatorització: reutilitza l'ítem només quan sigui necessari, canvia temporalment d'objectiu pedagògic o deixa el resultat com a limitat per la mida del banc.

Evidència d'un ítem reutilitzat:

- si l'ítem es reutilitza després que l'alumne hagi vist la seva correcció o explicació (inclòs el reintent immediat), l'encert posterior no és evidència plena de domini: pot reflectir només memòria del feedback;
- **en mode `Pràctica`, tracta'l sempre com les pistes: crèdit parcial, mai exclusió de l'actualització.** L'encert rep una `s` reduïda (tant menor com més explícita fou l'explicació mostrada) i la fallada compta completa (`s = 0`): fallar un ítem la correcció del qual ja es va veure és evidència clara de no domini. *Per què:* la pràctica és oberta i el banc és finit, així que tard o d'hora tot ítem disponible es repetirà; si els repetits no actualitzen, quan s'esgoten els ítems nous de la categoria menys dominada l'estimació es congela, aquella categoria continua sent la prioritària i el selector entra en un bucle de repassos sense efecte, mentre l'oblit degrada l'estat sense que cap evidència ho compensi. Excloure'ls violaria a més la regla de disseny que cada resposta modifica l'estat estimat;
- només en mode `Diagnòstic` (on els ítems no s'han de repetir llevat d'un reintent immediat deliberat) pots excloure el reintent de l'actualització i fer-lo servir només com a pràctica: la sessió és curta i el seu tancament no depèn d'aquesta evidència;
- la millor redundància local no és repetir el mateix ítem, sinó disposar de **variants parametritzades** del mateix tipus (mateix concepte, dificultat i format amb dades diferents): cada variant compta com a ítem nou i no arrossega la contaminació del feedback.

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

Després de complir el mínim de preguntes, aplica **un** d'aquests dos tancaments de confiança:

- **Poques hipòtesis** (`n <= 4`, l'habitual): tanca quan `max(p_i) >= p_min` (per defecte `0.80`).
- **Moltes hipòtesis**, on exigir `0.80` és poc pràctic: tanca quan `max(p_i)` assoleixi un valor moderat (per exemple `>= 0.50`) **i** la guanyadora se separi de la segona: `P(guanyadora) - P(segona) >= Δ_min`, amb `Δ_min ≈ 0.3`–`0.4`.

No combinis comprovacions redundants pensant que afegeixen exigència. *Per què:* quan `H_stop` es deriva del mateix `p_min` (fórmula anterior), `max(p_i) >= p_min` ja implica `H <= H_stop`; i amb `p_min = 0.80` la separació ja és `>= 0.60`, de manera que afegir-hi un `Δ_min` de `0.3`–`0.4` és inert (només afegiria exigència si `Δ_min > 2 * p_min - 1`). La separació és una **alternativa** a un `p_min` alt, no un afegit. Amb `n = 2` totes dues són equivalents (`sep = 2 * max - 1`).

En models multifactorials, avalua l'aturada **per factor**: un factor queda decidit quan la seva marginal surt de la zona indeterminada (per exemple, `P(error) >= 0.7` present, `<= 0.3` absent) i té la seva mostra mínima d'evidència. Tanca quan tots els factors estiguin decidits, quan cap ítem aporti guany apreciable sobre els indeterminats o en assolir el màxim pràctic; els factors sense decidir es reporten com a indeterminats, no es forcen.

Si es tanca per màxim de preguntes, banc exhaurit o utilitat marginal baixa sense assolir el criteri de confiança triat, presenta el resultat com a provisional.

Regles mínimes:

- no tanquis massa aviat;
- no allarguis artificialment la sessió quan la utilitat marginal ja és baixa;
- si la incertesa continua sent alta, el resultat s'ha d'indicar com a provisional.

## Reforç continu sense aturada

- En recursos de pràctica o reforç obert pot no haver-hi criteri d'aturada: la sessió continua mentre l'alumne practiqui.
- En aquest mode, l'estat estimat no és un diagnòstic tancat, sinó una estimació viva que s'actualitza amb cada resposta.
- No apliquis `H_stop` ni `p_min` per tancar la sessió; utilitza'ls, si de cas, només per informar del grau de confiança assolit.
- Mantén la selecció en dues fases durant tota la sessió: diagnòstic mínim per categoria i, després, reforç del que menys es domina.
- L'estat de l'alumne pot canviar mentre practica: aplica oblit exponencial perquè l'estimació segueixi el seu estat actual. Abans de cada actualització, atenua el posterior **cap al prior** `pi_i` i renormalitza: `p_i <- p_i^lambda * pi_i^(1 - lambda)` (dividit per la suma). Amb prior uniforme, això coincideix amb la forma simple `p_i <- p_i^lambda / Σ_j p_j^lambda`.
- **No atenuïs cap a la uniforme quan el prior sigui informatiu.** La forma simple té la uniforme com a punt fix: un factor d'error amb prior `P(error) ≈ 0.25` que passi un temps sense rebre evidència deriva sol cap al 50 % (≈ 0.40 després de 20 passos amb `lambda = 0.95`) i reapareix com a «indeterminat» o «probable» sense que l'alumne hagi fet res — el mateix fals positiu que el prior informatiu evita. Ancorat al prior, l'oblit descarta l'evidència antiga (la de fa `k` passos pesa `lambda^k`) però el prior conserva sempre pes complet, i una distribució sense evidència nova roman en el seu prior en lloc de degradar-se.
- Aplica l'atenuació a cada distribució en cada pas de la sessió, també a les que no reben evidència en aquella resposta: així totes segueixen el pas del temps i, gràcies a l'ancoratge, les no observades tornen al seu prior, no a la uniforme.
- Fixa primer la memòria efectiva `M` en **intents de la mateixa distribució** (per defecte `M = 20`), no en respostes de la sessió, i deriva `lambda` d'aquí. Si l'estat és una sola distribució que s'actualitza en cada resposta, `lambda = 1 - 1/M` (amb `M = 20`, `lambda = 0.95`, el rang habitual `0.9`–`0.98`).
- **Amb diverses distribucions paral·leles, calcula un `lambda` per distribució.** En atenuar-les totes en cada resposta, cadascuna envelleix un cop per resposta però només rep evidència quan aquella resposta li correspon. Si la distribució `d` s'actualitza en `1` de cada `K_d` respostes, un `lambda` comú li deixaria una memòria de `M / K_d` intents propis. Fes servir `lambda_d = (1 - 1/M)^(1 / K_d)`, on `K_d` és el nombre mitjà de respostes entre dues actualitzacions de `d` (`K_d = 1` per a les dimensions avaluades en cada resposta; `K_d = n` categories si cada pregunta pertany a una de `n` categories). Exemple: amb `M = 20` i `6` categories, `lambda = 0.95` per a una dimensió avaluada sempre i `lambda = 0.95^(1/6) ≈ 0.9915` per a cada categoria. Aplicar el `0.95` també a les categories els deixaria `20 / 6 ≈ 3.3` intents de memòria i esborraria el diagnòstic inicial (que sol tenir `2` intents per categoria).
- En recursos diagnòstics de sessió curta fes servir `lambda = 1` (sense oblit): allà només afegiria soroll.
- Amb oblit actiu, compta la mostra mínima per categoria o dimensió sobre una finestra recent, no sobre tota la sessió: l'evidència caduca amb l'oblit, però un comptador acumulat no, i una categoria mostrejada només al principi continuaria comptant com a diagnosticada amb el seu posterior ja degradat. La finestra de cada distribució dura el que la seva memòria: els intents dins de les últimes `1 / (1 - lambda_d)` respostes. Compta'ls sense ponderar, perquè els llindars enters conservin el seu significat: amb pes exponencial, dos intents consecutius sumen `1.95` i no complirien un llindar de `2`.
- Aquest comptador amb caducitat governa les portes de «mostra mínima» per afirmar domini, no allò que es mostra a l'alumne: si la interfície ensenya quants exercicis ha resolt, aquest nombre és el total real i no caduca.
- Si una categoria es queda sense evidència dins de la finestra, la seva creença ja ha tornat al prior: presenta-la com a **sense dades recents**, no com una debilitat. Marcar en vermell allò que el model ja no sosté és acusar l'alumne d'una cosa que no ha mostrat.
- Amb oblit actiu, presenta la confiança com a referida a l'estat recent de l'alumne.
- Si necessites modelar explícitament l'aprenentatge (per exemple, més probabilitat de pujar de nivell just després d'una explicació), fes servir un model de transició (Bayesian Knowledge Tracing); consulta `matematicas.html §3.5` (https://jjdeharo.github.io/recursos-adaptativos/matematicas_ca.html#s3).

## Itineraris per etapes

Si el recurs té fases, tècniques o etapes successives:

- distingeix entre estimació global i estimació local per etapa;
- no promociones una etapa utilitzant només la creença global acumulada;
- decideix la superació de l'etapa amb evidència generada dins d'aquesta etapa.

Per donar una etapa per superada, convé exigir almenys:

- confiança local suficient;
- entropia local suficientment baixa;
- i un mínim explícit de rendiment observat en aquesta etapa, mesurat sobre evidència **no** seleccionada per màxima informació.

Compte amb el percentatge brut d'encerts com a criteri: si dins de l'etapa tries els ítems per màxima guany d'informació, la taxa d'encert de tots els alumnes tendeix per disseny cap a `(1+c)/2` (≈ 50 % sense atzar, ≈ 62 % amb quatre opcions), de manera que un llindar fix com «60 % d'encerts» pot bloquejar alumnes que sí que dominen l'etapa i el seu efecte depèn del format de les preguntes. Per mesurar el rendiment de manera comparable, fes servir una d'aquestes dues vies:

- **Ítems de sortida (recomanat):** exigeix encertar 1-2 ítems de dificultat representativa de l'objectiu de l'etapa, seleccionats **sense** criteri informatiu (no per màxima IG). En fixar la dificultat, l'encert sí que informa sobre el domini. El format importa: evita vertader/fals per als ítems de sortida (un sol ítem V/F deixa passar per atzar la majoria dels qui no dominen, `P(encert | no domini) ≈ 0.6`); amb 4-5 opcions exigeix encertar-los tots dos; amb oberta tingues present que exigir 2 de 2 bloqueja ≈ 1 de cada 4 alumnes que sí que dominen. El filtre de sortida complementa la confiança local `p_min`, que ja criba: la seva funció és caçar una calibració dolenta, no decidir en solitari.
- **Consistència amb el model:** exigeix que la taxa observada no quedi gaire per sota de l'esperada sota la hipòtesi de domini local (és l'ajust de persona `l_z` dels fonaments aplicat com a criteri d'etapa). Amb els pocs ítems d'una etapa l'aproximació normal de `l_z` és feble: compara encerts observats davant d'esperats amb un marge de ~1 desviació típica (o un test binomial exacte) i tracta-ho com a senyal orientatiu, no com a bloqueig dur.

Tot això ho aplica el mateix recurs de manera automàtica, a partir de les dificultats que ja coneix: no requereix que el docent conegui la metodologia ni configuri res, llevat que ho vulgui fer.

Exemple raonable:

- `p_min = 0.80`
- encert de 1-2 ítems de sortida de dificultat representativa, en lloc d'un llindar fix de percentatge d'encerts

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
- Efecte de les pistes en l'evidència: si l'alumne encerta **després d'una pista**, no ho registris com a encert ple en l'actualització bayesiana. Una pista puja la probabilitat d'encert d'aquell ítem, així que tracta'l com a crèdit parcial amb `s < 1` (tant menor com més determinant sigui la pista; si la pista pràcticament dóna la resposta, l'evidència de domini és gairebé nul·la) i aplica la versemblança geomètrica del crèdit parcial. Els temps de resposta i l'ús d'ajudes es poden registrar a títol informatiu, però aquesta versió no defineix versemblança per a ells: no alteren per si sols l'actualització.

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

No declaris un domini alt basant-te en molt pocs intents. Si el resultat fa afirmacions per categoria o dimensió, exigeix una mostra mínima en aquestes categories o dimensions abans de presentar-les com a fermes (amb oblit actiu, aquesta mostra es compta sobre la finestra recent de la distribució, no sobre el total històric; vegeu «Reforç continu sense aturada»). Si l'estimació és global, la mostra mínima pot referir-se al conjunt de la sessió; limita o marca com a provisional l'estimació quan l'evidència sigui escassa.

No retornis només una nota o etiqueta.

Si dues hipòtesis acaben amb probabilitats properes, mostra la distribució posterior completa (per exemple, un diagrama de barres), no només l'etiqueta guanyadora.

En la vista de l'alumne, la recomanació pedagògica i el pas següent han de tenir **més pes visual** que l'etiqueta de nivell. Formula el resultat en termes de tasca («et convé practicar X abans que Y»), no de tret («ets nivell bàsic»): la literatura sobre expectatives indica que l'etiqueta de tret té més risc. L'etiqueta de nivell amb la seva probabilitat convé reservar-la per a la vista docent.

Si el model diagnostica un error concret (per exemple, «confon massa amb pes», amb la seva probabilitat), comunica-l'hi a l'alumne com una **hipòtesi a comprovar junts**, no com una sentència («comprovem si…»), especialment a primària. L'etiqueta d'error amb la seva probabilitat és apropiada per a la vista docent.

## Revisió docent del contingut

La validació sota el model no substitueix la revisió del contingut, que només el docent pot fer bé. En lliurar el recurs, genera una llista de comprovació breu en llenguatge docent, sense tecnicismes, centrada en allò que el docent pot validar millor que el sistema:

- Aquests errors, són errors reals del teu alumnat?
- Hi ha alguna pregunta massa fàcil o massa difícil per al curs?
- Les respostes correctes i les explicacions són correctes?
- Falta algun cas important del tema?
- El llenguatge és adequat per a l'edat?

Presenta-la a la conversa en lliurar el recurs o a la vista docent, mai a la de l'alumne. No demanis al docent que revisi paràmetres, priors ni probabilitats: si assenyala un problema amb les seves paraules, ajusta tu el banc o el model.

Si el banc es genera per plantilles, la revisió docent és **per plantilla**, no per ítem: n'hi ha prou de revisar una vegada la regla i una mostra de les seves variants. Digues-ho explícitament en lliurar, perquè el docent no cregui que està validant un banc tancat.

## Restriccions d'implementació

- La interfície ha de ser comprensible per a l'alumnat i el professorat.
- Ha de mostrar amb claredat el progrés i la retroalimentació.
- Si fas servir fórmules visibles, acompanya-les d'una interpretació llegible.
- Evita dependre d'un backend si no s'ha demanat.
- Evita preguntes obertes llargues sense correcció automàtica fiable.
- Accessibilitat mínima per defecte, encara que el docent no la demani: contrast suficient, navegació per teclat, no transmetre informació només mitjançant el color, text redimensionable i sense límit de temps per defecte (llevat que el disseny ho requereixi i s'avisi).
- Privacitat per defecte: no enviïs les dades de l'alumne fora del navegador. El recurs estàtic sense backend ja ho garanteix i és l'opció preferent en tractar dades de menors. Si es demana persistència de resultats, adverteix sobre la protecció de dades i prefereix l'exportació local (per exemple, descarregar un fitxer) en lloc de pujar-les a un servidor.

## Valors per defecte recomanats

Si el docent no especifica paràmetres:

- `n = 3` hipòtesis o nivells;
- discriminació efectiva objectiu `a_ef = 1.25`; calcula sempre `a = 1.25 / (1 - c_q)` per pregunta (amb `c_q = 0` dona `a = 1.25`; màxim `a = 2.5` en vertader/fals);
- `p_min = 0.80`;
- mínim de preguntes: entre `4` i `6`;
- mínim de diagnòstic per categoria en pràctica adaptativa: `2` intents;
- màxim pràctic: entre `10` i `20`, segons el tipus de recurs;
- pes del guany d'informació en la fase de reforç: `α = 0.65` (rang raonable `0.6`–`0.7`);
- oblit exponencial: memòria efectiva `M = 20` intents de cada distribució, d'on `lambda_d = (1 - 1/M)^(1 / K_d)` (`lambda = 0.95` si la distribució s'actualitza en cada resposta) en pràctica o reforç continu; `lambda = 1` en diagnòstic de sessió curta;
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

## Validació i fiabilitat

Aquestes comprovacions augmenten l'honestedat del diagnòstic sense requerir dades empíriques. Aplica-les quan la finalitat sigui diagnòstica o el resultat s'hagi d'usar per decidir. La validació del disseny ha de quedar preparada encara que la IA treballi en un xat sense capacitat d'execució.

- **Ajust del patró individual (person-fit).** En tancar una sessió, avalua si el patró de respostes és coherent amb el nivell estimat. Calcula l'índex estandarditzat `l_z` a partir de les probabilitats d'encert que el model assigna a les preguntes respostes sota la hipòtesi més probable. Si `l_z` és molt negatiu (orientativament `< -2`), marca el diagnòstic com a poc fiable encara que el posterior sigui alt: sol indicar respostes incoherents amb la dificultat, descuits o atzar. És un senyal de cautela, no una prova formal; amb poques preguntes és només orientatiu.
- **Detecció d'atzar o ansietat durant la sessió (no només al tancament).** No esperis al tancament per calcular l'ajust: si durant la sessió el person-fit o una ratxa inversemblant (fallar preguntes fàcils i encertar-ne de difícils, o respondre massa de pressa) suggereixen resposta a l'atzar o ansietat, el recurs hauria de poder **pausar i reconduir** («sembla que vas molt de pressa, seguim amb calma?») en lloc de continuar consumint banc. Fes-ho amb tacte, sense acusar, i reprèn amb normalitat.
- **Verificació del banc generat.** Si els ítems es produeixen per plantilles, executa abans de lliurar la comprovació descrita a «Bancs generats» sobre uns centenars de variants per plantilla, i tracta-la com a condició de lliurament, no com a prova opcional: mesura la correcció del generador, no el domini de l'alumne. Reporta quantes variants i quantes plantilles s'han comprovat. Si l'entorn no permet executar codi, deixa la utilitat escrita i marca el banc com a «variants pendents de verificar»; no afirmis que estan comprovades.
- **Separabilitat del disseny (Monte Carlo).** Com a propietat del test (no de l'alumne), estima amb quina fiabilitat el banc distingeix els nivells: genera respondents sintètics situats en el `theta` de cada hipòtesi, fes-los el test reutilitzant la mateixa selecció adaptativa i el mateix criteri d'aturada, i construeix la matriu de confusió (nivell real enfront de diagnosticat). Presenta-la com a fiabilitat sota el model, mai com a validesa empírica: els respondents surten del propi model, així que mesura si el disseny discrimina els nivells, no si els paràmetres reflecteixen la realitat. **Aquesta validació és una eina del creador del recurs, no de l'alumnat: no s'ha de mostrar en la interfície de l'alumne.**
  - Si l'entorn de la IA permet executar codi (CLI, entorn local, notebook), genera i executa la simulació en construir el recurs.
  - Si la IA treballa en xat sense execució, no afirmis que la validació està feta: implementa una utilitat de validació en un fitxer separat o en una vista docent/autora oculta a l'alumnat, amb un botó per executar-la al navegador, i marca el disseny com a «validació pendent d'executar».
  - Fes servir per defecte almenys `500` simulacions per hipòtesi o perfil (`1000` si el navegador ho suporta amb fluïdesa). Reporta matriu de confusió, exactitud equilibrada, taxa per hipòtesi/perfil, taxa de resultats indeterminats i longitud mitjana de la sessió.
  - Criteri orientatiu: si alguna hipòtesi rellevant queda per sota de `0.70` de classificació correcta sota el propi model, o si dues hipòtesis es confonen de manera sistemàtica, no presentis el banc com a ben separat; afegeix més ítems, revisa dificultats/versemblances o declara explícitament la limitació.

Consulta `matematicas.html §11.7–§11.8` (https://jjdeharo.github.io/recursos-adaptativos/matematicas_ca.html#s11) per a les fórmules i l'enquadrament complet.

## Verificació abans de lliurar

Comprova el recurs generat contra aquesta llista. Els blocs condicionals, només si corresponen al perfil i mode declarats a «Perfil i mode».

**Sempre:**

- Cada resposta actualitza el posterior (versemblança × prior, normalitzat); no hi ha regles ad hoc de pujar/baixar dificultat en el seu lloc.
- `a` derivada per ítem (`a = 1.25 / (1 - c_q)`) i sostre de domini `P(encert) <= 0.95` aplicats en la versemblança.
- Escala `theta` fixa segons `n` (si el perfil usa `theta`), amb les dificultats dins de la meitat central.
- El resultat no és només una nota: la recomanació i el pas següent pesen visualment més que l'etiqueta, i els errors es comuniquen com a hipòtesis a comprovar.
- Llista de comprovació docent generada (sobre contingut, no sobre paràmetres).
- Si el banc es genera per plantilles: resposta derivada de la regla (no transcrita), metadades del model fixes per plantilla, verificació automàtica de les variants executada i una mostra llegida a ull.
- Accessibilitat mínima i privacitat per defecte (cap dada de l'alumne surt del navegador).

**Si el perfil és `C` o `A+C`:**

- Prior d'error `0.2`–`0.3`, no uniforme; `0.40` si el docent descriu l'error com a freqüent, `0.10`–`0.15` si el descriu com a rar (mai `≥ 0.5` ni `< 0.10`).
- Cap factor declarat present o absent sense la seva mostra mínima d'evidència; el report distingeix «absent confirmat» de «sense evidència suficient».
- Aturada avaluada per factor; els no decidits es reporten com a indeterminats, no es forcen.
- En `A+C`: selecció per blocs pedagògics amb pesos segons la finalitat; l'aturada per guany mínim s'avalua sobre la IG crua, no sobre la utilitat normalitzada.

**Si el mode és `Pràctica`:**

- Oblit ancorat al prior, amb `lambda_d` per distribució segons la seva freqüència d'actualització.
- Mostra mínima comptada en finestra recent; el comptador visible per a l'alumne és el total real.
- Distribució sense evidència recent → «sense dades», mai vermell.
- Un ítem la correcció del qual ja s'ha mostrat no torna com a evidència plena: variants parametritzades o crèdit parcial reduït, mai exclusió de l'actualització (amb un banc finit, l'exclusió congela l'estimació i bloqueja la selecció).
- Cada categoria cobreix el rang de dificultats per si sola: cap no deixa `ajust = 0` per a un nivell sencer d'alumnat.

**Si el mode és `Itinerari`:**

- Promoció amb evidència local de l'etapa, més ítems de sortida (sense vertader/fals) o consistència amb el model; mai un llindar fix de percentatge d'encerts sota selecció per màxima IG.
- En repetir una etapa: estimació local reiniciada i exercicis regenerats com a variants.

**Si la finalitat és diagnòstica:**

- Utilitat de validació Monte Carlo a part (o en vista docent), sembrada i reproduïble, amb exactitud equilibrada, taxa d'indeterminats, longitud mitjana i alerta per sota de `0.70`.
- Si no s'ha pogut executar, queda marcada com a «validació pendent d'executar», sense afirmar que el banc està comprovat.

## Nota sobre el model

La funció IRT 3PL descrita aquí s'utilitza com a generador inicial de versemblances quan no hi ha dades empíriques. No equival a una calibració psicomètrica validada. Si s'acumulen suficients respostes reals, convé recalibrar dificultats, discriminació i pseudoatzar.
