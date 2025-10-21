### Cómo conectar un bucket de Supabase para almacenar media en un proyecto Django

Supabase ofrece un almacenamiento de objetos compatible con S3, lo que facilita su integración con Django para manejar archivos de media (como imágenes o videos). La forma más sencilla y recomendada es usar **django-storages** configurándolo para que apunte al endpoint de Supabase Storage, ya que es S3-compatible. Esto permite que Django use automáticamente el bucket para subir y servir archivos de media sin código adicional en tus modelos o vistas.

A continuación, te detallo los pasos paso a paso. Asumo que ya tienes un proyecto Django configurado y una cuenta en Supabase con un proyecto creado.

#### 1. **Crea un bucket en Supabase**

- Ve al **Dashboard de Supabase** > **Storage**.
- Haz clic en **New Bucket**.
- Ingresa un nombre para el bucket (ej: `media-bucket`). Asegúrate de que sea público si necesitas servir archivos directamente (o configúralo privado con políticas de RLS para acceso autenticado).
- Haz clic en **Create Bucket**.
- Nota el **URL del proyecto** (ej: `https://tu-proyecto.supabase.co`) y el **anon key** (clave pública) desde **Settings > API**. Para uploads, usa el **service_role key** (clave de servicio, secreta) para permisos completos.

#### 2. **Instala las dependencias necesarias**

   En tu entorno virtual de Django, instala:

```
pip install django-storages boto3
```

- `django-storages`: Backend para storages remotos.

- `boto3`: Cliente AWS S3 (funciona con Supabase por compatibilidad).
  
  Actualiza tu `requirements.txt` si lo usas.

#### 3. **Configura las variables de entorno**

   Crea un archivo `.env` en la raíz de tu proyecto Django (usa `python-decouple` o `django-environ` para cargarlo si no lo tienes). Agrega:

```
SUPABASE_URL=https://tu-proyecto.supabase.co
SUPABASE_ANON_KEY=tu-clave-anon-publica
SUPABASE_SERVICE_ROLE_KEY=tu-clave-service-role-secreta
AWS_ACCESS_KEY_ID=tu-clave-service-role-secreta  # Usa la service_role como access key
AWS_SECRET_ACCESS_KEY=tu-clave-service-role-secreta  # Igual, misma clave
AWS_STORAGE_BUCKET_NAME=media-bucket  # Nombre de tu bucket
AWS_S3_REGION_NAME=eu-central-1  # Región de Supabase (puedes usar cualquier, pero eu-central-1 es común)
AWS_ENDPOINT_URL=https://tu-proyecto.supabase.co/storage/v1/object/  # Endpoint S3 de Supabase
AWS_S3_FILE_OVERWRITE=False  # Opcional: Evita sobrescribir archivos
DEFAULT_FILE_STORAGE=storages.backends.s3boto3.S3Boto3Storage
```

- **Importante**: La `service_role key` actúa como tanto access key como secret key en la config S3. No la expongas en producción.
- En `settings.py`, carga las vars con:
  
  ```python
  from decouple import config  # Si usas python-decouple
  # O usa os.environ directamente
  ```

#### 4. **Configura settings.py en Django**

   Agrega o actualiza en tu `settings.py`:

```python
# Storages
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
STATICFILES_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'  # Opcional, para static files también

# AWS S3 (para Supabase)
AWS_ACCESS_KEY_ID = config('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = config('AWS_SECRET_ACCESS_KEY')
AWS_STORAGE_BUCKET_NAME = config('AWS_STORAGE_BUCKET_NAME')
AWS_S3_REGION_NAME = config('AWS_S3_REGION_NAME')
AWS_ENDPOINT_URL = config('AWS_ENDPOINT_URL')
AWS_S3_FILE_OVERWRITE = False

# Media y static URLs (ajusta según tu setup)
MEDIA_URL = f'{config("SUPABASE_URL")}/storage/v1/object/public/{config("AWS_STORAGE_BUCKET_NAME")}/'
STATIC_URL = f'{config("SUPABASE_URL")}/storage/v1/object/public/{config("AWS_STORAGE_BUCKET_NAME")}/static/'
```

- Esto hace que `MEDIA_ROOT` no se use localmente; todo va al bucket.
- Para desarrollo local, puedes condicionar con `if not DEBUG:` para fallback a local storage.

#### 5. **Integra en tus modelos y vistas**

- En tus modelos (ej: un modelo con `ImageField`):
  
  ```python
  from django.db import models
  
  class MiModelo(models.Model):
      imagen = models.ImageField(upload_to='imagenes/')  # Subirá automáticamente al bucket
  ```

- En `urls.py` principal, agrega para servir media en desarrollo:
  
  ```python
  from django.conf import settings
  from django.conf.urls.static import static
  
  urlpatterns = [
      # Tus URLs
  ] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)  # Solo en DEBUG=True
  ```

- En producción (ej: Heroku, Vercel), no necesitas esto; Supabase sirve los archivos directamente vía URL pública.

#### 6. **Prueba la conexión**

- Corre migraciones: `python manage.py makemigrations && python manage.py migrate`.
- Crea un superusuario y accede al admin: `python manage.py createsuperuser`.
- Sube un archivo vía admin o un form. Verifica en el Dashboard de Supabase > Storage si aparece en el bucket.
- Para acceder al archivo: La URL será algo como `https://tu-proyecto.supabase.co/storage/v1/object/public/media-bucket/imagenes/miarchivo.jpg`.

#### Notas adicionales

- **Seguridad**: Si el bucket es público, cualquiera puede leer archivos. Para privado, usa firmas RLS o tokens JWT en Supabase. Integra `supabase-py` si necesitas auth avanzado.
- **Alternativa con paquete custom**: Si prefieres algo más nativo, usa `django-storage-supabase` (instala via `pip install git+https://github.com/J0/django-storage-supabase.git`), pero la opción S3 es más estable y documentada.
- **Problemas comunes**:
  - Error de bucket: Verifica el nombre exacto y que el bucket exista.
  - Permisos: Usa `service_role` para uploads; `anon` solo para lecturas.
  - En Heroku: Agrega las vars en el dashboard de Heroku.
- Recursos: Consulta la [doc de Supabase Storage](https://supabase.com/docs/guides/storage) y [django-storages](https://django-storages.readthedocs.io/en/latest/backends/amazon-S3.html) para más opciones.

Si tienes errores específicos, comparte detalles para ayudarte más. ¡Éxito con tu proyecto!
