# general

Es para hacer test desde una perspectiva externa al objetivo (simular que somos un usuario real)

Tiene implementaciones en varios idiomas, pero nos vamos a centrar en typescript

# instalacion

Dentro de la carpeta del proyecto a probar /MiProyecto

```bash
pnpm create playwright
```

el va a instalarce con el node modules y va a crear una carpeta /tests y algunos archivos

Los archivos de test se van a inlcuir la terminacion `.spec.ts` ejemplo mi-test.spec.ts

# Ejecutar los test

```bash
pnpm exec playwright test
```

al final te redirige a una pestaña donde se ven los resultados `http://localhost:9323/`

## -g

filtra por los nombres de los test

```bash
pnpm exec playwright test -g "nombre del test"
```

## --ui

lanza el test pero mostrando informacion literalmente visual

```bash
npx playwright test --ui
```

![](./img_md/2025-06-06-12-14-50-image.jpg)

![](./img_md/2025-06-06-12-18-40-image.jpg)

# test

en los archivos `miPrueba.spect.ts` es necesario primero importar los elementos escenciales

```typescript
import { test, expect } from "@playwright/test";
import "dotenv/config"; // en caso de dotenv (recomendado)
```

un test individual se ejecuta dentro de una funcion `test` que toma dos argumentos, el titulo (string) del test y una funcion async cun un parametro `{ page }` que no retorna nada

```typescript
test("has title", async ({ page }) => { ... });
```

## page

representa a una pestaña del navegador

### goto

generalmente lo primero que se realiza en un test es ir a una direccion, y esto se hace a traves de la funcion `goto` de page que recibe una URL

```typescript
test("has title", async ({ page }) => {
  const EXPECTED_URL = process.env.EXPECTED_URL + "";

  await page.goto(EXPECTED_URL);
})
```

# Localizadores

son para obtener una manera de obtener los datos de un elmento para luego hacerle un test

Para obtenerlo se preciona sobre el boton (circulo) Pick Locator, y se lecciona el elmento a localizar (en este caso el boton blanco), luego en la pestaña de Locator sale el localizador de este elemento seleccionado

![](./img_md/2025-06-06-12-23-06-image.jpg)

## id

```typescript
const email = page.locator("#id-input-email");
```

## link

por contenido de boton relacionado a un `<a>`

```typescript
const boton = page.getByRole("link", { name: "Ir A Otra" });
```

## button

por boton con contenido

```typescript
const boton = page.getByRole("button", { name: "Send" });
```

# Esperas

Cuando es necesario esperar hasta que se cumpla algo

## waitForSelector

A beses es necesario esperar por la carga de algunos elementos

```javascript
await a_Page.waitForSelector("text=Send");
```

## waitForTimeout

esperar por un tiempo en ml segundos

```javascript
await a_Page.waitForTimeout(6000);
```

## waitForURL

Esperar porque carga una URL, generalmente atado a algun
evento que provoca la carga de esta URL

```javascript
await Promise.all([
    a_Page.waitForURL(`${URL_1}/verificar`),
    a_Page.keyboard.press("Enter"),
  ]);
```

# Acciones

se pueden usar desde los localizadores o desde el page pasandelo como primer argumento los datos del localizador

## click

```typescript
selector_boton.click();

// usando el localizador (1er arg el localizador, en este caso 
// por contenid de texto)
await a_Page.click("text=Send");
```

## fill

En el caso de los `<input>` para ponerle valores

```typescript
selector_input_email.fill("micorreo@example.com");

// usando el localizador (1er arg el localizador, en este caso por #id)
await a_Page.fill("#id-input-email", "asd@asd.com");
```

## press

para simular la utilizacion del teclado

```typescript
// usando el localizador (1er arg el localizador, en este caso por #id)
await a_Page.press("#id-input-email", "Enter");
```

### keyboard

luego de usar algun evento como `fill `o `click `, se puede mandar a precionar una tecla usando `page.keyboard` (osea este paso no incluye volver a seleccionar un elemento, sino que en el elemento que este previamente selecciona aplicar la tecla)

```javascript
await a_Page.fill("#id-input-email", "asd@asd.com");

//luego de tener seleccionado el elemento
await a_Page.keyboard.press("Enter");
```

# expect

Es en encargado de realizar las evaluaciones, recibe como argumento, un elemento de la pagina (como un page, o un localizador), o otro tipo de objeto. Y según su argumento provee de una serie de métodos de evaluación

```typescript
test("has title", async ({ page }) => {
  const EXPECTED_URL = process.env.EXPECTED_URL + "";

  await page.goto(EXPECTED_URL);
  await a_Page.click("text=Send");

  await expect(page.url()).toBe(`${EXPECTED_URL}/form`);
})
```

## toBeVisible

Comprueba si el elemento es visible

```typescript
await expect(paginaSuccess.successMessage).toBeVisible();
```

## toBeNull

```typescript
await expect(code_mail).toBeNull();
```

# describe

Para agrupar test, dentro de el van los test y los hooks comunes 

```typescript
test.describe("conjunto de test", () => {
  test.beforeAll(async ({ browser }) => {});
  test.afterAll(async ({ browser }) => {});
  beforeEach(async () => {});
  afterEach(async () => {});
  test("mi prueba interna1", async ({ page }) => {});
  test("mi prueba interna2", async ({ page }) => {});
});
```

# only

en un archivo de pruebas, si hay varias, y hay alguno marcado con esto, entonces de este archivo solo se va a ejecutar la prueba esta

```typescript
import { test, expect } from "@playwright/test";
import "dotenv/config";

test.only("prueba1", async ({ page }) => {});
test("prueba2", async ({ page }) => {});
```

# skip

Las pruebas marcadas con este no se ejecutarn, tambien es valido para descrive

```typescript
import { test, expect } from "@playwright/test";
import "dotenv/config";

test.skip("prueba1", async ({ page }) => {});
test("prueba2", async ({ page }) => {});
test.describe.skip("conjunto de test2", () => {});
```



# describe.serial

las pruebas dentro de este grupo se ejecutaran en serie (no de forma asincorna) (osea en orden, supongo que comenzando con la que mas arriba [cerca de la linea 0] este)

```typescript
test.describe.serial("conjunto de test2", () => {});
```

# Simular click (con class)

1- Vamos a almacenar los elementos comunes en una clase

/pages/PaginaAGuardar.ts

```typescript
import { Page, Locator } from "@playwright/test";

export default class PaginaAGuardar {
  readonly page: Page;
  readonly botonDeIr: Locator;
  constructor(page: Page) {
    this.page = page;
    this.botonDeIr = page.getByRole("link", { name: "Ir A Otra" });
  }
}
```

2- creamos el test

/test/mi-test.spec.ts

```typescript
import { test, expect } from "@playwright/test";

const URL = "http://localhost:3000/";
import PaginaAGuardar from "../pages/PaginaAGuardar";
let pagina: PaginaAGuardar;
test("move to page", async ({ page }) => {
  pagina = new PaginaAGuardar(page);
  await page.goto(URL);
  await paginaLanding.botonDeIr.click();
  await page.waitForTimeout(5000);
  expect(page.url()).toBe(`${URL}otra`);
});
```

al correr el test con --ui se ve como clickea sobre el boton y se ve la siguiente pagina. Si el boton no hubiese existido daria error con el test fallido

# Multiples paginas

se decalran todas al principio, (en este ejemplo como next se tarda en compilar pusimos entre todos los saltos un espera de 5s)

```typescript
import { test, expect } from "@playwright/test";

import PaginaLanding from "../pages/PaginaLanding";
import PaginaSendEmail from "../pages/PaginaSendEmail";
import PaginaSuccess from "../pages/PaginaSuccess";

const URL = "http://localhost:3000/";
let paginaLanding: PaginaLanding;
let paginaForm: PaginaSendEmail;
let paginaSuccess: PaginaSuccess;
test("sen email", async ({ page }) => {
  paginaLanding = new PaginaLanding(page);
  paginaForm = new PaginaSendEmail(page);
  paginaSuccess = new PaginaSuccess(page);
  await page.goto(URL);
  await paginaLanding.botonDeForm.click();
  await page.waitForTimeout(5000);
  expect(page.url()).toBe(`${URL}form`);
  await paginaForm.inputemail.fill("asd@asd.com");
  await paginaForm.botonSendEmail.click();
  await page.waitForTimeout(5000);
  expect(page.url()).toBe(`${URL}success`);
  expect(paginaSuccess.successMessage).toBeVisible();
});
```

# .env

si vas a usar variables de entorno en tus test, asegurate de tener instalado `dotenv`

```bash
pnpm install dotenv
```

y luego donde se vayan a usar las variables de entorno 

```typescript
import "dotenv/config";
test("has title", async ({ page }) => {
  const EXPECTED_URL = process.env.EXPECTED_URL + "";
  ...
}
```

# javascript

## Ejemplo Abrir y Cerrar navegador

Se puede usar para abrir una pestaña de un sitio web (obviamente este es el primer paso y lo siguiente es interactuar)

```bash
npx tsx  .\example\contexto_paginas.js
```

```javascript
const { chromium } = require("playwright");
URL_GOOGLE = "https://www.google.com";
URL_1 = "http://localhost:3000";
(async () => {
  const browser = await chromium.launch({ headless: false });
  //contexto google
  const googleContext = await browser.newContext();
  const googlePage = await googleContext.newPage();
  await googlePage.goto(URL_1);
  await googlePage.waitForTimeout(6000);
  console.log("contexto1 de google open");
  await browser.close();
})();
```

### multiple

```javascript
const { chromium } = require("playwright");
URL_GOOGLE = "https://www.google.com";
URL_1 = "http://localhost:3000";
URL_2 = `${URL_1}/form`;
(async () => {
  const browser = await chromium.launch({ headless: false });
  //contexto google
  const googleContext = await browser.newContext();
  const googlePage = await googleContext.newPage();
  await googlePage.goto(URL_1);
  await googlePage.waitForTimeout(6000);
  console.log("contexto1 de google open");

  //contexo wiki
  const wikipediaContext = await browser.newContext();
  const wikipediaPage = await wikipediaContext.newPage();
  await wikipediaPage.goto(URL_2);
  await wikipediaPage.waitForTimeout(6000);
  console.log("contexto1 de wikipedia open");

  await browser.close();
})();
```

# playwright.config.ts

almacena la configuracion, principalmente dentro del argumento metodo  `defineConfig` 

```typescript
import { defineConfig, devices } from "@playwright/test";


export default defineConfig({
  timeout: 90_000,
  testDir: "./tests",
  /* Run tests in files in parallel */
  fullyParallel: true,
    ......
  });
```

## fullyParallel

si es verdadero los test se ejecutan en paralelo 

## retries

cantidad de intentos para pruebas fallidas

El valor que se ve indica que en Integracion continua reintenta la prueba fallida 2 veces 

```typescript
export default defineConfig({
    retries: process.env.CI ? 2 : 0,
});
```

## workers

cantidad de workes, la configuracion que se ve indica que en CI seria 1, pero en local al ser undefined significa que segun el harware va a decidir la cantidad

```typescript
export default defineConfig({
    workers: process.env.CI ? 1 : undefined,
});
```

## webServer

Para cuando esta integrado directamente con un proyecto node (osea en el mismo directorio se encuentra esto de playwright y a la ves el proyecto que tal ves sea de react, vue o otro)

Esta configuracion por defecto esta comentada

```typescript
export default defineConfig({
    webServer: {
        command: 'npm run start',
        url: 'http://localhost:3000',
        reuseExistingServer: !process.env.CI,            
      },
});
```

## timeout

Para configurar el tiempo maximo del que dispone un prueba, por defecto son 30 segundos y esta propiedad no esta por defecto en la configuracion (si no esta se asume los 30s)

```typescript
export default defineConfig({
    timeout: 90_000,
});
```
