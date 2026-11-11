# Evidencias — Recetario UTEZ

**Nombre:** Pedro Fabian Ocampo Ramirez
**Matrícula:** 20233tn148
**Fecha:** 10 noviembre 2026
**Dispositivo donde se probó:** (Chrome)

## 1. Del JSON a Dart

| JSON | Campo Dart | Tipo | Regla (obligatorio, tolerado, valor por defecto) |
|---|---|---|---|
| `id` | `Receta.id` | `int` | Obligatorio; si falta o no es entero → `FormatException` |
| `name` | `Receta.nombre` | `String` | Obligatorio |
| `cuisine` | `Receta.cocina` | `String` | Obligatorio |
| `difficulty` | `Receta.dificultad` | `Dificultad` | Tolerado: `Easy/Medium/Hard` se traducen; cualquier otra cosa, `null` o ausente → `Dificultad.desconocida` |
| `prepTimeMinutes` | `Receta.minutosPreparacion` | `int` | Obligatorio |
| `cookTimeMinutes` | `Receta.minutosCoccion` | `int` | Obligatorio |
| `servings` | `Receta.porciones` | `int` | Obligatorio |
| `rating` | `Receta.calificacion` | `double` | Obligatorio; llega como `num` (entero o decimal) y se guarda con `.toDouble()` |
| `tags` | `Receta.etiquetas` | `List<String>` | Opcional → lista vacía; si no es lista de textos → `FormatException` |
| `ingredients` | `Receta.ingredientes` | `List<String>` | Opcional → lista vacía; si no es lista de textos → `FormatException` |
| `recipes` | `PaginaRecetas.recetas` | `List<Receta>` | Obligatorio (puede ser lista vacía); cada elemento debe ser un objeto |
| `total` | `PaginaRecetas.total` | `int` | Obligatorio |

Los demás campos del JSON (`instructions`, `image`, `userId`, etc.) se ignoran.

## 2. Escenarios probados con el servicio real

> Completa cada fila con lo que viste en TU ejecución: copia la línea real de la
> bitácora (consola de `flutter run`, filtrando por `recetario`) y guarda la captura
> en `docs/capturas/`.

| Escenario | Cómo lo provoqué | Qué vio la persona | Línea de la bitácora | Captura | Prueba del kit que lo cubre |
|---|---|---|---|---|---|
| Éxito | Busqué `chicken` | Encabezado "Mostrando N de M recetas · Rápidas: K" y tarjetas | (pega la línea `← 200 GET /recipes/search (… ms)`) | `capturas/01_exito.png` | etapa3: "200 → PaginaRecetas"; etapa5: "texto + botón → … → tarjetas" |
| Vacío | Busqué `zzzz` | `No encontramos recetas para "zzzz".` (no es un error) | (pega la línea `← 200 GET /recipes/search (… ms)`) | `capturas/02_vacio.png` | etapa3: "200 sin resultados → página vacía"; etapa5: "vacío con texto dice qué se buscó…" |
| Error del servidor | Cambio temporal A (`/http/503`) | Mensaje "El recetario no está disponible (503). Intenta más tarde." y botón Reintentar | (pega la línea `✗ badResponse 503 GET /http/503 (… ms)`) | `capturas/03_servidor.png` | etapa3: "503 → ErrorDelServidor(503)"; etapa4: "FallaRed → Fallidas…" |
| Sin red | Modo avión (o cambio temporal C) | "Sin conexión. Revisa tu red e intenta de nuevo." y botón Reintentar | (pega la línea `✗ connectionError GET /recipes (… ms)`) | `capturas/04_sin_red.png` | etapa3: "sin red → SinConexion"; etapa5: "error → mensaje y Reintentar…" |
| Tiempo agotado | Cambio temporal B (`delay=5000`, timeouts de 2 s) | "El recetario tardó demasiado en responder. Intenta de nuevo." | (pega la línea `✗ receiveTimeout GET /recipes (… ms)`) | `capturas/05_tiempo.png` | etapa3: "connectTimeout / receiveTimeout → TiempoAgotado" |

## 3. Preguntas de reflexión

1. **1. ¿En qué se diferencian "Error del servidor" y "Sin red" desde el punto de vista de HTTP?**

Cuando ocurre un error del servidor, significa que la solicitud sí llegó y el servidor respondió, pero presentó algún problema, por ejemplo, un código 503. En cambio, cuando no hay conexión, la solicitud no logra llegar al servidor y no se recibe ninguna respuesta HTTP. Por eso cada situación necesita un manejo distinto: una indica que se intente nuevamente después y la otra que se compruebe la conexión a Internet.

**2. ¿Por qué "Vacío" no es un error?**

Porque el sistema respondió correctamente, aunque no encontró recetas para mostrar. Por ejemplo, puede devolver un código 200 con una lista vacía. Esto no significa que algo haya fallado, sino que no existen resultados para esa búsqueda. Por esa razón se maneja como un estado independiente llamado `RecetasVacias`, en lugar de tratarlo como un problema de conexión.

**3. La búsqueda de la persona viaja en la query. ¿Por qué tu bitácora no la escribe?**

Porque el texto que se introduce en una búsqueda podría contener información privada. Si se guarda en los registros, existe el riesgo de que termine expuesto al compartirlos o revisarlos. Por seguridad, la bitácora únicamente almacena datos técnicos como el método HTTP, la ruta, el código de respuesta y el tiempo de ejecución, evitando guardar el contenido de la búsqueda.

**4. Si DummyJSON cambiara `rating` por `score`, ¿qué archivos tendría que modificar y por qué solo esos?**

El cambio principal se realizaría en `lib/datos/recetas_json.dart`, ya que ahí se interpreta la información que llega desde la API. Bastaría con ajustar el nombre de la propiedad para que coincida con `score`. Las demás partes del programa seguirían funcionando porque utilizan los atributos definidos en el modelo `Receta` y no dependen directamente de los nombres originales del JSON. También sería necesario actualizar los datos simulados en `test/soporte/json_dummy.dart`.
