# Informe — Clasificación de variables y población vs. muestra

**Para:** Gerencia de RetailAndina S.A. · **De:** Equipo de Business Intelligence
**Fecha:** 23/10/2026 · **Asunto:** Base de fidelización: tipos de variables y estimación por muestreo

---

## Resumen ejecutivo

Este informe clasifica formalmente las variables de la base de clientes de RetailAndina S.A. según su naturaleza estadística y evalúa el comportamiento de tres diseños de muestreo para estimar el ingreso mensual promedio frente al parámetro real de la población. Los resultados demuestran que los métodos no aleatorios, como la conveniencia, introducen graves sesgos de selección geográfica, por lo que la gerencia debe exigir diseños rigurosos y representativos antes de tomar decisiones estratégicas basadas en muestras.

## Parte A — Clasificación de las variables

| Variable | Tipo | Justificación (criterio) | Verificación en código |
|---|---|---|---|
| `cliente_id` |Categórica nominal |Identificador único de cada cliente; no admite operaciones aritméticas ni orden lógico. |str, 12 valores únicos. |
| `edad` |Numérica discreta |Representa una cantidad exacta de años cumplidos; los incrementos tienen significado numérico directo. | int64, 12 valores únicos.|
| `numero_hijos` | Numérica discreta|Conteo de elementos enteros; el cero y los valores absolutos reflejan cantidades exactas. | int64, 5 valores únicos ([0, 1, 2, 3, 4]).|
| `genero` | Categórica binaria / nominal| Clasifica en categorías discretas sin una jerarquía numérica subyacente.|str, 3 valores únicos (Femenino, Masculino, Otro). |
| `nivel_educacion` | Categórica ordinal| Existe una jerarquía clara de menor a mayor grado formativo, pero las distancias entre categorías no son cuantitativas.|str convertido a ordinal; min() devuelve "Primaria", 0 faltantes. |
| `ciudad` | Categórica nominal| Agrupación territorial sin relación de orden ni magnitud cuantitativa.|str, 3 valores únicos (Quito, Guayaquil, Cuenca). |
| `ingresos_mensuales` | Numérica continua|Admite valores decimales teóricos y tiene sentido analítico fraccionar o calcular la mitad del valor. | int64 con valores monetarios, 12 valores únicos.|
| `satisfaccion` |Categórica ordinal |Escala valorativa del 1 al 5 donde un nivel superior indica mayor satisfacción, pero una puntuación de 4 no es el doble de 2. |int64 convertido a ordinal, 0 faltantes. |

**P1.** ¿Qué te dijo el dtype de `satisfaccion` y por qué no basta para clasificarla? ¿Qué pasa si una ordinal queda como texto sin declarar el orden?

El dtype llegó originalmente como número entero (int64), lo cual es una trampa porque los lenguajes de programación tienden a tratar cualquier número como cuantitativo. Esto no basta para clasificarla porque no se pueden realizar operaciones aritméticas válidas con ella, como por ejemplo, promediar calificaciones ordinales de forma estricta requiere consideraciones especiales y "el doble de satisfacción" carece de sentido.

## Parte B — Población vs. muestra

### La población y su parámetro

| Concepto | Valor |
|---|---|
| Población (N) |400 clientes |
| Variable de interés |ingresos_mensuales |
| Parámetro poblacional (media) |1330.72 USD |

*¿Por qué se muestrea si aquí el parámetro existe y se calculó?*

Se muestrea con fines metodológicos y de validación pedagógica. Aunque en este entorno controlado conocemos el total de la población, en escenarios reales corporativos las poblaciones son masivas, costosas o inaccesibles, por lo que el muestreo es la única vía eficiente para estimar parámetros con un margen de error controlable.

### Los tres diseños

*Corre tu script y registra las salidas. La composición por ciudad es la que delata el sesgo: anótala tal como salió.*

| Diseño | n | Media estimada | Composición por ciudad | Diferencia vs. parámetro |
|---|---|---|---|---|
| Conveniencia (`head`) |20 | 1348.50 USD| {'Quito': 20}| +17.78 USD|
| Aleatorio simple (`sample`) | 20| 1336.00 USD| {'Cuenca': 8, 'Quito': 8, 'Guayaquil': 4}| +5.28 USD|
| Estratificado (`groupby` + 5 %) | 20| 1287.50 USD|{'Quito': 10, 'Guayaquil': 6, 'Cuenca': 4} | -43.22 USD|

*Anota también las medias de 5 sorteos distintos del aleatorio simple (semillas 1 a 5):*

| Semilla | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Media |1336.00 | 1318.40| 1342.10|1325.50 | 1331.80|

**P2.** ¿Cuánto vale el parámetro poblacional y cuánto estimó cada diseño? ¿Cuál quedó más cerca y por qué eso **no** alcanza para declarar un ganador?

El parámetro poblacional es de 1330.72 USD. La muestra por conveniencia estimó 1348.50 USD, el aleatorio simple 1336.00 USD y el estratificado 1287.50 USD. Aunque el aleatorio simple quedó numéricamente más cerca en esta ejecución específica, pero esto no declara un ganador absoluto, ya que el resultado del muestreo aleatorio varia según la semilla utilizada debido al azar.

**P3.** La muestra por conveniencia también tiene 20 filas. ¿Qué defecto tiene y cómo lo delatas en la composición por ciudad? ¿Se corrige tomando más filas?

La muestra por conveniencia tiene 20 filas, idéntico tamaño que las otras, pero tiene un sesgo de selección: el 100% de los registros pertenecen a una sola ciudad (Quito: 20), ignorando por completo a Guayaquil y Cuenca. Este error no se corrige simplemente tomando más filas, ya que el sesgo se debe al método defectuoso de recolección, no en el tamaño de la muestra.

**P4.** El sorteo aleatorio dio un número distinto en cada semilla. ¿Qué implica esa variabilidad para una empresa que va a decidir con un solo estudio?

La variabilidad de las medias entre las distintas semillas muestra la incertidumbre o error de muestreo. Para una empresa, esto implica que tomar decisiones basadas en una única muestra aleatoria sin conocer su margen de error o intervalo de confianza puede exponer a la organización a fluctuaciones aleatorias del azar.

**P5.** ¿Cuándo usarías cada diseño en la práctica? Relaciona cada uno con un uso real (piloto, subgrupos que deben aparecer, conclusión sobre toda la población).

•	Conveniencia: Únicamente para pruebas piloto rápidas, validación de formularios o fases exploratorias donde no se requieren inferencias estadísticas rigurosas.

•	Estratificado: Cuando la población tiene subgrupos conocidos, como ciudades o niveles socioeconómicos, que deben estar representados obligatoriamente en proporciones correctas para evitar sesgos estructurales.

•	Aleatorio simple: Cuando la población es relativamente homogénea y se busca un método neutral, imparcial y sencillo sin necesidad de segmentar a priori.


## Conclusiones

Los métodos no probabilísticos como la conveniencia pueden arrojar estimaciones engañosas al ignorar por completo sectores clave de la población. La gerencia jamás debe aceptar estudios basados en muestras de conveniencia para decisiones de alto impacto.

Un buen diseño estadístico como el muestreo estratificado o aleatorio controlado, garantiza una representación fiel de la realidad corporativa, superando las limitaciones inherentes al tamaño reducido de una muestra.

Se aconseja institucionalizar el uso de muestreos probabilísticos y exigir siempre el análisis de la variabilidad y los errores de estimación antes de aprobar presupuestos o estrategias comerciales basadas en datos muestrales.
