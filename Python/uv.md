### crear la carpeta del proyecto

```bash
uv init <nombreproyecto>
cd nombreproyecto
```

```
uv init
```

### Agregar Django y otros

dentro de la carpeta del proyecto

esto crea el entorno de forma automatica

```bash
uv add django
```

### Crear el proyecto de django

dentro de la carpeta del proyecto

```bash
uv run django-admin startproject config .
```

### comandos comunes

```bash
uv run manage.py migrate
uv run manage.py runserver
uv run manage.py createsuperuser
```

--------------

# Respuesta de IA

Aquí tienes una guía paso a paso para crear proyectos de Django REST Framework (DRF) usando `uv` como gestor de paquetes en Windows. Los comandos están optimizados para ser reutilizados en futuros proyectos.

---

### **1. Estructura básica del proyecto**

```bash
# Crea una carpeta para tu proyecto y entra en ella
mkdir mi_proyecto_drf
cd mi_proyecto_drf

# Inicializa un entorno virtual con uv (crea una carpeta .venv)
uv venv

# Activa el entorno virtual (en Windows)
.venv\Scripts\activate
```

---

### **2. Instalación de Django y DRF**

```bash
# Instala Django y Django REST Framework
uv pip install django djangorestframework

# (Opcional) Instala paquetes adicionales (ej: psycopg2 para PostgreSQL)
uv pip install psycopg2-binary
```

---

### **3. Crear el proyecto de Django**

```bash
# Genera el proyecto de Django
django-admin startproject config .

# Crea una aplicación Django (ej: "api")
uv run python manage.py startapp api
```

---

### **4. Configuración inicial**

- **En `config/settings.py`:**
  
  ```python
  # Agrega 'rest_framework' a INSTALLED_APPS
  INSTALLED_APPS = [
      ...
      'rest_framework',
      'api',  # Tu aplicación
  ]
  ```

---

### **5. Migraciones y servidor**

```bash
# Aplica las migraciones iniciales
uv run python manage.py migrate

# Crea un superusuario (opcional)
uv run python manage.py createsuperuser

# Inicia el servidor de desarrollo
uv run python manage.py runserver
```

---

### **6. Gestión de dependencias**

- **Genera un archivo de dependencias bloqueadas (lockfile):**
  
  ```bash
  uv lock
  ```
  
  Esto crea un archivo `uv.lock` con versiones exactas de todas las dependencias.

- **Instala dependencias desde un lockfile existente:**
  
  ```bash
  uv sync
  ```

- **Agrega una nueva dependencia:**
  
  ```bash
  uv pip install nombre_del_paquete
  ```

---

### **7. Comandos útiles de `uv`**

- **Verifica versiones instaladas:**
  
  ```bash
  uv pip list
  ```

- **Actualiza una dependencia:**
  
  ```bash
  uv pip install nombre_del_paquete --upgrade
  ```

- **Desinstala una dependencia:**
  
  ```bash
  uv pip uninstall nombre_del_paquete
  ```

- **Verifica el entorno virtual actual:**
  
  ```bash
  uv venv --help
  ```

---

### **8. Ejecutar comandos sin activar el entorno**

`uv run` permite ejecutar comandos directamente en el entorno virtual:

```bash
uv run python manage.py makemigrations
uv run python manage.py test
```

---

### **9. Archivos clave a incluir en `.gitignore`**

Crea un archivo `.gitignore` con:

```
.venv/
__pycache__
*.pyc
uv.lock
db.sqlite3
```

---

### **10. Flujo de trabajo recomendado**

1. **Iniciar un nuevo proyecto:**
   
   ```bash
   mkdir proyecto && cd proyecto
   uv venv && .venv\Scripts\activate
   uv pip install django djangorestframework
   django-admin startproject config .
   uv run python manage.py startapp api
   ```

2. **Trabajar en el proyecto:**
   
   - Activar el entorno: `.venv\Scripts\activate`
   - Ejecutar comandos con `uv run` o directamente en la terminal activada.

3. **Publicar dependencias:**
   
   ```bash
   uv lock  # Genera uv.lock
   ```

---

### **Notas importantes**

- `uv` es compatible con `pyproject.toml`, pero para proyectos simples, los comandos de `uv pip install` son suficientes.
- En Windows, asegúrate de que Python esté en el `PATH` (instálalo desde [python.org](https://www.python.org/downloads/)).
- Usa `uv --version` para verificar la instalación.

¡Listo! Con estos comandos tendrás un flujo de trabajo eficiente y reproducible para proyectos de Django REST Framework. 🚀
