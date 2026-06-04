# CÓDIGO COMPLETO — Ready To Cook

## Estructura del proyecto

```
Ready-To-Cook/
├── src/
│   ├── components/
│   │   ├── Auth/           Login.js, Register.js, Recovery.js
│   │   ├── Dishes/         History.js, PendingDishes.js
│   │   ├── Ingredients/    Inventory.js, RegisterIngredient.js
│   │   ├── Main/           MainMenu.js
│   │   └── Recipes/        GenerateRecipe.js, RecipeDetail.js, RecipeResults.js
│   ├── services/           firebase.js, foodDatabase.js, openaiService.js
│   ├── utils/              dateCalculations.js, Modal.js, recipeHelpers.js
│   ├── App.js
│   ├── index.js
│   └── index.css
├── Api/index.js            Proxy Express (desarrollo local)
├── api/openai.js           Función Serverless Vercel (producción)
└── package.json
```

---

## src/index.js

```js
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ minHeight: '100vh', display: 'flex', alignItems: 'center',
          justifyContent: 'center', flexDirection: 'column', gap: '16px' }}>
          <h2 style={{ color: '#333' }}>Algo salió mal</h2>
          <button onClick={() => window.location.reload()}>Recargar</button>
        </div>
      );
    }
    return this.props.children;
  }
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <ErrorBoundary>
      <App />
    </ErrorBoundary>
  </React.StrictMode>
);

reportWebVitals();
```

- `import React from 'react'` — importa la librería principal de React. Aunque en versiones modernas de React no siempre es obligatorio, es necesario aquí porque `ErrorBoundary` es un componente de clase que extiende `React.Component`.
- `import ReactDOM from 'react-dom/client'` — importa el módulo que se encarga de conectar el árbol de componentes de React con el DOM real del navegador. Se usa la versión `/client` que corresponde a React 18 y su nueva API `createRoot`.
- `import './index.css'` — importa el archivo de estilos globales. Al importarlo aquí, desde el punto de entrada de la app, los estilos aplican a todos los componentes sin necesidad de importarlos individualmente.
- `import App from './App'` — importa el componente raíz de la aplicación, que contiene toda la lógica de navegación y estado global.
- `import reportWebVitals from './reportWebVitals'` — importa una función de medición de rendimiento opcional que puede reportar métricas como CLS (Cumulative Layout Shift) y FCP (First Contentful Paint).
- `class ErrorBoundary extends React.Component` — declara un componente de clase que actúa como "límite de error". Los componentes de clase son los únicos que pueden usar el ciclo de vida `getDerivedStateFromError`, por eso se usa clase aquí en lugar de función.
- `state = { hasError: false }` — define el estado inicial del componente. `hasError` comienza en `false` porque al arrancar la app no hay ningún error; solo cambia a `true` cuando un componente hijo lanza una excepción durante el render.
- `static getDerivedStateFromError()` — método estático especial de React que se ejecuta automáticamente cuando cualquier componente descendiente lanza un error durante el renderizado. Al ser estático, no tiene acceso a `this`. Retorna `{ hasError: true }` para indicarle a React que actualice el estado y muestre la pantalla de error.
- `if (this.state.hasError)` — comprueba si el estado registró algún error. Si es `true`, en lugar de renderizar los hijos normales, muestra la pantalla de fallback con un botón para recargar.
- `<div style={{ minHeight: '100vh', display: 'flex', alignItems: 'center', justifyContent: 'center', flexDirection: 'column', gap: '16px' }}>` — contenedor centrado que ocupa toda la altura de la pantalla. Los estilos en línea se usan aquí intencionalmente porque en un error grave los estilos de CSS externos (como Tailwind) podrían no haberse cargado correctamente.
- `<button onClick={() => window.location.reload()}>` — botón que recarga la página completa del navegador cuando el usuario lo presiona, intentando recuperar la app del estado de error.
- `return this.props.children` — si no hay ningún error, el componente se vuelve "transparente" y simplemente renderiza todo lo que se le pase como hijos (en este caso, `<App />`).
- `ReactDOM.createRoot(document.getElementById('root'))` — encuentra el elemento `<div id="root">` en el archivo `public/index.html` y lo convierte en el punto de montaje de React. `createRoot` es la API de React 18 que habilita el renderizado concurrente.
- `root.render(...)` — toma el árbol de JSX que se le pasa y lo renderiza dentro del nodo raíz del DOM. A partir de aquí React toma el control del contenido de ese `div`.
- `<React.StrictMode>` — un componente especial de React que solo activa en modo desarrollo. Ejecuta los efectos y renders dos veces a propósito para detectar efectos secundarios inesperados. No afecta el comportamiento en producción.
- `<ErrorBoundary>` — envuelve toda la app con el límite de error. Si cualquier componente dentro de la aplicación lanza una excepción no capturada durante el render, `ErrorBoundary` la intercepta y muestra la pantalla de fallback en lugar de que la app se quede en blanco.
- `<App />` — el componente raíz que contiene toda la lógica de la aplicación: navegación, autenticación y estado compartido.
- `reportWebVitals()` — llama a la función de medición de métricas al final, después de que la app ya está montada. Puede configurarse para enviar datos a servicios de análisis.

---

## src/services/firebase.js

```js
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: process.env.REACT_APP_FIREBASE_API_KEY,
  authDomain: process.env.REACT_APP_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.REACT_APP_FIREBASE_PROJECT_ID,
  storageBucket: process.env.REACT_APP_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.REACT_APP_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.REACT_APP_FIREBASE_APP_ID
};

const app = initializeApp(firebaseConfig);

export const auth = getAuth(app);
export const db = getFirestore(app);

export default app;
```

- `import { initializeApp } from 'firebase/app'` — importa la función principal de Firebase que establece la conexión con el proyecto en la nube. Sin llamar a esta función primero, ningún otro módulo de Firebase funcionará.
- `import { getAuth } from 'firebase/auth'` — importa el módulo de autenticación de Firebase. Este módulo maneja todo lo relacionado con usuarios: crear cuentas, iniciar sesión, cerrar sesión, escuchar cambios de sesión y enviar correos de recuperación.
- `import { getFirestore } from 'firebase/firestore'` — importa el módulo de Firestore, la base de datos NoSQL en tiempo real de Firebase. Permite leer, escribir, actualizar y eliminar documentos organizados en colecciones.
- `const firebaseConfig = { ... }` — objeto de configuración que contiene las credenciales únicas del proyecto de Firebase. Estas credenciales identifican a qué proyecto de Firebase conectarse. Aunque son públicas en el sentido de que el navegador puede verlas, Google las protege mediante reglas de seguridad configuradas en la consola de Firebase.
- `apiKey: process.env.REACT_APP_FIREBASE_API_KEY` — lee la API key desde las variables de entorno del archivo `.env`. `process.env` es el objeto que Node.js usa para acceder a las variables de entorno. El prefijo `REACT_APP_` es obligatorio para que CRA (Create React App) exponga la variable al navegador; variables sin ese prefijo quedan ocultas en el servidor.
- `authDomain` — el dominio de Firebase que maneja el flujo de autenticación. Normalmente tiene la forma `proyecto.firebaseapp.com`.
- `projectId` — el identificador único del proyecto en Firebase. Todos los documentos de Firestore se almacenan bajo este proyecto.
- `storageBucket` — la URL del bucket de almacenamiento en la nube (aunque esta app no usa Firebase Storage directamente, se incluye en la configuración estándar).
- `messagingSenderId` — identificador para Firebase Cloud Messaging (notificaciones push). No se usa en esta app pero forma parte de la configuración estándar.
- `appId` — identificador único de la app web dentro del proyecto de Firebase. Un proyecto puede tener múltiples apps (web, Android, iOS) y cada una tiene su propio `appId`.
- `const app = initializeApp(firebaseConfig)` — inicializa la conexión con Firebase usando las credenciales del objeto de configuración. Retorna una instancia de la app de Firebase que los demás módulos necesitan para funcionar. Esta función solo se debe llamar una vez en toda la aplicación.
- `export const auth = getAuth(app)` — crea y exporta la instancia del módulo de autenticación asociada a esta app de Firebase. Al exportarla, cualquier componente puede importar `auth` directamente desde este archivo y usarla para operaciones de login, registro, etc.
- `export const db = getFirestore(app)` — crea y exporta la instancia de Firestore asociada a esta app. Con `db` se pueden hacer todas las operaciones de base de datos. Al centralizar esto en un solo archivo, se evita inicializar Firestore múltiples veces.
- `export default app` — exporta también la instancia de la app de Firebase como exportación por defecto. Algunos módulos avanzados de Firebase necesitan esta instancia directamente.

---

## src/utils/dateCalculations.js

```js
const TZ = 'America/Mexico_City';

const toLocalDateStr = (value) =>
  new Intl.DateTimeFormat('en-CA', { timeZone: TZ }).format(new Date(value));

const toLocalMidnight = (value) => {
  const [y, m, d] = toLocalDateStr(value).split('-').map(Number);
  return new Date(y, m - 1, d);
};

export const isPriority = (expirationDate) => {
  if (!expirationDate) return false;
  const today = toLocalMidnight(new Date());
  const expDate = toLocalMidnight(expirationDate);
  const diffDays = Math.round((expDate - today) / 864e5);
  return diffDays >= 0 && diffDays <= 3;
};

export const isExpired = (expirationDate) => {
  if (!expirationDate) return false;
  const today = toLocalMidnight(new Date());
  const expDate = toLocalMidnight(expirationDate);
  return expDate < today;
};

export const getDaysRemaining = (expirationDate) => {
  if (!expirationDate) return null;
  const today = toLocalMidnight(new Date());
  const expDate = toLocalMidnight(expirationDate);
  return Math.round((expDate - today) / 864e5);
};

export const formatDate = (dateString) => {
  if (!dateString) return 'Fecha desconocida';
  return new Date(dateString).toLocaleDateString('es-MX', {
    year: 'numeric', month: 'long', day: 'numeric', timeZone: TZ
  });
};

export const toISODateString = (date) => {
  if (!date) return '';
  if (typeof date === 'string' && /^\d{4}-\d{2}-\d{2}$/.test(date)) return date;
  return toLocalDateStr(date);
};

export const getTodayISO = () => toLocalDateStr(new Date());
```

- `const TZ = 'America/Mexico_City'` — define la zona horaria de Ciudad de México como constante. Esta zona tiene un desfase de UTC-6 (o UTC-5 en horario de verano). Si no se especifica la zona horaria en los cálculos de fechas, JavaScript usa UTC por defecto, lo que provoca que una fecha como `"2025-05-01"` se interprete como las 00:00 UTC, que equivale a las 18:00 del día anterior en México, causando que se muestre un día equivocado.
- `const toLocalDateStr = (value) => new Intl.DateTimeFormat('en-CA', { timeZone: TZ }).format(new Date(value))` — función auxiliar que convierte cualquier fecha (ISO string, objeto Date, Timestamp de Firestore) a un string en formato `"YYYY-MM-DD"` usando la zona horaria de México. Se usa el locale `'en-CA'` (inglés canadiense) porque ese locale produce automáticamente el formato `YYYY-MM-DD`, que es el formato estándar ISO para fechas sin hora. `Intl.DateTimeFormat` es la API nativa del navegador para internacionalización de fechas.
- `const toLocalMidnight = (value)` — función auxiliar que toma cualquier fecha y crea un nuevo objeto `Date` representando la medianoche local de ese día. Esto es fundamental para comparar fechas a nivel de día: si se compararan dos fechas con hora incluida, una con `12:00` y otra con `14:00` del mismo día podrían dar diferencias de horas en lugar de 0 días.
- `const [y, m, d] = toLocalDateStr(value).split('-').map(Number)` — descompone el string `"YYYY-MM-DD"` en tres números. `.split('-')` separa el string en `["2025","05","01"]` y `.map(Number)` convierte cada elemento a número, resultando en `[2025, 5, 1]`. La desestructuración asigna cada elemento a su variable correspondiente.
- `return new Date(y, m - 1, d)` — construye un objeto `Date` en hora local del navegador. Se usa el constructor con año, mes y día por separado (en lugar de parsear el string directamente) porque `new Date("2025-05-01")` trataría la fecha como UTC medianoche, mientras que `new Date(2025, 4, 1)` la crea en hora local. El `m - 1` es necesario porque el constructor `Date` numera los meses de 0 (enero) a 11 (diciembre).
- `export const isPriority = (expirationDate)` — función exportada que determina si un ingrediente está "en prioridad", es decir, si caduca en los próximos 3 días o ya hoy. Se usa para marcar ingredientes con badge rojo y destacarlos en la vista de generación de recetas.
- `if (!expirationDate) return false` — validación defensiva: si no hay fecha de caducidad (por ejemplo, ingredientes muy antiguos que se guardaron sin ese campo), la función retorna `false` de forma segura en lugar de lanzar un error.
- `const today = toLocalMidnight(new Date())` — obtiene la medianoche local de hoy. `new Date()` crea la fecha y hora actual, y `toLocalMidnight` la convierte a medianoche local para poder comparar solo el día.
- `const expDate = toLocalMidnight(expirationDate)` — convierte la fecha de caducidad (que viene de Firestore como string ISO o Timestamp) a medianoche local de ese día.
- `const diffDays = Math.round((expDate - today) / 864e5)` — calcula la diferencia en días. Restar dos objetos `Date` da la diferencia en milisegundos. `864e5` es la notación científica de 86,400,000 ms (24 horas × 60 min × 60 seg × 1000 ms = 1 día). `Math.round` redondea para evitar decimales por diferencias de microsegundos.
- `return diffDays >= 0 && diffDays <= 3` — retorna `true` solo si quedan entre 0 y 3 días. El `>= 0` excluye los ya caducados (que tendrían `diffDays` negativo); el `<= 3` limita a los próximos 3 días.
- `export const isExpired = (expirationDate)` — determina si un ingrediente ya caducó. Se usa para filtrar ingredientes en la vista de generación (no ofrecer caducados) y para cambiar su estilo en el inventario.
- `return expDate < today` — si la fecha de caducidad es antes de hoy, el ingrediente ya expiró. La comparación de objetos `Date` compara sus valores internos en milisegundos.
- `export const getDaysRemaining = (expirationDate)` — retorna el número de días que quedan hasta la caducidad. Puede ser negativo si ya caducó (útil para mostrar "Caducó hace X días"). Retorna `null` si no hay fecha para distinguir "no hay dato" de "0 días".
- `export const formatDate = (dateString)` — convierte un ISO string o Timestamp a una cadena legible en español, como "1 de mayo de 2025". Se usa en tablas y tarjetas donde mostrar la fecha en formato humano.
- `toLocaleDateString('es-MX', { year: 'numeric', month: 'long', day: 'numeric', timeZone: TZ })` — el locale `'es-MX'` produce el formato en español de México. `month: 'long'` escribe el mes completo ("mayo" en lugar de "5"). `timeZone: TZ` asegura que la conversión use la hora de México.
- `export const toISODateString = (date)` — convierte un objeto `Date` a string `"YYYY-MM-DD"` en hora local. Se usa principalmente para rellenar los atributos `value` de los inputs `<input type="date">`, que requieren exactamente ese formato.
- `if (typeof date === 'string' && /^\d{4}-\d{2}-\d{2}$/.test(date)) return date` — si el valor ya es un string en formato `"YYYY-MM-DD"`, lo retorna tal cual sin procesar. Esto evita el problema de pasar ese string por `new Date()`, que lo interpretaría como UTC medianoche y podría devolver el día anterior.
- `export const getTodayISO = () => toLocalDateStr(new Date())` — atajo que retorna la fecha de hoy en formato `"YYYY-MM-DD"` en hora local de México. Se usa para inicializar el campo de fecha de compra en el formulario de registro de ingredientes.

---

## src/utils/recipeHelpers.js

### Bloque 1 — cleanText, formatQuantity, parseSafeQuantity, isNumeric

```js
export function cleanText(value) {
  if (value === null || value === undefined) return null;
  if (typeof value === 'object') return null;
  const str = String(value).trim();
  if (str === '' || str.toLowerCase() === 'null' || str.toLowerCase() === 'undefined') {
    return null;
  }
  return str;
}

export function formatQuantity(value, decimals = 2) {
  const parsed = parseSafeQuantity(value);
  if (parsed.type === 'text') return parsed.text || '';
  if (decimals === 0) return String(Math.round(parsed.number));
  return parsed.number.toFixed(decimals);
}

export function parseSafeQuantity(value) {
  if (value === null || value === undefined) return { type: 'text', text: '' };
  const str = String(value).trim();
  if (str === '' || str.toLowerCase() === 'null' || str.toLowerCase() === 'undefined') {
    return { type: 'text', text: '' };
  }
  const num = Number(str);
  if (!isNaN(num) && isFinite(num)) return { type: 'number', number: num };
  if (str.length > 0) {
    if (str[0] === str[0].toLowerCase()) {
      return { type: 'text', text: str.charAt(0).toUpperCase() + str.slice(1) };
    }
    return { type: 'text', text: str };
  }
  return { type: 'text', text: '' };
}

export function isNumeric(value) {
  const parsed = parseSafeQuantity(value);
  return parsed.type === 'number';
}
```

- `export function cleanText(value)` — función que sanitiza cualquier valor para evitar que strings problemáticos lleguen a la interfaz de usuario. GPT a veces genera el literal `"null"` como string en lugar de `null` real, o devuelve objetos en lugar de strings.
- `if (value === null || value === undefined) return null` — maneja los casos donde no hay valor en absoluto. Retorna `null` (en lugar de el string `"null"`) para que los componentes puedan hacer comprobaciones simples con `if (valor)`.
- `if (typeof value === 'object') return null` — descarta objetos como Timestamps de Firestore que pudieran llegar por error a esta función. Un Timestamp de Firestore es un objeto con métodos `.toDate()`, no un string.
- `const str = String(value).trim()` — convierte el valor a string y elimina espacios en blanco del inicio y final. `String()` funciona con cualquier tipo primitivo: números, booleanos, etc.
- `str.toLowerCase() === 'null'` — compara en minúsculas para capturar `"null"`, `"NULL"`, `"Null"`, etc. El string literal `"null"` que GPT genera a veces debe tratarse como ausencia de valor.
- `str.toLowerCase() === 'undefined'` — mismo caso para `"undefined"` como string literal.
- `export function formatQuantity(value, decimals = 2)` — formatea una cantidad para mostrarla en la UI. Si el valor es texto como `"Al gusto"` lo muestra tal cual; si es número lo redondea al número de decimales indicado. El parámetro `decimals = 2` es un valor por defecto.
- `if (parsed.type === 'text') return parsed.text || ''` — si no es número, devuelve el texto. El `|| ''` garantiza que nunca se retorne `undefined` o `null`, lo que podría causar que React muestre `"undefined"` en pantalla.
- `if (decimals === 0) return String(Math.round(parsed.number))` — caso especial para cuando se quiere mostrar el número sin decimales. `Math.round` redondea al entero más cercano y `String()` lo convierte para retornar siempre un string.
- `return parsed.number.toFixed(decimals)` — formatea el número con exactamente `decimals` cifras decimales. Por ejemplo, `1.5` con `decimals = 2` produce `"1.50"`.
- `export function parseSafeQuantity(value)` — función central que analiza cualquier valor y determina si es un número válido o texto. Retorna un objeto discriminado: `{ type: 'number', number: N }` o `{ type: 'text', text: S }`. Este patrón se llama "discriminated union" y permite a quien llama saber exactamente qué recibió antes de operar.
- `const num = Number(str)` — intenta convertir el string a número. `Number("100")` → `100`, `Number("Al gusto")` → `NaN`, `Number("")` → `0`.
- `if (!isNaN(num) && isFinite(num))` — verifica que la conversión produjo un número real. `isNaN` verifica que no sea `NaN` (Not a Number), `isFinite` verifica que no sea `Infinity` o `-Infinity`, que también son valores inválidos para cantidades.
- `if (str[0] === str[0].toLowerCase())` — verifica si el primer carácter está en minúscula. Si el texto viene con minúscula inicial como `"al gusto"`, lo capitaliza. Esto es necesario porque el prompt de OpenAI pide que textos no numéricos empiecen con mayúscula, pero no siempre lo cumple.
- `str.charAt(0).toUpperCase() + str.slice(1)` — capitaliza solo el primer carácter y concatena el resto del string sin cambiar. `charAt(0)` es más seguro que `str[0]` para strings vacíos en algunas versiones de JavaScript.
- `export function isNumeric(value)` — atajo booleano que usa `parseSafeQuantity` internamente y retorna simplemente `true` o `false`. Se usa cuando no se necesita el valor numérico, solo saber si es número.

### Bloque 2 — normalizeOpenAIResponse

```js
export function normalizeOpenAIResponse(rawResponse) {
  if (rawResponse && typeof rawResponse === 'object' && rawResponse.error) {
    return { recipe: null, portionWarning: null,
      error: cleanText(rawResponse.error), isValid: false };
  }

  const recipe = rawResponse && rawResponse.recipe && Array.isArray(rawResponse.recipe)
    ? rawResponse.recipe[0]
    : (rawResponse && typeof rawResponse === 'object' ? rawResponse : null);

  if (!recipe) {
    return { recipe: null, portionWarning: null,
      error: 'No se pudo generar una receta válida', isValid: false };
  }

  let portionWarning = null;
  if (typeof recipe.portionWarning === 'string') {
    const cleaned = cleanText(recipe.portionWarning);
    if (cleaned) portionWarning = cleaned;
  }

  let allergenWarning = null;
  if (typeof recipe.allergenWarning === 'string') {
    const cleaned = cleanText(recipe.allergenWarning);
    if (cleaned) allergenWarning = cleaned;
  }

  let ingredients = [];
  if (Array.isArray(recipe.ingredients)) {
    ingredients = recipe.ingredients.map(ing => ({
      name: cleanText(ing.name) || 'Ingrediente',
      quantity: ing.quantity !== undefined ? ing.quantity : '',
      unit: cleanText(ing.unit) || ''
    }));
  }

  let missingIngredients = [];
  if (Array.isArray(recipe.missingIngredients)) {
    missingIngredients = recipe.missingIngredients.map(ing => ({
      name: cleanText(ing.name) || 'Ingrediente',
      quantity: ing.quantity !== undefined ? ing.quantity : '',
      unit: cleanText(ing.unit) || ''
    }));
  }

  let instructions = [];
  if (Array.isArray(recipe.instructions)) {
    instructions = recipe.instructions.map(inst => cleanText(inst)).filter(Boolean);
  }

  let categories = [];
  if (Array.isArray(recipe.categories)) {
    categories = recipe.categories.map(cat => cleanText(cat)).filter(Boolean);
  }

  const servings = isNumeric(recipe.servings) ? Number(recipe.servings) : 2;
  const prepTime = isNumeric(recipe.prepTime) ? Number(recipe.prepTime) : null;

  const normalizedRecipe = {
    name: cleanText(recipe.name) || 'Receta sin nombre',
    categories, ingredients, missingIngredients,
    instructions, prepTime, servings, portionWarning, allergenWarning
  };

  return { recipe: normalizedRecipe, portionWarning, error: null, isValid: true };
}
```

- `export function normalizeOpenAIResponse(rawResponse)` — función que toma la respuesta cruda de OpenAI y la convierte a una estructura limpia y predecible que los componentes pueden usar sin hacer validaciones adicionales.
- `if (rawResponse && typeof rawResponse === 'object' && rawResponse.error)` — comprueba si la respuesta es un objeto de error. Cuando las categorías seleccionadas son incompatibles con los ingredientes, OpenAI devuelve `{ "error": "No es posible..." }` en lugar de una receta. Este bloque lo detecta.
- `error: cleanText(rawResponse.error)` — limpia el mensaje de error para que no tenga strings problemáticos antes de mostrarlo al usuario.
- `isValid: false` — el flag `isValid` permite a quien llama saber de inmediato si la normalización fue exitosa sin tener que inspeccionar el objeto completo.
- `rawResponse.recipe && Array.isArray(rawResponse.recipe) ? rawResponse.recipe[0]` — el esquema JSON que se envía a OpenAI fuerza que la respuesta sea `{ recipe: [...] }` con un array de exactamente 1 receta. Esta línea extrae ese primer (y único) elemento. La verificación con `Array.isArray` es defensiva por si GPT ignorara el esquema y devolviera algo diferente.
- `rawResponse && typeof rawResponse === 'object' ? rawResponse : null` — fallback: si por alguna razón la respuesta no tiene el wrapper `recipe`, intenta tratar la respuesta completa como la receta directamente. Si tampoco es un objeto, retorna `null`.
- `if (!recipe)` — si tras todos los intentos no se pudo extraer una receta, retorna inmediatamente con `isValid: false` y un mensaje de error genérico.
- `if (typeof recipe.portionWarning === 'string')` — solo procesa `portionWarning` si es un string. Si es `null` (que el esquema permite) o cualquier otro tipo, simplemente se deja en `null`. Esto evita que `cleanText` reciba valores inesperados.
- `if (cleaned) portionWarning = cleaned` — solo asigna si `cleanText` devolvió algo no vacío. Si el warning era el string `"null"` o estaba vacío, `cleanText` retorna `null` y `portionWarning` permanece `null`.
- `ing.quantity !== undefined ? ing.quantity : ''` — para la cantidad se hace una verificación especial: si existe (aunque sea `0` o `false`), se preserva tal cual sin pasar por `cleanText`, porque la cantidad puede ser un número y `cleanText` solo funciona con strings. Si no existe, se usa string vacío.
- `cleanText(ing.name) || 'Ingrediente'` — si el nombre del ingrediente viene vacío o como `"null"`, usa el texto genérico `'Ingrediente'` como fallback para que la UI siempre tenga algo que mostrar.
- `.map(inst => cleanText(inst)).filter(Boolean)` — limpia cada instrucción con `cleanText` y luego `filter(Boolean)` elimina los elementos que quedaron `null`, `undefined`, `0` o string vacío. Esto asegura que el array de instrucciones solo contenga pasos reales.
- `const servings = isNumeric(recipe.servings) ? Number(recipe.servings) : 2` — convierte `servings` a número. Si OpenAI devolvió un string como `"4"` en lugar del número `4`, `Number()` lo convierte. Si es inválido, usa `2` como valor razonable por defecto.
- `const prepTime = isNumeric(recipe.prepTime) ? Number(recipe.prepTime) : null` — similar a `servings`, pero si el tiempo de preparación es inválido usa `null` (no `0` ni `2`), porque `null` indica "no disponible" y los componentes pueden ocultarlo.

### Bloque 3 — retryOperation

```js
export async function retryOperation(fn, options = {}) {
  const maxRetries = options.maxRetries || 3;
  const initialDelay = options.initialDelay || 500;
  const shouldRetry = options.shouldRetry || ((error) => {
    const status = error.response?.status || error.status;
    return !status || status >= 500 || status === 429;
  });

  let lastError;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      if (attempt === maxRetries || !shouldRetry(error)) break;
      const delay = initialDelay * Math.pow(2, attempt);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}
```

- `export async function retryOperation(fn, options = {})` — función de orden superior (higher-order function) que recibe otra función `fn` y la ejecuta con reintentos automáticos. Es `async` porque necesita esperar (`await`) tanto la función que reintenta como las pausas entre intentos. `options = {}` tiene un valor por defecto vacío para no requerir el segundo argumento.
- `const maxRetries = options.maxRetries || 3` — número máximo de reintentos adicionales. Con `maxRetries = 2`, habrá 1 intento inicial más 2 reintentos = 3 ejecuciones totales en el peor caso. El `|| 3` es el valor por defecto si no se especifica.
- `const initialDelay = options.initialDelay || 500` — tiempo en milisegundos que se espera antes del primer reintento. Los reintentos posteriores esperan más tiempo (backoff exponencial). El valor por defecto es 500 ms.
- `const shouldRetry = options.shouldRetry || ((error) => { ... })` — función que decide si un error específico justifica un reintento. Si no se pasa, usa la función por defecto que reintenta en tres casos: sin `status` (error de red como timeout o DNS), `status >= 500` (error interno del servidor, probablemente temporal), `status === 429` (rate limit de OpenAI, hay que esperar).
- `error.response?.status || error.status` — busca el status HTTP del error. `axios` lo pone en `error.response.status`; otros errores pueden tenerlo directamente en `error.status`. El operador `?.` evita un crash si `error.response` es `undefined`.
- `!status` — si no hay status en absoluto, es un error de red (sin respuesta del servidor), que es reintentable porque puede ser un problema temporal de conexión.
- `let lastError` — variable donde se guarda el último error capturado. Se declara fuera del ciclo para poder lanzarla después si todos los intentos fallaron.
- `for (let attempt = 0; attempt <= maxRetries; attempt++)` — el ciclo va de `0` a `maxRetries` inclusive. El intento `0` es la ejecución inicial (no un "reintento"); los intentos `1`, `2`, etc. son los reintentos reales. Con `maxRetries = 2`, el ciclo ejecuta: `attempt = 0` (intento inicial), `attempt = 1` (primer reintento), `attempt = 2` (segundo reintento).
- `return await fn()` — ejecuta la función y si tiene éxito retorna inmediatamente su resultado, saliendo del ciclo y de la función completa. El `await` es necesario porque `fn()` es asíncrona.
- `lastError = error` — guarda el error antes de decidir si reintentar. Así siempre tenemos el último error para relanzarlo al final.
- `if (attempt === maxRetries || !shouldRetry(error)) break` — condición de parada: si ya se agotaron los intentos (`attempt === maxRetries`) o si el error no es de los reintentables (`!shouldRetry(error)`), sale del ciclo sin esperar. Esto evita reintentar en errores como `400 Bad Request` que nunca van a resolverse solos.
- `const delay = initialDelay * Math.pow(2, attempt)` — calcula el tiempo de espera con backoff exponencial. Con `initialDelay = 500`: en `attempt = 0` espera `500 * 1 = 500ms`, en `attempt = 1` espera `500 * 2 = 1000ms`, en `attempt = 2` espera `500 * 4 = 2000ms`. Esto respeta a los servidores sobrecargados dándoles tiempo progresivamente mayor para recuperarse.
- `await new Promise(resolve => setTimeout(resolve, delay))` — pausa la ejecución el tiempo calculado. `setTimeout` normalmente usa callbacks, pero envuelto en `Promise` permite usar `await` para esperar de forma limpia sin bloquear el hilo principal.
- `throw lastError` — si el ciclo terminó sin retornar (todos los intentos fallaron), lanza el último error capturado para que el código que llamó a `retryOperation` pueda manejarlo.

---

## src/utils/Modal.js

### Bloque 1 — Configuración de tipos

```js
import React, { useState } from 'react';
import { X, CheckCircle } from 'lucide-react';

const TYPE_CONFIG = {
  confirm: {
    emoji: '🍳', animation: 'animate-bounce',
    headerGradient: 'from-food-50 to-cream-100',
    headerBorder: 'border-food-200',
    titleColor: 'text-food-800',
    confirmBtn: 'bg-gradient-to-r from-food-500 to-food-600 hover:from-food-600 hover:to-food-700',
    cancelBtn: 'bg-white border-2 border-food-200 text-food-700 hover:bg-food-50',
  },
  success: {
    emoji: '🎉', animation: 'animate-bounce',
    headerGradient: 'from-food-50 to-cream-100',
    headerBorder: 'border-food-200', titleColor: 'text-food-800',
    confirmBtn: 'bg-gradient-to-r from-food-500 to-food-600', cancelBtn: '',
  },
  error: {
    emoji: '😱', animation: 'animate-wiggle',
    headerGradient: 'from-tomato-50 to-tomato-100',
    headerBorder: 'border-tomato-200', titleColor: 'text-tomato-800',
    confirmBtn: 'bg-gradient-to-r from-tomato-500 to-tomato-600', cancelBtn: '',
  },
  pending: {
    emoji: '📦', animation: 'animate-bounce',
    headerGradient: 'from-orange-50 to-amber-50',
    headerBorder: 'border-orange-200', titleColor: 'text-orange-800',
    confirmBtn: 'bg-gradient-to-r from-orange-500 to-orange-600', cancelBtn: '',
  },
  welcome: {
    icon: null, animation: '',
    headerGradient: 'from-food-50 to-food-100',
    headerBorder: 'border-food-200', titleColor: 'text-food-800',
    confirmBtn: 'bg-gradient-to-r from-food-500 to-food-600', cancelBtn: '',
  },
};
```

- `import { X, CheckCircle } from 'lucide-react'` — importa dos íconos de la librería Lucide: `X` (la X para cerrar el modal) y `CheckCircle` (el círculo con palomita que se usa en el modal de bienvenida en lugar de emoji).
- `const TYPE_CONFIG = { ... }` — objeto de configuración que centraliza el estilo visual de cada tipo de modal. En lugar de escribir condicionales `if (type === 'confirm') { ... } else if (type === 'error') { ... }` dentro del componente, se define toda la variación aquí una sola vez. Cuando el componente necesita el estilo, simplemente accede a `TYPE_CONFIG[type]`.
- `emoji: '🍳'` en `confirm` — el emoji de la sartén aparece animado en los modales de confirmación, reforzando la temática de cocina de la app.
- `animation: 'animate-bounce'` — clase de Tailwind que aplica una animación de rebote al emoji. En los errores se usa `'animate-wiggle'` (definida en `index.css`) para dar una sensación de alarma.
- `headerGradient: 'from-food-50 to-cream-100'` — clases de Tailwind para el gradiente de fondo del encabezado del modal. El prefijo `from-` y `to-` son parte de la sintaxis de gradientes de Tailwind, y `food-50`, `cream-100` son colores personalizados definidos en `tailwind.config.js`.
- `headerBorder: 'border-food-200'` — clase de Tailwind para el color del borde inferior que separa el encabezado del cuerpo del modal.
- `titleColor: 'text-food-800'` — color del texto del título. En los errores es `text-tomato-800` (rojo oscuro) para comunicar visualmente el tipo de situación.
- `confirmBtn` — string con todas las clases de Tailwind del botón principal. Incluye el gradiente de fondo, el comportamiento en hover y la sombra. Se pasa directamente a `className` en el JSX.
- `cancelBtn` — clases del botón de cancelar. Solo existe en modales de tipo `confirm`; en los demás tipos (`success`, `error`, etc.) es string vacío porque no hay botón de cancelar.
- `welcome` con `icon: null` — el modal de bienvenida usa un ícono SVG (`CheckCircle`) en lugar de emoji, por eso `icon: null` indica que no hay emoji y el componente renderiza el ícono en su lugar.

### Bloque 2 — Componente Modal

```js
const Modal = ({ isOpen, onClose, onConfirm, title, message,
  type = 'confirm', confirmText = 'Confirmar', cancelText = 'Cancelar' }) => {

  const [emojiPopped, setEmojiPopped] = useState(false);

  if (!isOpen) return null;

  const cfg = TYPE_CONFIG[type] || TYPE_CONFIG.confirm;

  const handleBackdropClick = (e) => {
    if (e.target === e.currentTarget) onClose();
  };

  const handleConfirm = async () => {
    if (type === 'confirm') {
      onClose();
      if (onConfirm) await onConfirm();
    } else {
      if (onConfirm) await onConfirm();
      onClose();
    }
  };

  return (
    <div className="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-50 p-4"
      onClick={handleBackdropClick}>
      <div className="card-food rounded-2xl max-w-md w-full overflow-hidden modal-food-enter">

        <div className={`bg-gradient-to-r ${cfg.headerGradient} border-b-2 ${cfg.headerBorder} px-6 pt-6 pb-5`}>
          <div className="flex items-start justify-between gap-3">
            <div className="flex items-center gap-4">
              {type === 'welcome' ? (
                <CheckCircle size={28} className="text-food-600 flex-shrink-0" />
              ) : (
                <span
                  className={`text-5xl select-none cursor-default inline-block transition-transform duration-150
                    ${emojiPopped ? 'scale-150' : cfg.animation}`}
                  onMouseEnter={() => setEmojiPopped(true)}
                  onMouseLeave={() => setEmojiPopped(false)}>
                  {cfg.emoji}
                </span>
              )}
              <h3 className={`text-xl font-bold ${cfg.titleColor} font-cooking leading-tight`}>
                {title}
              </h3>
            </div>
            <button onClick={onClose}
              className="text-gray-400 hover:text-food-600 transition-all duration-300 hover:rotate-90">
              <X size={22} />
            </button>
          </div>
        </div>

        <div className="px-6 py-5 text-gray-600 text-sm leading-relaxed">{message}</div>

        <div className="px-6 pb-6 flex gap-3">
          {type === 'confirm' ? (
            <>
              <button onClick={onClose}
                className={`flex-1 ${cfg.cancelBtn} py-3 px-4 rounded-xl font-semibold transition-all duration-200 active:scale-95`}>
                {cancelText}
              </button>
              <button onClick={handleConfirm}
                className={`flex-1 ${cfg.confirmBtn} text-white py-3 px-4 rounded-xl font-semibold shadow-lg active:scale-95`}>
                {confirmText}
              </button>
            </>
          ) : (
            <button onClick={handleConfirm}
              className={`flex-1 ${cfg.confirmBtn} text-white py-3 px-4 rounded-xl font-semibold shadow-lg active:scale-95`}>
              Entendido
            </button>
          )}
        </div>
      </div>

      <style>{`
        @keyframes modal-food-enter {
          from { opacity: 0; transform: scale(0.88) translateY(12px); }
          to   { opacity: 1; transform: scale(1) translateY(0); }
        }
        .modal-food-enter {
          animation: modal-food-enter 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
        }
      `}</style>
    </div>
  );
};

export default Modal;
```

- `isOpen` — prop booleana que controla si el modal está visible. El componente padre lo maneja con su propio estado y lo pasa como prop.
- `onClose` — función que el componente padre pasa para cerrar el modal. El modal no puede cerrase a sí mismo porque no tiene acceso al estado del padre; necesita pedirle al padre que cambie `isOpen` a `false`.
- `onConfirm` — función asíncrona que se ejecuta cuando el usuario confirma. Puede contener operaciones de Firestore, llamadas a APIs, etc.
- `type = 'confirm'` — valor por defecto del tipo. Si el componente padre no especifica `type`, el modal se comporta como confirm con dos botones.
- `confirmText = 'Confirmar'` y `cancelText = 'Cancelar'` — textos por defecto de los botones, que el padre puede personalizar según el contexto (ej. "Eliminar", "Sí, continuar").
- `const [emojiPopped, setEmojiPopped] = useState(false)` — estado local que controla si el emoji está en el estado "pop" (agrandado). Es un estado simple booleano que solo vive dentro de este componente.
- `if (!isOpen) return null` — si el modal está cerrado, retorna `null`, que en React significa "no renderizar nada". Esto es importante: evita que el modal ocupe espacio en el DOM, que eventos de teclado lleguen al modal cerrado, y que se ejecuten efectos innecesarios.
- `const cfg = TYPE_CONFIG[type] || TYPE_CONFIG.confirm` — obtiene la configuración de estilo para el tipo actual. El `|| TYPE_CONFIG.confirm` es un fallback de seguridad: si se pasa un tipo desconocido, usa la configuración de confirm por defecto en lugar de dar error.
- `handleBackdropClick` — permite cerrar el modal haciendo click en el fondo oscuro (fuera de la tarjeta blanca). La condición `e.target === e.currentTarget` es crucial: `e.target` es el elemento que recibió el click (puede ser un botón dentro del modal), `e.currentTarget` es el elemento al que está vinculado el evento (el overlay). Solo si son iguales (el click fue directamente en el overlay, no en un hijo) se cierra el modal.
- `handleConfirm` con `if (type === 'confirm')` — el orden de operaciones importa. Para modales de confirmación, se cierra **primero** y luego se ejecuta `onConfirm`. Esto es necesario porque `onConfirm` puede abrir otro modal (ej. un modal de éxito), y si el modal actual no se cerró antes, el nuevo modal competiría con él.
- Para otros tipos (`success`, `error`, etc.) — se ejecuta `onConfirm` **primero** y luego se cierra. En estos casos `onConfirm` suele ser una navegación (ej. `() => setCurrentView('menu')`), y si se cerrara el modal antes de navegar el usuario vería un flash de la pantalla anterior.
- `fixed inset-0` — posición fija que cubre toda la pantalla. `inset-0` equivale a `top: 0; right: 0; bottom: 0; left: 0`, ocupando el 100% del viewport.
- `bg-black/50` — fondo negro al 50% de opacidad. El `/50` es la sintaxis de Tailwind para opacidad en colores.
- `backdrop-blur-sm` — aplica un efecto de desenfoque suave al contenido detrás del overlay, dando la sensación de que el fondo está "fuera de foco".
- `z-50` — z-index alto para que el modal aparezca por encima de todos los demás elementos de la página.
- `modal-food-enter` — clase CSS personalizada que activa la animación de entrada definida en el `<style>` embebido al final del componente.
- `${emojiPopped ? 'scale-150' : cfg.animation}` — expresión ternaria en el className: si el usuario pasó el cursor sobre el emoji (`emojiPopped = true`), aplica `scale-150` (agranda 150%); si no, aplica la animación del tipo (`animate-bounce` o `animate-wiggle`).
- `onMouseEnter={() => setEmojiPopped(true)}` — cuando el cursor entra al área del emoji, activa el estado "pop".
- `onMouseLeave={() => setEmojiPopped(false)}` — cuando el cursor sale, vuelve a la animación normal.
- `hover:rotate-90` — el botón X rota 90 grados al hacer hover. Es un efecto visual de Tailwind que indica interactividad.
- `active:scale-95` — cuando el botón está siendo presionado (estado `active`), se encoge al 95%. Este microefecto simula la sensación de presionar un botón físico.
- `@keyframes modal-food-enter` — animación CSS de entrada: el modal comienza con opacidad 0 y ligeramente encogido (`scale(0.88)`) y desplazado hacia abajo (`translateY(12px)`), y en 250ms llega a su estado final con opacidad 1 y tamaño normal.
- `cubic-bezier(0.34, 1.56, 0.64, 1)` — curva de animación personalizada que produce un efecto de "resorte" (spring): el modal llega a su posición y se pasa ligeramente antes de asentarse. El valor `1.56` mayor a `1.0` es lo que produce ese efecto de sobreimpulso.

---

## src/services/openaiService.js

### Bloque 1 — Helpers de error y sanitización

```js
import axios from 'axios';
import { normalizeOpenAIResponse, retryOperation } from '../utils/recipeHelpers';

const API_URL = '/api/openai';

const isTemporaryError = (error) => {
  const status = error.response?.status || error.status;
  if (!status) return true;
  if (status >= 500) return true;
  if (status === 429) return true;
  return false;
};

const sanitizeJsonString = (dirtyJson) => {
  if (typeof dirtyJson !== 'string') return dirtyJson;
  let cleaned = dirtyJson.trim();
  cleaned = cleaned.replace(/[""]/g, '"');
  cleaned = cleaned.replace(/['']/g, "'");
  cleaned = cleaned.replace(/\s*'([^']+)'\s*:/g, ' "$1": ');
  cleaned = cleaned.replace(/:\s*'([^']+)'/g, ': "$1"');
  cleaned = cleaned.replace(/[ --]/g, '');
  return cleaned;
};

const safeParseJsonFromResponse = (rawContent) => {
  if (!rawContent || typeof rawContent !== 'string') {
    throw new Error('Respuesta vacía o inválida de OpenAI');
  }
  const sanitized = sanitizeJsonString(rawContent);
  const jsonMatch = sanitized.match(/\{[\s\S]*\}/);
  if (!jsonMatch) throw new Error('No se encontró un JSON válido en la respuesta.');
  try {
    return JSON.parse(jsonMatch[0]);
  } catch (e) {
    throw new Error('JSON inválido en la respuesta.');
  }
};
```

- `import axios from 'axios'` — importa la librería HTTP Axios. Se prefiere sobre el `fetch` nativo porque permite configurar timeouts de forma sencilla (algo que `fetch` no soporta directamente), tiene mejor manejo de errores, e intercepta automáticamente respuestas de error.
- `import { normalizeOpenAIResponse, retryOperation } from '../utils/recipeHelpers'` — importa las dos funciones de utilidad que se usan después de recibir la respuesta de OpenAI: una para normalizar la estructura, otra para envolver la petición con reintentos.
- `const API_URL = '/api/openai'` — URL relativa del endpoint. Al ser relativa (sin dominio), funciona tanto en desarrollo (donde CRA la redirige al Express en puerto 3001 mediante el proxy configurado en `package.json`) como en producción (donde Vercel la sirve como función serverless). Esto hace que el código no necesite distinguir entre entornos.
- `const isTemporaryError = (error)` — función privada del módulo (no exportada) que determina si un error justifica reintentar la petición. Se pasa a `retryOperation` como el parámetro `shouldRetry`.
- `const status = error.response?.status || error.status` — busca el código de estado HTTP del error en dos lugares posibles. Axios pone el código en `error.response.status` cuando el servidor respondió (aunque con error). Si el servidor no respondió en absoluto (timeout, sin internet), `error.response` es `undefined` y se busca en `error.status`. El operador `?.` hace que si `error.response` es `undefined`, la expresión retorne `undefined` en lugar de lanzar `TypeError`.
- `if (!status) return true` — si no hay ningún código de estado, significa que el servidor no respondió en absoluto. Esto ocurre en timeouts, pérdida de conexión, o errores de DNS. Estos errores son temporales y vale la pena reintentar.
- `if (status >= 500) return true` — errores del servidor (500, 502, 503, 504, etc.) son generalmente temporales: el servidor sobrecargado, un reinicio, un bug intermitente. Vale reintentar.
- `if (status === 429) return true` — "Too Many Requests": OpenAI rechazó la petición porque se alcanzó el límite de tasa de la API. Después de esperar (con backoff exponencial), el reintento debería funcionar.
- `return false` — cualquier otro código de estado (400, 401, 403, 404, etc.) indica un error del lado del cliente que no se resolverá reintentando. Por ejemplo, un `400 Bad Request` significa que el prompt está mal formado; reintentar exactamente lo mismo daría el mismo error.
- `const sanitizeJsonString = (dirtyJson)` — función que limpia la respuesta de OpenAI antes de intentar parsearla como JSON. Aunque se usa `json_schema` structured output, GPT a veces devuelve caracteres problemáticos especialmente en textos largos como instrucciones de recetas.
- `if (typeof dirtyJson !== 'string') return dirtyJson` — si el valor ya fue procesado y no es un string (por ejemplo, ya es un objeto), lo retorna tal cual sin intentar limpiar.
- `let cleaned = dirtyJson.trim()` — elimina espacios en blanco y saltos de línea del inicio y final de la respuesta. GPT a veces agrega saltos de línea antes o después del JSON.
- `cleaned = cleaned.replace(/[""]/g, '"')` — los caracteres `"` (comilla izquierda tipográfica) y `"` (comilla derecha tipográfica) son caracteres Unicode distintos a la comilla estándar de JSON `"`. JSON solo acepta la comilla estándar; las tipográficas rompen el parser.
- `cleaned = cleaned.replace(/['']/g, "'")` — mismo caso para los apóstrofos tipográficos `'` y `'` que GPT usa a veces en contracciones dentro de textos.
- `cleaned = cleaned.replace(/\s*'([^']+)'\s*:/g, ' "$1": ')` — expresión regular que transforma claves con comillas simples. Por ejemplo, `'nombre':` se convierte en `"nombre":`. El patrón `[^']+` captura cualquier texto que no sea comilla simple (el nombre de la clave), y `$1` lo inserta en la posición correcta del reemplazo con comillas dobles.
- `cleaned = cleaned.replace(/:\s*'([^']+)'/g, ': "$1"')` — similar pero para los valores: `"clave": 'valor'` se convierte en `"clave": "valor"`.
- `cleaned = cleaned.replace(/[ --]/g, '')` — elimina caracteres de control binarios que son invisibles en editores de texto pero inválidos en JSON. Los caracteres dentro de la clase `[ --]` son NUL (0x00), US (0x1F), DEL (0x7F) y APC (0x9F).
- `const safeParseJsonFromResponse = (rawContent)` — función que extrae y parsea el JSON de la respuesta completa de OpenAI de forma segura, manejando los posibles errores con mensajes descriptivos.
- `if (!rawContent || typeof rawContent !== 'string')` — valida que la respuesta exista y sea un string antes de procesarla. `!rawContent` captura `null`, `undefined`, string vacío y `0`.
- `const sanitized = sanitizeJsonString(rawContent)` — limpia primero la respuesta de caracteres problemáticos antes de intentar encontrar el JSON.
- `const jsonMatch = sanitized.match(/\{[\s\S]*\}/)` — busca el primer bloque JSON en la respuesta usando regex. El patrón `\{` busca una llave de apertura, `[\s\S]*` captura cualquier carácter incluyendo saltos de línea (a diferencia de `.` que no captura newlines), y `\}` busca una llave de cierre. Retorna un array donde `jsonMatch[0]` es el texto que coincidió.
- `if (!jsonMatch)` — si no se encontró ningún bloque con `{}`, la respuesta no contiene JSON válido. Esto puede ocurrir si GPT devolvió texto plano en lugar de JSON.
- `return JSON.parse(jsonMatch[0])` — parsea el string JSON extraído a un objeto JavaScript. Si el JSON está malformado a pesar de la sanitización, `JSON.parse` lanzará un `SyntaxError`.
- `catch (e) { throw new Error('JSON inválido en la respuesta.') }` — captura el `SyntaxError` de `JSON.parse` y lo relanza como un error más descriptivo que indica qué salió mal.

### Bloque 2 — Esquema JSON estructurado

```js
const jsonSchema = {
  name: "recipe_response",
  schema: {
    type: "object",
    properties: {
      recipe: {
        type: "array",
        items: {
          type: "object",
          properties: {
            name: { type: "string" },
            categories: { type: "array", items: { type: "string" } },
            ingredients: {
              type: "array",
              items: {
                type: "object",
                properties: {
                  name: { type: "string" },
                  quantity: { type: ["string", "number"] },
                  unit: { type: "string" }
                },
                required: ["name", "quantity", "unit"]
              }
            },
            missingIngredients: {
              type: "array",
              items: {
                type: "object",
                properties: {
                  name: { type: "string" },
                  quantity: { type: ["string", "number"] },
                  unit: { type: "string" }
                },
                required: ["name", "quantity", "unit"]
              }
            },
            instructions: { type: "array", items: { type: "string" } },
            prepTime: { type: "number" },
            servings: { type: "number" },
            portionWarning: { type: ["string", "null"] },
            allergenWarning: { type: ["string", "null"] }
          },
          required: ["name","categories","ingredients","missingIngredients",
            "instructions","prepTime","servings","portionWarning","allergenWarning"]
        },
        minItems: 1,
        maxItems: 1
      }
    },
    required: ["recipe"],
    additionalProperties: false
  }
};
```

- `const jsonSchema = { name: "recipe_response", schema: { ... } }` — este objeto se envía a OpenAI con el parámetro `response_format: { type: "json_schema", json_schema: jsonSchema }`. Obliga a GPT-4 a devolver exactamente la estructura definida aquí. Sin este esquema, GPT podría devolver campos con nombres distintos, cambiar el tipo de datos, omitir campos obligatorios, etc.
- `name: "recipe_response"` — identificador del esquema. OpenAI lo usa internamente para validar la respuesta.
- `type: "object"` — el nivel raíz de la respuesta es un objeto JSON (lo que en JavaScript es un plain object `{}`).
- `recipe: { type: "array" }` — el campo `recipe` es un array, no un objeto directo. Aunque siempre tendrá 1 elemento, se modela como array para mantener coherencia con la API de OpenAI que permite devolver múltiples opciones.
- `items: { type: "object", properties: { ... } }` — cada elemento del array `recipe` es un objeto con las propiedades definidas en `properties`.
- `name: { type: "string" }` — el nombre de la receta debe ser string. Si GPT devolviera un número o `null`, OpenAI lo rechazaría antes de enviarlo.
- `categories: { type: "array", items: { type: "string" } }` — array de strings para las categorías. Por ejemplo `["Mexicana", "Saludable"]`.
- `ingredients` con `required: ["name", "quantity", "unit"]` — cada ingrediente DEBE tener los tres campos. Si faltara alguno, OpenAI lo forzaría a incluirlo.
- `quantity: { type: ["string", "number"] }` — la cantidad puede ser número (`100`) o string (`"Al gusto"`, `"Una pizca"`). El array de tipos le indica a OpenAI que ambos son válidos para este campo.
- `missingIngredients` — misma estructura que `ingredients`. Son los ingredientes que se necesitan para la receta pero que el usuario NO tiene en su inventario.
- `instructions: { type: "array", items: { type: "string" } }` — array de strings donde cada elemento es un paso de la receta.
- `prepTime: { type: "number" }` — tiempo de preparación en minutos como número (no string).
- `portionWarning: { type: ["string", "null"] }` — puede ser un string con una advertencia sobre las porciones (ej. "Esta receta es difícil de ajustar a 3 personas exactas") o `null` si no aplica advertencia.
- `allergenWarning: { type: ["string", "null"] }` — advertencia de alérgenos reconocidos por la FDA, o `null` si ningún ingrediente es alérgeno común.
- `required: ["name","categories","ingredients",...]` — todos estos campos son obligatorios en cada objeto de receta. GPT no puede omitirlos.
- `minItems: 1, maxItems: 1` — el array `recipe` debe tener exactamente 1 elemento. Esto asegura que siempre se genere exactamente una receta por petición.
- `additionalProperties: false` — prohíbe campos extras. Si GPT quisiera agregar un campo `"tips"` o `"calories"` que no está en el esquema, OpenAI lo rechazaría y GPT no podría incluirlo.

### Bloque 3 — generateRecipe

```js
export const generateRecipe = async ({
  ingredients, pendingDishes = [], categories, mealTime,
  servings, priorityOnly = false, regenerate = false, usedRecipeNames = []
}) => {
  try {
    const ingredientsList = ingredients.map(ing =>
      `${ing.name} (${ing.quantity} ${ing.unit})`).join(', ');

    const categoriesText = categories.length > 0
      ? `Categorías: ${categories.join(', ')}` : '';

    const regenerateText = regenerate
      ? '\n\n⚠️ IMPORTANTE: Genera una receta COMPLETAMENTE DIFERENTE a la anterior.' : '';

    const usedNamesText = usedRecipeNames.length > 0
      ? `\n\nRECETAS YA GENERADAS (NO REPITAS): ${usedRecipeNames.join(', ')}` : '';

    const makeRequest = async () => {
      const response = await axios.post(API_URL, {
        model: 'gpt-4.1',
        messages: [
          { role: 'system', content: 'Eres un generador de JSON estricto. Siempre responde con JSON válido.' },
          { role: 'user', content: prompt }
        ],
        response_format: { type: "json_schema", json_schema: jsonSchema },
        temperature: regenerate ? 0.7 : 0.5,
        max_completion_tokens: 1500
      }, { headers: { 'Content-Type': 'application/json' }, timeout: 30000 });

      if (response.status >= 400) {
        const err = new Error(response.data?.error?.message || `Error HTTP ${response.status}`);
        err.status = response.status;
        throw err;
      }

      const content = response.data?.choices?.[0]?.message?.content;
      return safeParseJsonFromResponse(content);
    };

    const result = await retryOperation(makeRequest, {
      maxRetries: 2, initialDelay: 500, shouldRetry: isTemporaryError
    });

    if (result.error) {
      const err = new Error(result.error);
      err.isAIError = true;
      err.isCompatibilityError = result.error.includes('No es posible');
      throw err;
    }

    const normalized = normalizeOpenAIResponse(result);
    if (!normalized.isValid) {
      const err = new Error(normalized.error || 'Error al procesar la respuesta de la IA');
      err.isAIError = true;
      throw err;
    }

    return [normalized.recipe];
  } catch (error) {
    throw error;
  }
};
```

- `export const generateRecipe = async ({ ... })` — función principal y más importante del servicio. Es `async` porque realiza peticiones HTTP que toman tiempo. Usa desestructuración de parámetros para recibir un objeto con propiedades nombradas, lo que hace las llamadas más legibles.
- `ingredients` — array de objetos con `name`, `quantity` y `unit`. Son los ingredientes que el usuario seleccionó de su inventario para usar en la receta.
- `pendingDishes = []` — platillos guardados previamente que el usuario quiere incorporar como base. El valor por defecto `[]` evita errores si no se pasa el argumento.
- `categories` — array de strings como `["Mexicana", "Saludable"]`. Máximo 3 categorías.
- `mealTime` — string como `"Desayuno"`, `"Comida"` o `"Cena"`.
- `servings` — número de personas para las que se genera la receta (1-20).
- `priorityOnly = false` — si es `true`, la generación se hizo con el botón "Usar ingredientes prioritarios", y el prompt lo indica explícitamente.
- `regenerate = false` — si es `true`, se está generando una receta alternativa a una ya mostrada. Cambia el prompt y aumenta la temperatura para mayor variedad.
- `usedRecipeNames = []` — lista de nombres de recetas ya generadas en esta sesión. Se pasan al prompt para que GPT no repita nombres.
- `const ingredientsList = ingredients.map(ing => \`${ing.name} (${ing.quantity} ${ing.unit})\`).join(', ')` — construye el texto de ingredientes para el prompt. Produce algo como `"Pollo (2 Piezas), Tomate (3 Piezas), Leche (0.5 Litros)"`. Incluir la cantidad y unidad ayuda a GPT a calcular las proporciones de la receta.
- `const categoriesText = categories.length > 0 ? \`Categorías: ${categories.join(', ')}\` : ''` — si hay categorías seleccionadas, construye el texto; si no, usa string vacío. Con el operador ternario se evita escribir `"Categorías: "` vacío en el prompt si no hay categorías.
- `const regenerateText = regenerate ? '...' : ''` — solo agrega la instrucción de "genera algo diferente" cuando sea un reintento. En la generación inicial, este texto no existe para no confundir a GPT.
- `const usedNamesText = usedRecipeNames.length > 0 ? \`\n\nRECETAS YA GENERADAS...\` : ''` — lista de nombres prohibidos. Solo se incluye si ya hay recetas generadas en esta sesión. Sin esto, GPT podría repetir el mismo nombre con variaciones mínimas.
- `const makeRequest = async ()` — función interna que encapsula una sola petición HTTP. Se define así para poder pasarla a `retryOperation`, que la llamará múltiples veces en caso de error temporal.
- `axios.post(API_URL, { model: 'gpt-4.1', ... }, { timeout: 30000 })` — realiza la petición POST al proxy. El primer argumento es la URL, el segundo es el body (que axios serializa automáticamente a JSON), el tercero son las opciones de axios como el timeout.
- `model: 'gpt-4.1'` — modelo de OpenAI que soporta `json_schema` structured output y tiene la capacidad necesaria para entender el contexto culinario y generar recetas coherentes.
- `messages: [{ role: 'system', content: '...' }, { role: 'user', content: prompt }]` — los mensajes del chat en formato estándar de OpenAI. El mensaje de sistema establece el comportamiento general del modelo; el mensaje de usuario contiene el prompt específico con los ingredientes, categorías, etc.
- `response_format: { type: "json_schema", json_schema: jsonSchema }` — indica a OpenAI que debe responder con JSON que cumpla el esquema definido. Esto activa el modo de "structured output" que garantiza la estructura de la respuesta.
- `temperature: regenerate ? 0.7 : 0.5` — controla la creatividad del modelo. `0.5` produce respuestas más consistentes y predecibles (bueno para la primera generación); `0.7` produce más variedad y creatividad (necesario en regeneraciones para que no repita la misma receta).
- `max_completion_tokens: 1500` — límite máximo de tokens en la respuesta. Una receta completa con instrucciones detalladas cabe dentro de 1500 tokens. Limitar los tokens reduce costos y evita respuestas excesivamente largas.
- `headers: { 'Content-Type': 'application/json' }` — le indica al proxy que el body es JSON. Aunque axios lo hace automáticamente con objetos, especificarlo explícitamente evita ambigüedades.
- `timeout: 30000` — si la petición tarda más de 30 segundos, axios la cancela automáticamente y lanza un error de timeout. Sin esto, la petición podría quedar colgada indefinidamente.
- `if (response.status >= 400)` — axios no lanza error automáticamente para respuestas con JSON (a diferencia de fetch). Esta verificación manual es necesaria para que errores como `429` o `500` activen el mecanismo de reintento.
- `err.status = response.status` — agrega el status HTTP al objeto de error para que `isTemporaryError` y el código de la UI puedan leer el tipo de error.
- `response.data?.choices?.[0]?.message?.content` — navega la estructura de respuesta de OpenAI de forma segura. `choices` es un array de completions; `[0]` es la primera (y única en este caso); `message.content` es el texto generado. Cada `?.` evita un crash si algún nivel de la jerarquía es `undefined`.
- `retryOperation(makeRequest, { maxRetries: 2, initialDelay: 500, shouldRetry: isTemporaryError })` — envuelve `makeRequest` con 2 reintentos adicionales, 500ms de delay inicial y usando `isTemporaryError` para decidir cuándo reintentar.
- `if (result.error)` — verifica si la respuesta de OpenAI es un error de incompatibilidad (como `{ "error": "No es posible generar una receta Vegana con carne" }`). Este es un "error de negocio" devuelto dentro del JSON, distinto al error HTTP.
- `err.isAIError = true` — flag personalizado que indica que el error vino de la lógica de la IA (no de la red). Los componentes lo usan para mostrar mensajes apropiados.
- `err.isCompatibilityError = result.error.includes('No es posible')` — flag adicional que indica incompatibilidad de categorías con ingredientes. Los componentes que lo reciben pueden mostrar un mensaje específico como "las categorías seleccionadas no son compatibles con los ingredientes".
- `const normalized = normalizeOpenAIResponse(result)` — pasa la respuesta por la función de normalización para validar y limpiar la estructura antes de retornarla.
- `if (!normalized.isValid)` — si la normalización detectó que la estructura de la receta es inválida (faltan campos obligatorios, tipos incorrectos, etc.), lanza un error.
- `return [normalized.recipe]` — retorna un array de 1 receta. Aunque siempre es 1, el array permite que el código del caller itere con `.map()` de forma uniforme y facilita la posible extensión futura a múltiples recetas.

### Bloque 4 — calculateDishShelfLife

```js
export const calculateDishShelfLife = async (ingredients) => {
  try {
    const ingredientsList = ingredients.map(ing => ing.name).join(', ');
    const prompt = `Como experto en seguridad alimentaria, calcula los días de refrigeración
      para: ${ingredientsList}. Responde SOLO con un número entero.`;

    const makeRequest = async () => {
      const response = await axios.post(API_URL, {
        model: 'gpt-4.1',
        messages: [
          { role: 'system', content: 'Responde SIEMPRE con un número entero. Nada más.' },
          { role: 'user', content: prompt }
        ],
        temperature: 0.2,
        max_completion_tokens: 10
      }, { headers: { 'Content-Type': 'application/json' }, timeout: 30000 });

      if (response.status >= 400) {
        const err = new Error(`Error ${response.status}`);
        err.status = response.status;
        throw err;
      }

      const text = response.data.choices[0].message.content.trim();
      const days = parseInt(text);
      if (isNaN(days) || days < 1 || days > 14) throw new Error('Día inválido');
      return days;
    };

    return await retryOperation(makeRequest, {
      maxRetries: 2, initialDelay: 500, shouldRetry: isTemporaryError
    });
  } catch (error) {
    return 3;
  }
};
```

- `export const calculateDishShelfLife = async (ingredients)` — función que pregunta a GPT cuántos días puede refrigerarse un platillo preparado dado su lista de ingredientes. Se llama cuando el usuario guarda una receta como platillo pendiente.
- `ingredients.map(ing => ing.name).join(', ')` — extrae solo los nombres de los ingredientes (sin cantidad ni unidad) y los une en una lista. GPT no necesita las cantidades para estimar la vida útil; solo le importa la combinación de ingredientes.
- `'Como experto en seguridad alimentaria, calcula los días de refrigeración para: ...'` — el prompt instruye a GPT a actuar como experto en seguridad alimentaria. El criterio de "días de refrigeración" es concreto y medible.
- `'Responde SIEMPRE con un número entero. Nada más.'` — mensaje de sistema muy restrictivo. Sin esta instrucción, GPT podría responder `"El platillo dura aproximadamente 4 días refrigerado, dependiendo..."`. Con la instrucción, responde `"4"`.
- `temperature: 0.2` — temperatura muy baja para que la respuesta sea determinista. La vida útil de un platillo es un hecho técnico, no algo que deba variar creativamente entre peticiones. Con `0.2`, GPT producirá esencialmente la misma respuesta cada vez para los mismos ingredientes.
- `max_completion_tokens: 10` — límite extremadamente bajo. Un número entero como `"3"` o `"7"` ocupa 1-2 tokens. Con 10 tokens, GPT no tiene espacio para agregar texto explicativo aunque quisiera. Esto reduce el costo de esta llamada a casi nada comparado con la generación de receta completa.
- `const text = response.data.choices[0].message.content.trim()` — extrae el texto de la respuesta y elimina espacios. Para esta llamada no se usa `safeParseJsonFromResponse` porque la respuesta esperada es un número, no JSON.
- `const days = parseInt(text)` — convierte el string al número entero. `parseInt` extrae el primer número que encuentre en el string, incluso si hay texto adicional (por ejemplo, si GPT devolvió `"4 días"` a pesar de la instrucción, `parseInt("4 días")` produce `4`).
- `if (isNaN(days) || days < 1 || days > 14) throw new Error('Día inválido')` — valida que el número esté dentro de un rango razonable. Menos de 1 día no tiene sentido para un platillo, y más de 14 días es poco realista para comida preparada refrigerada. Si está fuera de rango, lanza error para activar el reintento.
- `return await retryOperation(makeRequest, { maxRetries: 2, ... })` — igual que en `generateRecipe`, envuelve la petición con reintentos automáticos.
- `return 3` en el `catch` externo — si después de todos los reintentos la función falla (error de red, GPT no disponible, respuesta inválida repetida), retorna `3` días como valor conservador por defecto. Es mejor tener un valor razonable que lanzar un error al usuario en este punto, ya que el guardado del platillo es más importante que la precisión exacta de los días.

---

## src/services/foodDatabase.js

### Bloque 1 — Base de datos global y normalización

```js
import { collection, getDocs, addDoc, query, where } from 'firebase/firestore';
import { db } from './firebase';

export const foodDatabase = [
  { name: "Carnes frescas",     completo: 2,  fraccionado: 2,  category: "carnes" },
  { name: "Pollo crudo",        completo: 2,  fraccionado: 2,  category: "carnes" },
  { name: "Pollo cocido",       completo: 4,  fraccionado: 4,  category: "carnes" },
  { name: "Res cruda",          completo: 2,  fraccionado: 2,  category: "carnes" },
  { name: "Res cocida",         completo: 4,  fraccionado: 4,  category: "carnes" },
  { name: "Cerdo crudo",        completo: 2,  fraccionado: 2,  category: "carnes" },
  { name: "Cerdo cocido",       completo: 4,  fraccionado: 4,  category: "carnes" },
  { name: "Pescado fresco",     completo: 2,  fraccionado: 2,  category: "carnes" },
  { name: "Pescado cocido",     completo: 4,  fraccionado: 4,  category: "carnes" },
  { name: "Huevos con cáscara", completo: 28, fraccionado: 28, category: "lacteos" },
  { name: "Huevos duros",       completo: 7,  fraccionado: 7,  category: "lacteos" },
  { name: "Leche",              completo: 3,  fraccionado: 3,  category: "lacteos" },
  { name: "Yogur",              completo: 9,  fraccionado: 9,  category: "lacteos" },
  { name: "Queso fresco",       completo: 6,  fraccionado: 6,  category: "lacteos" },
  { name: "Queso curado",       completo: 23, fraccionado: 23, category: "lacteos" },
  { name: "Mantequilla",        completo: 18, fraccionado: 18, category: "lacteos" },
  { name: "Aguacate",           completo: 4,  fraccionado: 2,  category: "frutas" },
  { name: "Manzanas",           completo: 25, fraccionado: 25, category: "frutas" },
  { name: "Tomate",             completo: 7,  fraccionado: 7,  category: "verduras" },
  { name: "Cebolla",            completo: 14, fraccionado: 14, category: "verduras" },
  { name: "Ajo",                completo: 21, fraccionado: 21, category: "verduras" },
  { name: "Arroz cocido",       completo: 4,  fraccionado: 4,  category: "preparadas" },
  { name: "Pasta cocida",       completo: 4,  fraccionado: 4,  category: "preparadas" },
];

const normalizeText = (text) =>
  text.toLowerCase().normalize("NFD").replace(/[̀-ͯ]/g, "").trim();
```

- `import { collection, getDocs, addDoc, query, where }` — importa las funciones específicas de Firestore necesarias para leer y escribir en la base de datos personal del usuario. `collection` referencia una colección, `getDocs` lee documentos, `addDoc` agrega uno nuevo, `query` y `where` construyen consultas filtradas.
- `import { db } from './firebase'` — importa la instancia de Firestore inicializada en `firebase.js`. Al importar desde ese archivo central, no se crea una nueva conexión; se reutiliza la ya existente.
- `export const foodDatabase = [...]` — array exportado que contiene aproximadamente 90 alimentos con sus datos de vida útil. Se declara con `export` para que `Inventory.js` y `RecipeDetail.js` puedan acceder directamente a él cuando necesitan buscar datos de un alimento por nombre.
- Cada objeto `{ name, completo, fraccionado, category }` — representa un tipo de alimento. El campo `name` es el nombre exacto que debe coincidir con lo que el usuario escribe. `completo` son los días de vida útil cuando la cantidad es mayor o igual a 1 unidad entera. `fraccionado` son los días cuando la cantidad bajó de 1 (por ejemplo, medio aguacate se oxida más rápido que uno entero). `category` agrupa alimentos por tipo pero no se usa en la lógica actual.
- `{ name: "Aguacate", completo: 4, fraccionado: 2 }` — ejemplo donde `fraccionado` es diferente de `completo`. Un aguacate entero dura 4 días en el refrigerador, pero una vez cortado a la mitad solo dura 2 días porque la superficie expuesta se oxida.
- `{ name: "Huevos con cáscara", completo: 28, fraccionado: 28 }` — los huevos con cáscara duran igual completos o fraccionados porque la "cáscara" los protege; `fraccionado` igual a `completo` indica que no hay diferencia.
- `const normalizeText = (text)` — función interna (no exportada) que estandariza cualquier texto para poder compararlo sin importar acentos, mayúsculas o espacios extra. Se usa antes de cualquier búsqueda para que `"Aguacate"`, `"aguacate"` y `"aguácate"` sean equivalentes.
- `.toLowerCase()` — convierte todo a minúsculas. `"Tomate"` y `"tomate"` deben encontrar el mismo alimento.
- `.normalize("NFD")` — descompone los caracteres Unicode que tienen acento en dos partes: la letra base y el diacrítico (acento) como carácter separado. Por ejemplo, `"á"` se convierte en `"a"` + `"´"`.
- `.replace(/[̀-ͯ]/g, "")` — elimina todos los diacríticos que quedaron sueltos después de la descomposición NFD. El rango `[̀-ͯ]` corresponde a todos los modificadores de combinación Unicode (acentos, diéresis, tildes, etc.). El resultado es que `"á"` queda como `"a"`.
- `.trim()` — elimina espacios en blanco al inicio y al final. Si el usuario escribió `" Tomate "` con espacios, la búsqueda igual funciona.

### Bloque 2 — Búsqueda y cálculo global

```js
export const searchFood = (foodName) => {
  const normalizedSearch = normalizeText(foodName);
  return foodDatabase.find(food => normalizeText(food.name) === normalizedSearch);
};

export const getFoodSuggestions = (searchTerm) => {
  if (!searchTerm || searchTerm.length < 2) return [];
  const normalizedSearch = normalizeText(searchTerm);
  return foodDatabase
    .filter(food => normalizeText(food.name).includes(normalizedSearch))
    .slice(0, 5)
    .map(food => food.name);
};

export const calculateExpirationDate = (purchaseDate, foodName, quantity) => {
  const food = searchFood(foodName);
  if (!food) return null;
  const isFractioned = quantity < 1;
  const daysToAdd = isFractioned ? food.fraccionado : food.completo;
  if (daysToAdd === 0) return null;
  const dateOnly = typeof purchaseDate === 'string'
    ? purchaseDate.split('T')[0]
    : new Intl.DateTimeFormat('en-CA', { timeZone: 'America/Mexico_City' })
        .format(new Date(purchaseDate));
  const [y, m, d] = dateOnly.split('-').map(Number);
  const expDate = new Date(y, m - 1, d, 12);
  expDate.setDate(expDate.getDate() + daysToAdd);
  return expDate;
};
```

- `export const searchFood = (foodName)` — busca un alimento por nombre exacto (normalizado) en la base de datos global. Se usa en `RecipeDetail.js` cuando un ingrediente pasa de entero a fraccionado para recalcular su fecha de caducidad.
- `const normalizedSearch = normalizeText(foodName)` — normaliza el término de búsqueda antes de comparar. Así `"Pollo"` y `"pollo"` encuentran `"Pollo crudo"` o `"Pollo cocido"` si coinciden exactamente.
- `foodDatabase.find(food => normalizeText(food.name) === normalizedSearch)` — `Array.find` retorna el primer elemento que cumple la condición. La normalización se aplica también al nombre del alimento en la base de datos para que la comparación sea justa. Retorna `undefined` si no encuentra nada.
- `export const getFoodSuggestions = (searchTerm)` — retorna hasta 5 nombres de alimentos que contengan el término de búsqueda. Se usa para el autocompletado en el formulario de registro de ingredientes mientras el usuario escribe.
- `if (!searchTerm || searchTerm.length < 2) return []` — no busca si el campo está vacío o tiene menos de 2 caracteres. Con 1 solo carácter habría demasiados resultados poco útiles (casi todo coincide con una sola letra).
- `.filter(food => normalizeText(food.name).includes(normalizedSearch))` — a diferencia de `searchFood` que busca igualdad exacta, aquí se usa `includes` para coincidencia parcial. Si el usuario escribió `"pol"`, encuentra `"Pollo crudo"` y `"Pollo cocido"`.
- `.slice(0, 5)` — limita a 5 resultados máximo para no sobrecargar el dropdown de sugerencias con decenas de opciones.
- `.map(food => food.name)` — extrae solo el nombre de cada alimento. El dropdown muestra solo los nombres, no los días de vida útil.
- `export const calculateExpirationDate = (purchaseDate, foodName, quantity)` — calcula la fecha de caducidad de un ingrediente sumando los días de vida útil a la fecha de compra. Versión síncrona que solo busca en la base de datos global.
- `const isFractioned = quantity < 1` — determina si la cantidad es fraccionaria. En la app, una cantidad menor a 1 significa que el ingrediente está fraccionado (ej. medio aguacate = 0.5).
- `const daysToAdd = isFractioned ? food.fraccionado : food.completo` — selecciona los días correctos según si está fraccionado o no.
- `if (daysToAdd === 0) return null` — algunos alimentos pueden tener 0 días configurados, indicando que no se puede calcular su caducidad. En ese caso retorna `null` para que el formulario pida la fecha manualmente.
- `typeof purchaseDate === 'string' ? purchaseDate.split('T')[0] : ...` — extrae solo la parte de fecha. Si viene como ISO string completo `"2025-05-01T12:00:00.000Z"`, toma solo `"2025-05-01"`. Si viene como objeto `Date` o Timestamp, lo convierte a `"YYYY-MM-DD"` usando `Intl.DateTimeFormat` con la zona horaria de México.
- `const [y, m, d] = dateOnly.split('-').map(Number)` — descompone `"2025-05-01"` en `[2025, 5, 1]`.
- `const expDate = new Date(y, m - 1, d, 12)` — construye la fecha al mediodía (hora 12) en hora local. El mediodía evita que cambios de horario de verano (que ocurren a las 2:00 AM) afecten el cálculo del día.
- `expDate.setDate(expDate.getDate() + daysToAdd)` — suma los días de vida útil. `getDate()` retorna el día del mes actual; `setDate()` actualiza la fecha sumando los días. JavaScript maneja automáticamente el desbordamiento de meses (ej. día 31 + 5 días pasa correctamente al mes siguiente).

### Bloque 3 — Base de datos personal y funciones completas

```js
export const searchPersonalFood = async (foodName, userId) => {
  if (!userId) return null;
  const normalizedSearch = normalizeText(foodName);
  try {
    const q = query(
      collection(db, `users/${userId}/personalFoodDatabase`),
      where('normalizedName', '==', normalizedSearch)
    );
    const querySnapshot = await getDocs(q);
    if (!querySnapshot.empty) return querySnapshot.docs[0].data();
    return null;
  } catch (error) { return null; }
};

export const searchFoodComplete = async (foodName, userId) => {
  const globalFood = searchFood(foodName);
  if (globalFood) return globalFood;
  return await searchPersonalFood(foodName, userId);
};

export const addToPersonalFoodDatabase = async (foodName, days, userId) => {
  if (!userId) return false;
  const normalizedName = normalizeText(foodName);
  try {
    const existing = await searchPersonalFood(foodName, userId);
    if (existing) return false;
    await addDoc(collection(db, `users/${userId}/personalFoodDatabase`), {
      name: foodName,
      normalizedName,
      completo: days,
      fraccionado: days,
      category: 'personal',
      createdAt: new Date().toISOString()
    });
    return true;
  } catch (error) { return false; }
};

export const getFoodSuggestionsComplete = async (searchTerm, userId) => {
  const globalSuggestions = getFoodSuggestions(searchTerm);
  if (!userId) return globalSuggestions;
  try {
    const normalizedSearch = normalizeText(searchTerm);
    const querySnapshot = await getDocs(
      collection(db, `users/${userId}/personalFoodDatabase`)
    );
    const personalSuggestions = querySnapshot.docs
      .map(doc => doc.data().name)
      .filter(name => normalizeText(name).includes(normalizedSearch));
    const combined = [...new Set([...globalSuggestions, ...personalSuggestions])];
    return combined.slice(0, 5);
  } catch (error) { return globalSuggestions; }
};

export const calculateExpirationDateComplete = async (purchaseDate, foodName, quantity, userId) => {
  const food = await searchFoodComplete(foodName, userId);
  if (!food) return null;
  const isFractioned = quantity < 1;
  const daysToAdd = isFractioned ? food.fraccionado : food.completo;
  if (daysToAdd === 0) return null;
  const dateOnly = typeof purchaseDate === 'string'
    ? purchaseDate.split('T')[0]
    : new Intl.DateTimeFormat('en-CA', { timeZone: 'America/Mexico_City' })
        .format(new Date(purchaseDate));
  const [y, m, d] = dateOnly.split('-').map(Number);
  const expDate = new Date(y, m - 1, d, 12);
  expDate.setDate(expDate.getDate() + daysToAdd);
  return expDate;
};
```

- `export const searchPersonalFood = async (foodName, userId)` — busca un alimento en la base de datos personal del usuario almacenada en Firestore. Es `async` porque Firestore es una operación de red que toma tiempo.
- `if (!userId) return null` — si no hay usuario autenticado, no hay base de datos personal a la que buscar. Retorna `null` inmediatamente sin hacer ninguna petición a Firestore.
- `const q = query(collection(db, \`users/${userId}/personalFoodDatabase\`), where('normalizedName', '==', normalizedSearch))` — construye una consulta filtrada de Firestore. `collection(db, path)` referencia la subcolección `personalFoodDatabase` dentro del documento del usuario. `where('normalizedName', '==', normalizedSearch)` filtra solo los documentos donde el campo `normalizedName` es exactamente igual al término buscado normalizado. Firestore ejecuta este filtro en el servidor, sin descargar todos los documentos.
- `const querySnapshot = await getDocs(q)` — ejecuta la consulta y espera los resultados. `querySnapshot` es un objeto iterable que contiene los documentos que coincidieron.
- `if (!querySnapshot.empty) return querySnapshot.docs[0].data()` — si encontró al menos un resultado, retorna los datos del primer documento (el campo `data()` convierte el documento de Firestore a un objeto JavaScript plano). Se asume que no hay duplicados porque `addToPersonalFoodDatabase` verifica antes de insertar.
- `return null` — si la consulta no encontró ningún documento con ese nombre normalizado, retorna `null`.
- `catch (error) { return null }` — si Firestore falla (sin conexión, reglas de seguridad, etc.), retorna `null` silenciosamente en lugar de lanzar el error. Así el flujo de la app puede continuar (preguntando la fecha manualmente) en lugar de romperse.
- `export const searchFoodComplete = async (foodName, userId)` — versión completa que busca en ambas bases de datos. Primero intenta la base global (síncrona, instantánea, sin costo de red); solo si no encuentra, consulta la base personal en Firestore (asíncrona, requiere red). Esta estrategia prioriza la velocidad.
- `const globalFood = searchFood(foodName)` — búsqueda síncrona en el array en memoria. Si encuentra, retorna inmediatamente sin tocar Firestore.
- `if (globalFood) return globalFood` — cortocircuito: si la base global tiene el alimento, no se hace la petición a Firestore.
- `export const addToPersonalFoodDatabase = async (foodName, days, userId)` — guarda un nuevo alimento con su vida útil en la base de datos personal. Se llama cuando el usuario registra un ingrediente con fecha de caducidad manual; así la app aprende la vida útil de ese alimento para futuros registros.
- `const normalizedName = normalizeText(foodName)` — normaliza el nombre antes de guardar. Este campo `normalizedName` es el que se usa en las queries de `where('normalizedName', '==', ...)`.
- `const existing = await searchPersonalFood(foodName, userId)` — verifica que el alimento no exista ya antes de insertar. Evita duplicados en la base de datos personal.
- `if (existing) return false` — si ya existe, retorna `false` indicando que no se insertó. El llamador usa este retorno para decidir si mostrar el mensaje "(alimento agregado a tu base de datos personal)".
- `await addDoc(collection(db, \`users/${userId}/personalFoodDatabase\`), { ... })` — inserta el documento en Firestore. `addDoc` genera automáticamente un ID único para el documento. Se guarda tanto el `name` original (para mostrar al usuario) como `normalizedName` (para queries).
- `completo: days, fraccionado: days` — para alimentos personales, se usa el mismo número de días para entero y fraccionado. No hay información suficiente para distinguir, así que se asume igual para ambos.
- `category: 'personal'` — marca estos alimentos como personales para distinguirlos de los globales si en el futuro se quisieran listar por separado.
- `export const getFoodSuggestionsComplete = async (searchTerm, userId)` — versión completa del autocompletado que combina sugerencias de ambas bases de datos.
- `const globalSuggestions = getFoodSuggestions(searchTerm)` — obtiene sugerencias de la base global primero (síncrono).
- `if (!userId) return globalSuggestions` — si no hay usuario, retorna solo las sugerencias globales sin consultar Firestore.
- `await getDocs(collection(db, \`users/${userId}/personalFoodDatabase\`))` — descarga TODOS los documentos de la base personal. A diferencia de `searchPersonalFood`, aquí no se filtra en el servidor porque se necesitan todos los nombres para el autocompletado parcial. La base personal de un usuario típicamente tiene pocos documentos (decenas), por lo que descargar todos es aceptable.
- `.map(doc => doc.data().name)` — extrae solo el campo `name` de cada documento. Se usa el nombre original (no el normalizado) para mostrarlo al usuario.
- `.filter(name => normalizeText(name).includes(normalizedSearch))` — filtra en el cliente (en JavaScript) los nombres que contienen el término de búsqueda.
- `const combined = [...new Set([...globalSuggestions, ...personalSuggestions])]` — combina ambas listas. `new Set(...)` elimina duplicados automáticamente (si el mismo alimento está en ambas bases). El spread `[...new Set(...)]` convierte el Set de vuelta a array.
- `return combined.slice(0, 5)` — limita el total combinado a 5 sugerencias.
- `catch (error) { return globalSuggestions }` — si Firestore falla, retorna al menos las sugerencias globales en lugar de nada.
- `export const calculateExpirationDateComplete = async (purchaseDate, foodName, quantity, userId)` — versión `async` de `calculateExpirationDate` que busca en ambas bases antes de calcular. La lógica de cálculo es idéntica a la versión síncrona; la única diferencia es que `searchFoodComplete` es asíncrono.

---

## src/App.js

### Bloque 1 — Constantes de navegación

```js
import React, { useState, useEffect, useRef, useCallback } from 'react';
import { onAuthStateChanged } from 'firebase/auth';
import { auth } from './services/firebase';
import Login from './components/Auth/Login';
import Register from './components/Auth/Register';
import Recovery from './components/Auth/Recovery';
import Inventory from './components/Ingredients/Inventory';
import MainMenu from './components/Main/MainMenu';
import RegisterIngredient from './components/Ingredients/RegisterIngredient';
import GenerateRecipe from './components/Recipes/GenerateRecipe';
import RecipeResults from './components/Recipes/RecipeResults';
import RecipeDetail from './components/Recipes/RecipeDetail';
import PendingDishes from './components/Dishes/PendingDishes';
import History from './components/Dishes/History';

const VIEW_PATHS = {
  'login':               '/',
  'register':            '/registro',
  'recovery':            '/recuperar-cuenta',
  'menu':                '/menu',
  'inventory':           '/inventario',
  'register-ingredient': '/registrar-ingrediente',
  'generate-recipe':     '/generar-receta-con-ia',
  'recipe-results':      '/resultados',
  'recipe-detail':       '/detalle-receta',
  'pending-dishes':      '/platillos-pendientes',
  'history':             '/historial',
};

const PATH_TO_VIEW = Object.fromEntries(
  Object.entries(VIEW_PATHS).map(([view, path]) => [path, view])
);

const PUBLIC_VIEWS = new Set(['login', 'register', 'recovery']);

function getViewFromPath(pathname) {
  return PATH_TO_VIEW[pathname] ?? 'login';
}
```

- `import { useState, useEffect, useRef, useCallback }` — importa cuatro hooks de React. `useState` maneja estado reactivo. `useEffect` ejecuta código con efectos secundarios (como suscribirse a eventos o llamar APIs) después del render. `useRef` crea referencias mutables que no causan re-render cuando cambian. `useCallback` memoriza funciones para que su referencia no cambie entre renders.
- `import { onAuthStateChanged }` — función de Firebase que registra un listener que se activa cada vez que el estado de autenticación cambia: cuando el usuario inicia sesión, cierra sesión, o cuando Firebase restaura silenciosamente una sesión previa al cargar la app.
- Importaciones de componentes — cada vista de la app es un componente separado. `App.js` los importa todos porque es el único componente que decide cuál renderizar según `currentView`.
- `const VIEW_PATHS = { ... }` — objeto que mapea cada nombre interno de vista a su URL correspondiente. Este mapeo es la única fuente de verdad sobre qué URL corresponde a qué vista. Si se quiere cambiar la URL de una vista, solo se cambia aquí.
- `'login': '/'` — la vista de login corresponde a la raíz `/`. Es la primera pantalla que ve un usuario no autenticado.
- `'generate-recipe': '/generar-receta-con-ia'` — las URLs están en español para que sean descriptivas para el usuario. Usar un nombre descriptivo en la URL mejora la experiencia si el usuario la comparte o guarda.
- `const PATH_TO_VIEW = Object.fromEntries(Object.entries(VIEW_PATHS).map(([view, path]) => [path, view]))` — construye el mapa inverso automáticamente. `Object.entries(VIEW_PATHS)` convierte el objeto a un array de pares `[clave, valor]`. `.map(([view, path]) => [path, view])` intercambia cada par. `Object.fromEntries` convierte el array de vuelta a objeto. El resultado es `{ '/': 'login', '/registro': 'register', ... }`.
- `const PUBLIC_VIEWS = new Set(['login', 'register', 'recovery'])` — conjunto de vistas que no requieren autenticación. Se usa `Set` en lugar de `Array` porque la verificación `PUBLIC_VIEWS.has(view)` es O(1) en un Set pero O(n) en un Array.
- `function getViewFromPath(pathname)` — dado el `pathname` actual del URL (ej. `"/inventario"`), retorna el nombre interno de la vista (`"inventory"`). Si el pathname no está en el mapa (URL desconocida), retorna `'login'` como fallback seguro.
- `PATH_TO_VIEW[pathname] ?? 'login'` — el operador `??` (nullish coalescing) retorna el lado derecho solo si el lado izquierdo es `null` o `undefined`. A diferencia de `||`, no descarta `0` o `false`. Aquí retorna `'login'` si `pathname` no existe en el mapa.

### Bloque 2 — Estado y refs del componente App

```js
function App() {
  const [currentView, setCurrentViewRaw] = useState(() =>
    getViewFromPath(window.location.pathname)
  );

  const setCurrentView = useCallback((view) => {
    const path = VIEW_PATHS[view] ?? '/';
    window.history.pushState({ view }, '', path);
    setCurrentViewRaw(view);
  }, []);

  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  const registrationInProgress = useRef(false);
  const loginInProgress = useRef(false);
  const isInitialLoad = useRef(true);

  const [selectedRecipe, setSelectedRecipe] = useState(() => {
    try {
      const saved = sessionStorage.getItem('selectedRecipe');
      return saved ? JSON.parse(saved) : null;
    } catch { return null; }
  });

  const [generatedRecipes, setGeneratedRecipes] = useState([]);
  const [currentRecipeIndex, setCurrentRecipeIndex] = useState(0);
```

- `const [currentView, setCurrentViewRaw] = useState(() => getViewFromPath(window.location.pathname))` — inicializa la vista actual leyendo la URL del navegador. La función `() => getViewFromPath(...)` es una inicialización lazy: se ejecuta una sola vez cuando el componente monta, no en cada render. Esto es importante porque `window.location.pathname` solo necesita leerse una vez al inicio.
- `setCurrentViewRaw` — es el setter directo de React. Se nombra con `Raw` para indicar que es el setter interno y no debe usarse directamente en los componentes hijos; deben usar `setCurrentView` que también actualiza la URL.
- `const setCurrentView = useCallback((view) => { ... }, [])` — wrapper del setter que también actualiza la URL del navegador. `useCallback` memoriza la función y la misma referencia se mantiene entre renders (el array vacío `[]` como dependencias significa que nunca se recrea). Esto es importante porque `setCurrentView` se pasa como prop a múltiples componentes hijos y si cambiara en cada render causaría re-renders innecesarios en todos esos hijos.
- `window.history.pushState({ view }, '', path)` — cambia la URL en la barra del navegador sin recargar la página. El primer argumento `{ view }` es el objeto de estado asociado a esta entrada del historial (accesible en el evento `popstate`). El segundo argumento `''` es el título (obsoleto, se deja vacío). El tercero `path` es la nueva URL.
- `const [user, setUser] = useState(null)` — almacena el objeto de usuario de Firebase cuando está autenticado. Firebase retorna un objeto con propiedades como `uid`, `email`, `displayName`. Cuando el usuario cierra sesión, se pone a `null`.
- `const [loading, setLoading] = useState(true)` — comienza en `true` porque al iniciar la app no se sabe aún si hay una sesión activa. Firebase necesita un momento para verificar el token de sesión almacenado en el navegador. Mientras `loading` es `true`, la app muestra un spinner en lugar de redirigir al login precipitadamente.
- `const registrationInProgress = useRef(false)` — ref booleana que indica si hay un proceso de registro en curso. Se usa como "guard" para que `onAuthStateChanged` no navegue automáticamente cuando Firebase detecta al nuevo usuario recién registrado (porque ya hay un modal de éxito manejando la navegación). Es `useRef` y no `useState` porque cambiar su valor no debe provocar un re-render.
- `const loginInProgress = useRef(false)` — guard análogo al de registro, pero para el flujo de login. Cuando el usuario hace login exitoso, `Login.js` activa este guard antes de mostrar el modal de bienvenida, y `onAuthStateChanged` lo respeta.
- `const isInitialLoad = useRef(true)` — distingue la primera llamada de `onAuthStateChanged` (al cargar la app) de las siguientes. Solo en la primera llamada se restaura la sesión silenciosamente. Los cambios posteriores (login/logout) los manejan sus propios callbacks.
- `const [selectedRecipe, setSelectedRecipe] = useState(() => { ... })` — inicialización lazy que lee de `sessionStorage`. Si el usuario está en `/detalle-receta` y recarga la página, Firebase restaura la sesión pero React pierde el estado. `sessionStorage` guarda la receta seleccionada para que sobreviva la recarga.
- `sessionStorage.getItem('selectedRecipe')` — `sessionStorage` es similar a `localStorage` pero solo dura mientras la pestaña del navegador está abierta. Se elige sobre `localStorage` porque las recetas son datos de sesión, no configuración permanente.
- `JSON.parse(saved)` — convierte el string almacenado en `sessionStorage` de vuelta a objeto JavaScript.
- `catch { return null }` — si `sessionStorage` no está disponible (algunos navegadores lo bloquean en modo privado) o el JSON está corrupto, retorna `null` sin romper la app.
- `const [generatedRecipes, setGeneratedRecipes] = useState([])` — array de recetas generadas por la IA. Se almacena en App.js (estado global) para que tanto `GenerateRecipe.js` (que las escribe) como `RecipeResults.js` (que las muestra) puedan acceder a ellas.
- `const [currentRecipeIndex, setCurrentRecipeIndex] = useState(0)` — índice de la receta actualmente visible en el carrusel de `RecipeResults.js`. Se almacena en App.js para que el índice no se pierda si el usuario navega entre vistas.

### Bloque 3 — Efectos de sincronización

```js
  useEffect(() => {
    const handlePopState = () => {
      const view = getViewFromPath(window.location.pathname);
      if (!auth.currentUser && !PUBLIC_VIEWS.has(view)) {
        setCurrentView('login');
      } else {
        setCurrentViewRaw(view);
      }
    };
    window.addEventListener('popstate', handlePopState);
    return () => window.removeEventListener('popstate', handlePopState);
  }, [setCurrentView]);

  useEffect(() => {
    if (!loading && !user && !PUBLIC_VIEWS.has(currentView)) {
      setCurrentView('login');
    }
  }, [loading, user, currentView, setCurrentView]);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      setLoading(false);
      if (currentUser && isInitialLoad.current &&
          !registrationInProgress.current && !loginInProgress.current) {
        const urlView = getViewFromPath(window.location.pathname);
        const targetView = PUBLIC_VIEWS.has(urlView) ? 'menu' : urlView;
        setCurrentView(targetView);
      } else if (!currentUser) {
        setCurrentView('login');
      }
      isInitialLoad.current = false;
    });
    return () => unsubscribe();
  }, [setCurrentView]);

  useEffect(() => {
    try {
      if (selectedRecipe) {
        sessionStorage.setItem('selectedRecipe', JSON.stringify(selectedRecipe));
      } else {
        sessionStorage.removeItem('selectedRecipe');
      }
    } catch {}
  }, [selectedRecipe]);
```

- `useEffect(() => { ... }, [setCurrentView])` para `popstate` — registra un listener para el evento `popstate` del navegador. `popstate` se dispara cuando el usuario hace click en el botón "atrás" o "adelante" del navegador. Sin este listener, el URL cambiaría pero `currentView` no, mostrando la vista incorrecta. El array de dependencias `[setCurrentView]` incluye `setCurrentView` porque la función se usa dentro del efecto (regla de hooks).
- `const view = getViewFromPath(window.location.pathname)` — lee la URL actual después de que el navegador la cambió al presionar atrás/adelante.
- `if (!auth.currentUser && !PUBLIC_VIEWS.has(view))` — si el usuario no está autenticado e intenta navegar con el botón atrás a una vista privada (ej. `/inventario`), se redirige al login. `auth.currentUser` es el acceso síncrono al usuario actual de Firebase, sin necesidad de `await`.
- `setCurrentViewRaw(view)` — se usa el setter directo (sin `pushState`) porque el navegador ya actualizó la URL; no hay que actualizarla de nuevo.
- `return () => window.removeEventListener('popstate', handlePopState)` — función de cleanup del efecto. React la ejecuta cuando el componente desmonta o cuando las dependencias cambian. Evita que el listener quede activo después de que el componente ya no existe, lo que causaría memory leaks y comportamiento inesperado.
- Segundo `useEffect` (guard de autenticación) — se ejecuta cada vez que `loading`, `user` o `currentView` cambia. Si la app terminó de cargar (`!loading`), no hay usuario y la vista actual es privada, redirige al login. Es una red de seguridad para casos edge donde el estado llega a ser inconsistente.
- Tercer `useEffect` (Firebase auth listener) — el efecto más crítico de la app. Registra un listener con `onAuthStateChanged` que se activa en tres momentos: al cargar la app (para restaurar sesión), cuando el usuario hace login y cuando hace logout.
- `setUser(currentUser)` — actualiza el estado con el usuario de Firebase. Si hay sesión activa, `currentUser` es el objeto de usuario; si no, es `null`.
- `setLoading(false)` — marca que Firebase ya respondió, permitiendo que la app muestre contenido real.
- `if (currentUser && isInitialLoad.current && !registrationInProgress.current && !loginInProgress.current)` — condición para restaurar sesión silenciosamente. Todos los guards deben ser `false` para que App.js tome el control de la navegación. Si `loginInProgress` es `true`, significa que `Login.js` está manejando la navegación mediante su modal de bienvenida.
- `const targetView = PUBLIC_VIEWS.has(urlView) ? 'menu' : urlView` — si el usuario llega directamente a `/` (login) pero tiene sesión activa, se redirige al menú. Si llega a una URL privada como `/inventario`, se restaura directamente a esa vista.
- `else if (!currentUser) { setCurrentView('login') }` — si `onAuthStateChanged` reporta que no hay usuario (logout o sesión expirada), redirige al login inmediatamente.
- `isInitialLoad.current = false` — se ejecuta después de la primera llamada, independientemente del resultado. A partir de aquí, todas las llamadas subsiguientes de `onAuthStateChanged` son cambios de estado reales, no la restauración inicial.
- `return () => unsubscribe()` — Firebase retorna una función de "unsubscribe" al llamar `onAuthStateChanged`. Esta función cancela el listener. El cleanup del `useEffect` la llama cuando el componente desmonta.
- Cuarto `useEffect` (sincronización con `sessionStorage`) — se ejecuta cada vez que `selectedRecipe` cambia. Si hay receta, la serializa a JSON y la guarda. Si no hay receta, la elimina. El `try/catch` vacío maneja el caso donde `sessionStorage` no está disponible.

### Bloque 4 — Logout y renderizado

```js
  const handleLogout = async () => {
    try {
      await auth.signOut();
      loginInProgress.current = false;
      registrationInProgress.current = false;
      setSelectedRecipe(null);
      setCurrentView('login');
      setUser(null);
    } catch (error) {
      alert('Error al cerrar sesión');
    }
  };

  const renderView = () => {
    if (!user && !PUBLIC_VIEWS.has(currentView)) return <Login setCurrentView={setCurrentView} />;

    switch (currentView) {
      case 'login':
        return <Login setCurrentView={setCurrentView}
          onLoginComplete={() => { loginInProgress.current = true; }}
          onLoginReset={() => { loginInProgress.current = false; }} />;
      case 'register':
        return <Register setCurrentView={setCurrentView}
          onRegistrationComplete={() => { registrationInProgress.current = true; }}
          onRegistrationReset={() => { registrationInProgress.current = false; }} />;
      case 'recovery':
        return <Recovery setCurrentView={setCurrentView} />;
      case 'menu':
        return <MainMenu setCurrentView={setCurrentView} onLogout={handleLogout} />;
      case 'register-ingredient':
        return <RegisterIngredient setCurrentView={setCurrentView} userId={user?.uid} />;
      case 'inventory':
        return <Inventory setCurrentView={setCurrentView} userId={user?.uid} />;
      case 'generate-recipe':
        return <GenerateRecipe setCurrentView={setCurrentView} userId={user?.uid}
          setGeneratedRecipes={setGeneratedRecipes}
          setCurrentRecipeIndex={setCurrentRecipeIndex} />;
      case 'recipe-results':
        return <RecipeResults setCurrentView={setCurrentView}
          recipes={generatedRecipes} currentIndex={currentRecipeIndex}
          setCurrentIndex={setCurrentRecipeIndex}
          setSelectedRecipe={setSelectedRecipe}
          setGeneratedRecipes={setGeneratedRecipes} />;
      case 'recipe-detail':
        return <RecipeDetail setCurrentView={setCurrentView}
          recipe={selectedRecipe} userId={user?.uid} />;
      case 'pending-dishes':
        return <PendingDishes setCurrentView={setCurrentView} userId={user?.uid} />;
      case 'history':
        return <History setCurrentView={setCurrentView} userId={user?.uid} />;
      default:
        return <Login setCurrentView={setCurrentView} />;
    }
  };

  return <div className="App">{renderView()}</div>;
}

export default App;
```

- `const handleLogout = async ()` — función que cierra la sesión del usuario de forma limpia. Es `async` porque `auth.signOut()` es una operación asíncrona que espera que Firebase confirme el cierre de sesión.
- `await auth.signOut()` — llama al método de Firebase que invalida el token de sesión en el servidor y limpia los datos locales de autenticación.
- `loginInProgress.current = false` y `registrationInProgress.current = false` — resetea los guards de autenticación para que queden limpios para la próxima sesión.
- `setSelectedRecipe(null)` — limpia la receta seleccionada. El efecto de `sessionStorage` la eliminará automáticamente.
- `setCurrentView('login')` — navega al login y actualiza la URL.
- `setUser(null)` — actualiza el estado del usuario localmente sin esperar a `onAuthStateChanged`. Esto hace que la UI responda inmediatamente sin esperar la confirmación de Firebase.
- `alert('Error al cerrar sesión')` — en el caso raro de que Firebase falle al cerrar sesión, muestra un alerta nativo del navegador. Se usa `alert` (en lugar del Modal personalizado) porque si Firebase falla, posiblemente el estado de la app está en un estado indeterminado.
- `const renderView = ()` — función que decide qué componente renderizar según `currentView`. Se extrae en su propia función para mantener el `return` del componente limpio.
- `if (!user && !PUBLIC_VIEWS.has(currentView)) return <Login setCurrentView={setCurrentView} />` — guard de seguridad: si por alguna razón el estado es inconsistente (no hay usuario pero la vista es privada), renderiza el login sin props de autenticación como fallback.
- `case 'login': return <Login ... onLoginComplete={() => { loginInProgress.current = true; }}` — activa el guard `loginInProgress` cuando el usuario completa el login. La función de flecha `() => { ... }` se crea en cada render pero esto es aceptable porque `Login.js` no memoriza esta prop.
- `onLoginReset={() => { loginInProgress.current = false; }}` — si el usuario cierra el modal de bienvenida con la X (sin navegar), desactiva el guard para que la próxima acción de Firebase funcione normalmente.
- `case 'register-ingredient': return <RegisterIngredient ... userId={user?.uid} />` — `user?.uid` usa optional chaining: si por alguna razón `user` fuera `null` en una vista privada, retorna `undefined` en lugar de lanzar un `TypeError`. Aunque el guard debería prevenir esto, la defensive programming evita crashes.
- `case 'generate-recipe': return <GenerateRecipe ... setGeneratedRecipes={setGeneratedRecipes} setCurrentRecipeIndex={setCurrentRecipeIndex} />` — pasa los seteadores del estado de recetas a `GenerateRecipe`. Cuando el usuario genera recetas, este componente las almacena directamente en el estado de App.js, no en su propio estado, para que `RecipeResults.js` pueda accederlas.
- `case 'recipe-results': return <RecipeResults ... recipes={generatedRecipes} currentIndex={currentRecipeIndex} setCurrentIndex={setCurrentRecipeIndex} setSelectedRecipe={setSelectedRecipe} setGeneratedRecipes={setGeneratedRecipes} />` — pasa tanto los datos (recetas, índice) como los seteadores. `RecipeResults` puede agregar más recetas al array (`setGeneratedRecipes`) cuando el usuario regenera.
- `default: return <Login setCurrentView={setCurrentView} />` — fallback para cualquier valor desconocido de `currentView`. Nunca debería activarse en condiciones normales, pero es una red de seguridad.
- `return <div className="App">{renderView()}</div>` — el render final envuelve la vista activa en un `div` con clase `App`. Este div sirve como punto de referencia para estilos CSS globales.
- `export default App` — exporta el componente como exportación por defecto, que es lo que `index.js` importa para montar la aplicación.

---

## src/components/Auth/Login.js

```js
import React, { useState, useMemo } from 'react';
import { signInWithEmailAndPassword } from 'firebase/auth';
import { auth } from '../../services/firebase';
import Modal from '../../utils/Modal';
import { LogIn, Eye, EyeOff } from 'lucide-react';

const FOOD_DECORATIONS = [
  '🥗','🍳','🥘','🍲','🥙','🧆','🌮','🌯','🍕','🍔',
  '🍟','🥪','🧀','🥚','🥓','🥩','🍗','🍖','🐟','🦐',
  '🥬','🥦','🥕','🌽','🥒','🍅','🥔','🧅','🧄','🍎'
];

const Login = ({ setCurrentView, onLoginComplete, onLoginReset }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const [showPassword, setShowPassword] = useState(false);
  const [modalConfig, setModalConfig] = useState({
    isOpen: false, type: 'success', title: '', message: '', onConfirm: () => {}
  });

  const showModal = (type, title, message, onConfirm = () => {}) => {
    setModalConfig({ isOpen: true, type, title, message, onConfirm });
  };

  const closeModal = () => {
    onLoginReset?.();
    setModalConfig(prev => ({ ...prev, isOpen: false }));
  };

  const handleLogin = async (e) => {
    e.preventDefault();
    setError('');
    setLoading(true);
    if (!email || !password) {
      setError('Por favor completa todos los campos');
      setLoading(false);
      return;
    }
    try {
      const userCredential = await signInWithEmailAndPassword(auth, email, password);
      const displayName = userCredential.user.displayName || email;
      onLoginComplete?.();
      showModal('welcome', 'Sesión iniciada', `Bienvenido, ${displayName}.`,
        () => setCurrentView('menu'));
    } catch (error) {
      switch (error.code) {
        case 'auth/invalid-credential':
        case 'auth/user-not-found':
        case 'auth/wrong-password':
          setError('Correo o contraseña incorrectos'); break;
        case 'auth/invalid-email':
          setError('Correo electrónico inválido'); break;
        case 'auth/too-many-requests':
          setError('Demasiados intentos fallidos. Intenta más tarde'); break;
        case 'auth/user-disabled':
          setError('Esta cuenta ha sido deshabilitada'); break;
        default:
          setError('Error al iniciar sesión. Verifica tus credenciales');
      }
    } finally { setLoading(false); }
  };

  const decorationElements = useMemo(() =>
    FOOD_DECORATIONS.slice(0, 15).map((emoji) => ({
      emoji,
      style: {
        top: `${Math.random() * 80 + 10}%`,
        left: `${Math.random() * 80 + 10}%`,
        animationDelay: `${Math.random() * 3}s`,
        fontSize: `${Math.random() * 1.5 + 1.5}rem`,
      }
    }))
  , []);
```

- `import { useMemo }` — hook que memoriza el resultado de una función costosa. Se usa para calcular las posiciones aleatorias de los emojis decorativos solo una vez, no en cada render.
- `import { signInWithEmailAndPassword }` — función de Firebase Auth que valida un email y contraseña contra la base de datos de usuarios del proyecto.
- `import { LogIn, Eye, EyeOff }` — íconos de Lucide: `LogIn` para el botón de submit, `Eye` y `EyeOff` para alternar visibilidad de la contraseña.
- `const FOOD_DECORATIONS = [...]` — array de emojis de alimentos. Se define fuera del componente para que no se recree en cada render (es una constante estática).
- `const Login = ({ setCurrentView, onLoginComplete, onLoginReset })` — el componente recibe tres props. `setCurrentView` es la función de App.js para navegar. `onLoginComplete` es un callback que se llama cuando el login es exitoso, antes de mostrar el modal. `onLoginReset` se llama cuando el modal se cierra sin haber navegado.
- `const [email, setEmail] = useState('')` — estado del campo de correo electrónico, controlado por React (controlled component).
- `const [password, setPassword] = useState('')` — estado del campo de contraseña.
- `const [error, setError] = useState('')` — mensaje de error que se muestra bajo el formulario. Comienza vacío; se rellena cuando hay un error de autenticación.
- `const [loading, setLoading] = useState(false)` — controla si el formulario está esperando respuesta de Firebase. Cuando es `true`, el botón de submit se deshabilita para evitar múltiples peticiones simultáneas.
- `const [showPassword, setShowPassword] = useState(false)` — alterna entre mostrar la contraseña como texto plano o como puntos ocultos.
- `const [modalConfig, setModalConfig] = useState({ isOpen: false, ... })` — estado del modal de bienvenida. En lugar de múltiples estados separados (`isModalOpen`, `modalTitle`, etc.), se agrupa todo en un objeto para simplificar.
- `const showModal = (type, title, message, onConfirm = () => {})` — función auxiliar para abrir el modal con la configuración correcta. `onConfirm = () => {}` es un valor por defecto vacío para cuando no se necesita acción al confirmar.
- `const closeModal = ()` — cierra el modal y notifica a App.js que el flujo de login terminó sin navegar. Esto es importante para que App.js resetee el guard `loginInProgress`.
- `onLoginReset?.()` — llama la función solo si fue pasada como prop (`?.` evita error si es `undefined`). Si el usuario cerró el modal con la X, App.js necesita saber que el flujo terminó sin completarse.
- `setModalConfig(prev => ({ ...prev, isOpen: false }))` — usa un updater funcional que recibe el estado anterior. `...prev` copia todos los campos actuales del modal (tipo, título, mensaje) y solo cambia `isOpen` a `false`. Esto evita capturar un estado stale en la closure.
- `const handleLogin = async (e)` — manejador del submit del formulario. Es `async` porque `signInWithEmailAndPassword` es asíncrono.
- `e.preventDefault()` — cancela el comportamiento por defecto del formulario HTML, que sería recargar la página al hacer submit. En React se maneja el submit con JavaScript en lugar de dejar que el navegador lo procese.
- `setError('')` — limpia el error anterior antes de intentar el login. Si no se limpia, un error de un intento anterior podría seguir visible mientras se procesa el nuevo intento.
- `setLoading(true)` — activa el estado de carga para deshabilitar el botón y mostrar el spinner.
- `if (!email || !password)` — validación básica del lado del cliente antes de llamar a Firebase. Evita peticiones innecesarias cuando el formulario está incompleto.
- `const userCredential = await signInWithEmailAndPassword(auth, email, password)` — llama a Firebase Auth con las credenciales. Si son correctas, retorna un objeto `UserCredential` con información del usuario. Si son incorrectas, lanza un error con un `error.code` específico.
- `const displayName = userCredential.user.displayName || email` — usa el nombre del usuario si está configurado; si no, muestra el email. Algunos usuarios podrían no tener `displayName` si hubo algún problema al registrarse.
- `onLoginComplete?.()` — activa el guard en App.js ANTES de abrir el modal. Esto es el orden crítico: si se abriera el modal primero, Firebase podría detectar al usuario autenticado y navegar automáticamente antes de que el modal sea visible.
- `showModal('welcome', ...)` — muestra el modal de bienvenida. El `onConfirm` que se pasa es `() => setCurrentView('menu')`, que navega al menú cuando el usuario presiona "Entendido".
- `switch (error.code)` — mapea los códigos de error de Firebase (que son strings internos) a mensajes en español comprensibles para el usuario. `error.code` puede ser `'auth/invalid-credential'`, `'auth/too-many-requests'`, etc.
- `case 'auth/invalid-credential'` y `case 'auth/user-not-found'` y `case 'auth/wrong-password'` — todos muestran el mismo mensaje genérico para no revelar al usuario si el email existe o no (práctica de seguridad).
- `case 'auth/too-many-requests'` — Firebase bloquea temporalmente cuentas con muchos intentos fallidos para prevenir ataques de fuerza bruta.
- `finally { setLoading(false) }` — se ejecuta siempre, haya o no error. Garantiza que el loading se desactive aunque la petición falle a mitad de camino.
- `const decorationElements = useMemo(() => FOOD_DECORATIONS.slice(0, 15).map((emoji) => ({ ... })), [])` — calcula las posiciones y tamaños aleatorios de los 15 emojis del fondo una sola vez. `Math.random()` se llama durante ese cálculo inicial. Si no se usara `useMemo`, se llamaría en cada render y los emojis "saltarían" a posiciones diferentes constantemente.
- `top: \`${Math.random() * 80 + 10}%\`` — posición vertical aleatoria entre 10% y 90% de la pantalla. El rango `10-90` evita que los emojis queden cortados en los bordes.
- `animationDelay: \`${Math.random() * 3}s\`` — retraso aleatorio de hasta 3 segundos para que las animaciones de pulse no estén sincronizadas (se ve más natural).
- `fontSize: \`${Math.random() * 1.5 + 1.5}rem\`` — tamaño aleatorio entre 1.5rem y 3rem para crear profundidad visual.

---

## src/components/Auth/Register.js

```js
import React, { useState, useRef, useMemo } from 'react';
import { createUserWithEmailAndPassword, updateProfile } from 'firebase/auth';
import { doc, setDoc } from 'firebase/firestore';
import { auth, db } from '../../services/firebase';
import Modal from '../../utils/Modal';

const Register = ({ setCurrentView, onRegistrationComplete, onRegistrationReset }) => {
  const [formData, setFormData] = useState({
    username: '', email: '', password: '', confirmPassword: '', birthdate: ''
  });
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const [showPassword, setShowPassword] = useState(false);
  const [showConfirmPassword, setShowConfirmPassword] = useState(false);
  const [modalConfig, setModalConfig] = useState({
    isOpen: false, type: 'success', title: '', message: '', onConfirm: () => {}
  });
  const registrationCompleted = useRef(false);

  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const closeModal = () => {
    if (!registrationCompleted.current) onRegistrationReset?.();
    setModalConfig(prev => ({ ...prev, isOpen: false }));
  };

  const validateForm = () => {
    if (!formData.username || !formData.email || !formData.password ||
        !formData.confirmPassword || !formData.birthdate) {
      setError('Por favor complete todos los campos'); return false;
    }
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(formData.email)) {
      setError('Formato de correo electrónico inválido'); return false;
    }
    const passwordRegex = /^(?=.*[A-Z])(?=.*\d).{8,}$/;
    if (!passwordRegex.test(formData.password)) {
      setError('La contraseña debe tener al menos 8 caracteres, una mayúscula y un número');
      return false;
    }
    if (formData.password !== formData.confirmPassword) {
      setError('Las contraseñas no coinciden'); return false;
    }
    const birth = new Date(formData.birthdate);
    const today = new Date();
    const minDate = new Date();
    minDate.setFullYear(today.getFullYear() - 120);
    if (birth > today || birth < minDate) {
      setError('Fecha de nacimiento inválida'); return false;
    }
    return true;
  };

  const handleRegister = async (e) => {
    e.preventDefault();
    setError('');
    if (!validateForm()) return;
    setLoading(true);
    try {
      const userCredential = await createUserWithEmailAndPassword(
        auth, formData.email, formData.password);
      const user = userCredential.user;
      await updateProfile(user, { displayName: formData.username });
      try {
        await setDoc(doc(db, 'users', user.uid), {
          username: formData.username,
          email: formData.email,
          birthdate: formData.birthdate,
          createdAt: new Date().toISOString()
        });
      } catch (firestoreError) {
        try { await user.delete(); } catch {}
        throw firestoreError;
      }
      registrationCompleted.current = true;
      onRegistrationComplete?.();
      setModalConfig({ isOpen: true, type: 'welcome', title: 'Cuenta creada',
        message: 'Tu cuenta ha sido creada exitosamente. Bienvenido a Ready to Cook.',
        onConfirm: () => setCurrentView('menu') });
    } catch (error) {
      switch (error.code) {
        case 'auth/email-already-in-use':
          setError('Ya existe una cuenta con este correo electrónico'); break;
        case 'auth/weak-password':
          setError('La contraseña es muy débil'); break;
        default:
          setError('Error al crear la cuenta. Intente nuevamente');
      }
    } finally { setLoading(false); }
  };
```

- `import { createUserWithEmailAndPassword, updateProfile }` — `createUserWithEmailAndPassword` crea la cuenta en Firebase Auth. `updateProfile` actualiza el perfil del usuario (en este caso, el `displayName`) después de crearlo.
- `import { doc, setDoc }` — `doc` crea una referencia a un documento específico de Firestore. `setDoc` escribe un documento en esa referencia, creándolo si no existe o sobreescribiéndolo si ya existe.
- `const [formData, setFormData] = useState({ username: '', email: '', ... })` — un solo estado agrupa todos los campos del formulario. Esto simplifica el manejo: en lugar de 5 estados separados, hay uno solo que es un objeto.
- `const registrationCompleted = useRef(false)` — ref que rastrea si el registro se completó exitosamente. Es `useRef` (no `useState`) porque su cambio no debe causar un re-render. Su propósito es modificar el comportamiento de `closeModal` según si el registro fue o no exitoso.
- `const handleChange = (e)` — manejador genérico para todos los inputs del formulario. Funciona gracias a que cada `<input>` tiene el atributo `name` que coincide con la clave del objeto `formData`.
- `setFormData({ ...formData, [e.target.name]: e.target.value })` — crea un nuevo objeto copiando todos los campos del estado actual (`...formData`) y actualizando solo el campo que cambió. `[e.target.name]` es una computed property: si el usuario escribió en el campo `name="email"`, esto actualiza `formData.email`.
- `const closeModal = ()` — lógica especial: `if (!registrationCompleted.current) onRegistrationReset?.()` solo llama al reset si el registro NO se completó. Si el usuario hace login y cierra el modal con X, se llama al reset. Pero si el registro fue exitoso y el usuario cierra el modal de bienvenida, NO se llama al reset porque `registrationCompleted.current` es `true`.
- `const validateForm = ()` — valida todos los campos antes de hacer cualquier petición a Firebase. Hacerlo en el cliente evita llamadas innecesarias a la API cuando hay errores evidentes.
- `if (!formData.username || !formData.email || ...)` — verifica que ningún campo esté vacío. El operador `!` convierte strings vacíos `""` a `true` (porque `!""` es `true`), así que esta condición detecta campos vacíos.
- `const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/` — expresión regular para validar el formato del email. `^[^\s@]+` = uno o más caracteres que no sean espacio ni @. `@` = el símbolo @. `[^\s@]+` = dominio (sin espacios ni @). `\.` = punto literal. `[^\s@]+$` = extensión (sin espacios ni @).
- `const passwordRegex = /^(?=.*[A-Z])(?=.*\d).{8,}$/` — expresión regular con lookaheads. `(?=.*[A-Z])` = debe haber al menos una mayúscula en algún lugar. `(?=.*\d)` = debe haber al menos un dígito. `.{8,}` = mínimo 8 caracteres en total. Los lookaheads verifican condiciones sin consumir caracteres, permitiendo múltiples requisitos simultáneos.
- `const birth = new Date(formData.birthdate)` — convierte el string de la fecha de nacimiento a objeto Date para poder compararlo.
- `minDate.setFullYear(today.getFullYear() - 120)` — calcula la fecha mínima aceptable: 120 años atrás. Una fecha anterior sería imposiblemente antigua.
- `if (birth > today || birth < minDate)` — la fecha no puede ser en el futuro (nadie ha nacido mañana) ni hace más de 120 años.
- `const userCredential = await createUserWithEmailAndPassword(auth, formData.email, formData.password)` — crea la cuenta en Firebase Auth. Firebase valida el email y la fortaleza de la contraseña del lado del servidor. Si el email ya existe, lanza `'auth/email-already-in-use'`.
- `await updateProfile(user, { displayName: formData.username })` — guarda el nombre de usuario en el perfil de Firebase Auth. `displayName` es un campo estándar de Firebase Auth accesible desde cualquier parte de la app con `auth.currentUser.displayName`.
- `await setDoc(doc(db, 'users', user.uid), { ... })` — crea el documento del usuario en Firestore. `doc(db, 'users', user.uid)` crea una referencia al documento con ID igual al UID del usuario. Guardar datos adicionales en Firestore (como `birthdate`) es necesario porque Firebase Auth solo permite campos estándar (email, displayName, photoURL).
- `createdAt: new Date().toISOString()` — guarda la fecha y hora de creación de la cuenta en formato ISO. Útil para estadísticas y para saber desde cuándo es usuario.
- `catch (firestoreError) { try { await user.delete(); } catch {} throw firestoreError; }` — rollback en caso de fallo de Firestore. Si se creó la cuenta en Auth pero el documento en Firestore falló, se elimina la cuenta de Auth para evitar una cuenta huérfana (existe en Auth pero sin datos en Firestore). El `try/catch` interno alrededor de `user.delete()` evita que un fallo al eliminar oculte el error original de Firestore.
- `registrationCompleted.current = true` — se marca ANTES de mostrar el modal. Así si `closeModal` se llama mientras el modal está abierto, sabrá que el registro fue exitoso.
- `onRegistrationComplete?.()` — activa el guard en App.js para que `onAuthStateChanged` no navegue automáticamente (el modal de bienvenida maneja la navegación).

---

## src/components/Auth/Recovery.js

```js
import React, { useState } from 'react';
import { sendPasswordResetEmail } from 'firebase/auth';
import { auth } from '../../services/firebase';

const Recovery = ({ setCurrentView }) => {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  const [success, setSuccess] = useState(false);
  const [loading, setLoading] = useState(false);

  const handleRecovery = async (e) => {
    e.preventDefault();
    setError(''); setSuccess(false); setLoading(true);
    if (!email) {
      setError('Por favor ingrese su correo electrónico');
      setLoading(false); return;
    }
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      setError('Formato de correo electrónico inválido');
      setLoading(false); return;
    }
    try {
      await sendPasswordResetEmail(auth, email);
      setSuccess(true);
      setEmail('');
    } catch (error) {
      switch (error.code) {
        case 'auth/user-not-found':
          setError('No existe una cuenta asociada a este correo'); break;
        case 'auth/too-many-requests':
          setError('Demasiados intentos. Intente más tarde'); break;
        default:
          setError('Error al enviar el correo. Intente nuevamente');
      }
    } finally { setLoading(false); }
  };
```

- `import { sendPasswordResetEmail }` — función de Firebase que envía un correo electrónico al usuario con un enlace para restablecer su contraseña. El enlace tiene tiempo de expiración configurado en la consola de Firebase.
- `const [success, setSuccess] = useState(false)` — estado booleano que controla qué pantalla se muestra: `false` muestra el formulario, `true` muestra la confirmación de éxito. Este componente usa esta técnica en lugar del componente `Modal` compartido.
- `setError(''); setSuccess(false); setLoading(true)` — limpia estados previos al inicio de cada intento. Si el usuario intenta una vez, recibe un error, y luego intenta de nuevo, los estados anteriores deben limpiarse.
- `if (!email)` — validación de campo vacío con mensaje específico sobre qué campo falta.
- `emailRegex.test(email)` — valida el formato antes de llamar a Firebase para evitar peticiones con emails claramente inválidos.
- `await sendPasswordResetEmail(auth, email)` — envía el correo de recuperación. Firebase valida el email contra sus registros. Si el email no está registrado, Firebase puede lanzar `'auth/user-not-found'`.
- `setSuccess(true)` — cambia la pantalla al estado de éxito. El componente renderiza condicionalmente según este estado.
- `setEmail('')` — limpia el campo de email después del éxito por higiene visual y para facilitar un nuevo intento si el usuario lo necesita.
- `case 'auth/user-not-found'` — Firebase confirma que el email no está en su base de datos. En algunas implementaciones se evita este mensaje por privacidad (para no revelar si un email está registrado), pero aquí se muestra para mejor UX.
- La pantalla de éxito (en el JSX, no mostrado aquí) incluye un aviso de revisar la carpeta de spam porque muchos servidores de correo marcan los emails de Firebase como spam.

---

## src/components/Main/MainMenu.js

```js
import React from 'react';
import { LogOut } from 'lucide-react';

const MainMenu = ({ setCurrentView, onLogout }) => {
  return (
    <div className="min-h-screen bg-food-pattern p-6 relative overflow-hidden">
      <div className="max-w-4xl mx-auto relative z-10">
        <div className="flex justify-end mb-4">
          <button onClick={onLogout}
            className="flex items-center gap-2 bg-white px-4 py-2 rounded-xl shadow-md
              hover:shadow-lg hover:scale-105 transition-all duration-300
              text-food-700 font-medium border-2 border-transparent hover:border-food-300">
            <LogOut size={18} />
            Cerrar sesión
          </button>
        </div>

        <div className="text-center mb-8">
          <div className="text-6xl mb-4 animate-bounce">🥗</div>
          <h1 className="text-4xl font-bold text-food-800 mb-2 font-cooking">
            ¡Bienvenido a Ready to Cook!
          </h1>
          <p className="text-food-600 text-lg">¿Qué quieres hacer hoy con tus alimentos?</p>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          <button onClick={() => setCurrentView('generate-recipe')}
            className="md:col-span-2 bg-gradient-to-br from-food-50 via-cream-50 to-food-100
              border-2 border-food-600 rounded-2xl p-10 hover:animate-card-float
              transition-all duration-300 group relative overflow-hidden">
            <div className="bg-food-200 w-24 h-24 rounded-full flex items-center justify-center mb-4
              group-hover:bg-food-300 group-hover:scale-[1.15] mx-auto transition-all duration-300">
              <span className="text-5xl">🍳</span>
            </div>
            <h3 className="text-2xl font-bold text-center">Generar recetas con IA</h3>
            <p className="text-food-600 text-center">Crea recetas con tus ingredientes</p>
          </button>

          <button onClick={() => setCurrentView('register-ingredient')} className="card-food p-8 ...">
            <div className="bg-fresh-100 w-20 h-20 rounded-full flex items-center justify-center mb-4 mx-auto">
              <span className="text-4xl">🥬</span>
            </div>
            <h3 className="text-xl font-bold text-center">Registrar ingredientes</h3>
          </button>

          <button onClick={() => setCurrentView('inventory')} className="card-food p-8 ...">
            <span className="text-4xl">📦</span>
            <h3 className="text-xl font-bold text-center">Gestionar inventario</h3>
          </button>

          <button onClick={() => setCurrentView('pending-dishes')} className="card-food p-8 ...">
            <span className="text-4xl">⌛</span>
            <h3 className="text-xl font-bold text-center">Platillos almacenados</h3>
          </button>

          <button onClick={() => setCurrentView('history')} className="card-food p-8 ...">
            <span className="text-4xl">📚</span>
            <h3 className="text-xl font-bold text-center">Historial de recetas</h3>
          </button>
        </div>

        <div className="mt-8 text-center">
          <p className="text-food-500 text-sm font-medium">
            🌱 «Cada alimento ahorrado es un paso hacia un mundo mejor»
          </p>
        </div>
      </div>
    </div>
  );
};

export default MainMenu;
```

- `const MainMenu = ({ setCurrentView, onLogout })` — componente sin estado local. No tiene `useState` ni efectos; es puramente presentacional. Toda su lógica se delega al padre (App.js) mediante los props.
- `import { LogOut }` — ícono de Lucide para el botón de cerrar sesión.
- `min-h-screen bg-food-pattern` — la pantalla ocupa al menos el 100% de la altura del viewport y usa el fondo con patrón de cocina definido en `index.css`.
- `max-w-4xl mx-auto` — centra el contenido y lo limita a 896px de ancho para que no se vea excesivamente ancho en pantallas grandes.
- `relative z-10` — posiciona el contenido sobre los emojis decorativos del fondo (que tienen `z-index` menor).
- `onClick={onLogout}` — llama directamente a la función de App.js que cierra sesión. El componente no sabe cómo cerrar sesión; solo sabe que existe una función que lo hace.
- `hover:shadow-lg hover:scale-105 transition-all duration-300` — al hacer hover el botón crece ligeramente y su sombra aumenta, dando retroalimentación visual de interactividad.
- `border-2 border-transparent hover:border-food-300` — el borde existe siempre (evita que el elemento "salte" al aparecer el borde en hover) pero es transparente hasta el hover.
- `md:col-span-2` — en pantallas medianas (≥768px) la tarjeta de "Generar recetas" ocupa las 2 columnas del grid, haciéndola visualmente destacada sobre las demás.
- `hover:animate-card-float` — al hacer hover aplica la animación `card-float` definida en `tailwind.config.js`, que hace flotar suavemente la tarjeta.
- `group` — clase de Tailwind que permite que elementos hijos reaccionen al hover del padre. Por ejemplo, `group-hover:bg-food-300` cambia el color del círculo cuando se hace hover en la tarjeta completa, no solo en el círculo.
- `group-hover:scale-[1.15]` — el círculo del ícono crece al 115% cuando se hace hover en la tarjeta. `[1.15]` es una escala arbitraria especificada con la sintaxis de corchetes de Tailwind.
- `relative overflow-hidden` — necesario para que el pseudo-elemento de brillo (`absolute inset-0`) quede contenido dentro de la tarjeta sin desbordarse.
- `() => setCurrentView('...')` — cada botón navega a una vista diferente llamando a la función de App.js con el nombre de la vista destino.
- Frase motivacional al final — texto estático que refuerza el propósito de la app (reducir desperdicio de alimentos).

---

## src/components/Ingredients/RegisterIngredient.js

### Bloque 1 — Estado y autocompletado

```js
import React, { useState } from 'react';
import { collection, addDoc } from 'firebase/firestore';
import { db } from '../../services/firebase';
import {
  getFoodSuggestionsComplete,
  calculateExpirationDateComplete,
  addToPersonalFoodDatabase
} from '../../services/foodDatabase';
import { toISODateString, getTodayISO } from '../../utils/dateCalculations';

const normalizeDateForFirestore = (isoDate) => {
  const [year, month, day] = isoDate.split('-');
  return new Date(year, month - 1, day, 12).toISOString();
};

const RegisterIngredient = ({ setCurrentView, userId }) => {
  const [formData, setFormData] = useState({
    name: '', quantity: '', unit: 'Piezas',
    purchaseDate: getTodayISO(), expirationDate: ''
  });
  const [suggestions, setSuggestions] = useState([]);
  const [showSuggestions, setShowSuggestions] = useState(false);
  const [error, setError] = useState('');
  const [success, setSuccess] = useState('');
  const [loading, setLoading] = useState(false);
  const [manualExpiration, setManualExpiration] = useState(false);

  const handleNameChange = async (e) => {
    const value = e.target.value;
    setFormData({ ...formData, name: value });
    if (value.length >= 2) {
      const foodSuggestions = await getFoodSuggestionsComplete(value, userId);
      setSuggestions(foodSuggestions);
      setShowSuggestions(true);
    } else {
      setSuggestions([]); setShowSuggestions(false);
    }
  };

  const selectSuggestion = (suggestion) => {
    setFormData({ ...formData, name: suggestion });
    setSuggestions([]); setShowSuggestions(false);
  };
```

- `const normalizeDateForFirestore = (isoDate)` — convierte un string `"YYYY-MM-DD"` a un ISO string completo con hora 12:00 PM local. Es esencial para que las fechas se guarden correctamente en Firestore; sin esta función, guardar `"2025-05-01"` como string simple causaría que Firestore lo interprete como UTC medianoche, lo que en México corresponde a las 18:00 del 30 de abril.
- `const [year, month, day] = isoDate.split('-')` — divide el string en sus componentes. El resultado son strings, no números.
- `new Date(year, month - 1, day, 12).toISOString()` — crea una fecha al mediodía local. Los argumentos del constructor `Date` se interpretan en hora local (no UTC). `.toISOString()` convierte a ISO string UTC, pero como la fecha fue creada al mediodía en UTC-6, el ISO string resultante tiene hora `18:00:00Z`, garantizando que al leerla después siempre corresponda al día correcto en México.
- `unit: 'Piezas'` — unidad por defecto preseleccionada. "Piezas" es la más común para la mayoría de los alimentos que un usuario registraría.
- `purchaseDate: getTodayISO()` — inicializa la fecha de compra con la fecha de hoy en zona horaria México. El usuario casi siempre compra el ingrediente el mismo día que lo registra.
- `const [manualExpiration, setManualExpiration] = useState(false)` — controla si el campo de fecha de caducidad es visible. Por defecto está oculto porque el sistema la calcula automáticamente; el usuario puede activarlo si quiere ingresar una fecha diferente.
- `const handleNameChange = async (e)` — es `async` porque `getFoodSuggestionsComplete` consulta Firestore. El manejador actualiza el estado del campo y luego busca sugerencias.
- `if (value.length >= 2)` — solo busca sugerencias cuando el usuario ha escrito 2 o más caracteres. Con 1 carácter habría demasiados resultados poco específicos.
- `await getFoodSuggestionsComplete(value, userId)` — busca en la BD global (síncrono) y en la personal del usuario (Firestore), combinando ambos resultados.
- `setSuggestions(foodSuggestions)` — actualiza el estado con los nombres sugeridos para mostrar en el dropdown.
- `setShowSuggestions(true)` — hace visible el dropdown de sugerencias.
- `const selectSuggestion = (suggestion)` — cuando el usuario hace click en una sugerencia, actualiza el campo de nombre y cierra el dropdown.
- `setSuggestions([])` — limpia el array de sugerencias para que el dropdown no tenga contenido si se volviera a abrir accidentalmente.

### Bloque 2 — Envío del formulario

```js
  const handleSubmit = async (e) => {
    e.preventDefault();
    setError(''); setSuccess('');
    if (!formData.name || !formData.quantity || !formData.purchaseDate) {
      setError('Por favor completa todos los campos obligatorios'); return;
    }
    if (parseFloat(formData.quantity) <= 0) {
      setError('La cantidad debe ser mayor a 0'); return;
    }
    setLoading(true);
    try {
      const normalizedPurchaseDate = normalizeDateForFirestore(formData.purchaseDate);
      let finalExpirationDate = formData.expirationDate;
      let foodAddedToPersonalDB = false;

      if (!manualExpiration || !formData.expirationDate) {
        const calculatedDate = await calculateExpirationDateComplete(
          formData.purchaseDate, formData.name, parseFloat(formData.quantity), userId);
        if (calculatedDate) {
          finalExpirationDate = toISODateString(calculatedDate);
        } else {
          setError('No se pudo calcular la caducidad. Por favor ingrésela manualmente.');
          setManualExpiration(true); setLoading(false); return;
        }
      } else {
        const diffTime = Math.abs(
          new Date(formData.expirationDate) - new Date(formData.purchaseDate));
        const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
        foodAddedToPersonalDB = await addToPersonalFoodDatabase(
          formData.name, diffDays, userId);
      }

      const normalizedExpirationDate = normalizeDateForFirestore(finalExpirationDate);
      const finalQuantity = parseFloat(parseFloat(formData.quantity).toFixed(2));
      const expirationDateType = (manualExpiration && formData.expirationDate)
        ? 'manual' : 'calculada';

      await addDoc(collection(db, `users/${userId}/ingredients`), {
        name: formData.name,
        quantity: finalQuantity,
        unit: formData.unit,
        purchaseDate: normalizedPurchaseDate,
        expirationDate: normalizedExpirationDate,
        isFractioned: parseFloat(formData.quantity) < 1,
        expirationDateType,
        createdAt: new Date().toISOString(),
        userId
      });

      let successMessage = '¡Ingrediente registrado exitosamente!';
      if (foodAddedToPersonalDB) {
        successMessage += ' (alimento agregado a tu base de datos personal)';
      }
      setSuccess(successMessage);
      setFormData({ name: '', quantity: '', unit: 'Piezas',
        purchaseDate: getTodayISO(), expirationDate: '' });
      setManualExpiration(false);
      setTimeout(() => setSuccess(''), 3000);
    } catch (error) {
      setError('Error al registrar el ingrediente. Intente nuevamente.');
    } finally { setLoading(false); }
  };
```

- `parseFloat(formData.quantity) <= 0` — convierte el string del input a número y verifica que sea positivo. Cantidades negativas o cero no tienen sentido en un inventario.
- `if (!manualExpiration || !formData.expirationDate)` — calcula automáticamente si el usuario no activó el modo manual, o si lo activó pero no ingresó fecha. Esto permite que incluso en modo manual, si el campo queda vacío, se intente calcular automáticamente.
- `await calculateExpirationDateComplete(formData.purchaseDate, formData.name, parseFloat(formData.quantity), userId)` — busca el alimento en ambas BDs y suma los días de vida útil a la fecha de compra. Retorna un objeto `Date` o `null`.
- `if (calculatedDate)` — si se encontró el alimento y se calculó la fecha, la usa.
- `setError('No se pudo calcular...'); setManualExpiration(true)` — si el alimento no está en ninguna BD, activa el modo manual automáticamente y muestra un mensaje indicando al usuario que debe ingresar la fecha. Esto es un flujo de degradación elegante.
- `const diffTime = Math.abs(new Date(formData.expirationDate) - new Date(formData.purchaseDate))` — cuando el usuario ingresó la fecha manualmente, calcula cuántos milisegundos hay entre la compra y la caducidad. `Math.abs` asegura un valor positivo aunque el orden sea incorrecto.
- `const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))` — convierte milisegundos a días, redondeando hacia arriba. `1000 * 60 * 60 * 24` = milisegundos en un día.
- `await addToPersonalFoodDatabase(formData.name, diffDays, userId)` — guarda el alimento y su vida útil calculada en la BD personal. La próxima vez que el usuario registre el mismo alimento, el sistema lo encontrará y no pedirá la fecha.
- `foodAddedToPersonalDB` — booleano que indica si se guardó en la BD personal. Se usa al final para personalizar el mensaje de éxito.
- `parseFloat(parseFloat(formData.quantity).toFixed(2))` — doble conversión para redondear con precisión. `.toFixed(2)` retorna un string con 2 decimales exactos. El segundo `parseFloat` convierte ese string de vuelta a número. Por ejemplo, `1.999999999` se convierte a `2.00` (string) y luego a `2` (número).
- `isFractioned: parseFloat(formData.quantity) < 1` — marca si el ingrediente está fraccionado. Esta propiedad se usa en `RecipeDetail.js` para decidir si recalcular la fecha de caducidad cuando la cantidad baja de 1.
- `expirationDateType: (manualExpiration && formData.expirationDate) ? 'manual' : 'calculada'` — guarda si la fecha fue ingresada manualmente o calculada por el sistema. Este campo se usa en `Inventory.js` para mostrar un ícono de recalcular y para decidir si respetar o recalcular la fecha al editar la cantidad.
- `addDoc(collection(db, \`users/${userId}/ingredients\`), {...})` — inserta el documento en la subcolección `ingredients` del usuario. `addDoc` genera un ID único automáticamente (a diferencia de `setDoc` que requiere especificar el ID).
- `userId` — aunque se puede obtener el `userId` del documento (está en la ruta de la colección), se guarda también como campo para facilitar posibles consultas futuras.
- `setTimeout(() => setSuccess(''), 3000)` — el mensaje de éxito desaparece automáticamente después de 3 segundos. Esto evita que el usuario tenga que cerrar el mensaje manualmente.

---

## src/components/Ingredients/Inventory.js

### Bloque 1 — Carga y ordenamiento

```js
import React, { useState, useEffect } from 'react';
import { collection, getDocs, deleteDoc, doc, updateDoc } from 'firebase/firestore';
import { db } from '../../services/firebase';
import { isPriority, isExpired, formatDate, toISODateString } from '../../utils/dateCalculations';
import { searchFood, calculateExpirationDateComplete } from '../../services/foodDatabase';
import Modal from '../../utils/Modal';

const Inventory = ({ setCurrentView, userId }) => {
  const [ingredients, setIngredients] = useState([]);
  const [loading, setLoading] = useState(true);
  const [editingId, setEditingId] = useState(null);
  const [editForm, setEditForm] = useState({});

  useEffect(() => {
    loadIngredients();
    const interval = setInterval(() => { loadIngredients(); }, 60000);
    return () => clearInterval(interval);
  }, [userId]);

  const loadIngredients = async () => {
    try {
      const querySnapshot = await getDocs(
        collection(db, `users/${userId}/ingredients`));
      const ingredientsData = querySnapshot.docs.map(doc => ({
        id: doc.id, ...doc.data()
      }));
      const sorted = ingredientsData.sort((a, b) => {
        const aExpired = isExpired(a.expirationDate);
        const bExpired = isExpired(b.expirationDate);
        const aPriority = isPriority(a.expirationDate);
        const bPriority = isPriority(b.expirationDate);
        if (aExpired && !bExpired) return 1;
        if (!aExpired && bExpired) return -1;
        if (aPriority && !bPriority) return -1;
        if (!aPriority && bPriority) return 1;
        return 0;
      });
      setIngredients(sorted);
    } catch (error) {
      showModal('error', 'Error', 'Error al cargar el inventario');
    } finally { setLoading(false); }
  };
```

- `import { deleteDoc, updateDoc }` — `deleteDoc` elimina un documento de Firestore por su referencia. `updateDoc` actualiza campos específicos de un documento existente sin afectar los demás campos.
- `const [editingId, setEditingId] = useState(null)` — almacena el ID del ingrediente que está siendo editado en la tabla. Cuando es `null`, ningún ingrediente está en modo edición. Cuando tiene un valor, la fila de ese ingrediente muestra inputs en lugar de texto.
- `const [editForm, setEditForm] = useState({})` — almacena temporalmente los valores del formulario de edición inline. Cuando el usuario termina de editar, estos valores se guardan en Firestore.
- `useEffect(() => { loadIngredients(); const interval = setInterval(..., 60000); return () => clearInterval(interval); }, [userId])` — este efecto hace dos cosas: carga los ingredientes al montar el componente y establece un intervalo de 60 segundos. El intervalo recarga los ingredientes periódicamente para actualizar los estados de caducidad en tiempo real (sin que el usuario tenga que recargar la página). `return () => clearInterval(interval)` cancela el intervalo cuando el componente desmonta o cuando `userId` cambia.
- `querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }))` — convierte cada documento de Firestore a un objeto JavaScript. `doc.id` es el ID del documento (generado por `addDoc`). `doc.data()` retorna un objeto con todos los campos del documento. Se combinan en un solo objeto para poder acceder al ID junto con los datos.
- Lógica de ordenamiento `sort((a, b) => { ... })` — ordena los ingredientes en tres grupos. Los caducados van al final (son informativos, no urgentes). Los prioritarios (≤ 3 días) van primero porque necesitan atención inmediata. Los frescos van en el medio. Dentro de cada grupo, el orden es el original de Firestore.
- `if (aExpired && !bExpired) return 1` — si `a` está caducado pero `b` no, `a` va después de `b` (al final). `return 1` significa "a es mayor (va después)".
- `if (aPriority && !bPriority) return -1` — si `a` es prioritario pero `b` no, `a` va antes de `b`. `return -1` significa "a es menor (va primero)".
- `return 0` — si ambos tienen el mismo estado, mantiene el orden relativo original.

### Bloque 2 — Edición inline

```js
  const startEdit = (ingredient) => {
    setEditingId(ingredient.id);
    setEditForm({
      quantity: parseFloat(ingredient.quantity || 0).toFixed(2),
      unit: ingredient.unit,
      expirationDate: ingredient.expirationDate,
      purchaseDate: ingredient.purchaseDate,
      name: ingredient.name,
      expirationDateType: ingredient.expirationDateType || 'calculada',
      dateManuallyChanged: false
    });
  };

  const cancelEdit = () => { setEditingId(null); setEditForm({}); };

  const normalizeDateForFirestore = (isoDate) => {
    const [year, month, day] = isoDate.split('-');
    return new Date(year, month - 1, day, 12).toISOString();
  };

  const saveEdit = (id) => {
    const newQuantity = parseFloat(editForm.quantity);
    if (!newQuantity || newQuantity < 0.25) {
      showModal('error', 'Cantidad inválida',
        'La cantidad debe ser mayor o igual a 0.25');
      return;
    }
    showModal('confirm', 'Guardar cambios', '¿Desea guardar los cambios realizados?',
      async () => {
        try {
          const ingredientRef = doc(db, `users/${userId}/ingredients`, id);
          const isFractioned = newQuantity < 1;
          const rawDate = editForm.expirationDate || '';
          const dateOnly = rawDate.length > 10 ? rawDate.split('T')[0] : rawDate;
          let newExpirationDate = dateOnly ? normalizeDateForFirestore(dateOnly) : rawDate;
          let newExpirationDateType = editForm.expirationDateType || 'calculada';

          if (editForm.dateManuallyChanged) {
            newExpirationDateType = 'manual';
          } else if (newExpirationDateType !== 'manual') {
            const purchaseDateStr = typeof editForm.purchaseDate === 'string'
              ? editForm.purchaseDate.split('T')[0]
              : editForm.purchaseDate;
            const calculatedDate = await calculateExpirationDateComplete(
              purchaseDateStr, editForm.name, newQuantity, userId);
            if (calculatedDate) {
              newExpirationDate = calculatedDate.toISOString();
              newExpirationDateType = 'calculada';
            }
          }

          await updateDoc(ingredientRef, {
            quantity: parseFloat(newQuantity.toFixed(2)),
            unit: editForm.unit,
            expirationDate: newExpirationDate,
            expirationDateType: newExpirationDateType,
            isFractioned
          });

          setIngredients(prev => prev.map(ing =>
            ing.id === id ? {
              ...ing,
              quantity: parseFloat(newQuantity.toFixed(2)),
              unit: editForm.unit,
              expirationDate: newExpirationDate,
              expirationDateType: newExpirationDateType,
              isFractioned
            } : ing
          ));
          setEditingId(null); setEditForm({});
          showModal('success', '¡Actualizado!', 'Ingrediente actualizado exitosamente');
        } catch (error) {
          showModal('error', 'Error', 'Error al actualizar el ingrediente');
        }
      }
    );
  };
```

- `startEdit(ingredient)` — inicializa el formulario de edición con los valores actuales del ingrediente. Esto permite que el usuario vea los valores actuales y los modifique sin tener que escribirlos desde cero.
- `parseFloat(ingredient.quantity || 0).toFixed(2)` — formatea la cantidad con 2 decimales para el input. `|| 0` maneja el caso donde la cantidad es `null` o `undefined`. El resultado es un string como `"2.50"`.
- `expirationDateType: ingredient.expirationDateType || 'calculada'` — si el campo no existe en el documento (ingredientes guardados antes de agregar esta funcionalidad), usa `'calculada'` como valor por defecto para mantener compatibilidad hacia atrás.
- `dateManuallyChanged: false` — flag que comienza en `false` y se activa si el usuario cambia el input de fecha. Permite distinguir entre "el usuario no tocó la fecha" (recalcular automáticamente si la cantidad cambió) y "el usuario cambió la fecha explícitamente" (respetar el nuevo valor).
- `cancelEdit()` — limpia `editingId` y `editForm`. Al poner `editingId` en `null`, React deja de renderizar los inputs y muestra el texto normal.
- `normalizeDateForFirestore` (redefinida localmente) — misma lógica que en `RegisterIngredient.js`. Se redefine localmente para que este componente sea independiente.
- `if (!newQuantity || newQuantity < 0.25)` — validación ANTES de mostrar el modal de confirmación. Esto cumple el invariante del modal: no hacer validaciones dentro de `onConfirm` porque `onConfirm` se ejecuta después de que el modal de confirmación ya se cerró.
- `showModal('confirm', ..., async () => { ... })` — si la cantidad es válida, muestra un modal de confirmación. El `onConfirm` es una función asíncrona que hace las operaciones de Firestore.
- `const ingredientRef = doc(db, \`users/${userId}/ingredients\`, id)` — crea una referencia al documento específico que se va a actualizar. `id` es el ID de Firestore del ingrediente.
- `const rawDate = editForm.expirationDate || ''` — obtiene la fecha de caducidad del formulario. Si está vacía, usa string vacío.
- `rawDate.length > 10 ? rawDate.split('T')[0] : rawDate` — la fecha puede estar en formato ISO completo `"2025-05-01T18:00:00.000Z"` (como la almacena Firestore) o solo `"2025-05-01"` (como lo pone el `<input type="date">`). Se extrae solo la parte de fecha.
- `if (editForm.dateManuallyChanged)` — si el usuario cambió la fecha en el input, se marca como `'manual'` y se usa la nueva fecha sin recalcular.
- `else if (newExpirationDateType !== 'manual')` — si la fecha era calculada y el usuario NO la cambió, se recalcula con la nueva cantidad. Esto es importante porque si el usuario cambió la cantidad de 2 a 0.5 Kilogramos, la fecha de caducidad debe actualizarse porque ahora está fraccionado.
- `await calculateExpirationDateComplete(purchaseDateStr, editForm.name, newQuantity, userId)` — recalcula la fecha con la nueva cantidad.
- `await updateDoc(ingredientRef, { ... })` — actualiza solo los campos especificados. A diferencia de `setDoc` que reemplaza el documento completo, `updateDoc` merge los cambios con los campos existentes.
- `setIngredients(prev => prev.map(ing => ing.id === id ? { ...ing, ...actualizados } : ing))` — actualización optimista: actualiza el estado local inmediatamente sin esperar a recargar de Firestore. Si el `updateDoc` falla, el `catch` muestra el error.

---

## src/components/Recipes/GenerateRecipe.js

### Bloque 1 — Estado y carga de datos

```js
import React, { useState, useEffect } from 'react';
import { collection, getDocs } from 'firebase/firestore';
import { db } from '../../services/firebase';
import { isPriority, isExpired, getDaysRemaining } from '../../utils/dateCalculations';
import { generateRecipe } from '../../services/openaiService';
import { formatQuantity } from '../../utils/recipeHelpers';

const GenerateRecipe = ({ setCurrentView, userId, setGeneratedRecipes, setCurrentRecipeIndex }) => {
  const [ingredients, setIngredients] = useState([]);
  const [pendingDishes, setPendingDishes] = useState([]);
  const [selectedIngredients, setSelectedIngredients] = useState([]);
  const [selectedDishes, setSelectedDishes] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');
  const [generating, setGenerating] = useState(false);
  const [generatingMode, setGeneratingMode] = useState('');
  const [error, setError] = useState('');
  const [errorType, setErrorType] = useState('');
  const [selectedCategories, setSelectedCategories] = useState([]);
  const [mealTime, setMealTime] = useState('Comida');
  const [servings, setServings] = useState(2);

  const categories = ['Snack','Postre','Saludable','Rápida',
    'Internacional','Mexicana','Vegana','Vegetariana','Alta en proteína'];

  useEffect(() => { loadData(); }, [userId]);

  const loadData = async () => {
    try {
      const ingredientsSnapshot = await getDocs(
        collection(db, `users/${userId}/ingredients`));
      const ingredientsData = ingredientsSnapshot.docs
        .map(doc => ({ id: doc.id, ...doc.data() }))
        .filter(ing => !isExpired(ing.expirationDate));
      setIngredients(ingredientsData);

      const dishesSnapshot = await getDocs(
        collection(db, `users/${userId}/pendingDishes`));
      const dishesData = dishesSnapshot.docs
        .map(doc => {
          const data = doc.data();
          return { id: doc.id, ...data,
            daysRemaining: getDaysRemaining(data.expirationDate) || 0 };
        })
        .filter(dish => !isExpired(dish.expirationDate));
      setPendingDishes(dishesData);
    } catch (error) { setError('Error al cargar ingredientes'); }
  };
```

- `setGeneratedRecipes` y `setCurrentRecipeIndex` — props que son seteadores del estado de App.js. Este componente los recibe para poder escribir directamente en el estado global sin que App.js tenga que observar un estado intermedio.
- `const [generating, setGenerating] = useState(false)` — indica si hay una generación de receta en curso. Cuando es `true`, los botones se deshabilitan y se muestra texto de "Generando...".
- `const [generatingMode, setGeneratingMode] = useState('')` — puede ser `'ia'` o `'priority'`. Permite que cada botón muestre su propio indicador de carga independientemente. Si el usuario hace click en "Usar ingredientes prioritarios", ese botón muestra "Generando..." pero el otro botón también se deshabilita.
- `const [errorType, setErrorType] = useState('')` — clasifica el error para mostrar un mensaje de ayuda apropiado. `'ai'` sugiere ajustar categorías; `'technical'` sugiere verificar la conexión; `'validation'` indica que falta seleccionar algo.
- `const categories = [...]` — array estático de categorías disponibles. Se define dentro del componente pero fuera del render principal. Son los tipos de receta que el usuario puede filtrar.
- `useEffect(() => { loadData(); }, [userId])` — carga los datos cuando el componente monta o cuando cambia el `userId`. Sin `userId` en las dependencias, si el usuario cambiara (improbable pero posible), los datos no se actualizarían.
- `.filter(ing => !isExpired(ing.expirationDate))` — excluye ingredientes caducados de la lista. No tiene sentido ofrecerlos para generar recetas si no se pueden consumir.
- `getDaysRemaining(data.expirationDate) || 0` — calcula cuántos días le quedan al platillo pendiente. El `|| 0` convierte `null` a `0` para el caso donde no hay fecha de caducidad.
- `.filter(dish => !isExpired(dish.expirationDate))` — similar a los ingredientes, excluye platillos caducados.

### Bloque 2 — Selección y generación

```js
  const priorityIngredients = ingredients.filter(ing => isPriority(ing.expirationDate));
  const normalIngredients   = ingredients.filter(ing => !isPriority(ing.expirationDate));

  const toggleIngredient = (ingredientId) => {
    setSelectedIngredients(prev =>
      prev.includes(ingredientId)
        ? prev.filter(id => id !== ingredientId)
        : [...prev, ingredientId]);
  };

  const toggleDish = (dishId) => {
    setSelectedDishes(prev =>
      prev.includes(dishId)
        ? prev.filter(id => id !== dishId)
        : [...prev, dishId]);
  };

  const toggleCategory = (category) => {
    if (selectedCategories.includes(category)) {
      setSelectedCategories(prev => prev.filter(cat => cat !== category));
    } else {
      if (selectedCategories.length >= 3) return;
      setSelectedCategories(prev => [...prev, category]);
    }
  };

  const handleGenerate = async (priorityOnly = false) => {
    setError('');
    let ingredientsToUse = priorityOnly
      ? priorityIngredients.map(ing => ing.id)
      : selectedIngredients;

    if (ingredientsToUse.length === 0 && selectedDishes.length === 0) {
      setError('Por favor selecciona al menos un ingrediente o platillo');
      setErrorType('validation'); return;
    }
    if (selectedCategories.length === 0) {
      setError('Por favor selecciona al menos una categoría');
      setErrorType('validation'); return;
    }
    setGenerating(true); setGeneratingMode(priorityOnly ? 'priority' : 'ia');
    try {
      const selectedIngredientsData = ingredients.filter(
        ing => ingredientsToUse.includes(ing.id));
      const selectedDishesData = pendingDishes.filter(
        dish => selectedDishes.includes(dish.id));
      const allItems = selectedIngredientsData.map(ing => ({
        name: ing.name, quantity: ing.quantity, unit: ing.unit
      }));
      const effectiveServings = Math.max(1, Math.min(20, parseInt(servings) || 1));
      const params = {
        ingredients: allItems, pendingDishes: selectedDishesData,
        categories: selectedCategories, mealTime,
        servings: effectiveServings, priorityOnly
      };
      sessionStorage.setItem('lastRecipeParams', JSON.stringify(params));
      const recipes = await generateRecipe(params);
      const recipesWithPendingInfo = recipes.map(r => ({
        ...r,
        usedPendingDishIds: selectedDishesData.map(d => d.id),
        usedPendingDishNames: selectedDishesData.map(d => d.name)
      }));
      setGeneratedRecipes(recipesWithPendingInfo);
      setCurrentRecipeIndex(0);
      setCurrentView('recipe-results');
    } catch (error) {
      if (error.isCompatibilityError || error.message?.includes('No es posible')) {
        setError(error.message); setErrorType('ai');
      } else if (error.status === 429) {
        setError('Límite de uso de IA alcanzado.'); setErrorType('technical');
      } else {
        setError(error.message || 'Error al generar recetas.'); setErrorType('technical');
      }
    } finally { setGenerating(false); setGeneratingMode(''); }
  };
```

- `const priorityIngredients = ingredients.filter(...)` — calculado en cada render (no en un efecto). Al ser derivado del estado `ingredients`, siempre está actualizado automáticamente sin necesidad de gestionarlo por separado.
- `toggleIngredient(ingredientId)` — usa un updater funcional `prev => ...` para evitar closures stale. `prev.includes(ingredientId)` verifica si ya está seleccionado: si sí, lo filtra del array (deselecciona); si no, lo agrega (selecciona).
- `toggleCategory` con `if (selectedCategories.length >= 3) return` — límite de 3 categorías máximo. Si ya hay 3 y el usuario intenta agregar otra, la función retorna sin hacer nada (no hace falta mostrar error, la UI puede deshabilitar las opciones no seleccionadas).
- `let ingredientsToUse = priorityOnly ? priorityIngredients.map(ing => ing.id) : selectedIngredients` — en modo prioritario, usa todos los ingredientes prioritarios ignorando la selección manual del usuario. En modo normal, usa solo los seleccionados manualmente.
- `if (ingredientsToUse.length === 0 && selectedDishes.length === 0)` — debe haber al menos un ingrediente O un platillo seleccionado. Con solo categorías y tiempo no hay suficiente información para generar una receta.
- `setGeneratingMode(priorityOnly ? 'priority' : 'ia')` — registra qué botón activó la generación para mostrar el indicador de carga en el botón correcto.
- `const selectedIngredientsData = ingredients.filter(ing => ingredientsToUse.includes(ing.id))` — recupera los objetos completos de los ingredientes seleccionados. `ingredientsToUse` solo tiene los IDs; aquí se recuperan los datos completos (nombre, cantidad, unidad).
- `const allItems = selectedIngredientsData.map(ing => ({ name, quantity, unit }))` — extrae solo los campos relevantes para el prompt de la IA. No se envía el ID de Firestore ni la fecha de caducidad a OpenAI.
- `Math.max(1, Math.min(20, parseInt(servings) || 1))` — asegura que las porciones estén entre 1 y 20. `parseInt(servings) || 1` maneja el caso donde el input está vacío o tiene un valor no numérico.
- `sessionStorage.setItem('lastRecipeParams', JSON.stringify(params))` — guarda todos los parámetros de generación. `RecipeResults.js` los lee para poder regenerar con los mismos parámetros sin necesidad de que el usuario regrese a esta pantalla.
- `const recipesWithPendingInfo = recipes.map(r => ({ ...r, usedPendingDishIds: ..., usedPendingDishNames: ... }))` — agrega a cada receta los IDs y nombres de los platillos pendientes que se usaron. `RecipeDetail.js` necesita esta información para eliminar esos platillos de `pendingDishes` cuando el usuario complete o guarde la receta.
- `error.isCompatibilityError || error.message?.includes('No es posible')` — detecta si el error fue por incompatibilidad de categorías con ingredientes. `error.isCompatibilityError` es el flag que pone `openaiService.js`; el `.includes` es un fallback en caso de que el mensaje llegue sin el flag.

---

## src/components/Recipes/RecipeResults.js

```js
import React, { useState, useEffect, useCallback, useRef } from 'react';
import { generateRecipe } from '../../services/openaiService';
import { formatQuantity, parseSafeQuantity, cleanText } from '../../utils/recipeHelpers';

const RecipeResults = ({ setCurrentView, recipes, currentIndex, setCurrentIndex,
  setSelectedRecipe, setGeneratedRecipes }) => {

  const [generating, setGenerating] = useState(false);
  const isGeneratingRef = useRef(false);
  const [error, setError] = useState('');
  const [usedRecipeNames, setUsedRecipeNames] = useState([]);

  const [lastParams] = useState(() => {
    try {
      const saved = sessionStorage.getItem('lastRecipeParams');
      return saved ? JSON.parse(saved) : null;
    } catch { return null; }
  });

  const currentRecipe = recipes && recipes.length > 0 ? recipes[currentIndex] : null;
  const portionWarning = currentRecipe?.portionWarning
    ? cleanText(currentRecipe.portionWarning) : null;
  const allergenWarning = currentRecipe?.allergenWarning
    ? cleanText(currentRecipe.allergenWarning) : null;
  const missingIngredients = Array.isArray(currentRecipe?.missingIngredients)
    ? currentRecipe.missingIngredients.filter(ing => ing && ing.name) : [];

  const handleNext = () =>
    setCurrentIndex(prev => (prev < (recipes?.length || 1) - 1 ? prev + 1 : 0));
  const handlePrevious = () =>
    setCurrentIndex(prev => (prev > 0 ? prev - 1 : (recipes?.length || 1) - 1));

  const handleViewDetails = () => {
    setSelectedRecipe(currentRecipe);
    setCurrentView('recipe-detail');
  };

  const handleGenerateAnother = useCallback(async () => {
    if (!lastParams || isGeneratingRef.current) {
      if (!lastParams) setCurrentView('generate-recipe');
      return;
    }
    isGeneratingRef.current = true;
    setGenerating(true); setError('');
    try {
      const newRecipes = await generateRecipe({
        ...lastParams, regenerate: true, usedRecipeNames });
      if (!newRecipes || newRecipes.length === 0)
        throw new Error('No se generó ninguna receta.');
      const merged = [...(recipes || []), ...newRecipes];
      setGeneratedRecipes(merged);
      setCurrentIndex(merged.length - 1);
      const newNames = newRecipes.map(r => cleanText(r?.name)).filter(Boolean);
      setUsedRecipeNames(prev => [...prev, ...newNames]);
    } catch (error) {
      if (error.isCompatibilityError) setError(error.message);
      else if (error.status === 429) setError('Límite de uso alcanzado. Espera un momento.');
      else setError(error.message || 'Error al generar otra receta.');
    } finally { isGeneratingRef.current = false; setGenerating(false); }
  }, [lastParams, usedRecipeNames, recipes, setGeneratedRecipes, setCurrentIndex, setCurrentView]);

  useEffect(() => {
    if (recipes && recipes.length > 0) {
      setUsedRecipeNames(recipes.map(r => cleanText(r?.name)).filter(Boolean));
    }
  }, [recipes]);
```

- `const isGeneratingRef = useRef(false)` — ref booleana que actúa como mutex (semáforo) para prevenir ejecuciones concurrentes de `handleGenerateAnother`. A diferencia de `generating` (estado), esta ref se actualiza síncronamente y no causa re-render, lo que es crucial para prevenir race conditions en el manejador de eventos.
- `const [lastParams] = useState(() => { ... })` — estado que se inicializa una sola vez con los parámetros guardados en `sessionStorage`. El array destructurado sin setter `[lastParams]` indica que este estado nunca cambia (es solo lectura). La inicialización lazy lee `sessionStorage` solo una vez al montar.
- `const currentRecipe = recipes && recipes.length > 0 ? recipes[currentIndex] : null` — extrae la receta actual de forma segura. Primero verifica que `recipes` exista y tenga elementos; si no, retorna `null` en lugar de causar un error de acceso a índice.
- `currentRecipe?.portionWarning ? cleanText(...) : null` — el operador `?.` evita error si `currentRecipe` es `null`. Solo limpia el warning si existe y no es `null`.
- `missingIngredients.filter(ing => ing && ing.name)` — filtra ingredientes faltantes que tengan al menos el campo `name`. Descarta entradas malformadas que OpenAI pudiera haber generado.
- `handleNext` y `handlePrevious` — implementan un carrusel circular. `prev < (recipes?.length || 1) - 1` verifica si hay una siguiente receta; si no, vuelve al índice 0 (primera receta). `prev > 0` verifica si hay una anterior; si no, va al último índice.
- `handleViewDetails` — guarda la receta actual en el estado de App.js (mediante `setSelectedRecipe`) y navega al detalle. La receta también se guarda en `sessionStorage` vía el efecto de App.js.
- `const handleGenerateAnother = useCallback(async () => { ... }, [lastParams, usedRecipeNames, recipes, setGeneratedRecipes, setCurrentIndex, setCurrentView])` — función memorizada que genera una receta adicional. `useCallback` evita que se recree en cada render; las dependencias en el array son todos los valores que usa internamente.
- `if (!lastParams || isGeneratingRef.current)` — dos condiciones de parada al inicio. Sin `lastParams`, no hay cómo regenerar; sin este guard redirige de vuelta a la pantalla de generación. `isGeneratingRef.current` previene que el usuario haga click varias veces rápidamente.
- `isGeneratingRef.current = true` — se activa síncronamente (antes del `await`) para que si el usuario hace doble click, el segundo click encuentre el ref ya activado y retorne.
- `{ ...lastParams, regenerate: true, usedRecipeNames }` — copia todos los parámetros originales (mismo inventario, categorías, etc.) pero agrega `regenerate: true` y la lista de nombres ya generados.
- `const merged = [...(recipes || []), ...newRecipes]` — agrega la nueva receta al array existente. Usar `...` (spread) crea un nuevo array en lugar de mutar el existente, respetando la inmutabilidad de React.
- `setCurrentIndex(merged.length - 1)` — apunta al índice de la última receta (la recién generada) para que sea la visible de inmediato.
- `setUsedRecipeNames(prev => [...prev, ...newNames])` — agrega los nuevos nombres a la lista acumulada. Con updater funcional para evitar closures stale.
- El `useEffect` que escucha `recipes` — inicializa `usedRecipeNames` con los nombres de las recetas que ya existen cuando el componente monta. Esto garantiza que si el usuario vuelve a esta pantalla, los nombres anteriores no se repitan.

---

## src/components/Recipes/RecipeDetail.js

### Bloque 1 — Helpers de unidades

```js
const normalizeUnit = (unit) => {
  const u = String(unit || '').toLowerCase().trim();
  if (['g','gr','grs','gramo','gramos'].includes(u))             return 'gramos';
  if (['kg','kilo','kilos','kilogramo','kilogramos'].includes(u)) return 'kilogramos';
  if (['ml','mililitro','mililitros'].includes(u))               return 'mililitros';
  if (['l','litro','litros'].includes(u))                        return 'litros';
  return u;
};

const convertToInventoryUnit = (quantity, fromUnit, toUnit) => {
  const from = normalizeUnit(fromUnit);
  const to   = normalizeUnit(toUnit);
  if (from === to) return quantity;
  if (from === 'gramos'     && to === 'kilogramos') return quantity / 1000;
  if (from === 'kilogramos' && to === 'gramos')     return quantity * 1000;
  if (from === 'mililitros' && to === 'litros')     return quantity / 1000;
  if (from === 'litros'     && to === 'mililitros') return quantity * 1000;
  return quantity;
};

const FEMININE_UNITS = new Set(['piezas', 'pieza']);

const getUsedWord = (rawUnit, quantity) => {
  const unitNorm = String(rawUnit || '').toLowerCase().trim();
  const isFeminine = FEMININE_UNITS.has(unitNorm);
  const isSingular = quantity === 1;
  if (isFeminine) return isSingular ? 'usada' : 'usadas';
  return isSingular ? 'usado' : 'usados';
};
```

- `normalizeUnit(unit)` — estandariza las variantes escritas de la misma unidad. OpenAI puede escribir "gramos", "g", "gr" o "Gramos" para referirse a lo mismo. Esta función las convierte todas a una representación canónica para poder comparar y convertir entre unidades.
- `String(unit || '').toLowerCase().trim()` — primero convierte a string (en caso de que `unit` sea `null` o `undefined`), luego a minúsculas y elimina espacios. Triple transformación defensiva.
- `['g','gr','grs','gramo','gramos'].includes(u)` — verifica si la unidad normalizada está en el array de variantes conocidas de "gramos". `.includes()` hace búsqueda lineal en el array.
- `return u` al final — si la unidad no coincide con ninguna conocida, la retorna tal cual. Esto maneja unidades desconocidas sin lanzar error (por ejemplo, una unidad personalizada que el usuario pudiera tener).
- `convertToInventoryUnit(quantity, fromUnit, toUnit)` — convierte una cantidad de la unidad de la receta a la unidad del inventario. Por ejemplo, si la receta pide "200 gramos" pero el inventario tiene "0.5 Kilogramos", necesita convertir para poder restar correctamente.
- `if (from === to) return quantity` — caso trivial: si las unidades ya son iguales, no hay conversión necesaria.
- `quantity / 1000` y `quantity * 1000` — conversiones estándar entre unidades del sistema métrico. 1 kg = 1000 g, 1 L = 1000 mL.
- `return quantity` al final — si no hay conversión conocida entre las dos unidades (por ejemplo, de "piezas" a "kilogramos"), retorna la cantidad original sin convertir. Esto puede causar descuentos incorrectos, pero es preferible a crashear.
- `const FEMININE_UNITS = new Set(['piezas', 'pieza'])` — conjunto de unidades gramaticalmente femeninas en español. Se usa para elegir el participio correcto en el reporte de ingredientes usados.
- `getUsedWord(rawUnit, quantity)` — retorna el participio pasado gramaticalmente correcto: "usada/usadas" para unidades femeninas (pieza/piezas), "usado/usados" para masculinas (gramo, kilogramo, litro). El número (singular/plural) depende de si `quantity === 1`.

### Bloque 2 — Estado e interacción

```js
const RecipeDetail = ({ setCurrentView, recipe, userId }) => {
  const [usedIngredients, setUsedIngredients] = useState(
    recipe?.ingredients?.map(ing => ({
      ...ing,
      used: true,
      usedQuantity: typeof ing.quantity === 'number'
        ? Math.round(ing.quantity * 100) / 100
        : ing.quantity,
      usedUnit: ing.unit
    })) || []
  );
  const [savingAction, setSavingAction] = useState(null);

  if (!recipe) {
    return (
      <div className="...">
        <p>No hay receta seleccionada</p>
        <button onClick={() => setCurrentView('recipe-results')}>← Volver</button>
      </div>
    );
  }

  const showModal = (type, title, message, onConfirm = () => {}) => {
    setModalConfig({ isOpen: true, type, title, message, onConfirm });
  };

  const toggleIngredient = (index) => {
    setUsedIngredients(prev => prev.map((ing, i) =>
      i === index ? { ...ing, used: !ing.used } : ing));
  };

  const handleTempChange = (index, value) => {
    setUsedIngredients(prev => prev.map((ing, i) =>
      i === index ? { ...ing, usedQuantity: value } : ing));
  };

  const handleQuantityBlur = (index) => {
    const value = usedIngredients[index].usedQuantity;
    const parsed = parseSafeQuantity(value);
    if (parsed.type !== 'number' || isNaN(parsed.number) || parsed.number < 0.25) {
      showModal('error', 'Cantidad inválida',
        'La cantidad mínima permitida es 0.25. Se ajustará automáticamente.',
        () => {
          setUsedIngredients(prev => prev.map((ing, i) =>
            i === index ? { ...ing, usedQuantity: '0.25' } : ing));
        }
      );
    }
  };
```

- `recipe?.ingredients?.map(ing => ({ ...ing, used: true, usedQuantity: ..., usedUnit: ing.unit })) || []` — inicializa el estado con una copia enriquecida de los ingredientes. Se agrega `used` (si se usó el ingrediente), `usedQuantity` (cantidad usada, inicialmente la de la receta) y `usedUnit` (unidad, inicialmente la de la receta). El `|| []` maneja el caso donde `recipe` es `null` o no tiene ingredientes.
- `typeof ing.quantity === 'number' ? Math.round(ing.quantity * 100) / 100 : ing.quantity` — si la cantidad es número, la redondea a 2 decimales. `Math.round(x * 100) / 100` es más preciso que `.toFixed(2)` para evitar artefactos de punto flotante. Si es string ("Al gusto"), lo deja tal cual.
- `const [savingAction, setSavingAction] = useState(null)` — puede ser `'complete'`, `'pending'` o `null`. Cuando no es `null`, ambos botones de acción se deshabilitan, previniendo que el usuario haga click en "Completar" mientras ya está guardando "Pendiente" o viceversa.
- `if (!recipe) { return <div>...</div> }` — pantalla de fallback si no hay receta. Esto ocurre si el usuario navega directamente a `/detalle-receta` sin haber seleccionado una receta primero (y `sessionStorage` también está vacío).
- `toggleIngredient(index)` — marca o desmarca un ingrediente como usado. Solo los marcados se descuentan del inventario. El usuario puede desmarcar un ingrediente si en realidad no lo usó.
- `handleTempChange(index, value)` — actualiza la cantidad usada mientras el usuario escribe. Solo almacena el valor temporalmente sin validarlo (la validación ocurre en `handleQuantityBlur`).
- `handleQuantityBlur(index)` — valida cuando el campo pierde el foco. Si la cantidad es menor a 0.25, muestra un modal de error cuyo `onConfirm` corrige el valor a `'0.25'`. Poner la corrección en `onConfirm` permite que el usuario vea primero el mensaje antes de que el valor cambie.
- `parseSafeQuantity(value)` — valida la cantidad del input. Si el usuario escribió texto o un número inválido, `parsed.type` será `'text'` y la validación falla.

### Bloque 3 — Completar receta

```js
  const handleMarkAsCompleted = () => {
    showModal('confirm', 'Marcar como terminada',
      '¿Está seguro de marcar esta receta como terminada? Se actualizará su inventario.',
      async () => {
        setSavingAction('complete');
        try {
          const ingredientsSnapshot = await getDocs(
            collection(db, `users/${userId}/ingredients`));
          const batch = writeBatch(db);

          for (const ing of usedIngredients) {
            if (!ing.used) continue;
            const ingredientDoc = ingredientsSnapshot.docs.find(
              d => d.data().name?.toLowerCase().trim() === ing.name.toLowerCase().trim());
            if (ingredientDoc) {
              const currentData = ingredientDoc.data();
              const parsedQty = parseSafeQuantity(ing.usedQuantity);
              if (parsedQty.type !== 'number' || parsedQty.number <= 0) {
                showModal('error', 'Cantidad inválida',
                  `La cantidad de "${ing.name}" no es válida.`);
                setSavingAction(null); return;
              }
              const quantityUsed = Math.round(parsedQty.number * 100) / 100;
              const convertedUsed = convertToInventoryUnit(
                quantityUsed, ing.usedUnit, currentData.unit);
              const newQuantity = Math.round(
                (currentData.quantity - convertedUsed) * 100) / 100;

              if (newQuantity <= 0) {
                batch.delete(doc(db,
                  `users/${userId}/ingredients`, ingredientDoc.id));
              } else {
                const isFractioned = newQuantity < 1;
                const updateData = { quantity: newQuantity, isFractioned };
                if (currentData.unit === 'Piezas' && isFractioned
                    && !currentData.isFractioned) {
                  const { searchFood } = await import('../../services/foodDatabase');
                  const food = searchFood(currentData.name);
                  if (food?.fraccionado > 0) {
                    const purchaseDate = currentData.purchaseDate?.toDate
                      ? currentData.purchaseDate.toDate()
                      : new Date(currentData.purchaseDate);
                    const newExpDate = new Date(purchaseDate);
                    newExpDate.setDate(newExpDate.getDate() + food.fraccionado);
                    updateData.expirationDate = newExpDate.toISOString();
                  }
                }
                batch.update(doc(db,
                  `users/${userId}/ingredients`, ingredientDoc.id), updateData);
              }
            }
          }

          const historyRef = doc(collection(db, `users/${userId}/history`));
          batch.set(historyRef, {
            name: recipe.name,
            ingredients: usedIngredients.filter(i => i.used).map(i => ({
              name: i.name, quantity: i.usedQuantity, unit: i.usedUnit
            })),
            instructions: recipe.instructions || [],
            categories: recipe.categories || [],
            prepTime: recipe.prepTime || null,
            servings: recipe.servings || 2,
            completedAt: new Date().toISOString(),
            favorite: false
          });

          await batch.commit();

          if (recipe.usedPendingDishIds?.length > 0) {
            for (const dishId of recipe.usedPendingDishIds) {
              try {
                await deleteDoc(doc(db, `users/${userId}/pendingDishes`, dishId));
              } catch {}
            }
          }

          showModal('success', '¡Receta completada!', '...', () => setCurrentView('menu'));
        } catch (error) {
          showModal('error', 'Error', `Error al guardar: ${error.message}`);
        } finally { setSavingAction(null); }
      }
    );
  };
```

- `showModal('confirm', ..., async () => { ... })` — muestra confirmación antes de ejecutar. El `onConfirm` es `async` porque contiene múltiples operaciones de Firestore.
- `setSavingAction('complete')` — bloquea ambos botones de acción mientras se procesa. Se hace dentro del `onConfirm`, DESPUÉS de que el modal de confirmación se cerró.
- `await getDocs(collection(db, \`users/${userId}/ingredients\`))` — carga todo el inventario actual. Es necesario para comparar las cantidades actuales con las que se van a descontar. Se hace dentro del `onConfirm` para asegurar que los datos son los más recientes al momento de guardar.
- `const batch = writeBatch(db)` — inicializa un batch de escritura. Un batch agrupa múltiples operaciones (decrementar ingredientes + escribir historial) en una sola transacción atómica. Si cualquier operación falla, ninguna se aplica.
- `for (const ing of usedIngredients)` — itera sobre todos los ingredientes del estado local (no de Firestore). El estado local ya fue modificado por el usuario (cantidades ajustadas, ingredientes desmarcados).
- `if (!ing.used) continue` — salta los ingredientes no marcados como usados. Solo se descuentan del inventario los que el usuario confirmó que usó.
- `ingredientsSnapshot.docs.find(d => d.data().name?.toLowerCase().trim() === ing.name.toLowerCase().trim())` — busca el documento de Firestore correspondiente al ingrediente. La búsqueda es por nombre normalizado para tolerar diferencias de capitalización o espacios.
- `parsedQty.type !== 'number' || parsedQty.number <= 0` — validación de último momento antes de operar. Aunque `handleQuantityBlur` ya validó, es posible que el estado esté inconsistente; esta validación previene corrupciones del inventario.
- `convertToInventoryUnit(quantityUsed, ing.usedUnit, currentData.unit)` — convierte la cantidad usada a la unidad del inventario. Si la receta pidió "200 gramos" pero el inventario tiene "0.5 Kilogramos", convierte 200 g a 0.2 kg antes de restar.
- `Math.round((currentData.quantity - convertedUsed) * 100) / 100` — resta y redondea a 2 decimales para evitar resultados como `0.30000000000000004` por aritmética de punto flotante.
- `newQuantity <= 0 → batch.delete(...)` — si después de descontar no queda nada (o queda negativo por imprecisión), elimina el documento completo del inventario. Es mejor eliminarlo que dejarlo con cantidad 0 o negativa.
- `currentData.unit === 'Piezas' && isFractioned && !currentData.isFractioned` — detecta la transición específica de "entero a fraccionado" para ingredientes medidos en piezas. Por ejemplo, el usuario tenía 2 piezas de aguacate, usó 1.5, quedan 0.5 piezas. Esto activa el recálculo de caducidad.
- `const { searchFood } = await import('../../services/foodDatabase')` — import dinámico. Solo carga el módulo `foodDatabase` cuando es necesario (cuando hay transición a fraccionado). Esto reduce el tamaño del bundle inicial porque este código no se ejecuta en la mayoría de los casos.
- `currentData.purchaseDate?.toDate ? currentData.purchaseDate.toDate() : new Date(currentData.purchaseDate)` — guard contra Timestamps de Firestore. Si `purchaseDate` es un objeto Timestamp de Firestore, tiene el método `.toDate()` que lo convierte a `Date`. Si es un ISO string (guardado antes de que Firestore usara Timestamps), se usa `new Date()`.
- `newExpDate.setDate(newExpDate.getDate() + food.fraccionado)` — suma los días de vida útil fraccionado a la fecha de compra original. Se usa la fecha de compra (no la actual) porque la vida útil se cuenta desde la compra.
- `const historyRef = doc(collection(db, \`users/${userId}/history\`))` — genera una referencia con ID automático para el documento de historial. Dentro de un batch no se puede usar `addDoc` (que también genera ID automático), por eso se usa `doc(collection(...))` que genera la referencia sin escribir.
- `batch.set(historyRef, { ... })` — agrega la escritura del historial al batch. `set` crea el documento con todos los campos especificados.
- `await batch.commit()` — ejecuta todas las operaciones del batch en una sola transacción. Si alguna operación falla (por ejemplo, un ingrediente fue eliminado por otro dispositivo simultáneamente), todas se revierten.
- `recipe.usedPendingDishIds?.length > 0` — verifica si hubo platillos pendientes usados como base de esta receta.
- `await deleteDoc(...)` fuera del batch — la eliminación de platillos pendientes es "best-effort": se intenta pero si falla no es crítico. Por eso va fuera del batch con su propio `try/catch` silencioso. Si fallara dentro del batch, revertiría también el descuento del inventario y el historial.

---

## src/components/Dishes/PendingDishes.js

```js
import React, { useState, useEffect } from 'react';
import { collection, getDocs, doc, writeBatch } from 'firebase/firestore';
import { db } from '../../services/firebase';
import { getDaysRemaining, isExpired } from '../../utils/dateCalculations';
import Modal from '../../utils/Modal';

const PendingDishes = ({ setCurrentView, userId }) => {
  const [dishes, setDishes] = useState([]);
  const [loading, setLoading] = useState(true);
  const [expandedId, setExpandedId] = useState(null);

  useEffect(() => { loadDishes(); }, [userId]);

  const loadDishes = async () => {
    try {
      const querySnapshot = await getDocs(
        collection(db, `users/${userId}/pendingDishes`));
      const dishesData = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      const updated = dishesData.map(dish => ({
        ...dish,
        daysRemaining: getDaysRemaining(dish.expirationDate) || 0,
        expired: isExpired(dish.expirationDate)
      }));
      setDishes(updated.sort((a, b) => {
        if (a.expired && !b.expired) return 1;
        if (!a.expired && b.expired) return -1;
        return a.daysRemaining - b.daysRemaining;
      }));
    } catch (error) { showModal('error', 'Error', 'Error al cargar platillos'); }
    finally { setLoading(false); }
  };

  const handleComplete = (id, name) => {
    const dish = dishes.find(d => d.id === id);
    if (!dish) { showModal('error', 'Error', 'Platillo no encontrado.'); return; }
    showModal('confirm', 'Marcar como terminado',
      `¿Deseas marcar "${name}" como terminado?`,
      async () => {
        try {
          const batch = writeBatch(db);
          const historyRef = doc(collection(db, `users/${userId}/history`));
          batch.set(historyRef, {
            name: dish.name,
            ingredients: dish.ingredients || [],
            instructions: dish.instructions || [],
            categories: dish.categories || [],
            prepTime: dish.prepTime ?? null,
            servings: dish.servings ?? null,
            completedAt: new Date().toISOString(),
            favorite: false
          });
          batch.delete(doc(db, `users/${userId}/pendingDishes`, id));
          await batch.commit();
          setDishes(prev => prev.filter(d => d.id !== id));
          showModal('success', '¡Platillo terminado!',
            'Puedes consultarlo en tu historial.');
        } catch (error) {
          showModal('error', 'Error', 'Error al marcar como terminado');
        }
      });
  };

  const toggleExpand = (id) => {
    setExpandedId(expandedId === id ? null : id);
  };
```

- `const [expandedId, setExpandedId] = useState(null)` — ID del platillo cuyo acordeón está expandido. Solo un platillo puede estar expandido a la vez (acordeón de apertura única). `null` significa que ninguno está expandido.
- `loadDishes` — carga los platillos y calcula propiedades derivadas (`daysRemaining` y `expired`) que se necesitan en la UI. Estas propiedades no se guardan en Firestore; se calculan cada vez que se cargan los datos.
- `getDaysRemaining(dish.expirationDate) || 0` — calcula días restantes y usa `0` como fallback si retorna `null` (platillo sin fecha de caducidad).
- `expired: isExpired(dish.expirationDate)` — booleano calculado para determinar el estado visual del platillo (caducado, próximo a caducar, fresco).
- `updated.sort((a, b) => { ... })` — ordena: caducados al final, activos ordenados por días restantes (los que caducan más pronto primero). `a.daysRemaining - b.daysRemaining` es un comparador numérico estándar.
- `const dish = dishes.find(d => d.id === id)` — recupera el objeto completo del platillo antes de abrir el modal de confirmación. Se hace aquí (en el manejador de click) y no dentro del `onConfirm` porque el estado podría cambiar entre el click y la confirmación, y necesitamos el objeto al momento del click.
- `if (!dish) { showModal('error', ...) }` — guard de seguridad. Si por alguna razón el platillo no se encuentra en el estado local (improbable pero posible si hubo un error de sincronización), muestra error en lugar de crashear.
- `batch.set(historyRef, { ... })` — escribe el platillo en el historial como receta completada. Los campos son los mismos que una receta completada desde `RecipeDetail.js`.
- `dish.prepTime ?? null` — el operador `??` (nullish coalescing) usa `null` si `dish.prepTime` es `null` o `undefined`. A diferencia de `||`, no convierte `0` a `null` (lo que sería incorrecto para un tiempo de 0 minutos).
- `batch.delete(doc(db, \`users/${userId}/pendingDishes\`, id))` — elimina el platillo de pendientes en el mismo batch que escribe en historial. Las dos operaciones son atómicas: si falla una, la otra también se revierte.
- `setDishes(prev => prev.filter(d => d.id !== id))` — actualización optimista: elimina el platillo del estado local inmediatamente sin esperar a recargar de Firestore. Hace la UI más responsive.
- `toggleExpand(id)` — implementación del acordeón: si el platillo clickeado ya está expandido (`expandedId === id`), lo colapsa poniendo `null`; si no, lo expande poniendo su ID. Solo un platillo puede estar expandido a la vez.

---

## src/components/Dishes/History.js

```js
import React, { useState, useEffect } from 'react';
import { collection, getDocs, deleteDoc, doc, updateDoc } from 'firebase/firestore';
import { db } from '../../services/firebase';
import { formatDate } from '../../utils/dateCalculations';
import Modal from '../../utils/Modal';

const History = ({ setCurrentView, userId }) => {
  const [recipes, setRecipes] = useState([]);
  const [loading, setLoading] = useState(true);
  const [expandedId, setExpandedId] = useState(null);

  useEffect(() => { loadHistory(); }, [userId]);

  const loadHistory = async () => {
    try {
      const querySnapshot = await getDocs(
        collection(db, `users/${userId}/history`));
      const recipesData = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      const toMs = (val) => {
        if (!val) return 0;
        const d = val?.toDate ? val.toDate() : new Date(val);
        return isNaN(d) ? 0 : d.getTime();
      };
      setRecipes(recipesData.sort((a, b) =>
        toMs(b.completedAt) - toMs(a.completedAt)));
    } catch (error) { showModal('error', 'Error', 'Error al cargar el historial'); }
    finally { setLoading(false); }
  };

  const toggleFavorite = async (id, currentFavorite) => {
    try {
      await updateDoc(doc(db, `users/${userId}/history`, id),
        { favorite: !currentFavorite });
      setRecipes(prev => prev.map(recipe =>
        recipe.id === id
          ? { ...recipe, favorite: !currentFavorite }
          : recipe));
    } catch (error) { showModal('error', 'Error', 'Error al actualizar favorito'); }
  };

  const handleDelete = (id, name) => {
    showModal('confirm', 'Eliminar del historial',
      `¿Deseas eliminar "${name}" del historial?`,
      async () => {
        try {
          await deleteDoc(doc(db, `users/${userId}/history`, id));
          setRecipes(prev => prev.filter(recipe => recipe.id !== id));
          showModal('success', '¡Eliminada!', 'Receta eliminada del historial');
        } catch (error) {
          showModal('error', 'Error', 'Error al eliminar la receta');
        }
      });
  };

  const toggleExpand = (id) => { setExpandedId(expandedId === id ? null : id); };
```

- `const toMs = (val)` — función local para convertir `completedAt` a milisegundos. Es necesaria porque `completedAt` puede ser un Timestamp de Firestore o un string ISO, dependiendo de cuándo y cómo se guardó.
- `val?.toDate ? val.toDate() : new Date(val)` — la doble verificación: `?.toDate` verifica si el método existe (solo los Timestamps de Firestore lo tienen). Si existe, lo llama para convertir a `Date`. Si no existe, asume que es un string ISO y usa el constructor `Date`.
- `isNaN(d) ? 0 : d.getTime()` — si la conversión produjo una fecha inválida (por datos corruptos), usa `0` como milisegundos. Esto coloca el elemento al principio del array al ordenar de más reciente a más antiguo, donde `0` es el más antiguo posible.
- `.sort((a, b) => toMs(b.completedAt) - toMs(a.completedAt))` — orden descendente: `b - a` (en lugar del ascendente `a - b`). Las recetas más recientes aparecen primero.
- `const toggleFavorite = async (id, currentFavorite)` — función que alterna el estado de favorito. Recibe el valor actual para poder negarlo.
- `await updateDoc(doc(db, ...), { favorite: !currentFavorite })` — actualiza solo el campo `favorite` en Firestore. `updateDoc` merge el cambio sin afectar los demás campos del documento.
- `setRecipes(prev => prev.map(recipe => recipe.id === id ? { ...recipe, favorite: !currentFavorite } : recipe))` — actualización optimista local. Si `updateDoc` falla, el catch muestra el error pero el estado local ya cambió. Esto es un trade-off aceptable: en caso de error, el ícono de favorito mostrará el estado incorrecto hasta que el usuario recargue.
- `handleDelete` — flujo típico con confirmación: primero muestra el modal de confirm, y si el usuario acepta, elimina de Firestore y actualiza el estado local.
- `await deleteDoc(doc(db, \`users/${userId}/history\`, id))` — elimina el documento de historial de Firestore.
- `showModal('success', ...)` llamado después de `deleteDoc` — muestra confirmación de éxito después de eliminar. Nota: esto abre un nuevo modal mientras el de confirmación ya se cerró (el cierre ocurrió antes de ejecutar `onConfirm`).

---

## Api/index.js — Proxy Express (desarrollo local)

```js
import dotenv from 'dotenv';
dotenv.config();
import express from 'express';
import fetch from 'node-fetch';
import cors from 'cors';

const app = express();

app.use(cors({ origin: 'http://localhost:3000' }));
app.use(express.json());

app.post('/api/openai', async (req, res) => {
  try {
    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${process.env.OPENAI_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(req.body)
    });
    const data = await response.json();
    return res.status(response.status).json(data);
  } catch (error) {
    console.error('[Proxy Error]:', error.message);
    return res.status(503).json({
      error: 'Error de conexión',
      message: 'No se pudo conectar con el servicio de IA.',
      type: 'network_error'
    });
  }
});

app.listen(3001, () => console.log('Proxy OpenAI listo en http://localhost:3001'));
```

- `import dotenv from 'dotenv'` — importa la librería que lee archivos `.env` y carga sus variables en `process.env`. Sin esto, `process.env.OPENAI_API_KEY` sería `undefined`.
- `dotenv.config()` — ejecuta la carga del archivo `Api/.env`. Se llama antes de importar los demás módulos porque estos pueden necesitar las variables de entorno ya cargadas.
- `import express from 'express'` — importa el framework web Express para crear el servidor HTTP.
- `import fetch from 'node-fetch'` — importa `fetch` para Node.js. Las versiones de Node anteriores a 18 no tienen `fetch` nativo; este paquete lo provee. (En producción con Vercel y Node 18+, `fetch` ya es nativo).
- `import cors from 'cors'` — middleware que agrega los headers HTTP necesarios para permitir peticiones de origen cruzado (Cross-Origin Resource Sharing).
- `const app = express()` — crea la instancia del servidor Express. A partir de aquí se le pueden agregar middlewares y rutas.
- `app.use(cors({ origin: 'http://localhost:3000' }))` — configura CORS para aceptar peticiones solo desde `http://localhost:3000` (el servidor de desarrollo de CRA). Si se intentara hacer una petición desde cualquier otro origen, el navegador la rechazaría.
- `app.use(express.json())` — middleware que parsea automáticamente el body de las peticiones con `Content-Type: application/json`. Sin esto, `req.body` sería `undefined`.
- `app.post('/api/openai', async (req, res) => { ... })` — define la ruta POST. Cualquier petición POST a `/api/openai` es manejada por esta función. Es `async` porque hace peticiones HTTP asíncronas.
- `await fetch('https://api.openai.com/v1/chat/completions', { ... })` — reenvía la petición al endpoint real de OpenAI. El servidor actúa como intermediario (proxy) entre el frontend y OpenAI.
- `Authorization: \`Bearer ${process.env.OPENAI_API_KEY}\`` — agrega la API key de OpenAI en el header de autorización. El frontend nunca tiene acceso a esta key; solo el servidor la conoce.
- `body: JSON.stringify(req.body)` — serializa el body recibido del frontend (que Express ya parseó) para enviarlo a OpenAI. El frontend construye el objeto de petición y el proxy lo reenvía tal cual.
- `const data = await response.json()` — parsea la respuesta de OpenAI a objeto JavaScript.
- `return res.status(response.status).json(data)` — responde al frontend con el mismo status HTTP que OpenAI dio. Si OpenAI respondió `429 Too Many Requests`, el frontend recibe el mismo `429`. Si respondió `200 OK`, el frontend recibe `200`. Esto permite que la lógica de reintento del frontend distinga entre tipos de error.
- `res.status(503).json({ error: '...', type: 'network_error' })` — si el proxy no pudo conectarse a OpenAI (timeout, DNS, red caída), responde con `503 Service Unavailable`. El campo `type: 'network_error'` ayuda al frontend a diagnosticar el error.
- `app.listen(3001, () => console.log(...))` — inicia el servidor en el puerto 3001. CRA tiene configurado en `package.json` un proxy (`"proxy": "http://localhost:3001"`) que redirige peticiones a `/api/*` a este servidor.
- Módulo ESM — este archivo usa `import/export` porque `Api/package.json` tiene `"type": "module"`. Esto lo hace incompatible con `require()`, pero más moderno y alineado con el frontend React.

---

## api/openai.js — Función Serverless Vercel (producción)

```js
module.exports = async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).end();
  }

  try {
    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${process.env.OPENAI_API_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(req.body),
    });

    const data = await response.json();
    return res.status(response.status).json(data);
  } catch (error) {
    console.error('[Serverless Proxy Error]', error.message);
    return res.status(503).json({
      error: 'Error de conexión',
      message: 'No se pudo conectar con el servicio de IA.',
      type: 'network_error',
    });
  }
};
```

- `module.exports = async function handler(req, res)` — exporta la función usando CommonJS (`module.exports`). Vercel requiere este formato para sus funciones serverless en Node.js. A diferencia de `Api/index.js` que usa ESM (`export`), este archivo usa CommonJS porque el entorno de Vercel así lo requiere para la carpeta `api/`.
- `async function handler` — el nombre `handler` es una convención de Vercel. Vercel detecta automáticamente archivos en la carpeta `api/` y los convierte en funciones serverless; el archivo exportado se convierte en el handler de la función.
- `if (req.method !== 'POST')` — verifica el método HTTP de la petición. A diferencia del servidor Express donde solo se define `app.post(...)`, una función serverless de Vercel recibe TODOS los métodos HTTP (GET, POST, PUT, DELETE, etc.) para esa ruta. Hay que filtrar manualmente los métodos no permitidos.
- `return res.status(405).end()` — `405 Method Not Allowed`. `.end()` finaliza la respuesta sin cuerpo. Si alguien intenta hacer GET a `/api/openai`, recibe este error.
- `const response = await fetch(...)` — usa el `fetch` nativo de Node.js 18. Vercel despliega funciones serverless en Node.js 18 por defecto, que ya incluye `fetch` sin necesidad de importar `node-fetch`.
- `Authorization: \`Bearer ${process.env.OPENAI_API_KEY}\`` — igual que en el Express. Las variables de entorno en Vercel se configuran en el panel de control de Vercel (no en un archivo `.env` en el repositorio).
- `body: JSON.stringify(req.body)` — Vercel parsea automáticamente el body JSON de las peticiones entrantes, igual que `express.json()`. `req.body` ya es un objeto JavaScript que hay que serializar para enviarlo a OpenAI.
- `res.status(response.status).json(data)` — preserva el status de OpenAI, exactamente igual que el Express.
- `console.error('[Serverless Proxy Error]', error.message)` — los logs de `console.error` en funciones serverless de Vercel aparecen en el panel de logs de Vercel y en los registros de la función, útil para debugging en producción.
- `res.status(503).json({ ... })` — mismo manejo de errores de red que el Express.
- **Diferencia fundamental con `Api/index.js`**: no hay `app.listen()`, no hay CORS manual, no hay `express.json()`. Vercel maneja todo eso automáticamente. El archivo es completamente stateless: cada invocación es independiente, sin estado compartido entre peticiones. Esto lo hace escalable y tolerante a fallos.

---

## src/index.css

### Bloque 1 — Tailwind y reset global

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto',
    'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans',
    'Helvetica Neue', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

- `@tailwind base` — directiva de PostCSS que inyecta los estilos base de Tailwind (Preflight). Preflight es una normalización CSS derivada de `normalize.css` que elimina inconsistencias entre navegadores (márgenes, tamaños de fuente, estilos de elementos HTML).
- `@tailwind components` — inyecta todas las clases definidas con `@layer components` en cualquier archivo CSS del proyecto. Aquí se inyectarán las clases como `.card-food`, `.btn-food`, `.badge-fresh`, etc. que se definen más adelante en este mismo archivo.
- `@tailwind utilities` — inyecta todas las clases utilitarias de Tailwind como `flex`, `p-4`, `text-red-500`, etc. Va al final para que las utilities tengan mayor especificidad y puedan sobreescribir los estilos de componentes si es necesario.
- `* { margin: 0; padding: 0; box-sizing: border-box; }` — reset universal. `margin: 0; padding: 0` elimina los márgenes y paddings por defecto que los navegadores aplican a elementos como `<h1>`, `<p>`, `<ul>`, etc. `box-sizing: border-box` cambia el modelo de caja: el padding y border se incluyen dentro del ancho declarado, haciendo el layout mucho más predecible.
- `font-family: -apple-system, BlinkMacSystemFont, ...` — stack de fuentes del sistema. Usa la fuente nativa del sistema operativo del usuario, lo que hace que la app se vea familiar y cargue instantáneamente (sin descargar fuentes externas). `-apple-system` y `BlinkMacSystemFont` son las fuentes del sistema en macOS/iOS; `Segoe UI` en Windows; `Roboto` en Android; y las demás son fallbacks para Linux y sistemas más antiguos.
- `-webkit-font-smoothing: antialiased` — activa el suavizado de fuentes en macOS usando antialiasing de subpíxel. Hace que el texto se vea más delgado y nítido en pantallas Retina.
- `-moz-osx-font-smoothing: grayscale` — equivalente para Firefox en macOS. Produce el mismo efecto de suavizado.

### Bloque 2 — Spinner, scrollbar y controles

```css
@keyframes spin {
  to { transform: rotate(360deg); }
}
.animate-spin { animation: spin 1s linear infinite; }

::-webkit-scrollbar { width: 8px; height: 8px; }
::-webkit-scrollbar-track { background: #fef7ee; border-radius: 10px; }
::-webkit-scrollbar-thumb { background: #f29442; border-radius: 10px; }
::-webkit-scrollbar-thumb:hover { background: #ed751d; }

input[type="checkbox"] {
  cursor: pointer;
  width: 18px;
  height: 18px;
  accent-color: #f29442;
}

select { cursor: pointer; }

input[type="password"]::-ms-reveal,
input[type="password"]::-ms-clear { display: none; }
```

- `@keyframes spin { to { transform: rotate(360deg); } }` — define la animación de giro. Solo especifica el frame final (`to`); el frame inicial es implícitamente el estado natural del elemento (sin rotación). El resultado es una rotación continua de 360 grados.
- `.animate-spin { animation: spin 1s linear infinite; }` — aplica la animación. `1s` es la duración de una vuelta completa. `linear` mantiene velocidad constante (sin aceleración ni desaceleración). `infinite` hace que se repita indefinidamente. Se usa en todos los spinners de carga de la app.
- `::-webkit-scrollbar` — pseudo-elemento CSS que permite estilizar el scrollbar en navegadores basados en WebKit (Chrome, Edge, Safari). `width: 8px` y `height: 8px` reducen el scrollbar a 8px, más discreto que el default del sistema.
- `::-webkit-scrollbar-track` — el "carril" por donde se desliza el thumb. Color crema `#fef7ee` (el mismo que el fondo `food-50`). `border-radius: 10px` redondea los extremos del carril.
- `::-webkit-scrollbar-thumb` — la barra deslizante. Color naranja `#f29442` (food-400) con bordes redondeados. El color naranja hace que el scrollbar sea parte coherente del diseño de la app.
- `::-webkit-scrollbar-thumb:hover` — al pasar el cursor sobre el thumb, se oscurece a naranja más intenso `#ed751d` (food-500). Feedback visual de que el thumb es interactivo.
- `input[type="checkbox"] { ... }` — estiliza todos los checkboxes de la app. `cursor: pointer` cambia el cursor a mano al pasar sobre él. `width: 18px; height: 18px` establece un tamaño consistente (los navegadores tienen tamaños por defecto diferentes). `accent-color: #f29442` es una propiedad CSS moderna que colorea el interior del checkbox cuando está marcado, sin necesidad de reemplazarlo completamente con CSS custom.
- `select { cursor: pointer; }` — todos los selectores (`<select>`) muestran cursor de mano al pasar sobre ellos, indicando que son interactivos.
- `input[type="password"]::-ms-reveal, input[type="password"]::-ms-clear { display: none; }` — oculta los botones nativos que Microsoft Edge y Internet Explorer agregan a los campos de contraseña: el "ojo" para mostrarla y la X para limpiarla. La app tiene su propia implementación de estos controles (los botones `Eye`/`EyeOff` de Lucide), por lo que los botones nativos son redundantes y crean confusión visual.

### Bloque 3 — Tarjeta y bordes

```css
.card-food {
  @apply bg-white rounded-2xl shadow-lg;
  box-shadow:
    0 4px 6px -1px rgba(0,0,0,0.1),
    0 2px 4px -1px rgba(0,0,0,0.06),
    inset 0 1px 0 rgba(255,255,255,0.8);
}

.card-food:hover {
  box-shadow:
    0 10px 15px -3px rgba(0,0,0,0.1),
    0 4px 6px -2px rgba(0,0,0,0.05),
    inset 0 1px 0 rgba(255,255,255,0.8);
}

.border-cooking {
  border: 2px solid #f29442;
  position: relative;
}

.border-cooking::before {
  content: '';
  position: absolute;
  inset: -4px;
  border: 2px dashed #fcd34d;
  border-radius: inherit;
  opacity: 0.5;
}
```

- `@apply bg-white rounded-2xl shadow-lg` — usa la directiva `@apply` de Tailwind para aplicar clases utilitarias dentro de CSS personalizado. `bg-white` es fondo blanco. `rounded-2xl` son esquinas redondeadas de 16px. `shadow-lg` es una sombra predefinida de Tailwind. Usar `@apply` en lugar de duplicar estilos mantiene la consistencia con el sistema de diseño.
- `box-shadow` con tres valores separados por coma — CSS permite múltiples sombras en una sola propiedad, separadas por coma. Se aplican en orden (primero encima, luego debajo).
- `0 4px 6px -1px rgba(0,0,0,0.1)` — primera sombra: sin desplazamiento horizontal, 4px vertical, 6px de blur, -1px de spread (encoge la sombra), negro al 10% de opacidad. Simula la sombra principal del objeto.
- `0 2px 4px -1px rgba(0,0,0,0.06)` — segunda sombra: más pequeña y sutil, añade detalle al borde inferior de la tarjeta.
- `inset 0 1px 0 rgba(255,255,255,0.8)` — sombra interior. `inset` hace que la sombra vaya hacia adentro en lugar de hacia afuera. `0` horizontal, `1px` vertical (arriba), `0` blur, `rgba(255,255,255,0.8)` blanco al 80% de opacidad. Crea un sutil brillo en el borde superior de la tarjeta, simulando luz viniendo de arriba.
- `.card-food:hover box-shadow` — en hover se aumenta la primera sombra (`10px` y `15px` en lugar de `4px` y `6px`). Esto da la sensación de que la tarjeta se eleva al pasar el cursor. La sombra interior se mantiene igual.
- `.border-cooking { position: relative; }` — `position: relative` es necesario para que el pseudo-elemento `::before` (que tiene `position: absolute`) se posicione relativo a este elemento y no al documento completo.
- `.border-cooking::before { content: ''; }` — `content: ''` es requerido para que el pseudo-elemento exista aunque esté vacío. Sin esto, el navegador no renderiza el pseudo-elemento.
- `position: absolute; inset: -4px` — el pseudo-elemento sale 4px fuera del elemento en todos los lados. `inset: -4px` es shorthand de `top: -4px; right: -4px; bottom: -4px; left: -4px`.
- `border: 2px dashed #fcd34d` — borde punteado amarillo. El efecto combinado con el borde sólido naranja del elemento padre crea la apariencia de "doble borde de cocción" que refuerza la temática culinaria.
- `border-radius: inherit` — el pseudo-elemento hereda el `border-radius` del padre. Si el padre tiene `rounded-2xl` (16px de radio), el pseudo-elemento también tendrá 16px, haciendo que el segundo borde siga exactamente la misma forma redondeada.

### Bloque 4 — Efectos de brillo y animaciones

```css
.fresh-glow   { box-shadow: 0 0 15px rgba(34,197,94,0.3); }
.warning-glow { box-shadow: 0 0 15px rgba(248,113,113,0.3); }
.expired-glow { box-shadow: 0 0 15px rgba(220,38,38,0.3); }

@keyframes fresh-pulse {
  0%, 100% { box-shadow: 0 0 0 0 rgba(34,197,94,0.4); }
  50%       { box-shadow: 0 0 0 8px rgba(34,197,94,0); }
}
.animate-fresh-pulse { animation: fresh-pulse 2s ease-in-out infinite; }

@keyframes food-bounce {
  0%, 100% { transform: translateY(0) scale(1); }
  25%       { transform: translateY(-3px) scale(1.02); }
  50%       { transform: translateY(0) scale(1); }
  75%       { transform: translateY(-1px) scale(1.01); }
}
.animate-food-bounce { animation: food-bounce 1s ease-in-out infinite; }
```

- `.fresh-glow { box-shadow: 0 0 15px rgba(34,197,94,0.3) }` — aura verde para ingredientes frescos. `0 0` sin desplazamiento (la sombra es simétrica). `15px` de blur (difusión amplia). `rgba(34,197,94,0.3)` verde al 30% de opacidad (el color `fresh-500` de Tailwind). El resultado es un resplandor verde difuso alrededor del elemento.
- `.warning-glow` — aura roja para ingredientes próximos a caducar. `rgba(248,113,113,0.3)` es `red-400` de Tailwind al 30%.
- `.expired-glow` — aura roja más intensa para ingredientes caducados. `rgba(220,38,38,0.3)` es `red-600` de Tailwind, un rojo más oscuro y saturado.
- `@keyframes fresh-pulse` — animación de "ping" inspirada en los indicadores de estado de algunas apps. Simula el pulso de un corazón o señal de radio.
- `0%, 100% { box-shadow: 0 0 0 0 rgba(34,197,94,0.4) }` — en el inicio y final del ciclo, la sombra tiene `spread = 0` (no se ve) pero `opacity = 0.4` (lista para expandirse).
- `50% { box-shadow: 0 0 0 8px rgba(34,197,94,0) }` — en el punto medio, el spread creció a `8px` pero la opacidad bajó a `0`. El efecto visual es que el brillo se expande y desaparece, como una onda.
- `@keyframes food-bounce` — bounce personalizado con 4 keyframes que produce un movimiento más orgánico y realista que el `bounce` predefinido de Tailwind, que tiene un rebote muy pronunciado.
- `translateY(-3px) scale(1.02)` en el 25% — el elemento sube 3px y se agranda ligeramente. La combinación de traslación y escala imita el comportamiento de un objeto real que al subir se aleja un poco del observador.
- `translateY(-1px) scale(1.01)` en el 75% — rebote menor después del punto más alto. El elemento sube menos en el rebote que en la subida inicial, como si perdiera energía gradualmente.

### Bloque 5 — Fondos de pantalla

```css
.bg-kitchen {
  background:
    linear-gradient(135deg, rgba(253,251,247,0.95) 0%, rgba(254,240,213,0.95) 100%),
    url("data:image/svg+xml,...");
}

.bg-food-pattern {
  background-color: #fdfbf7;
  background-image:
    radial-gradient(circle at 25% 25%, rgba(242,148,66,0.1) 0%, transparent 50%),
    radial-gradient(circle at 75% 75%, rgba(34,197,94,0.1) 0%, transparent 50%),
    url("data:image/svg+xml,...");
}
```

- `.bg-kitchen` — fondo de degradado diagonal sobre un SVG de círculos. CSS permite múltiples valores en `background` separados por coma; el primero (el gradiente) se renderiza encima del SVG.
- `linear-gradient(135deg, rgba(253,251,247,0.95) 0%, rgba(254,240,213,0.95) 100%)` — gradiente diagonal de 135 grados (esquina superior izquierda a inferior derecha). Va de un blanco casi puro a un crema cálido. Ambos colores tienen `0.95` de opacidad, dejando ver apenas el SVG de fondo.
- `.bg-food-pattern` — el fondo principal usado en casi todas las pantallas de la app. Combina 3 capas de `background-image`.
- `background-color: #fdfbf7` — color sólido de base. Si las imágenes tardan en cargar o no pueden mostrarse, este color se ve. Es el mismo crema muy claro de la paleta.
- `radial-gradient(circle at 25% 25%, rgba(242,148,66,0.1) 0%, transparent 50%)` — gradiente radial circular centrado en el 25% horizontal y 25% vertical (esquina superior izquierda). Va de naranja al 10% de opacidad en el centro a completamente transparente en el 50% del radio. Crea un sutil resplandor naranja en esa esquina.
- `radial-gradient(circle at 75% 75%, rgba(34,197,94,0.1) 0%, transparent 50%)` — gradiente radial verde centrado en la esquina inferior derecha. Complementa el naranja de la esquina superior izquierda, creando un efecto de luz suave de dos colores.
- `url("data:image/svg+xml,...")` — SVG embebido directamente en el CSS como data URI. Evita la necesidad de un archivo de imagen separado y su correspondiente petición HTTP. El SVG es un patrón de estrellas en naranja muy tenue (3% de opacidad).

### Bloque 6 — Botón, input y badges

```css
.btn-food {
  @apply px-6 py-3 rounded-xl font-semibold transition-all duration-300;
  background: linear-gradient(145deg, #ffffff 0%, #fef7ee 100%);
  border: 2px solid #f29442;
  color: #b8440f;
}

.btn-food:hover {
  background: linear-gradient(145deg, #f29442 0%, #ed751d 100%);
  color: white;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(242,148,66,0.4);
}

.input-food {
  @apply w-full px-4 py-3 rounded-xl border-2 transition-all duration-300;
  background: linear-gradient(145deg, #ffffff 0%, #fdfbf7 100%);
  border-color: #eeddb3;
}

.input-food:focus {
  border-color: #f29442;
  box-shadow: 0 0 0 4px rgba(242,148,66,0.15);
  outline: none;
}

.badge-fresh {
  @apply px-3 py-1 rounded-full text-xs font-bold uppercase tracking-wide;
  background: linear-gradient(145deg, #dcfce7 0%, #bbf7d0 100%);
  color: #15803d;
}

.badge-priority {
  @apply px-3 py-1 rounded-full text-xs font-bold uppercase tracking-wide;
  background: linear-gradient(145deg, #fee2e2 0%, #fecaca 100%);
  color: #b91c1c;
}

.badge-expired {
  @apply px-3 py-1 rounded-full text-xs font-bold uppercase tracking-wide;
  background: linear-gradient(145deg, #7f1d1d 0%, #991b1b 100%);
  color: #fef2f2;
}
```

- `.btn-food` — botón principal de la app. Combina clases de Tailwind (via `@apply`) con CSS personalizado para el gradiente y borde.
- `@apply px-6 py-3 rounded-xl font-semibold transition-all duration-300` — padding horizontal de 24px, vertical de 12px, esquinas de 12px, texto semi-negrita, y transición de todas las propiedades en 300ms.
- `background: linear-gradient(145deg, #ffffff 0%, #fef7ee 100%)` — gradiente diagonal de blanco puro a crema muy suave. El ángulo `145deg` da un sutil efecto tridimensional.
- `border: 2px solid #f29442` — borde naranja de 2px. Define el color principal del botón en estado reposo.
- `color: #b8440f` — texto marrón oscuro que contrasta bien con el fondo claro y combina con el naranja del borde.
- `.btn-food:hover background: linear-gradient(145deg, #f29442 0%, #ed751d 100%)` — en hover, el fondo cambia a naranja sólido. El gradiente va de naranja medio a naranja más oscuro, manteniendo profundidad.
- `transform: translateY(-2px)` — el botón sube 2px al hacer hover, dando sensación de que se levanta. La transición de 300ms hace este movimiento suave.
- `box-shadow: 0 6px 20px rgba(242,148,66,0.4)` — sombra naranja amplia que aparece al hacer hover. Simula la luz que proyectaría el botón elevado. `6px` vertical, `20px` de blur, `rgba(242,148,66,0.4)` naranja al 40% de opacidad.
- `.input-food:focus { box-shadow: 0 0 0 4px rgba(242,148,66,0.15); outline: none; }` — anillo de enfoque personalizado. `outline: none` elimina el borde azul nativo del navegador (que es accesible pero visualmente inconsistente con el diseño). Lo reemplaza con `box-shadow: 0 0 0 4px ...` que produce el mismo efecto visual pero con el color de la app. `4px` de spread, `0` blur (anillo nítido), naranja al 15% de opacidad.
- `.badge-fresh, .badge-priority, .badge-expired` — los tres badges comparten la misma base de Tailwind (`@apply`) y solo difieren en colores de fondo y texto.
- `@apply ... rounded-full text-xs font-bold uppercase tracking-wide` — pastilla completamente ovalada (`rounded-full`), texto muy pequeño, negrita, mayúsculas, con espaciado entre letras amplio. El tracking amplio en mayúsculas mejora la legibilidad de textos cortos como "FRESCO", "PRIORITARIO", "CADUCADO".
- `.badge-expired` con fondo oscuro `#7f1d1d → #991b1b` y texto claro `#fef2f2` — contraste invertido. Los badges de fresco y prioritario tienen fondo claro y texto oscuro; el de caducado tiene fondo oscuro (rojo muy oscuro) y texto casi blanco. Esto hace al badge de caducado visualmente más severo e importante.

### Bloque 7 — Tabla

```css
.table-food { @apply w-full border-collapse; }

.table-food th {
  @apply text-left py-3 px-4 font-bold text-sm uppercase tracking-wider;
  background: linear-gradient(180deg, #fdfbf7 0%, #f5ead4 100%);
  color: #8d6b35;
  border-bottom: 2px solid #eeddb3;
}

.table-food td {
  @apply py-3 px-4 text-sm;
  border-bottom: 1px solid #f5ead4;
}

.table-food tr:hover td {
  background: linear-gradient(90deg, #fef7ee 0%, #fdfbf7 100%);
}
```

- `@apply w-full border-collapse` — la tabla ocupa el 100% del ancho disponible. `border-collapse` colapsa los bordes de celdas adyacentes en uno solo, en lugar de mostrar doble borde entre celdas vecinas.
- `.table-food th` — encabezados de la tabla con estilo consistente con la paleta de colores de la app.
- `text-left` — los encabezados se alinean a la izquierda. El comportamiento por defecto en HTML es centrado para `<th>`.
- `uppercase tracking-wider` — texto en mayúsculas con mayor espaciado entre letras. Convención de diseño para encabezados de tabla; hace los títulos más claros y formales.
- `background: linear-gradient(180deg, #fdfbf7 0%, #f5ead4 100%)` — gradiente vertical (de arriba abajo) de crema casi blanco a crema más cálido. Da profundidad al encabezado.
- `color: #8d6b35` — marrón dorado que complementa los tonos crema del fondo.
- `border-bottom: 2px solid #eeddb3` — borde inferior más grueso (2px) que el de las celdas (1px), haciendo visualmente más prominente la separación entre encabezado y datos.
- `.table-food td border-bottom: 1px solid #f5ead4` — borde inferior fino entre filas. `#f5ead4` es un crema muy claro que crea separación sin ser intrusivo.
- `.table-food tr:hover td` — cuando el cursor pasa sobre una fila, TODAS las celdas de esa fila cambian de color. El selector `tr:hover td` es necesario porque CSS no permite hacer hover en múltiples celdas simultáneamente con solo `:hover` en las `td`.
- `background: linear-gradient(90deg, #fef7ee 0%, #fdfbf7 100%)` — gradiente horizontal (izquierda a derecha) de crema cálido a blanco. Da la sensación de que la luz entra desde la izquierda al hacer hover, efecto sutil pero elegante.
