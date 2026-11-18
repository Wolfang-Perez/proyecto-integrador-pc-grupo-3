[README-entregable-1.md](https://github.com/user-attachments/files/35052849/README-entregable-1.md)
# Proyecto integrador — Grupo 3: Entregas de última milla en el Valle de Aburrá (2025)

> **Este archivo es el `README.md` del repositorio.** Se pega tal cual en GitHub
> (reemplazando el README que crea GitHub al abrir el repositorio).
> Antes de pegarlo, reemplaza los dos nombres de los integrantes por los reales.

## Integrantes

- **Wolfang Pérez** — usuario de GitHub: `Wolfang-Perez`
- **Juan Blanco** — usuario de GitHub: `JuanBlanco19`

**Curso:** Programación científica
**Entregable 1:** análisis y limpieza con pandas (15 %)
**Notebook:** [`notebooks/analisis.ipynb`](notebooks/analisis.ipynb)

## El conjunto de datos

La **última milla** es el último tramo de una entrega: de la bodega a la puerta del
cliente. Es el tramo más caro y el que más se retrasa. Cada fila del archivo es
**un pedido** de una tienda en línea, entregado por uno de cuatro operadores
logísticos en los municipios del Valle de Aburrá entre septiembre y diciembre de
2025 (incluye la temporada de Black Friday y Navidad).

- **Archivo:** `data/logistica_entregas_valle_aburra_2025.csv`
- **Tamaño:** 4.100 filas × 21 columnas
- **Origen:** datos **sintéticos**, generados para el curso a partir de fenómenos
  reales de Colombia. Los lugares existen, las cifras no: las conclusiones valen
  para el dataset, no para la realidad.

**Decisión que apoya:** la tienda debe decidir con qué operador renovar el
contrato, qué vehículo asignar según el pedido y en qué franjas o temporadas
reforzar la operación para reducir retrasos y mejorar la calificación del cliente.

### Resumen de las columnas

| Columna | Tipo | Descripción |
|---|---|---|
| `id_pedido` | texto | Identificador del pedido. |
| `fecha` | fecha (AAAA-MM-DD) | Día de la entrega. |
| `dia_semana` | categórica | Día de la semana. |
| `franja_horaria` | categórica | `Mañana`, `Tarde` o `Noche`. |
| `municipio_destino` | categórica | Municipio del cliente. |
| `canal_compra` | categórica | `App`, `Web`, `WhatsApp`, `Teléfono`. |
| `categoria_producto` | categórica | `Alimentos`, `Farmacia`, `Ropa`, `Electrónica`, `Hogar`. |
| `peso_kg` | numérica (kg) | Peso del paquete. |
| `distancia_km` | numérica (km) | Distancia de la bodega al cliente. |
| `tipo_vehiculo` | categórica | `Moto`, `Bicicleta eléctrica`, `Van`, `Camión pequeño`. |
| `operador` | categórica | Operador logístico (A, B, C, D). |
| `paradas_en_ruta` | entero | Entregas que hizo el vehículo en esa ruta. |
| `lluvia` | categórica | `Sí` si llovió durante la entrega. |
| `tiempo_estimado_min` | numérica (min) | Tiempo prometido al cliente. |
| `tiempo_real_min` | numérica (min) | Tiempo que tardó realmente. **Vacío si la entrega falló.** |
| `retraso_min` | numérica (min) | `tiempo_real_min − tiempo_estimado_min` (negativo = llegó antes). **Vacío si la entrega falló.** |
| `estado_entrega` | categórica | `A tiempo`, `Con retraso` (más de 15 min), `Fallido`, `Devuelto`. |
| `motivo_no_entrega` | categórica | Motivo del fallo o la devolución. **Vacío si el pedido se entregó.** |
| `costo_envio_cop` | numérica (pesos) | Costo del envío. |
| `calificacion_cliente` | numérica (1-5) | Calificación que dejó el cliente (no todos califican). |
| `propina_cop` | numérica (pesos) | Propina dejada al repartidor. |

## Preguntas de análisis

1. ¿Qué operador tiene más retrasos y fallos? ¿Y la peor calificación?
2. ¿Cuánto aumenta el retraso cuando llueve? ¿Afecta igual a todos los vehículos?
3. ¿Cómo cambia el volumen de pedidos y el retraso en Black Friday y diciembre?
4. ¿Qué relación hay entre la distancia, el peso y el costo del envío?
5. ¿El retraso explica la calificación del cliente?

## Calidad de los datos y limpieza

### Faltantes: dos tipos que no se tratan igual

| Tipo de faltante | Columnas | % | Decisión |
|---|---|---|---|
| **"No aplica"** (el vacío tiene significado) | `motivo_no_entrega` 92,49 % · `propina_cop` 77,39 % · `calificacion_cliente` 27,02 % · `tiempo_real_min` 5,10 % · `retraso_min` 5,10 % | — | **No se imputan** |
| **Reales** (el dato se perdió) | `peso_kg` 6,02 % · `canal_compra` 5,44 % · `distancia_km` 2,73 % | — | Se imputan con criterio |

Los vacíos de `motivo_no_entrega` coinciden **exactamente** con los 3.792 pedidos
entregados (1.791 "A tiempo" + 2.001 "Con retraso"): no es un dato perdido, es
"no aplica". Lo mismo pasa con `tiempo_real_min` y `retraso_min`, cuyos 209 vacíos
son los 209 pedidos `Fallido`.

### Tabla de decisiones

| Columna | Problema | % afectado | Decisión | Justificación |
|---|---|---|---|---|
| `id_pedido` | 0 repetidos | 0 % | Conservar | Es la llave de cada pedido. |
| `fecha` | Se leyó como texto (`object`) | 100 % | Convertir a `datetime` | Sin fecha no se puede filtrar por rango (Black Friday, diciembre) ni agrupar por día o mes. |
| `dia_semana` | Coherente con `fecha` | 0 % | Conservar | 0 inconsistencias al recalcular el día desde la fecha. |
| `franja_horaria` | Sin problemas | 0 % | Conservar | Se usa en el análisis. |
| `municipio_destino` | Sin problemas | 0 % | Conservar | Define la distancia y el costo. |
| `canal_compra` | Faltantes reales | 5,44 % (223) | Imputar con la **moda** ("App", 1.746) | Es categórica: la moda es el valor más frecuente y no inventa un número. |
| `categoria_producto` | Sin problemas | 0 % | Conservar | Es el grupo para imputar el peso. |
| `peso_kg` | 247 faltantes + 3 centinelas (`999` y `350` kg) | 6,10 % (250) | Centinelas → `NaN`; imputar con la **mediana por `categoria_producto`** | El peso depende de la categoría (mediana 0,77 kg en Farmacia vs 9,20 kg en Hogar; general 3,16 kg). 350 kg en una moto y 999 kg son imposibles. |
| `distancia_km` | 112 faltantes + 3 imposibles (`0`, `250`, `480` km) | 2,80 % (115) | Imposibles → `NaN`; imputar con la **mediana por `municipio_destino`** | La distancia depende del municipio (6,0 km en Medellín vs 36,85 km en Barbosa; general 8,7 km). El Valle de Aburrá mide ~70 km: 250 y 480 km son imposibles, y 0 km también. |
| `tipo_vehiculo` | Sin problemas | 0 % | Conservar | Explica el efecto de la lluvia. |
| `operador` | Desbalance (A 1.299 vs D 667) | — | Conservar y reportar `count` | El desbalance se controla mostrando cuántos casos hay en cada promedio. |
| `paradas_en_ruta` | Sin problemas | 0 % | Conservar | Aporta al tiempo real. |
| `lluvia` | Sin problemas | 0 % | Conservar | Es el factor del que más depende el retraso. |
| `tiempo_estimado_min` | Sin problemas | 0 % | Conservar | Base del cálculo del retraso. |
| `tiempo_real_min` | 209 vacíos = pedidos fallidos | 5,10 % | **No imputar** | "No aplica": si la entrega falló, no hubo tiempo real. Imputar inventaría un dato. |
| `retraso_min` | 209 vacíos = pedidos fallidos | 5,10 % | **No imputar** | Igual que el anterior: sin tiempo real no hay retraso. |
| `estado_entrega` | Sin problemas | 0 % | Conservar | Es la variable de resultado del negocio. |
| `motivo_no_entrega` | 92,49 % de vacíos | 92,49 % | **No eliminar y no imputar** | Se justifica con datos: los 3.792 vacíos son exactamente las entregas exitosas, y entre las 308 no exitosas hay 7 motivos repartidos (`Cliente ausente` 102, `Dirección incorrecta` 49, `Zona de difícil acceso` 40, `Producto dañado` 35, `Pedido equivocado` 33, `Cliente rechazó` 31, `Vehículo averiado` 18). Es la única fuente que dice **por qué** falla una entrega. |
| `costo_envio_cop` | Sin problemas | 0 % | Conservar | Es la variable de costo a explicar. |
| `calificacion_cliente` | 1.108 vacíos = el cliente no calificó | 27,02 % | **No imputar** | "No aplica": imputar una nota inventaría opiniones. |
| `propina_cop` | 3.173 vacíos = no dejó propina | 77,39 % | **No eliminar y no imputar** | Se justifica: el vacío significa "propina = 0", no un dato perdido; la columna entra en el mapa de calor de correlaciones. |

### Costo de las decisiones

| Columna | Faltantes iniciales | Centinelas → `NaN` | Valores imputados | Estrategia |
|---|---|---|---|---|
| `canal_compra` | 223 | 0 | **223** | Moda |
| `peso_kg` | 247 | 3 | **250** | Mediana por `categoria_producto` |
| `distancia_km` | 112 | 3 | **115** | Mediana por `municipio_destino` |
| **Total** | **582** | **6** | **588** | — |

- **Filas antes:** 4.100 → **filas después:** 4.100 (**0 eliminadas**).
- **Columnas eliminadas:** 0 (las dos con más faltantes se conservan con
  justificación).
- **Valores imputados:** 588 (14,3 % del dataset).
- Se conservan los 243 pedidos `Fallido`/`Devuelto` que **sí** tienen
  `calificacion_cliente`: se documenta como inconsistencia semántica (el cliente
  calificó un intento fallido), pero no se borran porque son evidencia real.

### Otras verificaciones

- **Duplicados:** 0 filas duplicadas y 0 `id_pedido` repetidos.
- **Coherencia fecha ↔ día:** 0 inconsistencias.
- **Coherencia del retraso:** `retraso_min = tiempo_real_min − tiempo_estimado_min`
  en las 3.891 filas con dato, sin excepciones.
- **Coherencia del estado:** `A tiempo` nunca pasa de 15 min de retraso y
  `Con retraso` siempre pasa de 15 min (0 excepciones).
- **Categorías escritas de varias formas:** 0 variantes por espacios o mayúsculas
  en las diez columnas categóricas; no hubo nada que unificar.

## Hallazgos

### 1. ¿Qué operador tiene más retrasos y fallos? ¿Y la peor calificación?

| Operador | Pedidos | Retraso promedio (min) | % con retraso | % fallido/devuelto | Calificación | N.º de calificaciones |
|---|---|---|---|---|---|---|
| A | 1.299 | 18,35 | 44,34 % | 7,78 % | **3,50** | 935 |
| B | 1.107 | 21,61 | 48,33 % | **5,69 %** | 3,46 | 830 |
| C | 1.027 | 20,28 | 46,84 % | 6,82 % | 3,47 | 724 |
| D | 667 | **34,18** | **61,32 %** | **11,09 %** | **2,79** | 503 |

**El operador D es el peor en las cuatro dimensiones**: casi el doble de retraso
que A (34,18 vs 18,35 min), el mayor porcentaje de pedidos con retraso (61,32 %) y
de fallidos/devueltos (11,09 %), y la peor calificación (2,79 frente a ~3,5 de los
otros tres). A es el mejor en tiempo y en calificación; B es el que menos falla.

### 2. ¿Cuánto aumenta el retraso cuando llueve? ¿Afecta igual a todos los vehículos?

| Lluvia | Pedidos | Retraso promedio (min) | % con retraso |
|---|---|---|---|
| No | 2.479 | 10,93 | 30,46 % |
| Sí | 1.621 | **39,78** | **76,87 %** |

La lluvia **casi cuadruplica** el retraso (+28,85 min) y más que duplica el
porcentaje de pedidos con retraso. Afecta a **todos** los vehículos:

| Vehículo | Sin lluvia (min) | Con lluvia (min) | Aumento |
|---|---|---|---|
| Bicicleta eléctrica | 5,58 | 33,66 | +28,08 |
| Moto | 10,34 | 39,05 | +28,71 |
| Van | 16,14 | 45,65 | +29,51 |
| Camión pequeño | 21,92 | 53,44 | +31,52 |

No hay vehículo "inmune": el aumento es de ~28 a ~32 min en todos.

### 3. ¿Cómo cambia el volumen de pedidos y el retraso en Black Friday y diciembre?

| Periodo | Pedidos | Retraso promedio (min) | % con retraso |
|---|---|---|---|
| Sep – 27 nov | 2.720 | 21,31 | 47,83 % |
| Black Friday (28 nov) | 65 | **36,10** | **69,23 %** |
| Diciembre | 1.315 | 23,40 | 49,81 % |

El volumen crece mes a mes: **814** (sep) → **862** (oct) → **1.109** (nov) →
**1.315** (dic), un +62 % de septiembre a diciembre; el día más cargado fue el
**25 de noviembre, con 92 pedidos**. El retraso también sube: 23,40 min en
diciembre frente a 21,31 min del resto. Black Friday tiene el peor retraso
(36,10 min y 69,23 % con retraso), pero con solo 65 pedidos: la cifra es
indicativa, no concluyente.

### 4. ¿Qué relación hay entre la distancia, el peso y el costo del envío?

- Correlación **distancia ↔ costo: 0,83** (fuerte).
- Correlación **peso ↔ costo: 0,48** (moderada).
- Distancia y peso entre sí: ≈ 0 (son ejes independientes del precio).
- Costo de envío promedio: **14.026 COP**.

| Rango de distancia (km) | Pedidos | Costo promedio (COP) |
|---|---|---|
| 0 – 5 | 575 | 9.390 |
| 5 – 10 | 1.873 | 11.553 |
| 10 – 20 | 1.222 | 15.854 |
| 20 – 40 | 375 | 23.807 |
| 40 – 100 | 55 | 39.389 |

La **distancia manda en el costo** (r = 0,83): el costo se multiplica por ~4 entre
el tramo más corto y el más largo. El peso influye, pero menos.

### 5. ¿El retraso explica la calificación del cliente?

| Estado | Pedidos | Calificación promedio | N.º de calificaciones |
|---|---|---|---|
| A tiempo | 1.791 | **4,29** | 1.100 |
| Con retraso | 2.001 | 3,03 | 1.649 |
| Devuelto | 99 | 2,10 | 78 |
| Fallido | 209 | **1,06** | 165 |

| Tramo de retraso (min) | Calificaciones | Promedio |
|---|---|---|
| ≤ −30 | 8 | 4,25 |
| −30 a 0 | 472 | 4,40 |
| 0 a 15 | 658 | 4,11 |
| 15 a 30 | 700 | 3,62 |
| 30 a 60 | 697 | 2,91 |
| > 60 | 292 | **1,71** |

La correlación retraso ↔ calificación es **−0,76**: el retraso **sí explica** en
buena medida la calificación, y el descenso es monótono tramo a tramo. Entre un
pedido puntual (4,40) y uno con más de una hora de retraso (1,71) hay **2,7 puntos**
de diferencia.

## Cómo ejecutar el notebook

1. Abre el notebook `notebooks/analisis.ipynb` del repositorio (botón
   **Open in Colab** del encabezado, o descárgalo y súbelo a
   [colab.research.google.com](https://colab.research.google.com)).
2. Sube el archivo `data/logistica_entregas_valle_aburra_2025.csv` al panel de
   archivos de Colab (icono de carpeta → botón de subir). El notebook lo busca en
   tres ubicaciones relativas, así que funciona si lo dejas en la raíz de Colab.
3. Menú **Entorno de ejecución → Ejecutar todas**. Debe correr de principio a fin
   sin errores y sin necesidad de ejecutar celdas en desorden.
4. Las celdas de texto explican qué hace cada sección y qué dicen los números.

> Si ejecutas el notebook desde tu computador, tenlo en la misma carpeta `data/`
> del repositorio y usa un entorno con `pandas`, `numpy`, `matplotlib` y `seaborn`.
