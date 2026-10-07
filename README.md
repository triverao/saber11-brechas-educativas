# Brechas educativas en las pruebas Saber 11 (2010–2022)

Proyecto de ciencia de datos que usa datos abiertos del Estado colombiano para medir qué tanto se asocian las condiciones del hogar y las características del colegio con el puntaje de los estudiantes en el examen Saber 11.

> **Estado del proyecto:** fase inicial (comprensión de los datos). El repositorio contiene el planteamiento, la consulta reproducible a la fuente y las primeras verificaciones. Todavía no hay modelo ni conclusiones.

## Tabla de contenido

1. [Pregunta del proyecto](#pregunta-del-proyecto)
2. [Datos](#datos)
3. [Metodología](#metodología)
4. [Estructura del repositorio](#estructura-del-repositorio)
5. [Cómo reproducirlo](#cómo-reproducirlo)
6. [Primeras verificaciones](#primeras-verificaciones)
7. [Limitaciones y uso responsable](#limitaciones-y-uso-responsable)
8. [Hoja de ruta](#hoja-de-ruta)
9. [Autoría y licencia](#autoría-y-licencia)
10. [Referencias](#referencias)

## Pregunta del proyecto

¿Qué tan grandes son las diferencias en el puntaje global de Saber 11 entre grupos de estudiantes, y cuáles características las explican mejor?

Se comparan grupos definidos por:

- **El colegio:** naturaleza (oficial o no oficial), zona (urbana o rural), jornada y departamento.
- **El hogar:** estrato de la vivienda, nivel educativo de la madre y del padre, acceso a internet y a computador.

El resultado esperado es una medición descriptiva de esas brechas y de su evolución en el tiempo, útil para quien diseña o evalúa política educativa. El proyecto **no** busca establecer causalidad.

## Datos

| Campo | Detalle |
| --- | --- |
| Conjunto | Resultados únicos Saber 11 |
| Entidad | Instituto Colombiano para la Evaluación de la Educación (ICFES) |
| Portal | Datos Abiertos Colombia: <https://www.datos.gov.co/d/kgxf-xxbe> |
| Cobertura | Periodos 2010-1 a 2022-4 |
| Tamaño | 7.109.704 filas y 51 columnas (consulta del 7 de octubre de 2026) |
| Última actualización de los datos | 23 de agosto de 2023 |
| Licencia de los datos | Creative Commons Atribución-CompartirIgual 4.0 (CC BY-SA 4.0) |

Cada fila es un estudiante que presentó el examen. Las variables se agrupan por prefijo:

| Prefijo | Qué describe | Ejemplos |
| --- | --- | --- |
| `cole_` | El colegio | `cole_naturaleza`, `cole_area_ubicacion`, `cole_jornada`, `cole_depto_ubicacion` |
| `estu_` | El estudiante | `estu_genero`, `estu_fechanacimiento`, `estu_depto_reside` |
| `fami_` | El hogar | `fami_estratovivienda`, `fami_educacionmadre`, `fami_tieneinternet` |
| `punt_` | Los puntajes | `punt_matematicas`, `punt_lectura_critica`, `punt_ingles`, `punt_global` |

Los datos no se copian en este repositorio: se consultan directamente a la fuente, de modo que cualquier persona puede verificar las cifras.

## Metodología

El trabajo sigue las seis fases de CRISP-DM, el proceso estándar para proyectos de minería y ciencia de datos:

1. **Comprensión del problema.** Definir la pregunta y a quién le sirve la respuesta.
2. **Comprensión de los datos.** Revisar cobertura, valores faltantes y posibles duplicados. *(Fase actual.)*
3. **Preparación de los datos.** Convertir tipos, unificar categorías y resolver duplicados.
4. **Modelado.** Estadística descriptiva de las brechas y, después, una regresión lineal del puntaje global sobre las variables del hogar y del colegio.
5. **Evaluación.** Revisar supuestos, tamaño de los efectos y estabilidad entre periodos.
6. **Comunicación.** Tablas, gráficas y un informe breve con los hallazgos.

## Estructura del repositorio

La organización toma como referencia la plantilla Cookiecutter Data Science, reducida a lo que el proyecto necesita hoy:

```
.
├── README.md                  Presentación del proyecto (este archivo)
├── LICENSE                    Licencia del código (MIT)
├── .gitignore                 Archivos que Git no debe versionar
├── requirements.txt           Dependencias de Python
├── src/
│   └── consultar_saber11.py   Consulta agregada a la API de datos.gov.co
├── tests/
│   └── test_consultar.py      Pruebas del script de consulta
└── reports/                   Tablas generadas por el script (se crea al ejecutarlo)
```

## Cómo reproducirlo

Se necesita Python 3.10 o superior.

```bash
git clone https://github.com/triverao/saber11-brechas-educativas.git
cd saber11-brechas-educativas
python -m venv .venv
source .venv/bin/activate        # En Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Consultar el puntaje promedio por naturaleza del colegio en un periodo:

```bash
python src/consultar_saber11.py cole_naturaleza --periodo 20224
```

Cambiar la variable de agrupación, o traer todos los periodos:

```bash
python src/consultar_saber11.py fami_estratovivienda
```

Ejecutar las pruebas:

```bash
python -m unittest discover tests
```

El script no descarga los 7 millones de filas. Le pide a la API que agregue en el servidor y guarda el resultado como CSV en `reports/`.

## Primeras verificaciones

Cifras obtenidas de la API del portal el 7 de octubre de 2026. Son verificaciones de calidad de los datos, no resultados del proyecto.

**1. Faltan periodos.** El conjunto tiene 23 periodos, pero no trae registros del segundo semestre de 2018, de 2020 ni de 2021. Cualquier serie de tiempo debe mostrar esos vacíos en lugar de interpolarlos.

**2. Posibles registros duplicados.** Los periodos 2019-4 y 2022-4 tienen cerca del doble de filas que los segundos semestres anteriores:

| Periodo | Filas |
| --- | ---: |
| 2015-2 | 544.491 |
| 2016-2 | 550.275 |
| 2017-2 | 548.269 |
| 2019-4 | 1.096.524 |
| 2022-4 | 1.065.888 |

Antes de calcular cualquier brecha hay que revisar si hay estudiantes repetidos (`estu_consecutivo`) en esos dos periodos.

**3. Consulta de ejemplo (periodo 2022-4).** Puntaje global promedio por naturaleza del colegio, sin depurar duplicados:

| Naturaleza del colegio | Filas | Puntaje global promedio |
| --- | ---: | ---: |
| No oficial | 240.956 | 272,55 |
| Oficial | 824.930 | 243,60 |
| Sin dato | 2 | 192,00 |

La diferencia bruta es de unos 29 puntos. Es una comparación sin ajustar: no descuenta el estrato, la zona ni la educación de los padres, y por eso no dice nada sobre la calidad de un tipo de colegio frente al otro.

## Limitaciones y uso responsable

- **Asociación, no causalidad.** Los datos son observacionales. Las diferencias entre grupos reflejan también quiénes asisten a cada tipo de colegio.
- **Comparación entre años.** La estructura del examen cambió en el segundo semestre de 2014, así que los puntajes anteriores y posteriores no son directamente comparables.
- **Datos autorreportados.** Las variables del hogar provienen del formulario que diligencia el estudiante y tienen valores faltantes.
- **Datos personales.** El conjunto es público y no trae nombres ni documentos, pero sí información por estudiante. El proyecto solo publica resultados agregados y no intenta identificar personas ni señalar colegios individuales.

## Hoja de ruta

- [x] Plantear la pregunta y elegir la fuente
- [x] Consulta reproducible a la API
- [x] Primeras verificaciones de cobertura y calidad
- [ ] Depurar duplicados y unificar categorías
- [ ] Análisis exploratorio con gráficas por estrato, zona y departamento
- [ ] Regresión del puntaje global y evaluación del modelo
- [ ] Informe final

## Autoría y licencia

Proyecto académico del curso Introducción a la ciencia de datos (DATA1001), Universidad de los Andes, 2026-20.

Autor: [@triverao](https://github.com/triverao)

El código se publica bajo licencia MIT (ver [LICENSE](LICENSE)). Los datos pertenecen al ICFES y se usan bajo licencia CC BY-SA 4.0, que exige citar la fuente y compartir las obras derivadas con la misma licencia.

## Referencias

- Instituto Colombiano para la Evaluación de la Educación (ICFES). *Resultados únicos Saber 11* [conjunto de datos]. Datos Abiertos Colombia. <https://www.datos.gov.co/d/kgxf-xxbe>
- Chapman, P., Clinton, J., Kerber, R., Khabaza, T., Reinartz, T., Shearer, C. y Wirth, R. (2000). *CRISP-DM 1.0: Step-by-step data mining guide*. SPSS.
- DrivenData. *Cookiecutter Data Science*. <https://cookiecutter-data-science.drivendata.org/>
- Tyler Technologies. *Queries using SODA* (documentación de la API Socrata). <https://dev.socrata.com/docs/queries/>
- GitHub Docs. *About READMEs*. <https://docs.github.com/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes>
# saber11-brechas-educativas
Proyecto de ciencia de datos con datos abiertos de Colombia: brechas en los resultados de las pruebas Saber 11 (ICFES, 2010-2022).
