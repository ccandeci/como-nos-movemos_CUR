# Trivia · Semana de la Movilidad Sostenible (CUR)

Trivia interactiva de una sola página (single-file HTML) sobre movilidad sostenible y los resultados del relevamiento de movilidad del CUR (Ciudad Universitaria Rosario), realizado por el Instituto de Estudios de Transporte (IET) el 20/05/2026.

Archivo principal: **`como-nos-movemos_CUR.html`**

No requiere build ni dependencias: es un único archivo HTML con CSS y JavaScript embebidos (vanilla JS, sin frameworks). Se puede abrir directamente en el navegador o subir a cualquier hosting estático.

## Cómo funciona

1. **Registro**: la persona ingresa su nombre y selecciona su facultad.
2. **Trivia**: se arma un cuestionario de 6 preguntas por partida:
   - 2 preguntas elegidas al azar del **bloque 1** (conceptos generales de movilidad sostenible).
   - 4 preguntas elegidas al azar del **bloque 2** (datos concretos del relevamiento del CUR).
   - En cada pregunta, las opciones de respuesta también se mezclan al azar.
3. **Feedback por pregunta**: al responder, se marca si fue correcta o incorrecta y, si la pregunta tiene un dato asociado (`dato`), se muestra un cartel "¿SABÍAS QUE...?" con información adicional.
4. **Pantalla final**: muestra el puntaje, un resumen de todas las respuestas, y envía los resultados a una planilla de Google Sheets.

## Estructura de los datos (`QUIZ_DATA`)

Todas las preguntas viven en el objeto `QUIZ_DATA`, dividido en `block1` y `block2`. Cada pregunta tiene esta forma:

```js
{
  "q": "Texto de la pregunta",
  "options": ["Opción A", "Opción B", "Opción C"],
  "correct": 0,          // índice (en options) de la respuesta correcta
  "dato": "Dato curioso que se muestra después de responder (o null si no hay)"
}
```

### Agregar una imagen a una pregunta

Cualquier pregunta puede llevar una imagen opcional agregando el campo `img` con un [data URI](https://developer.mozilla.org/es/docs/Web/URI/Schemes/data) en base64. Por ejemplo, la pregunta 3 del bloque 1 (`QUIZ_DATA.block1[2]`) tiene su imagen asignada así, justo antes de `function shuffle(arr){`:

```js
const Q3_BLOCK1_IMG = "data:image/jpeg;base64,....";
QUIZ_DATA.block1[2].img = Q3_BLOCK1_IMG;
```

Para agregar una imagen a otra pregunta:

1. Convertí la imagen a base64 (por ejemplo con `base64 -i imagen.jpg` en la terminal, o cualquier conversor online).
2. Agregá una constante con el data URI (`data:image/jpeg;base64,...` o `data:image/png;base64,...`).
3. Asignala al campo `img` de la pregunta correspondiente dentro de `QUIZ_DATA.block1[i]` o `QUIZ_DATA.block2[i]`.

> Recomendación: comprimí y reducí el ancho de la imagen (~900px) antes de convertirla a base64, para no inflar demasiado el tamaño del HTML.

## Registro de resultados en Google Sheets

Al terminar la trivia, se envían los resultados (nombre, facultad, puntaje, total y detalle de respuestas) a un Google Apps Script Web App mediante `fetch`, definido en:

```js
const GOOGLE_SHEET_ENDPOINT = "https://script.google.com/macros/s/AKfycby.../exec";
```

El envío se hace en modo `no-cors`. Esto es intencional: Google Apps Script no siempre devuelve los headers CORS correctos en la respuesta (por la redirección interna que hace internamente), lo que puede hacer que el navegador no pueda *leer* la respuesta aunque el dato sí se haya guardado correctamente en la planilla. Con `no-cors` no se puede leer la respuesta, pero evitamos que eso se reporte como un error falso al usuario: si el `fetch` no lanza una excepción, se considera que el envío fue exitoso.

Si en algún momento se cambia el endpoint (por ejemplo, para apuntar a una nueva implementación de Apps Script), solo hay que reemplazar el valor de `GOOGLE_SHEET_ENDPOINT`.

## Personalización visual

Los colores institucionales del IET están centralizados como variables CSS en `:root` (escala de teal, `--c100` a `--c10`). Los logos (IET y UNR) están embebidos como base64 en las constantes `IET_LOGO` y `UNR_LOGO`.

## Deploy

Al ser un único archivo HTML autocontenido, se puede:

- Abrir directamente en el navegador (doble clic).
- Subir a GitHub Pages, Netlify, Vercel, o cualquier hosting estático.
- Compartir el link una vez publicado (por ejemplo, para usarse desde el celular durante la Semana de la Movilidad Sostenible).

## Historial de cambios relevantes

- Se agregó soporte para imágenes opcionales por pregunta (`item.img`).
- Se corrigió el envío a Google Sheets para evitar el falso mensaje "No pudimos registrar tus respuestas" cuando el dato sí se había guardado (cambio a modo `no-cors`).
