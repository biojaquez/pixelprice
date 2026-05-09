# PixelPrice por Sakate en .ES

PixelPrice es una app web para analizar el mercado de furnis en Habbo usando histórico de precios y ventas. Te permite buscar un furni por nombre, filtrar por hotel y rango de días, comparar resultados con el mismo nombre, y ver gráficos interactivos + métricas que ayudan a decidir si conviene comprar, esperar o evitar entrar en un precio inflado.

> Interfaz en español. Puedes consultar información de todos los hoteles de Habbo disponibles en el selector (com, de, es, fi, fr, it, nl, br, tr).

---

## ¿Para qué sirve?

PixelPrice sirve para:
- **Ver el histórico de precio promedio (avgPrice)** de un furni en un hotel específico.
- **Ver el volumen de ventas (soldItems)** para medir demanda/liquidez.
- Detectar **inflaciones** (picos anormalmente altos) y **bajadas forzadas** (caídas anormalmente bajas).
- Obtener un **rango recomendado de compra** basado en el comportamiento del período seleccionado.
- Estimar el **riesgo de volatilidad** y el **sesgo** del precio (si el promedio está “jalado” por outliers).
- Exportar el histórico en **CSV** (para Excel / Google Sheets / análisis propio).
- Tener una **predicción heurística** de tendencia a 30 días (SUBE / BAJA / NEUTRO) con probabilidad.

---

## Requisitos

- Un navegador moderno (Chrome, Edge, Firefox).
- Conexión a internet (la app consulta la API).
- El proyecto corre como HTML estático, pero por seguridad del navegador se recomienda abrirlo con un servidor local.

---

## Cómo usar la app (paso a paso)

### 1) Abrir PixelPrice
Abre el archivo `index.html` en un navegador.
- Si estás local, lo ideal es usar un servidor:
  - VS Code: extensión “Live Server”
  - O cualquier servidor estático (por ejemplo, en GitHub Pages / Netlify)

Esto evita problemas típicos de rutas y carga de recursos.

---

### 2) Buscar un furni por nombre
En la parte superior verás los controles:

**Buscar por nombre**
- Escribe el nombre del furni.
- Ejemplos: `Trono`, `Sofá`, `Dragon`, etc.
- Tip: puedes presionar **Enter** para buscar (además del botón).

**Hotel**
- Elige el hotel donde quieres consultar el mercado:
  - `es`, `com`, `br`, `de`, etc.

**Días (aprox)**
- Define cuántos días quieres analizar.
- Por defecto: `60`
- Puedes escribir un número (ej. 30, 90, 180).
- La app interpreta el rango como “los últimos N días” dentro del histórico disponible.

Luego presiona:
- **Buscar**

---

### 3) Elegir un resultado de la lista “Resultados”
Muchos furnis pueden compartir nombre o tener variantes. Por eso, después de buscar, PixelPrice llena el selector:

**Resultados**
- Aquí se listan coincidencias filtradas (se excluyen tokens tipo NFT/collectible/web3).
- Cada opción incluye datos útiles:
  - `FurniName`
  - `classname=...`
  - `type=...`
  - `line=...`

Selecciona el furni correcto (si hay varios).

---

### 4) Cargar el furni seleccionado
Haz clic en:
- ** Cargar seleccionado**

Esto hace que la app:
1) Lea el `history` del furni
2) Limpie y ordene los datos por fecha
3) Aplique el filtro de días si corresponde
4) Calcule estadísticas
5) Dibuje los gráficos
6) Genere la predicción
7) Intente cargar la imagen por `classname`

---

### 5) Interpretar lo que aparece en el panel izquierdo

#### A) Imagen del furni
- Se intenta descargar desde el endpoint de imágenes con el `classname`.
- Si la API responde `202` significa “imagen en proceso”; reintenta después.

#### B) Información
Muestra:
- Nombre del furni
- Classname
- Tipo
- Hotel
- Rango mostrado (inicio a fin del período)
- Última actualización (según API)

Botón:
- **📋 Ver datos en tabla**: abre una tabla paginada con los datos exactos usados para graficar.

#### C) Estadísticas de precio (créditos)
Incluye:
- **Precio mínimo / máximo**: extremos del rango.
- **Precio promedio**: media (sensible a picos).
- **Mediana**: valor central (más robusto).
- **Percentil 95**: techo “típico” sin irse al máximo absoluto.
- **Último precio (serie)**: el punto más reciente del rango.
- **Precio justo**: rango recomendado basado en:
  - mínimo del período
  - y como techo: el menor entre (promedio, mediana)
- **Número de inflaciones**: puntos donde el z-score > 1.96
- **Número de bajadas forzadas**: z-score < -1.96
- **Riesgo de volatilidad (%)**: (std / promedio) × 100
- **Sesgo (media - mediana)**:
  - positivo: promedio “jalado” hacia arriba (inflaciones)
  - negativo: promedio “jalado” hacia abajo (bajadas)
- **Índice de inflación / bajadas (%)**: inflaciones o bajadas dividido entre total de puntos del rango

#### D) Estadísticas de volumen de ventas
Incluye:
- **Ventas totales**
- **Ventas promedio por punto**
- **Máximo en un punto**
- **Puntos en la serie**
- **Créditos totales (creditSum)**
- **Liquidez (ventas por día)**:
  - ventas totales / número de días del rango
- **Presión vendedora (ofertas/ventas)**:
  - openOffers promedio / soldItems promedio
  - Si las ventas promedio son 0, se muestra ∞

#### E) Predicción (próximos 30 días)
Actualmente muestra el modelo:
- **Heurístico**
- Etiqueta:
  - **SUBE** (verde)
  - **BAJA** (rojo)
  - **NEUTRO** (azul)
- Probabilidad (50% a 95%)

La predicción se basa en:
- retorno vs hace ~30 días
- pendiente (tendencia) de los últimos ~60 puntos
- volatilidad reciente para definir una banda neutral

---

## Gráficos (panel derecho)

### 1) Precio promedio (avgPrice)
- Línea principal = precio promedio por fecha.
- Marcadores rojos = inflaciones (outliers altos).
- Marcadores azules = bajadas forzadas (outliers bajos).
- Rectángulo verde = zona sugerida de compra (entre mínimo y el menor de (media, mediana)).

Interacciones:
- Puedes hacer zoom/drag como cualquier gráfico Plotly.
- La app mantiene el rango dentro del histórico (clamp del eje X) para evitar perderte fuera del rango real.

### 2) Volumen de ventas (soldItems)
- Barras = ventas por fecha.
- Puntos = refuerzo visual para el conteo.

---

## Tabla y exportación CSV

### Abrir tabla
Botón:
- ** Ver datos en tabla**

Dentro verás:
- Fecha
- Precio (avgPrice)
- Ventas (soldItems)
- Créditos (creditSum)
- OpenOffers

Incluye paginación:
- ⟵ Anterior / Siguiente ⟶
- Cambiar página con número.

### Descargar CSV
En la tabla:
- **⬇️ Descargar CSV**

Genera un archivo con nombre:
`habbo_<classname>_<hotel>_<dias>d.csv`

Incluye encabezados:
`date,avgPrice,soldItems,creditSum,openOffers`

---

## Botón “♻️ Limpiar”
Este botón reinicia TODO:
- borra resultados y selección
- limpia cards (valores “—”)
- borra imagen
- reinicia gráficos vacíos
- desactiva botones dependientes (como “Ver datos en tabla”)

Útil si quieres comparar varios furnis sin recargar la página.

---

## Notas importantes / Problemas comunes

- **Sin resultados**: el nombre puede variar (acentos, plural, etc.). Prueba con otra forma.
- **Sin histórico**: algunos resultados pueden venir sin `history` por parte de la API.
- **Imagen 202**: la imagen está “en proceso”. Espera un poco y vuelve a cargar.
- **Días muy altos**: si pides más días de los que existen, verás el máximo disponible.
- **Datos raros/picos**: el mercado puede estar manipulado o tener pocas transacciones. Revisa liquidez y presión vendedora.

---

## Disclaimer
PixelPrice muestra estadísticas basadas en datos públicos del mercado y reglas heurísticas. No es consejo financiero; úsalo como herramienta de apoyo para comparar tendencias y riesgo.

---

## Créditos
- UI y lógica: biojaquez (Sakate in .ES)
- Gráficos: Plotly
- Datos e imágenes: Habbo API (según el endpoint configurado en la app)
