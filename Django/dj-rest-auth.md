# Configuracion username - password



#### **Paso 1: Preparación del Entorno y Proyecto Django**

Primero, asegúrate de tener un entorno virtual activado. Luego, crea un nuevo proyecto de Django y una aplicación dentro de él.

1. **Crea el proyecto:**
   
   ```
   django-admin startproject mi_proyecto
   cd mi_proyecto
   ```

2. **Crea la aplicación:**
   
   ```
   python manage.py startapp core
   ```

#### **Paso 2: Instalación de Dependencias**

Ahora, instalaremos todas las librerías necesarias. `dj-rest-auth` depende de `djangorestframework` y `django-allauth`, así que las instalaremos todas juntas.

```
pip install django-allauth dj-rest-auth
```

#### **Paso 3: Configuración en `settings.py`**

Este es el paso más importante. Abre el archivo `mi_proyecto/settings.py` y realiza las siguientes modificaciones.

1. **Añade las aplicaciones a `INSTALLED_APPS`:** Asegúrate de que el orden sea el correcto, ya que algunas apps dependen de otras.
   
   ```python
   # mi_proyecto/settings.py
   INSTALLED_APPS = [  
     
   'django.contrib.admin',    
   'django.contrib.auth',    
   'django.contrib.contenttypes',    
   'django.contrib.sessions',    
   'django.contrib.messages',    
   'django.contrib.staticfiles',    
   
   # Aplicación local    
   'core',    
   
   # Apps de terceros    
   'rest_framework',    
   'rest_framework.authtoken',    
   'allauth',    
   'allauth.account',    
   'allauth.socialaccount',    
   'dj_rest_auth',    
   'dj_rest_auth.registration',
       
   # Necesario para dj-rest-auth    
   'django.contrib.sites',
   
   ]
   ```

2. **Configura `REST_FRAMEWORK`:** Le diremos a Django REST Framework que use la autenticación por Token.
   
   ```python
   REST_FRAMEWORK = {
       'DEFAULT_AUTHENTICATION_CLASSES': [
           'rest_framework.authentication.TokenAuthentication',
       ],
   }
   ```

3. **Añade `SITE_ID`:** `django.contrib.sites` requiere que esta variable esté definida.
   
   ```python
   # Al final de settings.py
   SITE_ID = 1
   ```

4. **Configuración de `django-allauth`:** Para este tutorial, desactivaremos la verificación por correo electrónico para simplificar el proceso de registro. En un proyecto real, es recomendable mantenerla activada.
   
   ```python
   # Desactiva la verificación de email para que el registro sea más rápido
   ACCOUNT_EMAIL_VERIFICATION = 'none'
   # Requiere que el email sea único para evitar registros duplicados
   ACCOUNT_UNIQUE_EMAIL = True
   # Permite el registro con solo un nombre de usuario y contraseña
   ACCOUNT_AUTHENTICATION_METHOD = 'username'
   # El email no es obligatorio para el registro
   ACCOUNT_EMAIL_REQUIRED = False
   ```

#### **Paso 4: Configuración de las URLs**

Ahora necesitamos exponer los endpoints de la API. Modifica el archivo `mi_proyecto/urls.py` para incluir las URLs de `dj_rest_auth`.

generalmente se pone:

```python
    path('dj-rest-auth/', include('dj_rest_auth.urls')),
    path('dj-rest-auth/registration/', include('dj_rest_auth.registration.urls'))
```

pero para este caso:

```python
    path('api/auth/', include('dj_rest_auth.urls')),
    path('api/auth/registration/', include('dj_rest_auth.registration.urls'))
```

Con esta configuración, `dj-rest-auth` creará automáticamente los siguientes endpoints:

- `/api/auth/login/` (POST): Para iniciar sesión.

- `/api/auth/logout/` (POST): Para cerrar sesión.

- `/api/auth/user/` (GET): Para obtener los detalles del usuario.

- `/api/auth/password/change/` (POST): Para cambiar la contraseña.

- `/api/auth/registration/` (POST): Para registrar un nuevo usuario.

#### **Paso 5: Aplicar las Migraciones**

Las nuevas aplicaciones que hemos añadido (`allauth`, `authtoken`, etc.) necesitan crear sus propias tablas en la base de datos.

```
python manage.py makemigrations
python manage.py migrate
```

#### **Paso 6: ¡A Probar la API!**

¡Tu sistema de autenticación ya está listo! Ahora puedes probarlo usando una herramienta como **Postman**, **Insomnia** o `curl` desde la terminal.

Primero, inicia el servidor de desarrollo:

```
python manage.py runserver
```

**1. Registrar un nuevo usuario**

- **Método:** `POST`

- **URL:** `http://127.0.0.1:8000/api/auth/registration/`

- **Body (JSON):**
  
  ```
  {
      "username": "nuevo_usuario",
      "password": "una_clave_segura_123",
      "password2": "una_clave_segura_123"
  }
  ```

- **Respuesta Exitosa (HTTP 201 Created):** Recibirás un token de autenticación. ¡Guárdalo!
  
  ```
  {
      "key": "tu_token_de_autenticacion_aqui"
  }
  ```

**2. Iniciar Sesión (Login)**

- **Método:** `POST`

- **URL:** `http://127.0.0.1:8000/api/auth/login/`

- **Body (JSON):**
  
  ```
  {
      "username": "nuevo_usuario",
      "password": "una_clave_segura_123"
  }
  ```

- **Respuesta Exitosa (HTTP 200 OK):** Obtendrás un nuevo token.
  
  ```
  {
      "key": "otro_token_de_autenticacion"
  }
  ```

**3. Acceder a Datos del Usuario (Ruta Protegida)**

- **Método:** `GET`

- **URL:** `http://127.0.0.1:8000/api/auth/user/`

- **Headers:** Para acceder a esta ruta, debes incluir el token en los encabezados de la petición.
  
  - **Key:** `Authorization`
  
  - **Value:** `Token tu_token_de_autenticacion_aqui`

- **Respuesta Exitosa (HTTP 200 OK):**
  
  ```
  {
      "pk": 1,
      "username": "nuevo_usuario",
      "email": ""
  }
  
  ```

**4. Cerrar Sesión (Logout)**

- **Método:** `POST`

- **URL:** `http://127.0.0.1:8000/api/auth/logout/`

- **Headers:** También requiere el token de autorización.
  
  - **Key:** `Authorization`
  
  - **Value:** `Token tu_token_de_autenticacion_aqui`

- **Respuesta Exitosa (HTTP 200 OK):**
  
  ```
  {    "detail": "Successfully logged out."}
  ```
  
  Después de esto, el token ya no será válido.
