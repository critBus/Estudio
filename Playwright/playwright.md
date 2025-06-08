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

## --ui

lanza el test pero mostrando informacion literalmente visual

```bash
npx playwright test --ui
```

![](./img_md/2025-06-06-12-14-50-image.jpg)

![](./img_md/2025-06-06-12-18-40-image.jpg)

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

## Esperas

Cuando es necesario esperar hasta que se cumpla algo

### waitForSelector

A beses es necesario esperar por la carga de algunos elementos

```javascript
await a_Page.waitForSelector("text=Send");
```

### waitForTimeout

esperar por un tiempo en ml segundos

```javascript
await a_Page.waitForTimeout(6000);
```

### waitForURL

Esperar porque carga una URL, generalmente atado a algun
evento que provoca la carga de esta URL

```javascript
await Promise.all([
    a_Page.waitForURL(`${URL_1}/verificar`),
    a_Page.keyboard.press("Enter"),
  ]);
```

## Acciones

se pueden usar desde los localizadores o desde el page pasandelo como primer argumento los datos del localizador

### click

```typescript
selector_boton.click();

// usando el localizador (1er arg el localizador, en este caso 
// por contenid de texto)
await a_Page.click("text=Send");
```

### fill

En el caso de los `<input>` para ponerle valores

```typescript
selector_input_email.fill("micorreo@example.com");

// usando el localizador (1er arg el localizador, en este caso por #id)
await a_Page.fill("#id-input-email", "asd@asd.com");
```

### press

para simular la utilizacion del teclado

```typescript
// usando el localizador (1er arg el localizador, en este caso por #id)
await a_Page.press("#id-input-email", "Enter");
```

#### keyboard

luego de usar algun evento como `fill `o `click `, se puede mandar a precionar una tecla usando `page.keyboard` (osea este paso no incluye volver a seleccionar un elemento, sino que en el elemento que este previamente selecciona aplicar la tecla)

```javascript
await a_Page.fill("#id-input-email", "asd@asd.com");

//luego de tener seleccionado el elemento
await a_Page.keyboard.press("Enter");
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
