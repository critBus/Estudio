Para conectar tu proyecto Django con un bucket de Supabase y almacenar archivos multimedia (como imágenes, videos, etc.), necesitas utilizar el **Storage API** de Supabase. Supabase no tiene un sistema de archivos local como los backends de almacenamiento tradicionales de Django (por ejemplo, `FileSystemStorage`), por lo que deberás usar un **backend personalizado** o interactuar directamente con la API de Supabase Storage.

A continuación, te explico paso a paso cómo hacerlo:

---

### ✅ 1. **Configuración previa en Supabase**

1. Ve a tu proyecto en [Supabase Dashboard](https://app.supabase.com/).
2. Ve a **Storage** > **Buckets**.
3. Crea un nuevo bucket (por ejemplo: `media`) o usa uno existente.
4. Asegúrate de que el bucket tenga los permisos adecuados. Por ejemplo, para permitir subidas desde el backend (tu servidor Django), puedes configurar una política como:

```sql
-- Permite a los usuarios autenticados subir archivos
CREATE POLICY "Allow upload for authenticated users" ON storage.objects
FOR INSERT TO authenticated
WITH CHECK (bucket_id = 'media');

-- O si quieres permitir acceso público (solo para desarrollo o casos específicos)
CREATE POLICY "Public Access" ON storage.objects
FOR SELECT USING (bucket_id = 'media');
```

> ⚠️ **Importante**: No expongas tu `anon key` o `service_role key` en el frontend si no es necesario. Para operaciones desde el backend (Django), usa la **`service_role` key** (más permisos), pero **nunca la expongas en el cliente**.

---

### ✅ 2. **Instalar dependencias en tu proyecto Django**

```bash
pip install supabase
```

Esto instalará el SDK oficial de Supabase para Python.

---

### ✅ 3. **Configurar credenciales en Django**

En tu archivo `settings.py`:

```python
# settings.py

SUPABASE_URL = 'https://tu-proyecto.supabase.co'
SUPABASE_KEY = 'tu-service_role-key-o-anon-key'  # Usa service_role para operaciones de escritura
SUPABASE_BUCKET = 'media'  # Nombre de tu bucket
```

> 🔐 **Recomendación**: Usa variables de entorno para las claves:

```python
import os
SUPABASE_URL = os.getenv('SUPABASE_URL')
SUPABASE_KEY = os.getenv('SUPABASE_KEY')
SUPABASE_BUCKET = os.getenv('SUPABASE_BUCKET', 'media')
```

---

### ✅ 4. **Crear un backend de almacenamiento personalizado (opcional pero recomendado)**

Puedes crear un backend personalizado para que Django use Supabase Storage como si fuera un sistema de archivos.

Crea un archivo: `storage_backends.py` (por ejemplo, en tu app `core` o en la raíz del proyecto):

```python
# storage_backends.py
from django.core.files.storage import Storage
from django.core.files.base import ContentFile
from supabase import create_client, Client
from django.conf import settings
import os

class SupabaseStorage(Storage):
    def __init__(self):
        self.supabase_url = settings.SUPABASE_URL
        self.supabase_key = settings.SUPABASE_KEY
        self.bucket_name = settings.SUPABASE_BUCKET
        self.supabase: Client = create_client(self.supabase_url, self.supabase_key)

    def _open(self, name, mode='rb'):
        # Opcional: implementar descarga
        response = self.supabase.storage.from_(self.bucket_name).download(name)
        return ContentFile(response)

    def _save(self, name, content):
        # Subir archivo
        file_data = content.read()
        self.supabase.storage.from_(self.bucket_name).upload(
            path=name,
            file=file_data,
            file_options={"content-type": content.content_type}
        )
        return name

    def url(self, name):
        # Generar URL pública (asegúrate de que el bucket sea público o usa signed URLs)
        return f"{self.supabase_url}/storage/v1/object/public/{self.bucket_name}/{name}"

    def exists(self, name):
        try:
            self.supabase.storage.from_(self.bucket_name).download(name)
            return True
        except:
            return False

    def delete(self, name):
        self.supabase.storage.from_(self.bucket_name).remove([name])
```

> ⚠️ **Nota**: El método `url()` asume que el bucket tiene acceso público. Si no, necesitarás generar URLs firmadas o usar autenticación.

---

### ✅ 5. **Usar el backend en tus modelos**

En tu modelo de Django:

```python
# models.py
from django.db import models
from .storage_backends import SupabaseStorage

class MiModelo(models.Model):
    imagen = models.ImageField(upload_to='imagenes/', storage=SupabaseStorage())
```

O si quieres usarlo globalmente para todos los archivos multimedia, en `settings.py`:

```python
DEFAULT_FILE_STORAGE = 'myproject.storage_backends.SupabaseStorage'
```

(Asegúrate de ajustar la ruta al archivo `storage_backends.py`)

---

### ✅ 6. **Alternativa: Subir manualmente sin backend personalizado**

Si no quieres crear un backend, puedes subir archivos directamente en tus vistas:

```python
# views.py
from supabase import create_client
from django.conf import settings
from django.core.files.storage import default_storage

def subir_archivo(request):
    if request.method == 'POST' and request.FILES:
        archivo = request.FILES['archivo']
        nombre = archivo.name
        contenido = archivo.read()

        supabase = create_client(settings.SUPABASE_URL, settings.SUPABASE_KEY)
        supabase.storage.from_(settings.SUPABASE_BUCKET).upload(
            path=f"uploads/{nombre}",
            file=contenido,
            file_options={"content-type": archivo.content_type}
        )
        # Guardar la ruta en tu modelo si es necesario
        url = f"{settings.SUPABASE_URL}/storage/v1/object/public/{settings.SUPABASE_BUCKET}/uploads/{nombre}"
        return JsonResponse({'url': url})
```

---

### 🛡️ Seguridad

- **Nunca uses la `service_role` key en el frontend**.
- Si tu bucket no es público, necesitarás autenticar las descargas (por ejemplo, usando `create_signed_url`).
- Considera usar RLS (Row Level Security) en Supabase Storage para controlar quién puede subir/ver archivos.

---

### 📌 Resumen

- Usa el SDK de Supabase (`supabase-py`).
- Crea un backend personalizado o sube archivos manualmente.
- Configura políticas de seguridad en tu bucket.
- Usa variables de entorno para las claves.

¿Quieres que te dé un ejemplo completo con un modelo y una vista funcional?
