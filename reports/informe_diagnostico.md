# Informe técnico: diagnóstico inicial de calidad de datos

**Proyecto:** Distribución de la propiedad horizontal en las comunas de Medellín
**Evidencia:** GA1-220501094-01-AA1-EV06 – Diagnosticar la calidad de datos: preparación, exploración y documentación
**Aprendiz:** Maicol Esneider Posada Callejas · Ficha 3223873 · Tecnólogo en Análisis y Desarrollo de Software (SENA)
**Instructor:** Carlos Navia · **Fecha:** noviembre de 2026

Todos los resultados de este informe salen del notebook [`notebooks/diagnostico_calidad.ipynb`](../notebooks/diagnostico_calidad.ipynb).

---

## 1. Nombre del proyecto y pregunta analítica

**Proyecto:** Distribución de la propiedad horizontal en las comunas de Medellín.

**Propósito:** identificar en qué zonas de Medellín se concentran los conjuntos residenciales y edificios en propiedad horizontal, para decidir dónde enfocar primero Domux, un software de administración de propiedades horizontales que se desarrolla como proyecto de grado.

**Pregunta analítica:** ¿En qué comunas de Medellín se concentran los predios habitacionales en propiedad horizontal y qué proporción representan frente al total de predios habitacionales de cada comuna?

**Tipo de análisis:** descriptivo.

## 2. Descripción de los datasets utilizados

| N.º | Dataset | Fuente | Contenido | Uso en el proyecto |
|---|---|---|---|---|
| 1 | Información Predios (`informacion_predios.csv`) | MEData, Subsecretaría de Catastro de Medellín. Licencia CC BY-SA 4.0. Última actualización: 2021 | Un registro por predio: comuna, barrio, manzana, lote (CBML), matrícula, porcentaje de copropiedad (`PorDes`), estrato, uso predial y avalúo | Fuente principal para contar predios por comuna y estimar la propiedad horizontal |
| 2 | Información Lotes (`informacion_lotes.csv`) | MEData, Subsecretaría de Catastro de Medellín. Última actualización: 2021 | Un registro por lote: área del lote, área construida, zona geoeconómica y número de pisos | Complemento de los predios, unido por el `CBML` |
| 3 | Medellín Properties 2023 (`medellin_properties.csv`) | Kaggle (cesaregr). Origen no documentado | Inmuebles en venta en 2023 con barrio, coordenadas, precio, área, administración y estrato | Apoyo: oferta de inmuebles y cobro de administración |
| 4 | Precios inmuebles Medellín (`houses_medellin.csv`) | Kaggle (valentinafeve). Origen no documentado | Inmuebles en venta con habitaciones, estrato, antigüedad, administración, precio y área | Apoyo: relación entre administración, estrato y área |

## 3. Resultados de la inspección estructural

Para cada dataset se ejecutaron `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`, `describe()` y `describe(include='object')`.

| Dataset | Filas | Columnas | Tipos de datos | Observaciones |
|---|---|---|---|---|
| Información Predios | 1.041.413 | 15 | 9 enteros, 2 decimales, 4 texto | Códigos (`CBML`, `Matricula`) leídos como números; `NroPropietarios` siempre 0; estrato entre 0 y 6; avalúos con valores extremos |
| Información Lotes | 308.129 | 8 | 4 enteros, 3 decimales, 1 texto | Sin nulos; `CBML` como número y `ZonaGeo` como decimal; hasta 39 pisos |
| Medellín Properties 2023 | 9.999 | 12 | 2 enteros, 5 decimales, 5 texto | Coordenadas y `garages` como texto; `age` es un código de 0 a 4; precios y áreas extremos |
| Precios inmuebles Medellín | 430 | 14 | 2 enteros, 4 decimales, 8 texto | Columna de índice sobrante (`Unnamed: 0`); montos y áreas como texto; sin columna de ubicación |

## 4. Valores faltantes, tipos de datos y posibles inconsistencias

### 4.1 Valores faltantes

- **Información Predios:** 5.800 nulos en `Urbanizacion`, más 590.022 registros con el valor de relleno "--" (en total, el 57,2 %). `NroPropietarios` vale 0 en todos los registros y 391.024 predios tienen estrato 0.
- **Información Lotes:** no tiene nulos.
- **Medellín Properties 2023:** `age` (2.637 nulos, 26,4 %), `garages` (3.206, 32,1 %), `baths` (122) y `rooms` (63). Además, 3.930 inmuebles tienen `administration_price` igual a 0 y 54 tienen coordenadas "INVALID".
- **Precios inmuebles Medellín:** todas las columnas, salvo el índice y `Precio`, tienen entre el 28 % y el 96 % de vacíos. 121 filas (28,1 %) solo tienen el precio.

**Impacto:** los promedios y conteos se calculan solo con los valores disponibles. Los valores de relleno ("--", 0, "No definida", "INVALID") no aparecen como nulos en `info()`, así que pueden pasar desapercibidos y distorsionar los resultados.

### 4.2 Tipos de datos

| Dataset | Columna | Tipo actual | Tipo esperado | Causa |
|---|---|---|---|---|
| Predios y Lotes | `CBML` | `int64` | texto | Pandas quita el cero inicial: 353.005 predios de las comunas 1 a 9 quedan con 10 dígitos en lugar de 11 |
| Predios | `Matricula`, `UsoPredial` | `int64` | texto o categoría | Son códigos |
| Lotes | `ZonaGeo` | `float64` | texto | Código de 15 dígitos convertido a decimal |
| Medellín Properties | `latitude`, `longitude` | texto | decimal | Valores "INVALID" |
| Medellín Properties | `garages` | texto | entero | Valor "Más de 10" |
| Medellín Properties | `age` | decimal | categoría | Código de rango de antigüedad (0 a 4) |
| Precios inmuebles | `Administración`, `Precio m²`, áreas | texto | decimal | Símbolos (`$`, `COP`, `*m²`), puntos de miles y espacios |

### 4.3 Posibles inconsistencias

- Los nombres de varias columnas de MEData no coinciden con su diccionario de datos: `UsoPredial` (diccionario: `Uso Predial`), `nomComuna` (`NomComuna`), `DS_BARRIO` (`DS_Barrio`) y `ZonaGeo` (`Zona Geo`).
- Nombres de comuna mal escritos en Predios: "MANRRIQUE" y "LA AMERIA".
- La suma del porcentaje de copropiedad por lote debería ser 100, pero en 430 de 308.129 lotes no lo es (la suma máxima encontrada es 7.400). Además, 16.868 predios tienen `PorDes` igual a 0.
- Lotes: 80 con área del lote igual a 0, 696 sin área construida pero con pisos y 415 con área construida pero 0 pisos.
- Medellín Properties: 100 filas duplicadas; 104 inmuebles con coordenadas fuera del área de Medellín (algunas de otros países); áreas de 0 m² (4) y de más de 2.000 m² (21); 11 precios por encima de 100.000 millones de pesos; 5 inmuebles con estrato 0.
- Precios inmuebles: 185 filas duplicadas sin contar el índice y un precio máximo de 1.500.000.000.000 pesos.

## 5. Principales hallazgos de calidad por dataset

**Información Predios.** Es completo (solo una columna con nulos) y cubre las 16 comunas y los 5 corregimientos de Medellín. Sus problemas son de tipo de dato (`CBML`), valores de relleno (`Urbanizacion`, `Estrato` 0), una columna sin información (`NroPropietarios`) y nombres mal escritos. La limitación más importante para la pregunta es que **no existe una columna que indique la propiedad horizontal**: el criterio propuesto (porcentaje de copropiedad menor que 100 en un lote compartido) marca 820.173 predios (78,8 %), lo que incluye parqueaderos, cuartos útiles y locales. Tampoco hay una tabla que explique los 11 códigos de `UsoPredial`, necesaria para separar lo habitacional.

**Información Lotes.** No tiene nulos y todos sus `CBML` coinciden con los de Predios, así que las dos tablas se pueden unir. Tiene códigos leídos como números y algunos lotes con áreas o pisos incoherentes.

**Medellín Properties 2023.** Es reciente y tiene barrio y administración, pero su origen no está documentado. Presenta tipos incorrectos, duplicados, coordenadas fuera de la ciudad y valores extremos. No tiene comuna.

**Precios inmuebles Medellín.** Es el de menor calidad: muy pequeño, con la mayoría de columnas vacías o como texto, duplicados y sin ninguna columna de ubicación. No permite responder la pregunta por comunas.

## 6. Acciones de limpieza y preparación propuestas

| Dataset | Problema | Acción propuesta |
|---|---|---|
| Predios y Lotes | `CBML` y `ZonaGeo` leídos como números | Leer con `dtype={"CBML": str, "ZonaGeo": str}` |
| Predios y Lotes | Columnas con nombres distintos al diccionario | Renombrar con un estándar único |
| Predios | `NroPropietarios` siempre 0 | Eliminar la columna |
| Predios | "--" en `Urbanizacion` y estrato 0 | Convertir a nulo y tratar el estrato 0 como "sin estrato" |
| Predios | "MANRRIQUE" y "LA AMERIA" | Corregir con un diccionario de reemplazos |
| Predios | Lotes cuya copropiedad no suma 100 | Marcarlos y revisarlos antes de usarlos en el criterio |
| Predios | Sin marca de propiedad horizontal ni tabla de usos | Solicitar la tabla de códigos de uso predial a Catastro y validar el criterio con una muestra de fichas catastrales |
| Lotes | Áreas y pisos incoherentes | Marcar como inconsistentes y excluirlos de los cálculos de área |
| Medellín Properties | Coordenadas como texto y fuera de Medellín | Convertir con `pd.to_numeric(errors="coerce")` y filtrar por el rango de la ciudad |
| Medellín Properties | Duplicados y valores extremos | `drop_duplicates()` y definir rangos válidos de precio y área |
| Medellín Properties | `garages` y `age` | Convertir `garages` a número ("Más de 10" → 11 o categoría) y `age` a categoría |
| Precios inmuebles | Índice sobrante, montos como texto, filas vacías | Leer con `index_col=0`, quitar símbolos y convertir a número, eliminar filas sin datos |

## 7. Conclusiones

1. **Los datasets de MEData son aptos para continuar**, después de una limpieza moderada. Cubren toda la ciudad, casi no tienen nulos y se pueden unir por el `CBML`. Antes de analizar hay que corregir los tipos de datos, los valores de relleno y los nombres de las comunas.
2. **La pregunta analítica todavía no se puede responder con certeza**, porque los datos no marcan la propiedad horizontal y los usos prediales están codificados sin explicación. El paso siguiente es conseguir la tabla de códigos de uso predial y validar el criterio de copropiedad.
3. **Los datasets de Kaggle solo sirven como apoyo.** Tienen más problemas de calidad, su origen no está documentado y no tienen comuna; *Precios inmuebles Medellín* ni siquiera tiene ubicación. Se pueden usar para explorar el cobro de administración, pero no para la conclusión principal.
4. **Los datos describen 2021.** Cualquier resultado debe presentarse como la situación de ese año.
