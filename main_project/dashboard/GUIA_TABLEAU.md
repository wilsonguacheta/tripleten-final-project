# Guía paso a paso — Dashboard de CallMeMaybe en Tableau Public

Esta guía construye el dashboard que pide la consigna del caso principal, con las dos sugerencias combinadas en un único panel. Está escrita para seguirse de principio a fin sin conocimientos previos de Tableau.

**Tiempo estimado:** 45–60 minutos.

**Necesitas:**
- [Tableau Public Desktop](https://public.tableau.com/app/discover) instalado (es gratuito).
- Una cuenta en Tableau Public (se crea gratis desde la misma web).
- El archivo `tableau_extract.csv`, que está en esta misma carpeta.

> **Aviso importante:** al publicar en Tableau Public, **los datos quedan visibles para cualquiera**. Es lo que la consigna pide, pero conviene saberlo. No subas nunca datos confidenciales a Tableau Public.

---

## Qué contiene el archivo de datos

`tableau_extract.csv` tiene 49 002 filas y 13 columnas, ya limpio y con las métricas derivadas calculadas. Cada fila es un registro diario agregado de llamadas.

| Campo | Tipo | Contenido |
|---|---|---|
| `Fecha` | Fecha | Día de la actividad (2019-08-02 a 2019-11-28) |
| `Cliente` | Número entero | Identificador de la organización cliente |
| `Operador` | Número entero | Identificador del operador — **vacío en las llamadas entrantes perdidas** |
| `Plan tarifario` | Texto | A, B o C |
| `Direccion` | Texto | Entrante / Saliente |
| `Tipo de llamada` | Texto | Interna / Externa / Sin dato |
| `Estado` | Texto | Contestada / Perdida |
| `Numero de llamadas` | Número entero | Llamadas agregadas en ese registro |
| `Duracion total (s)` | Número entero | Segundos de conversación |
| `Duracion con espera (s)` | Número entero | Conversación más espera |
| `Espera total (s)` | Número entero | Solo la espera |
| `Duracion por llamada (s)` | Decimal | Duración media por llamada del registro |
| `Espera por llamada (s)` | Decimal | Espera media por llamada del registro |

> **Clave para no equivocarse:** el archivo está **agregado**, no tiene una fila por llamada. Para contar llamadas hay que usar **SUMA de `Numero de llamadas`**, nunca "Número de registros". Este es el error más habitual y falsea todas las cifras.

---

## Paso 1 — Conectar el archivo

1. Abre Tableau Public Desktop.
2. En el panel izquierdo, bajo **Conectar → A un archivo**, pulsa **Archivo de texto**.
3. Selecciona `tableau_extract.csv` de esta carpeta.
4. Tableau muestra la vista de origen de datos con una previsualización.

### Verificar los tipos de campo

En la previsualización, cada columna tiene un icono con su tipo. Compruébalos y corrige los que no coincidan (clic en el icono → elegir el tipo correcto):

- `Fecha` debe ser **Fecha** (icono de calendario). Si aparece como texto (`Abc`), cámbialo.
- `Cliente` y `Operador` deben ser **Número (entero)**.
- `Plan tarifario`, `Direccion`, `Tipo de llamada` y `Estado` deben ser **Texto**.

### Convertir los identificadores en dimensiones

Tableau interpreta `Cliente` y `Operador` como cifras que se pueden sumar, lo cual no tiene sentido. Ve a una hoja nueva (pestaña **Hoja 1** abajo) y, en el panel de datos:

- Arrastra **Cliente** desde *Medidas* hasta *Dimensiones*.
- Haz lo mismo con **Operador**.

---

## Paso 2 — Hoja 1: histograma de duración de llamada

*Corresponde a la sugerencia 1.1 de la consigna.*

1. Renombra la hoja actual: doble clic en la pestaña **Hoja 1** → escribe `Duración de llamada`.
2. En el panel de datos, clic derecho sobre **Duracion por llamada (s)** → **Crear → Grupos de bins…**
3. En el diálogo:
   - Nombre: `Duración por llamada (bins)`
   - Tamaño del bin: **15** (agrupa en tramos de 15 segundos)
   - Pulsa **Aceptar**.
4. Arrastra **Duración por llamada (bins)** a **Columnas**.
5. Arrastra **Numero de llamadas** a **Filas**. Comprueba que aparece como `SUMA(Numero de llamadas)`; si no, clic derecho sobre la píldora → **Medida → Suma**.
6. La distribución tiene una cola muy larga que aplasta el gráfico. Para limitarla:
   - Arrastra **Duracion por llamada (s)** al estante **Filtros**.
   - Elige **Todos los valores** → **Siguiente** → selecciona **Rango de valores** y fija el máximo en **600** segundos.
   - Pulsa **Aceptar**.
7. Añade un filtro de contexto útil: arrastra **Estado** a **Filtros**, marca solo **Contestada** y acepta. Una llamada perdida no tiene duración real, así que incluirlas distorsiona el histograma.
8. Da formato:
   - Clic en el título del gráfico → escribe `Distribución de la duración de las llamadas contestadas`.
   - Clic derecho en el eje horizontal → **Editar eje** → título: `Segundos por llamada`.
   - Clic derecho en el eje vertical → **Editar eje** → título: `Llamadas`.

**Resultado esperado:** una distribución muy asimétrica, con la mayoría de las llamadas por debajo de los 150 segundos y una cola larga hacia la derecha.

---

## Paso 3 — Hoja 2: gráfico circular de internas vs. externas

*Corresponde a las sugerencias 1.2 y 2.2.*

1. Crea una hoja nueva (icono de hoja con `+` en la barra inferior) y llámala `Internas vs externas`.
2. En la tarjeta **Marcas**, despliega el menú de tipo de marca (dice *Automático*) y elige **Círculo**.
3. Arrastra **Tipo de llamada** al botón **Color** de la tarjeta Marcas.
4. Arrastra **Numero de llamadas** al botón **Ángulo**. Verifica que sea `SUMA`.
5. Excluye la categoría sin información: en la leyenda de color, clic derecho sobre **Sin dato** → **Excluir**.
6. Muestra las cifras:
   - Arrastra **Numero de llamadas** a **Etiqueta**.
   - Clic en **Etiqueta** → marca **Mostrar etiquetas de marca**.
   - Para verlas en porcentaje: clic derecho sobre la píldora `SUMA(Numero de llamadas)` que está en Etiqueta → **Cálculo rápido de tabla → Porcentaje del total**.
7. Agranda el gráfico: en la barra superior, cambia el desplegable de tamaño de *Estándar* a **Ajustar → Ajustar toda la vista**.
8. Título: `Reparto entre llamadas internas y externas`.

**Resultado esperado:** las llamadas externas dominan claramente sobre las internas.

---

## Paso 4 — Hoja 3: histograma de llamadas por día

*Corresponde a la sugerencia 2.1.*

1. Nueva hoja, nómbrala `Llamadas por día`.
2. Arrastra **Fecha** a **Columnas**. Aparecerá como `AÑO(Fecha)`.
3. Clic derecho sobre esa píldora → elige **Día** en el **segundo bloque** del menú (el que muestra fechas completas como `2 de agosto de 2019`, no el que dice simplemente `Día`). Así se obtiene la fecha exacta y no el día del mes agregado.
4. Arrastra **Numero de llamadas** a **Filas** (como `SUMA`).
5. En la tarjeta **Marcas**, elige el tipo **Barra**.
6. Distingue entrantes de salientes: arrastra **Direccion** al botón **Color**.
7. Título: `Volumen diario de llamadas`.
8. Eje vertical: clic derecho → **Editar eje** → título `Llamadas`.

**Resultado esperado:** una tendencia creciente a lo largo de los cuatro meses, con un patrón semanal visible (caídas los fines de semana).

---

## Paso 5 — Hoja 4: tasa de llamadas perdidas por plan

*No está en las sugerencias de la consigna, pero es el hallazgo principal del análisis y da valor real al dashboard.*

1. Nueva hoja, nómbrala `Pérdida por plan`.
2. Crea un campo calculado: menú **Análisis → Crear campo calculado…**
   - Nombre: `Tasa de pérdida`
   - Fórmula (cópiala tal cual):
     ```
     SUM(IF [Estado] = 'Perdida' THEN [Numero de llamadas] END)
     / SUM([Numero de llamadas])
     ```
   - Pulsa **Aceptar**. Debe aparecer el mensaje *El cálculo es válido*.
3. Arrastra **Plan tarifario** a **Columnas**.
4. Arrastra **Tasa de pérdida** a **Filas**.
5. Filtra a las entrantes, que son las únicas donde "perdida" indica un fallo del servicio: arrastra **Direccion** a **Filtros** y marca solo **Entrante**.
6. Formatea como porcentaje: clic derecho sobre la píldora `Tasa de pérdida` en Filas → **Formato…** → en el panel izquierdo, apartado **Números** → **Porcentaje** con 1 decimal.
7. Arrastra **Tasa de pérdida** a **Etiqueta** para que la cifra se vea sobre cada barra.
8. Título: `Llamadas entrantes perdidas según plan tarifario`.

**Resultado esperado:** el plan B destaca con una tasa de pérdida claramente superior a la de A y C.

---

## Paso 6 — Montar el dashboard

1. En la barra inferior, pulsa el icono **Nuevo dashboard** (el del centro, con dos rectángulos).
2. En el panel izquierdo, apartado **Tamaño**, elige **Automático**. Así se adapta a la pantalla de quien lo consulte.
3. Arrastra las hojas desde el panel izquierdo al lienzo, en este orden:
   - `Llamadas por día` → arriba, ocupando todo el ancho.
   - `Duración de llamada` → abajo a la izquierda.
   - `Internas vs externas` → abajo en el centro.
   - `Pérdida por plan` → abajo a la derecha.
4. Añade un título general: activa la casilla **Mostrar título del dashboard** (abajo a la izquierda), luego doble clic sobre el título y escribe:
   `CallMeMaybe — Actividad del servicio de telefonía (ago–nov 2019)`

---

## Paso 7 — Añadir los filtros interactivos

La consigna pide un filtro de dirección de llamada y otro de tipo (interna/externa).

1. Selecciona en el lienzo la hoja `Llamadas por día` (clic sobre ella; aparecerá un borde gris).
2. En su esquina superior derecha aparece una pequeña flecha desplegable → **Filtros → Direccion**.
3. Repite la operación para **Tipo de llamada**.
4. Ahora hay que hacer que esos filtros afecten a **todas** las hojas, no solo a una:
   - Sobre el filtro de **Direccion** que apareció en el lienzo, pulsa su flecha desplegable → **Aplicar a hojas de trabajo → Todos los que usan esta fuente de datos**.
   - Repite con **Tipo de llamada**.
5. Cambia el formato de los filtros para que sean cómodos: flecha desplegable de cada filtro → **Lista de valores únicos** (permite marcar y desmarcar).
6. Coloca ambos filtros a la derecha del dashboard, uno debajo del otro.

> **Cuidado con una interacción:** el filtro de Dirección afecta también a la hoja `Pérdida por plan`, que ya tiene su propio filtro fijado en *Entrante*. Si desmarcas *Entrante* en el filtro global, esa hoja se quedará vacía. Es el comportamiento correcto, pero conviene saberlo antes de que parezca un fallo.

---

## Paso 8 — Revisar antes de publicar

Comprueba estos puntos, que son los tres criterios que la consigna evalúa (`.ai/docs/dashboard.md`):

**¿Resuelve la tarea?**
- [ ] Están el histograma de duración, el circular de internas/externas y el histograma de llamadas por día.
- [ ] Los dos filtros pedidos funcionan y afectan a todas las hojas.

**¿Aporta variedad de información?**
- [ ] Se ve el volumen en el tiempo, la composición del tráfico, la distribución de duraciones y la calidad del servicio por plan.

**¿Es fácil de usar?**
- [ ] Todos los gráficos tienen título descriptivo y ejes etiquetados.
- [ ] No hay barras de desplazamiento internas ni textos cortados.
- [ ] Los filtros están agrupados en un mismo lugar.
- [ ] Prueba a mover un filtro y confirma que todos los gráficos reaccionan.

---

## Paso 9 — Publicar en Tableau Public

1. Menú **Archivo → Guardar en Tableau Public como…**
2. Inicia sesión con tu cuenta de Tableau Public.
3. Nombre del libro de trabajo: `CallMeMaybe - Analisis de operadores`.
4. Pulsa **Guardar**. Tableau sube el trabajo y abre el navegador con la vista publicada.
5. **Copia la URL** de la barra de direcciones del navegador.
6. Pega esa URL en el archivo `dashboard.txt` de esta carpeta, y añade una línea con la fecha de publicación.

### Comprobación final

Abre la URL en una ventana de incógnito (sin sesión iniciada) y verifica que el dashboard carga y que los filtros funcionan. Si no carga, es que el libro se guardó como privado: entra en tu perfil de Tableau Public → pestaña de trabajos → ajustes del libro → marca la visibilidad como pública.

---

## Problemas frecuentes

| Síntoma | Causa y solución |
|---|---|
| Las cifras de llamadas parecen demasiado bajas | Estás usando *Número de registros* en lugar de `SUMA(Numero de llamadas)`. El archivo está agregado |
| El histograma sale como una sola barra gigante | Falta el filtro de rango sobre `Duracion por llamada (s)`; la cola larga aplasta la escala |
| `Fecha` aparece como texto | Cámbiala a tipo Fecha en la vista de origen de datos y refresca la extracción |
| El circular sale como un único color | `Tipo de llamada` no está en **Color**, o el tipo de marca no es *Círculo* |
| El campo calculado da error | Revisa que los nombres entre corchetes coincidan exactamente, incluidos los espacios: `[Numero de llamadas]`, `[Estado]` |
| Un filtro solo afecta a un gráfico | Falta aplicarlo con **Aplicar a hojas de trabajo → Todos los que usan esta fuente de datos** |
| El operador aparece como `880022,0` | El extracto ya lo exporta como entero; si lo ves con decimales, cambia el tipo del campo a *Número (entero)* |

---

## Si quieres ir más allá

Estas adiciones no las pide la consigna, pero refuerzan el dashboard:

- **Tarjetas de indicadores** arriba del todo: total de llamadas, tasa de pérdida global y espera media. Se hacen con una hoja por indicador, arrastrando la medida a **Texto** y quitando los encabezados.
- **Filtro por cliente** para que cada supervisor vea solo su organización, que es el caso de uso real de la función.
- **Mapa de calor por día de la semana y semana**, útil para detectar en qué franjas se concentra la pérdida de llamadas.
