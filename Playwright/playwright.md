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

![](C:\Users\Rene\AppData\Roaming\marktext\images\2025-06-06-12-14-50-image.png)

![](C:\Users\Rene\AppData\Roaming\marktext\images\2025-06-06-12-18-40-image.png)

# Localizadores

son para obtener una manera de obtener los datos de un elmento para luego hacerle un test

Para obtenerlo se preciona sobre el boton (circulo) Pick Locator, y se lecciona el elmento a localizar (en este caso el boton blanco), luego en la pestaña de Locator sale el localizador de este elemento seleccionado

![](C:\Users\Rene\AppData\Roaming\marktext\images\2025-06-06-12-23-06-image.png)



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
test("has title", async ({ page }) => {
  pagina = new PaginaAGuardar(page);
  await page.goto(URL);
  await paginaLanding.botonDeIr.click();
  await page.waitForTimeout(5000);
  expect(page.url()).toBe(`${URL}otra`);
});
```

al correr el test con --ui se ve como clickea sobre el boton y se ve la siguiente pagina. Si el boton no hubiese existido daria error con el test fallido
