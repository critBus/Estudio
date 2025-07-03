# Guía Completa de React

## Crear un Proyecto

- Usando Yarn:
  
  ```bash
  yarn create react-app NOMBRE_CARPETA_PROYECTO
  ```

- Usando Vite:
  
  ```bash
  yarn create vite
  ```
  
  Luego seleccionar React.

- Usando PNPM:
  
  ```bash
  pnpm create vite
  ```

## Ejecutar el Proyecto

- Para iniciar el servidor de desarrollo:
  
  ```bash
  npm start
  ```
  
  Esto ejecuta el contenido de `src/App.js`, específicamente la función `App()`.

## Crear el Compilado

- Para generar los archivos de producción:
  
  ```bash
  npm run build
  ```
  
  Esto crea los elementos en la carpeta `dist`.

## Ejecutar Tests

- Para correr los tests:
  
  ```bash
  npm run test
  ```

## Estructura de Archivos

- **/src/package.json**: Contiene las dependencias del proyecto.
  
  - `"react"`: Librería base de React.
  - `"react-dom"`: Para usar React en la web.

- **Instalar Dependencias**:
  
  - Yarn: `yarn`
  - NPM: `npm install`
  - PNPM: `pnpm install`

- **/src/main.jsx**: Punto de entrada del proyecto.

- **/src/index.html**: Contiene la etiqueta con `id="root"` donde se renderizan los componentes.

- **/src/index.js**: Equivalente a `main.jsx` cuando se usa `create-react-app`.

- **/src/vite.config.js**: Configuración de Vite.

## CSS Generales

- Los estilos generales se importan en `main.jsx`:
  
  ```jsx
  import './css/estilos_generales.css';
  ```

- Renderizado del componente principal:
  
  ```jsx
  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
  ```

## Carpetas para Archivos Estáticos

- **/public/**: Para contenido estático como imágenes. Las rutas relativas no necesitan incluir "public".

- **/assets/**: También para archivos estáticos.

## Elementos y Componentes

- **Elemento**: Lo que se renderiza, por ejemplo, JSX.

- **Componentes**: Funciones que retornan JSX, permitiendo la reutilización de código. Los datos fluyen de padres a hijos.
  
  - Deben nombrarse en PascalCase.
  
  - Ejemplo:
    
    ```jsx
    export const CustomButton = ({ textoEnElBoton }) => (
      <button style={{ backgroundColor: "blue" }}>{textoEnElBoton}</button>
    );
    ```
  
  - Uso:
    
    ```jsx
    <CustomButton textoEnElBoton="texto1" />
    ```

- **Children**: Permite pasar contenido dentro del componente.
  
  - Ejemplo:
    
    ```jsx
    export const CustomButton = ({ children }) => (
      <button style={{ backgroundColor: "blue" }}>{children}</button>
    );
    ```
  
  - Uso:
    
    ```jsx
    <CustomButton>Texto1a</CustomButton>
    ```

- **PropTypes**: Para definir tipos de propiedades (solo en desarrollo).
  
  - Ejemplo:
    
    ```jsx
    import PropTypes from "prop-types";
    
    export default function Componente2({ title, id, usuario }) {
      return <>Componente2</>;
    }
    
    Componente2.propTypes = {
      title: PropTypes.string.isRequired,
      id: PropTypes.number.isRequired,
      usuario: PropTypes.shape({
        nombre: PropTypes.string.isRequired,
        edad: PropTypes.number.isRequired,
      }).isRequired,
    };
    ```

- **Variables Externas**: Se pueden declarar fuera de la función para que sean constantes.
  
  - Ejemplo:
    
    ```jsx
    const fecha = new Date();
    
    export const ComponentePrueba = () => {
      return <>{JSON.stringify(fecha)}</>;
    };
    ```

## JSX

- Lenguaje de plantillas de React.

- Debe estar dentro de una sola etiqueta. Se puede usar `<React.Fragment>` o `<>` para envolver múltiples elementos.

- **Atributos**:
  
  - `class` → `className`
  
  - Estilos en línea: Se pasan como objetos.
    
    ```jsx
    <button style={{ backgroundColor: "blue" }}>button</button>
    ```

- **True por Defecto**: Atributos sin valor son `true`.
  
  - Ejemplo:
    
    ```jsx
    <CustomButton ponerTextoGrande />
    ```

- **Pasar Múltiples Argumentos**:
  
  - Usar spread operator: `<CustomButton {...props} />`

- **Arreglos**: Necesitan una `key`.
  
  - Ejemplo:
    
    ```jsx
    {datos_en_arreglo.map(v => <span key={v}>{v}</span>)}
    ```

- **JSON.stringify**: Para renderizar objetos.
  
  - Ejemplo:
    
    ```jsx
    {JSON.stringify(objeto)}
    ```

## Hooks

- Funciones especiales para acceder a características de React, como el estado o el ciclo de vida.

### Rutas

- Instalar: `pnpm install react-router-dom`

- Configuración con `createBrowserRouter`:
  
  ```jsx
  import { createBrowserRouter, RouterProvider } from "react-router-dom";
  
  const router = createBrowserRouter([
    { path: "/", element: <PruebaTWcss /> },
    { path: "/doc/", element: <DocTWcss /> },
  ]);
  
  const Rutas = () => <RouterProvider router={router} />;
  ```

- Alternativa con `BrowserRouter` y `Routes`:
  
  ```jsx
  import { BrowserRouter, Routes, Route } from "react-router-dom";
  
  <BrowserRouter>
    <Routes>
      <Route path="/" element={<Fib />} />
      <Route path="/otherpage" element={<OtherPage />} />
    </Routes>
  </BrowserRouter>
  ```

- **Switch**: Para detener en la primera coincidencia (en versiones anteriores).

- **Exact**: Para coincidencia exacta.

- **Link** y **NavLink**: Para navegación.

- **useParams**: Para parámetros en la URL.

- **useLocation**: Para obtener información de la URL.

- **useHistory**: Para manipular el historial.

- **Redirect** y **Navigate**: Para redirecciones.

### Estado

- El estado maneja la lógica y datos internos de un componente.

- Es inmutable y asíncrono.

- Se maneja con `useState`:
  
  ```jsx
  import { useState } from "react";
  
  const [getFollowing, setFollowing] = useState(false);
  ```

- Actualizar estado:
  
  ```jsx
  setLista([...lista, nuevoElemento]);
  ```

- Ejemplo:
  
  ```jsx
  const [lista, setLista] = useState([]);
  
  const agregarElemento = (nuevo) => {
    setLista([...lista, { cantidad: lista.length, elemento: `${nuevo} - ${lista.length}` }]);
  };
  
  return (
    <>
      <button onClick={() => agregarElemento("a")}>Agregar</button>
      <ul>
        {lista.map((v, i) => <li key={i}>{JSON.stringify(v)}</li>)}
      </ul>
    </>
  );
  ```

## Eventos

- Se pasan métodos a atributos de eventos HTML.
  
  ```jsx
  const botonPulsado = () => alert('hola mundo');
  
  <button onClick={botonPulsado}>Click me</button>
  ```

## Operadores Booleanos en JSX

- Evaluación de corto circuito:
  
  ```jsx
  {variableBooleana && <span>agrega este jsx si es true</span>}
  ```

## Videos

- Usar `useRef` para controlar elementos multimedia.
  
  ```jsx
  import { useRef } from 'react';
  
  const videoRef = useRef(null);
  
  const videoPlay = () => videoRef.current.play();
  
  return (
    <video ref={videoRef}>
      <source src={referencia_al_video} type='video/mp4' />
    </video>
    <button onClick={videoPlay}>Play</button>
  );
  ```

## Ciclo de Vida de Componentes

- **Montaje**: Cuando se visualiza.
- **Actualización**: Cuando cambia el estado o props.
- **Desmontaje**: Cuando se deja de visualizar.

### useEffect

- Para ejecutar código en momentos específicos.
  
  ```jsx
  useEffect(() => {
    // Código al montar o actualizar
    return () => {
      // Código al desmontar
    };
  }, [dependencias]);
  ```

## Formularios

- Manejar formularios con estado.
  
  ```jsx
  const [formulario, setFormulario] = useState({ nombre: "", edad: 0 });
  
  const onInputChange = ({ target }) => {
    const { name, value } = target;
    setFormulario({ ...formulario, [name]: value });
  };
  
  return (
    <form onSubmit={onSubmit}>
      <input type="text" name="nombre" value={formulario.nombre} onChange={onInputChange} />
      <input type="text" name="edad" value={formulario.edad} onChange={onInputChange} />
      <button type="submit">Enviar</button>
    </form>
  );
  ```

## useMemo y useCallback

- **useMemo**: Memoriza resultados de funciones.
  
  ```jsx
  const resultado = useMemo(() => computeExpensiveValue(a, b), [a, b]);
  ```

- **useCallback**: Memoriza funciones.
  
  ```jsx
  const memoizedCallback = useCallback(() => {
    doSomething(a, b);
  }, [a, b]);
  ```

## useReducer

- Para manejar estados complejos.
  
  ```jsx
  const [state, dispatch] = useReducer(reducer, initialState);
  ```

- Ejemplo de reducer:
  
  ```jsx
  const reducer = (state, action) => {
    switch (action.type) {
      case 'increment':
        return { count: state.count + 1 };
      default:
        return state;
    }
  };
  ```

## createContext

- Para proveer datos a componentes hijos.
  
  ```jsx
  const UsuarioContext = createContext(initialValue);
  
  const UsuarioProvider = ({ children }) => (
    <UsuarioContext.Provider value={value}>
      {children}
    </UsuarioContext.Provider>
  );
  
  // Uso
  const { usuario } = useContext(UsuarioContext);
  ```

## Redux

- Instalar: 
  
  ```bash
  pnpm install redux react-redux @reduxjs/toolkit
  ```

- **Slice**:
  
  ```jsx
  import { createSlice } from "@reduxjs/toolkit";
  
  const contadorSlice = createSlice({
    name: "contador",
    initialState: { cantidad: 10 },
    reducers: {
      incrementarContador: (state) => { state.cantidad += 1; },
      restarContador: (state, action) => { state.cantidad -= action.payload.restar; },
    },
  });
  
  export const { incrementarContador, restarContador } = contadorSlice.actions;
  export default contadorSlice.reducer;
  ```

- **Store**:
  
  ```jsx
  import { configureStore } from "@reduxjs/toolkit";
  import contadorReducer from "./slices/contadorSlice";
  
  export const store = configureStore({
    reducer: { contador: contadorReducer },
  });
  ```

- **Provider**:
  
  ```jsx
  import { Provider } from "react-redux";
  import { store } from "./store";
  
  <Provider store={store}>
    <App />
  </Provider>
  ```

- **useSelector y useDispatch**:
  
  ```jsx
  import { useSelector, useDispatch } from "react-redux";
  
  const contador = useSelector((state) => state.contador.cantidad);
  const dispatch = useDispatch();
  
  dispatch(incrementarContador());
  ```

## Persist

- Para persistir el estado de Redux.
  
  ```bash
  pnpm install redux-persist redux-thunk
  ```

## styled-components

- Librería para estilos en componentes.
  
  ```bash
  pnpm install styled-components
  ```

- Ejemplo:
  
  ```jsx
  import styled from "styled-components";
  
  export const MiCaja = styled.div`
    width: 170px;
    height: 90px;
    margin: 150px;
    background-color: blue;
    &:hover {
      background-color: red;
    }
  `;
  
  // Uso
  <MiCaja />
  ```

- **Estilos Dinámicos**:
  
  ```jsx
  export const MiCaja = styled.div`
    background-color: ${({ atributo }) => (atributo ? 'blue' : 'green')};
  `;
  
  // Uso
  <MiCaja atributo={true} />
  ```

- **Herencia**:
  
  ```jsx
  export const CajaHija = styled(MiCaja)`
    width: 170px;
  `;
  ```

- **Conjunto de Estilos**:
  
  ```jsx
  import styled, { css } from "styled-components";
  
  const EstilosComunes = css`
    width: 170px;
    height: 90px;
    margin: 150px;
  `;
  
  export const MiCaja = styled.div`
    ${({ atributoConjunto }) =>
      atributoConjunto
        ? css`
            ${EstilosComunes}
            text-align: center;
            font-size: large;
          `
        : css`
            ${EstilosComunes}
            text-align: start;
            font-size: medium;
          `};
  `;
  ```