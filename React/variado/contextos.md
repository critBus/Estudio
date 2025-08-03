### **¿Qué es el Contexto en React?**

El **Context API** de React permite compartir datos (como estado, configuraciones, temas, etc.) entre componentes **sin pasar props manualmente** a través de cada nivel del árbol de componentes (evitando el *prop drilling*).

---

### **Pasos para usar Context en React**

#### 1. **Crear el Contexto**

Usa `createContext` para definir un nuevo contexto.  

```jsx
// Archivo: ThemeContext.js
import { createContext } from 'react';

// Creamos el contexto con un valor por defecto (opcional)
const ThemeContext = createContext({
  theme: 'light', // Valor por defecto
  toggleTheme: () => {} // Función vacía como placeholder
});

export default ThemeContext;
```

---

#### 2. **Crear un Proveedor (Provider)**

El **Provider** envuelve a los componentes hijos y les provee el valor del contexto.  

```jsx
// Archivo: ThemeProvider.js
import { useState, useContext } from 'react';
import ThemeContext from './ThemeContext';

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prevTheme => (prevTheme === 'light' ? 'dark' : 'light'));
  };

  // Valor que se compartirá con los consumidores
  const value = { theme, toggleTheme };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};
```

---

#### 3. **Envolver la App con el Provider**

En tu componente principal (ej: `App.js`), envuelve los componentes que necesitan acceso al contexto.  

```jsx
// Archivo: App.js
import { ThemeProvider } from './ThemeProvider';
import Toolbar from './Toolbar';

function App() {
  return (
    <ThemeProvider> {/* ¡Aquí se inyecta el contexto! */}
      <div>
        <Toolbar />
      </div>
    </ThemeProvider>
  );
}
```

---

#### 4. **Acceder a las Variables del Contexto**

Usa el **hook `useContext`** en cualquier componente hijo para acceder a los valores.  

```jsx
// Archivo: Toolbar.js
import { useContext } from 'react';
import ThemeContext from './ThemeContext';

const Toolbar = () => {
  // Accedemos al valor del contexto
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <div>
      <p>Tema actual: {theme}</p>
      <button onClick={toggleTheme}>
        Cambiar tema
      </button>
    </div>
  );
};
```

---

### **¿Cómo acceder a las variables?**

1. **Importa el contexto** en el componente donde lo necesites:
   
   ```jsx
   import ThemeContext from './ThemeContext';
   ```

2. **Usa `useContext`** para obtener el valor:
   
   ```jsx
   const { theme, toggleTheme } = useContext(ThemeContext);
   ```
   
   - `theme`: Variable de estado (ej: `'light'` o `'dark'`).
   - `toggleTheme`: Función para modificar el estado.

---

### **Ejemplo Completo**

```jsx
// 1. Crear contexto (ThemeContext.js)
import { createContext } from 'react';
export default createContext({ theme: 'light', toggleTheme: () => {} });

// 2. Proveedor (ThemeProvider.js)
import { useState } from 'react';
import ThemeContext from './ThemeContext';

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  const toggleTheme = () => setTheme(prev => (prev === 'light' ? 'dark' : 'light'));
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// 3. App (App.js)
import { ThemeProvider } from './ThemeProvider';
import Toolbar from './Toolbar';

function App() {
  return (
    <ThemeProvider>
      <Toolbar />
    </ThemeProvider>
  );
}

// 4. Componente que usa el contexto (Toolbar.js)
import { useContext } from 'react';
import ThemeContext from './ThemeContext';

const Toolbar = () => {
  const { theme, toggleTheme } = useContext(ThemeContext);
  return (
    <div style={{ background: theme === 'dark' ? '#333' : '#fff' }}>
      <button onClick={toggleTheme}>
        {theme === 'light' ? 'Modo Oscuro' : 'Modo Claro'}
      </button>
    </div>
  );
};
```

---

### **Errores Comunes y Soluciones**

1. **Acceder al contexto fuera del Provider**  
   Si usas `useContext` en un componente **no envuelto** por el `Provider`, React usará el **valor por defecto** (definido en `createContext`).  
   ✅ **Solución:** Asegúrate de envolver los componentes con el `Provider`.

2. **Valores complejos sin memoización**  
   Si el valor del `Provider` es un objeto/array creado en cada render, puede causar re-renders innecesarios.  
   ✅ **Solución:** Usa `useMemo` para memoizar el valor:
   
   ```jsx
   const value = useMemo(() => ({ theme, toggleTheme }), [theme]);
   ```

---

### **Resumen**

- **`createContext`**: Define el contexto.
- **`Provider`**: Envuelve componentes y provee el valor.
- **`useContext`**: Accede al valor en componentes hijos.
- **Estructura ideal**: Separa la lógica del contexto en archivos específicos (ej: `ThemeContext.js` y `ThemeProvider.js`).

¡Con esto podrás compartir estado global sin complicaciones! 🚀
