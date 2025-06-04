# Guía de Tailwind CSS

Tailwind CSS es un framework de CSS utilitario que elimina los estilos por defecto de CSS y utiliza clases predefinidas para estilizar componentes de manera rápida y eficiente. Esta guía cubre sus conceptos fundamentales y ejemplos prácticos.

---

## 1. Reglas Generales

- **Sintaxis de valores personalizados**:
  - Usa `[]` para valores personalizados (ej: `bg-[#b33030]`).
  - No uses espacios dentro de los corchetes; reemplázalos por guiones bajos (`_`).
  - Ejemplo: `font-[Times_New_Roman]`.

---

## 2. Colores de Fondo (`bg-`)

- **Colores fijos**:
  
  ```html
  <div className="bg-red-500">...</div> <!-- Color rojo predefinido -->
  ```
- **Colores personalizados**:
  
  ```html
  <div className="bg-[#b33030]">...</div> <!-- Hex personalizado -->
  ```
- **Opacidad**:
  - Sintaxis: `bg-{color}/{opacidad}` (0-100) o `rgba()`.
    
    ```html
    <div className="bg-red-500/50">...</div> <!-- 50% de opacidad -->
    <div className="bg-[rgba(41,12,62,0.5)]">...</div> <!-- RGBA personalizado -->
    ```

---

## 3. Gradientes

- **Lineales**:
  
  ```html
  <div className="bg-linear-to-r from-cyan-500 to-blue-500">...</div> <!-- Gradiente horizontal -->
  <div className="bg-[linear-gradient(to_right,#3b82f6,#22c55e)]">...</div> <!-- Personalizado -->
  ```
- **Radiales**:
  
  ```html
  <div className="rounded-full bg-radial from-pink-400 from-40% to-fuchsia-700">...</div>
  ```

---

## 4. Texto (`text-`)

- **Color de texto**:
  
  ```html
  <div className="text-red-500">Texto rojo</div>
  ```
- **Tamaño**:
  
  ```html
  <div className="text-xs">Extra pequeño</div>
  <div className="text-xl">Extra grande</div>
  <div className="text-2xl">2xl (múltiplos de xl)</div>
  ```
- **Alineación**:
  
  ```html
  <div className="text-center">Centrado</div>
  <div className="text-right">Alineado a la derecha</div>
  ```

---

## 5. Flexbox

- **Alineación básica**:
  
  ```html
  <div className="flex justify-center items-center">...</div> <!-- Centro H y V -->
  ```
- **Dirección de la cuadrícula**:
  
  ```html
  <div className="flex flex-col">...</div> <!-- Columna -->
  ```
- **Espaciado entre elementos**:
  
  ```html
  <div className="flex gap-4">...</div> <!-- 1rem (16px) de separación -->
  ```

---

## 6. Tamaño (Alto/Ancho: `h-`, `w-`)

- **Tamaños predefinidos**:
  - Escala: 0, 1, 2, 4, 8, 16, 32, 64 (ej: `h-16` = 64px).
    
    ```html
    <div className="w-full h-screen">Ocupa todo el ancho y alto de la pantalla</div>
    ```
- **Personalizados**:
  
  ```html
  <div className="w-[200px] h-[100px]">...</div>
  ```

---

## 7. Margen y Padding (`m-`, `p-`)

- **Sintaxis**:
  - `m{dirección}-{tamaño}` (ej: `mt-4` = margen superior de 1rem).
  - Direcciones: `t` (top), `b` (bottom), `l` (left), `r` (right), `x` (horizontal), `y` (vertical).
    
    ```html
    <div className="px-4 py-2">Padding horizontal y vertical</div>
    <div className="mx-auto">Centrado horizontal</div>
    ```

---

## 8. Grid

- **Columnas**:
  
  ```html
  <div className="grid grid-cols-3">...</div> <!-- 3 columnas -->
  ```
- **Span de columnas/filas**:
  
  ```html
  <div className="col-span-2">Ocupa 2 columnas</div>
  <div className="row-span-2">Ocupa 2 filas</div>
  ```

---

## 9. Responsive Design

- **Prefijos por breakpoint**:
  - `sm:`, `md:`, `lg:`, `xl:`, `2xl:`.
    
    ```html
    <div className="md:text-center lg:text-left">...</div>
    ```

---

## 10. Posicionamiento (`relative`, `absolute`)

- **Posición absoluta**:
  
  ```html
  <div className="relative">
    <div className="absolute top-0 right-0">...</div>
  </div>
  ```
- **Z-Index**:
  
  ```html
  <div className="z-10">...</div> <!-- Por encima de otros elementos -->
  ```

---

## 11. Bordes (`rounded-`, `border-`)

- **Redondeado**:
  
  ```html
  <div className="rounded-full">Círculo</div>
  <div className="rounded-tl-lg">Esquina superior izquierda redondeada</div>
  ```
- **Bordes**:
  
  ```html
  <div className="border-2 border-red-500">Borde rojo de 2px</div>
  ```

---

## 12. Efectos Hover y Transiciones

- **Hover**:
  
  ```html
  <div className="hover:bg-blue-500">Cambia color al pasar el mouse</div>
  ```
- **Transiciones**:
  
  ```html
  <div className="transition-all duration-300 ease-in-out">...</div>
  ```

---

## 13. Sombras (`shadow-`)

- **Predefinidas**:
  
  ```html
  <div className="shadow-lg">Sombra grande</div>
  <div className="shadow-2xl">Sombra muy grande</div>
  ```
- **Color personalizado**:
  
  ```html
  <div className="shadow-md shadow-blue-500">Sombra azul</div>
  ```

---

## 14. Animaciones

- **Clases predefinidas**:
  
  ```html
  <div className="animate-spin">Gira infinitamente</div>
  <div className="animate-bounce">Rebota</div>
  <div className="animate-pulse">Pulso de opacidad</div>
  ```

---

## `@apply`

Para reutilizar un conjunto de clases de Tailwind CSS, usa la directiva `@apply` en tu archivo CSS (ejemplo: `index.css` en React).

```css
@import "tailwindcss";

.clase_comun {
  @apply text-3xl font-bold text-amber-200;
}
```

---

## Estilos de Texto (Bold, Italic, Normal)

Controla el grosor (`font-weight`) y el estilo (`font-style`) del texto.

- **Clases Principales**:
  
  - Normal: `font-normal`
  - Bold: `font-bold`
  - Italic: `italic`
  - Underline: `underline`
  - Line-through: `line-through`

- **Ejemplos Combinados**:
  
  ```html
  <div className="font-bold italic">Texto en negrita y cursiva</div>
  <div className="underline line-through">Texto subrayado y tachado</div>
  ```

- **Variantes de Grosor (Font-Weight)**:
  
  - `font-thin` (100)
  - `font-light` (300)
  - `font-medium` (500)
  - `font-semibold` (600)
  - `font-black` (900)

- **Ejemplo**:
  
  ```html
  <div className="font-light">Texto ligero</div>
  <div className="font-black">Texto muy grueso</div>
  ```

- **Notas Adicionales**:
  
  - Personalización: Extiende la configuración en `tailwind.config.js` para grosores o estilos no incluidos.
  - Combinación con colores: Ejemplo: `text-red-500 font-bold`.

---

## Group

Permite disparar eventos como `hover` en elementos hijos basados en el estado del padre.

- **Ejemplo**:
  
  ```html
  <div className="group mt-2 border-2 border-red-500 p-5 hover:bg-amber-300">
    <div className="group-hover:bg-amber-600 border-2 border-red-500 bg-black">
      Boton
    </div>
  </div>
  ```

- **Subgrupos**:
  Para evitar la propagación de eventos en subgrupos, usa identificadores:
  
  ```html
  <div className="group/general mt-3 border-2 border-red-500 p-5 hover:bg-blue-950">
    <span className="text-red-500">group/general</span>
    <div className="group-hover/general:bg-amber-950 mt-5 border-2 border-red-500 bg-black">
      <span className="text-red-500">group-hover/general:bg-amber-950</span>
    </div>
    <div className="group/individual mt-3 border-2 border-red-500 p-5 hover:bg-blue-950 bg-black">
      <span className="text-red-500">group/individual</span>
      <div className="group-hover/individual:bg-amber-950 mt-5 border-2 border-red-500 bg-black">
        <span className="text-red-500">group-hover/individual:bg-amber-950</span>
      </div>
    </div>
  </div>
  ```

---

## Otros Conceptos

### `overflow-hidden`

Oculta el contenido que se desborda del contenedor. Útil para ajustar imágenes dentro de contenedores con formas específicas.

### `-mt-32`

Usa márgenes negativos para desplazar elementos fuera de su posición normal, permitiendo superposiciones.

### `text-transparent` y `bg-clip-text`

- `text-transparent`: Hace el texto transparente.
- `bg-clip-text`: Recorta el fondo para que coincida con el texto, útil para gradientes o imágenes.
  
  ```html
  <div className="bg-gradient-to-r from-pink-500 to-violet-500 font-extrabold bg-clip-text text-transparent">
    con el color de un fondo gradiente
  </div>
  ```

### `disabled`

Aplica propiedades cuando un elemento está deshabilitado.

- Sintaxis: `disabled:propiedad`

### `outline` y `ring`

- **Outline**: Dibuja un contorno alrededor del elemento sin afectar el diseño.
  
  - Clases: `outline`, `outline-none`, `outline-dashed`, etc.

- **Ring**: Proporciona un efecto de anillo más visual y personalizable.
  
  - Clases: `ring`, `ring-2`, `ring-red-500`, `ring-offset-4`, etc.

- **Ejemplo de outline**:
  
  ```html
  <div className="outline outline-2 outline-red-500 outline-offset-4">
    Este elemento tiene un contorno rojo.
  </div>
  ```

- **Ejemplo de ring**:
  
  ```html
  <div className="ring ring-2 ring-green-500">Anillo grueso</div>
  ```

### `invalid`

Aplica propiedades a inputs inválidos.

- Sintaxis: `invalid:propiedad`
- Ejemplo:
  
  ```html
  <input className="invalid:focus:ring-red-400 focus:outline-none focus:ring-1 focus:ring-purple-600 border border-gray-300 w-full px-3 py-2 mb-4 rounded-md" type="email" placeholder="Correo" />
  ```

---

Esta guía ofrece una referencia rápida y completa para usar Tailwind CSS. Para más detalles, consulta la [documentación oficial](https://tailwindcss.com/docs).