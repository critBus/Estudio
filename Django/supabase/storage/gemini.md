Claro, para conectar un bucket de Supabase y usarlo como almacenamiento para tus archivos multimedia en Django, debes utilizar la librería **`django-storages`**. La razón es que el almacenamiento de Supabase es compatible con la API de S3 de Amazon Web Services (AWS), y `django-storages` te permite "engañar" a Django para que, en lugar de guardar los archivos en tu servidor, los envíe a un servicio compatible con S3 como Supabase.

Aquí tienes el proceso completo, paso a paso.

---

### Paso 0: Prerrequisitos en Supabase

Antes de tocar Django, asegúrate de tener lo siguiente en tu panel de Supabase:

1. **Un Proyecto Creado:** Obviamente, necesitas un proyecto en Supabase.

2. **Un Bucket Creado:**
   
   - Ve a la sección "Storage" en el menú de la izquierda.
   
   - Crea un nuevo bucket. Dale un nombre simple, por ejemplo, `media`.
   
   - **Importante:** Haz clic en el bucket que creaste, ve a "Bucket settings" y en "Permissions", asegúrate de que el bucket sea **público** para que los archivos se puedan ver desde tu web. Puedes crear políticas más específicas, pero para empezar, una política pública de lectura es lo más sencillo.

---

### Paso 1: Instalar las Librerías Necesarias

En tu entorno virtual de Django, necesitas instalar dos paquetes: `django-storages` para manejar la lógica de almacenamiento y `boto3`, que es el SDK que `django-storages` usa por debajo para comunicarse con servicios tipo S3.

Bash

```
pip install django-storages boto3
```

---

### Paso 2: Obtener tus Credenciales de Supabase 🔑

Necesitarás cuatro piezas de información de tu proyecto de Supabase. Encuéntralas en **Project Settings > API**:

1. **URL del Proyecto (Endpoint):** Es la URL principal de tu API. La usaremos para construir el endpoint de almacenamiento. Luce así: `https://xxxxxxxxxxxxxx.supabase.co`.

2. **Clave anónima (`anon` key):** La usaremos como el "Access Key ID".

3. **Clave de servicio (`service_role` key):** **¡Usa esta!** La usaremos como el "Secret Access Key", ya que tiene los permisos necesarios para subir y borrar archivos.

4. **Nombre de tu Bucket:** El que creaste en el paso 0 (ej. `media`).

---

### Paso 3: Configurar `settings.py` ⚙️

Esta es la parte más importante. Abre el archivo `settings.py` de tu proyecto de Django y añade o modifica lo siguiente.

#### 1. Añade `storages` a tus `INSTALLED_APPS`

Asegúrate de que `storages` esté en la lista de aplicaciones instaladas.

Python

```
# settings.py

INSTALLED_APPS = [
    # ... otras apps
    'storages',
]
```

#### 2. Configura el Almacenamiento por Defecto

Esta línea le dice a Django que deje de usar el sistema de archivos local y utilice el backend de S3 de `django-storages` para todos los campos `FileField` e `ImageField`.

Python

```
# settings.py

# Indica a Django que use S3Boto3Storage para los archivos por defecto
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
```

#### 3. Añade las Credenciales de Supabase (estilo S3)

`django-storages` espera variables con el prefijo `AWS_`. Simplemente las "mapeamos" a nuestras credenciales de Supabase.

**¡Recomendación!** No escribas las claves directamente en `settings.py`. Usa variables de entorno para la seguridad.

Python

```
# settings.py

# Credenciales y configuración para Supabase Storage (usando nombres de AWS)
AWS_ACCESS_KEY_ID = 'TU_CLAVE_ANON_DE_SUPABASE'  # Pega tu clave anon aquí
AWS_SECRET_ACCESS_KEY = 'TU_CLAVE_SERVICE_ROLE_DE_SUPABASE' # Pega tu clave service_role aquí
AWS_STORAGE_BUCKET_NAME = 'media'  # El nombre que le diste a tu bucket

# Esta es la URL del endpoint de almacenamiento de tu proyecto Supabase.
# Formato: https://<tu-ref-de-proyecto>.supabase.co/storage/v1
AWS_S3_ENDPOINT_URL = 'https://xxxxxxxxxxxxxx.supabase.co/storage/v1'

# Con esta configuración, los archivos no se sobreescriben si ya existen
AWS_S3_FILE_OVERWRITE = False
AWS_DEFAULT_ACL = None # No es necesario para Supabase

# Parámetros para que los archivos se sirvan con las cabeceras correctas en el navegador
AWS_S3_OBJECT_PARAMETERS = {
    'CacheControl': 'max-age=86400', # Cache de 1 día
}

# URL desde la que se servirán los archivos multimedia.
# Es la URL pública de tu bucket.
# Formato: https://<tu-ref-de-proyecto>.supabase.co/storage/v1/object/public/<tu-bucket>/
MEDIA_URL = 'https://xxxxxxxxxxxxxx.supabase.co/storage/v1/object/public/media/'

# MEDIA_ROOT no es necesario cuando usas un almacenamiento remoto, 
# pero es buena práctica mantenerlo definido.
MEDIA_ROOT = ''
```

**Reemplaza:**

- `TU_CLAVE_ANON_DE_SUPABASE` con tu `anon` key.

- `TU_CLAVE_SERVICE_ROLE_DE_SUPABASE` con tu `service_role` key.

- `xxxxxxxxxxxxxx` con la referencia única de tu proyecto Supabase en las dos URLs.

- `media` con el nombre de tu bucket si usaste uno diferente.

---

### Paso 4: ¡Úsalo en tus Modelos! ✅

¡Ya está todo configurado! Ahora, cuando uses un `ImageField` o `FileField` en tus modelos, Django automáticamente subirá el archivo a tu bucket de Supabase.

Python

```
# en tu models.py

from django.db import models

class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    # Al subir una imagen, se guardará en la carpeta 'productos/' dentro de tu bucket 'media'
    imagen_producto = models.ImageField(upload_to='productos/')

    def __str__(self):
        return self.nombre
```

Cuando un usuario suba una imagen a través del admin de Django o un formulario, sucederá lo siguiente:

1. Django interceptará la subida.

2. Usará `django-storages` y `boto3`.

3. Se conectará a tu endpoint de Supabase con tus credenciales.

4. Subirá el archivo a la carpeta `productos/` dentro de tu bucket `media`.

5. En la base de datos de Django, en el campo `imagen_producto`, guardará la ruta relativa: `productos/nombre_del_archivo.jpg`.

6. Cuando pidas la URL del archivo en una plantilla (ej. `{{ producto.imagen_producto.url }}`), Django usará la `MEDIA_URL` que definiste para construir la URL completa: `https://..../media/productos/nombre_del_archivo.jpg`.

¡Y listo! Tu proyecto Django ya está utilizando Supabase para almacenar y servir tus archivos multimedia. 🚀
