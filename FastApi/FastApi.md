# Guía de FastAPI

FastAPI es un framework web moderno y rápido para construir APIs con Python, similar a Flask en su flexibilidad, pero con características adicionales como la documentación automática y el soporte para operaciones asíncronas. A continuación, se presenta una guía detallada sobre cómo utilizar FastAPI.

## Introducción

FastAPI permite crear APIs de manera rápida y eficiente. Al igual que Flask, ofrece flexibilidad para estructurar el proyecto como desees, pero incluye ventajas como la validación automática de datos y la generación de documentación interactiva.

## Requisitos

Para trabajar con FastAPI, necesitas instalar las siguientes bibliotecas y dependencias:

- `fastapi`
- `uvicorn[standard]`
- `sqlalchemy`
- `python-decouple`
- `cryptography`
- `python-jose[cryptography]`
- `passlib[bcrypt]`
- `bcrypt==3.2.0`
- `pydantic`
- `python-multipart`
- `httpx`
- `pytest`

Instálalas con el siguiente comando:

```bash
pip install fastapi uvicorn[standard] sqlalchemy python-decouple cryptography python-jose[cryptography] passlib[bcrypt] bcrypt==3.2.0 pydantic python-multipart httpx pytest
```

## Estructura Básica

### Archivo `main.py`

Es común tener un archivo `main.py` como punto de entrada de la aplicación. Aquí se instancia FastAPI:

```python
from fastapi import FastAPI

app = FastAPI()
```

En este archivo también puedes definir los endpoints, aunque en proyectos más grandes se recomienda organizarlos en módulos separados.

### Ejecución de la Aplicación

Para ejecutar la aplicación, usa Uvicorn, un servidor ASGI. El comando depende de la ubicación de `main.py`:

- Si está en la raíz:
  
  ```bash
  uvicorn main:app --reload
  ```

- Si está en un directorio como `src/`:
  
  ```bash
  uvicorn src.main:app --reload
  ```

Para especificar un puerto diferente:

```bash
uvicorn main:app --reload --port 5000
```

## Endpoints

### Método GET

#### Respuesta en Texto Plano

```python
from fastapi import FastAPI
from fastapi.responses import PlainTextResponse

app = FastAPI()

@app.get("/")
def home():
    return PlainTextResponse("Hola Mundo")
```

#### Respuesta en JSON

```python
@app.get("/home")
def home():
    return {"data": "Hola Mundo"}
```

Alternativamente, con `JSONResponse`:

```python
from fastapi.responses import JSONResponse

@app.get("/home")
def home():
    return JSONResponse({"data": "Hola Mundo"})
```

#### Respuesta en HTML

```python
from fastapi.responses import HTMLResponse

@app.get("/hola")
def home_con_html():
    return HTMLResponse("<h1>Hola</h1>")
```

#### Parámetros en la Ruta

Los parámetros deben ser tipados:

```python
@app.get("/parametro/{id}")
def home(id: int):
    return f"El parámetro - {id}"
```

Para validaciones adicionales, usa `Path`:

```python
from fastapi import Path

@app.get("/parametro/{id}")
def home(id: int = Path(gt=0)):
    return f"El parámetro - {id}"
```

#### Parámetros de Consulta (Query Params)

Los parámetros de consulta no están en la ruta y deben ser tipados. Si no tienen valor por defecto, son obligatorios:

```python
@app.get("/query")
def home(qp_category: str, qp_opcional: str = "zxc"):
    return f"El query param - {qp_category}"
```

Para validarlos, usa `Query`:

```python
from fastapi import Query

@app.get("/query")
def home(qp_category: str = Query(max_length=5)):
    return f"El query param - {qp_category}"
```

### Método POST

Para manejar datos en el cuerpo de la solicitud, usa `Body`:

```python
from fastapi import Body

@app.post("/create")
def create_data(name: str = Body(), age: int = Body()):
    data.append({"name": name, "age": age})
    return data
```

### Método PUT

Para actualizar recursos:

```python
@app.put("/data/{id}")
def edit_data(id: int, name: str = Body(), age: int = Body()):
    for d in data:
        if d["id"] == id:
            d["name"] = name
            d["age"] = age
    return data
```

### Método DELETE

Para eliminar recursos:

```python
@app.delete("/data/{id}")
def delete_data(id: int):
    for d in data:
        if d["id"] == id:
            data.remove(d)
    return data
```

## Respuestas Especiales

### RedirectResponse

Para redirigir a otra ruta, usa `status_code=303` para redirecciones locales:

```python
from fastapi.responses import RedirectResponse

@app.post("/data")
def create_data(data_entry: Data):
    new_data = data_entry.model_dump()
    if "id" in new_data and new_data["id"] is None:
        new_data["id"] = len(data)
    data.append(new_data)
    return RedirectResponse("/data", status_code=303)
```

### FileResponse

Para servir archivos:

```python
from fastapi.responses import FileResponse

@app.get("/file")
def get_file():
    url = "path/to/file"
    return FileResponse(url)
```

## Personalización de Respuestas

### Código de Estado por Defecto

Especifica un código de estado por defecto en el endpoint:

```python
@app.post("/data", status_code=201)
def create_data(data_entry: Data):
    new_data = data_entry.model_dump()
    if "id" in new_data and new_data["id"] is None:
        new_data["id"] = len(data)
    data.append(new_data)
    return new_data
```

### HTTPException

Para lanzar errores personalizados (4XX):

```python
from fastapi import HTTPException

@app.get("/error")
def get_data():
    raise HTTPException(status_code=400, detail="porque sí")
```

## Documentación con Swagger

FastAPI genera documentación interactiva en `http://localhost:8000/docs`. Personalízala así:

```python
app = FastAPI(title="El título en Swagger", version="2.0.0")
```

### Etiquetas (Tags)

Agrupa rutas en la documentación:

```python
@app.get("/", tags=["Home"])
def home():
    return "Hola Mundo"
```

## Modelos de Datos con Pydantic

### BaseModel

Define la estructura de datos:

```python
from pydantic import BaseModel

class Data(BaseModel):
    name: str
    age: int
```

Úsalo en endpoints para validar el cuerpo:

```python
@app.post("/data")
def create_data(data_entry: Data):
    new_data = data_entry.model_dump()
    data.append({"id": len(data), **new_data})
    return data
```

### Campos Opcionales

```python
from typing import Optional

class Data(BaseModel):
    id: Optional[int] = None
    name: str
    age: int
```

### Validación con Field

Para validaciones específicas:

```python
from pydantic import Field

class Data(BaseModel):
    name: str = Field(min_length=5, max_length=15)
    age: int = Field(ge=18)
```

### Ejemplo en Swagger

Personaliza el ejemplo en la documentación:

```python
class Fecha(BaseModel):
    year: int = Field(le=datetime.date.today().year, ge=1900, default=2025)
    month: int = Field(ge=1, le=12)
    day: int = Field(ge=1, le=31)

    model_config = {
        'json_schema_extra': {
            'example': {
                'year': 2024,
                'month': 11,
                'day': 25,
            }
        }
    }
```

### Validadores Personalizados (Deprecado)

Aunque está deprecado, puedes usar `@validator`:

```python
from pydantic import validator

class Data(BaseModel):
    name: str

    @validator('name')
    def validate_name(cls, value):
        if value == "aaaaa":
            raise ValueError("Este nombre no está permitido")
        return value
```

## Organización de Rutas con APIRouter

Organiza rutas en módulos separados:

### Archivo `routes/data_routes.py`

```python
from fastapi import APIRouter

data_router = APIRouter()

@data_router.get("/data", tags=["Data"])
def get_data():
    return JSONResponse(data)
```

### Inclusión en `main.py`

```python
from .routes.data_routes import data_router

app.include_router(data_router)
```

Agrega un prefijo:

```python
app.include_router(prefix="/data", router=data_router)
```

O en el router:

```python
data_router = APIRouter(prefix="/data", tags=["Data"])
```

## Manejo de Errores

### BaseHTTPMiddleware

Para errores no controlados:

```python
from starlette.middleware.base import BaseHTTPMiddleware
from fastapi import status
from fastapi.responses import JSONResponse

class HTTPErrorHandler(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        try:
            return await call_next(request)
        except Exception as e:
            content = str(e)
            status_code = status.HTTP_500_INTERNAL_SERVER_ERROR
            return JSONResponse(content=content, status_code=status_code)

app.add_middleware(HTTPErrorHandler)
```

### Decorador `@app.middleware`

Otra forma de manejar errores:

```python
@app.middleware('http')
async def http_error_handler(request: Request, call_next):
    try:
        return await call_next(request)
    except Exception as e:
        content = str(e) + "--2"
        status_code = 500
        return JSONResponse(content=content, status_code=status_code)
```



## static

para exponer los archivos estaticos

/static/misacrhivosYcarpetas

```python
from fastapi.staticfiles import StaticFiles
app=FastAPI()
app.mount("/static",StaticFiles(directory="static"),name="static")
```

Luego puedes acceder al archivo desde

http://localhost:8000/static/example.PNG
