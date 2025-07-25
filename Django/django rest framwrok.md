# Label en Serializable

En Django REST Framework (DRF), la propiedad **`label`** en los campos de un `Serializer` se utiliza para **definir un nombre descriptivo y legible por humanos** para el campo. Este valor aparece en:

### 1. **Interfaz web navegable de DRF**

- En la interfaz web que DRF genera automáticamente para probar la API (por ejemplo, en endpoints como `/api/endpoint/`), el `label` se muestra como el **nombre del campo** en formularios y tablas.  

- Ejemplo:  
  
  ```python
  name = serializers.CharField(label=_("Nombre"), read_only=True)
  ```
  
  Aquí, en la interfaz web, el campo se mostrará como **"Nombre"** en lugar de `"name"` (el nombre del campo en el código).

---

### 2. **Mensajes de error**

- Si hay un error de validación, el mensaje usa el `label` para hacerlo más claro.  

- Ejemplo:  
  Si el campo `name` es requerido y no se proporciona, el error podría decir:  
  
  ```json
  {
    "name": ["Este campo es obligatorio."]
  }
  ```
  
  Pero si usas un `label`, el mensaje podría mostrarse como:  
  
  ```json
  {
    "name": ["El campo 'Nombre' es obligatorio."]
  }
  ```

---

### 3. **Documentación automática**

- Herramientas como **Swagger/OpenAPI** (usadas con `drf-yasg` o `drf-spectacular`) pueden usar el `label` para describir el campo en la documentación de la API.

---

### 4. **Internacionalización (i18n)**

- Es común usar `gettext_lazy` (`_`) con el `label` para soportar traducciones:  
  
  ```python
  from django.utils.translation import gettext_lazy as _
  
  name = serializers.CharField(label=_("Nombre"))
  ```
  
  Esto permite que el texto `"Nombre"` se traduzca automáticamente según el idioma del usuario.

---

### ¿Es obligatorio usar `label`?

No, pero es una **buena práctica** por:

- Mejorar la usabilidad de la interfaz web de DRF.
- Facilitar la comprensión de errores para usuarios finales o desarrolladores.
- Soportar internacionalización.

---

### Ejemplo completo:

```python
class UserSerializer(serializers.Serializer):
    name = serializers.CharField(
        label=_("Nombre de usuario"),  # Etiqueta legible
        help_text=_("Ingresa tu nombre completo"),  # Texto de ayuda
        read_only=True
    )
```

En la interfaz web de DRF, verías algo como:  

```
Nombre de usuario: [valor] (solo lectura)
Ayuda: Ingresa tu nombre completo
```

-------

# Tipado campo UUID

El campo `models.UUIDField` en Django **almacena internamente un objeto `UUID` de Python**, pero cuando se utiliza para filtrar en consultas (por ejemplo, con `.filter(id=valor)`), **puedes usar tanto un objeto `UUID` como una cadena (`str`) que represente el UUID** (en formato estándar como `"123e4567-e89b-12d3-a456-426614174000"`).

### ¿Qué tipo documentar?

Para tipar el parámetro de tu función en la documentación:

- **Recomendado**: Usa `str` si el valor proviene de una entrada externa (como una API o formulario), ya que los UUID suelen pasarse como cadenas en esos casos.
- **Alternativa**: Usa `uuid.UUID` si trabajas internamente con objetos `UUID` generados por Python.

Ejemplo:

```python
def mi_funcion(uuid_valor: str) -> None:
    # Aquí usas uuid_valor para filtrar el modelo
    MiModelo.objects.filter(id=uuid_valor)
```

### ¿Por qué funciona con `str`?

Django automáticamente convierte cadenas en objetos `UUID` cuando filtras con el ORM. Por ejemplo:

```python
MiModelo.objects.filter(id="123e4567-e89b-12d3-a456-426614174000")
```

es válido incluso si el campo `id` es de tipo `UUIDField`.

### Resumen

- **Tipo esperado**: `str` (para casos prácticos y documentación).
- **Tipado estricto**: Si usas `from typing import Union` y quieres ser explícito, puedes usar `Union[str, uuid.UUID]`.

-------------

# dynamic_db_router

## @in_database

El decorador `@in_database("default")` proviene del paquete **`dynamic_db_router`**, que se usa para **gestionar rutas dinámicas de bases de datos en Django** (por ejemplo, en entornos de múltiples bases de datos o arquitecturas de tipo *multi-tenant*). Este decorador asegura que una función o método se ejecute **usando una conexión específica a una base de datos** definida por el alias que le pasas (en este caso, `"default"`).

---

### ¿Qué hace `@in_database("default")`?

Cuando aplicas este decorador a un método (como `create()` en un serializador), **forza a Django a usar la base de datos con el alias `"default"`** para todas las operaciones ORM dentro de ese método, incluso si hay otros routers de base de datos configurados o si el contexto actual usaría otra base de datos por defecto.

#### Ejemplo:

```python
from dynamic_db_router import in_database

class MiSerializer(serializers.ModelSerializer):
    @in_database("default")
    def create(self, validated_data):
        return super().create(validated_data)
```

Aquí, aunque el sistema podría estar usando otro router o configuración para seleccionar la base de datos, el método `create()` **siempre usará la base de datos `"default"`**.

---

### ¿Por qué es útil?

Este decorador es clave en escenarios donde necesitas **control explícito sobre la base de datos** utilizada, como:

1. **Entornos multi-tenant**  
   Cada *tenant* (cliente) tiene su propia base de datos. Puedes usar `@in_database("tenant_123")` para asegurar que las operaciones se realicen en la base de datos correcta.

2. **Operaciones críticas**  
   Algunas acciones (como auditorías o logs) deben escribirse en una base de datos compartida, aunque el resto de la aplicación use bases de datos específicas.

3. **Tareas asíncronas o background jobs**  
   En contextos asíncronos (ej: Celery), la conexión a la base de datos no siempre está bien configurada. Este decorador garantiza que uses la conexión correcta.

---

### ¿Cómo funciona internamente?

`dynamic_db_router` modifica temporalmente el **router de bases de datos de Django** para forzar el uso del alias especificado durante la ejecución del método decorado. Esto afecta a todas las consultas ORM dentro de ese método, incluso si usas `Model.objects.create()` sin `.using("default")`.

---

### Ejemplo práctico en un serializador

Imagina un sistema multi-tenant donde cada cliente tiene su propia base de datos:

```python
from dynamic_db_router import in_database

class TenantAwareSerializer(serializers.ModelSerializer):
    @in_database("tenant_123")  # Fuerza el uso de la base de datos del tenant
    def create(self, validated_data):
        return super().create(validated_data)
```

Sin este decorador, el router por defecto de Django podría seleccionar una base de datos incorrecta, causando errores de escritura/lectura.

---

### Comparación con `.using("default")`

Puedes lograr algo similar con el método nativo de Django:

```python
def create(self, validated_data):
    return MiModelo.objects.using("default").create(**validated_data)
```

Pero `@in_database("default")` es más poderoso en casos donde:

- Hay múltiples consultas dentro del método.
- Usas relaciones o métodos que no son explícitos (ej: `save()`, `bulk_create`, etc.).
- Quieres evitar repetir `.using("default")` en cada consulta.

---

### Consideraciones importantes

1. **Alias de base de datos válido**  
   El alias que pasas (ej: `"default"`) debe estar definido en `DATABASES` en `settings.py`.

2. **Transacciones**  
   Si usas transacciones (`@transaction.atomic`), asegúrate de combinarlo con `@in_database` para evitar inconsistencias.

3. **Contexto asíncrono**  
   En vistas o tareas asíncronas, asegúrate de usar `sync_to_async` si el decorador no es compatible con async.

4. **Routers personalizados**  
   Si usas otros routers de bases de datos (ej: para multi-tenant), `@in_database` sobreescribe temporalmente su lógica.

---

### Resumen

| Característica              | `@in_database("default")` | `.using("default")`             |
| --------------------------- | ------------------------- | ------------------------------- |
| **Control explícito**       | ✅ Sí                      | ✅ Sí                            |
| **Aplica a todo el método** | ✅ Sí                      | ❌ Solo a la consulta específica |
| **Multi-tenant**            | ✅ Ideal                   | ❌ Requiere repetir código       |
| **Código limpio**           | ✅ Menos verboso           | ❌ Más código repetido           |

---

### Conclusión

Usa `@in_database("default")` cuando necesites **garantizar que un método o función use una base de datos específica**, especialmente en sistemas complejos como multi-tenant o cuando las rutas de bases de datos son dinámicas. Es una herramienta poderosa de `dynamic_db_router` para evitar errores de conexión o escritura en bases de datos incorrectas.

--------

# api_settings.URL_FIELD_NAME

El `api_settings.URL_FIELD_NAME` en Django REST Framework (DRF) es una **configuración global** que define el nombre del campo utilizado para representar las **URLs de los recursos en serializadores hipermediales** (como `HyperlinkedModelSerializer`). Su valor predeterminado es `"url"`.

---

### ¿Para qué se usa?

Se utiliza para acceder al campo que contiene la **URL de un recurso** en serializaciones que usan hiperenlaces (no IDs). Esto permite escribir código que sea **independiente del nombre específico del campo URL**, incluso si se cambia en la configuración global.

---

### Ejemplo de uso:

```python
from rest_framework.settings import api_settings

def get_location_header(data):
    return {"Location": str(data[api_settings.URL_FIELD_NAME])}
```

Aquí, `data[api_settings.URL_FIELD_NAME]` accede al campo que contiene la URL del recurso serializado. Si el campo se llama `"url"` (por defecto), esto equivale a `data["url"]`.

---

### Contexto: Serializadores hipermediales

Cuando usas `HyperlinkedModelSerializer`, DRF incluye automáticamente un campo con la URL del recurso. Por ejemplo:

```python
class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ['url', 'username', 'email']  # 'url' es el campo por defecto
```

La salida podría ser:

```json
{
    "url": "http://api.example.com/users/1/",
    "username": "john_doe",
    "email": "john@example.com"
}
```

---

### ¿Por qué usar `api_settings.URL_FIELD_NAME` en lugar de `"url"`?

1. **Flexibilidad**: Si cambias el nombre del campo URL en tus ajustes globales, tu código seguirá funcionando sin modificaciones.
   
   ```python
   # En settings.py
   REST_FRAMEWORK = {
       'URL_FIELD_NAME': 'resource_url',  # Cambio personalizado
   }
   ```
   
   Ahora, `api_settings.URL_FIELD_NAME` devolverá `"resource_url"` en lugar de `"url"`.

2. **Reutilización**: Evita hardcodear `"url"` en múltiples lugares, lo que facilita mantener el código si cambia la configuración.

---

### Caso práctico: Respuestas HTTP con encabezado `Location`

Es común usar este campo para construir respuestas `201 Created` que incluyan el encabezado `Location` apuntando al recurso creado:

```python
from rest_framework.response import Response
from rest_framework import status
from rest_framework.settings import api_settings

def create(self, request):
    serializer = self.get_serializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    instance = serializer.save()

    # Obtener la URL del recurso creado
    data = serializer.data
    headers = {"Location": str(data[api_settings.URL_FIELD_NAME])}

    return Response(data, status=status.HTTP_201_CREATED, headers=headers)
```

Esto garantiza que el encabezado `Location` siempre use el campo correcto, incluso si se cambia su nombre en `settings.py`.

---

### Resumen:

- **`api_settings.URL_FIELD_NAME`** es una constante de DRF que define el nombre del campo URL en serializadores hipermediales.
- **Por defecto**: `"url"`.
- **Uso**: Acceder al campo URL de forma dinámica para evitar dependencias hardcodeadas.
- **Ventaja**: Facilita cambios globales en la configuración y mejora la mantenibilidad del código.

Si estás usando `HyperlinkedModelSerializer` o necesitas referencias hipermediales, este patrón es clave para seguir buenas prácticas en DRF.

# drf-yasg

## Vistas falsas

La línea `if getattr(self, "swagger_fake_view", False):` en un `ViewSet` de Django REST Framework (DRF) se utiliza para **detectar si la ejecución actual es parte de una vista falsa generada por `drf-yasg`** (una herramienta para generar documentación automática de APIs). Esta comprobación es útil para evitar operaciones costosas o innecesarias (como consultas a la base de datos) cuando se está generando la documentación de la API.

---

### ¿Qué es `swagger_fake_view`?

- Es un atributo **inyectado por `drf-yasg`** durante el proceso de generación de la documentación (Swagger/OpenAPI).
- Indica que el código está siendo ejecutado en un contexto "falso" para inspeccionar la estructura de la API, **no en una solicitud real**.
- Su propósito es evitar que el código real (como consultas a la base de datos) se ejecute durante la generación de la documentación.

---

### Ejemplo de uso en `get_queryset`

```python
def get_queryset(self):
    if getattr(self, "swagger_fake_view", False):
        return Affiliate.objects.none()
    # Lógica normal para devolver queryset
```

#### ¿Por qué hacer esto?

1. **Evitar consultas a la base de datos**:  
   `drf-yasg` necesita inspeccionar la estructura de los datos (por ejemplo, campos de un serializador), pero no necesita datos reales. Usar `objects.none()` evita ejecutar consultas innecesarias.

2. **Prevenir errores**:  
   Algunas lógicas en `get_queryset` pueden depender de datos de la solicitud (`request`) que no existen en el contexto de documentación. Devolver un queryset vacío evita errores.

3. **Mejor rendimiento**:  
   Generar la documentación es más rápido si no hay que esperar a que se ejecuten consultas complejas.

---

### ¿Cómo funciona internamente?

- Cuando `drf-yasg` genera la documentación, simula llamadas a las vistas para extraer información de los serializadores, URLs, métodos HTTP, etc.
- Durante esta simulación, inyecta el atributo `swagger_fake_view=True` en el `ViewSet` o vista para indicar que no se trata de una solicitud real.
- Tu código puede usar esta comprobación para personalizar el comportamiento en este contexto.

---

### Alternativas y contexto actual

- **`drf-yasg` está obsoleto**:  
  Se recomienda migrar a [`drf-spectacular`](https://github.com/tfranzel/drf-spectacular), que es más moderno, compatible con OpenAPI 3.0 y no requiere este tipo de comprobaciones manuales. En `drf-spectacular`, la generación de la documentación es más limpia y no depende de atributos como `swagger_fake_view`.

- **Uso en otros escenarios**:  
  Esta técnica también es útil en validaciones de serializadores, permisos o vistas personalizadas donde necesitas evitar operaciones costosas durante la generación de la documentación.

---

### Ejemplo completo

```python
class AffiliateViewSet(viewsets.ModelViewSet):
    serializer_class = AffiliateSerializer

    def get_queryset(self):
        if getattr(self, "swagger_fake_view", False):
            return Affiliate.objects.none()  # Evita consultas reales
        return Affiliate.objects.filter(owner=self.request.user)
```

Sin esta comprobación, al generar la documentación, el método `get_queryset` intentaría acceder a `self.request.user`, lo cual fallaría porque `request` no existe en el contexto de `drf-yasg`.

---

### Resumen

- **`swagger_fake_view`** es un atributo usado por `drf-yasg` para indicar que el código se ejecuta en un contexto de documentación, no en una solicitud real.
- **Uso principal**: Evitar operaciones costosas o errores durante la generación de la documentación (como consultas a la base de datos o dependencias de `request`).
- **Recomendación**: Si usas `drf-yasg`, esta comprobación es útil. Si migras a `drf-spectacular`, ya no será necesaria.

-----

# values_list("id", flat=True)

En Django, el método `.values_list("id", flat=True)` se utiliza para **obtener una lista plana de valores de un campo específico (en este caso, `"id"`) desde la base de datos**, en lugar de objetos completos o tuplas. Aquí te explico su propósito y uso:

---

### 1. **¿Qué hace exactamente?**

- **`values_list("id")`**: Devuelve una lista de tuplas con los valores del campo `"id"`.  
  Ejemplo:  
  
  ```python
  MyModel.objects.values_list("id")
  # Resultado: [(1,), (2,), (3,)]
  ```

- **`values_list("id", flat=True)`**: Devuelve una lista plana (sin tuplas) con los valores del campo `"id"`.  
  Ejemplo:  
  
  ```python
  MyModel.objects.values_list("id", flat=True)
  # Resultado: [1, 2, 3]
  ```

---

### 2. **¿Para qué sirve?**

- **Optimización**: Evita cargar objetos completos de la base de datos cuando solo necesitas un campo específico (como `"id"`). Esto reduce el uso de memoria y mejora el rendimiento.
- **Facilita operaciones con listas**: Es útil para comparaciones, filtrados o cuando necesitas pasar los IDs a otra consulta.

---

### 3. **Casos de uso comunes**

#### a. **Filtrar objetos usando `__in`**

```python
# Obtener IDs de usuarios activos
active_user_ids = User.objects.filter(is_active=True).values_list("id", flat=True)

# Usar esos IDs para filtrar otro queryset
orders = Order.objects.filter(user_id__in=active_user_ids)
```

#### b. **Verificar pertenencia**

```python
user_ids = User.objects.values_list("id", flat=True)
if some_user_id in user_ids:
    print("El ID existe")
```

#### c. **Serialización rápida**

Si necesitas solo los IDs para una respuesta JSON:

```python
from django.http import JsonResponse

def get_ids(request):
    ids = MyModel.objects.values_list("id", flat=True)
    return JsonResponse(list(ids), safe=False)
```

---

### 4. **Comparación con otros métodos**

| Método                          | Resultado                                       | Uso típico                                                             |
| ------------------------------- | ----------------------------------------------- | ---------------------------------------------------------------------- |
| `.values_list("id")`            | Lista de tuplas: `[(1,), (2,)]`                 | Cuando necesitas múltiples campos como tuplas.                         |
| `.values_list("id", flat=True)` | Lista plana: `[1, 2, 3]`                        | Para trabajar directamente con valores individuales.                   |
| `.values("id")`                 | Lista de diccionarios: `[{"id": 1}, {"id": 2}]` | Cuando prefieres acceso por clave (menos eficiente que `values_list`). |

---

### 5. **Precauciones**

- **Solo funciona con un único campo**: Si intentas usar `flat=True` con múltiples campos, lanzará un error.  
  Ejemplo inválido:  
  
  ```python
  MyModel.objects.values_list("id", "name", flat=True)  # ❌ Error
  ```

- **No es compatible con campos relacionales directamente**:  
  Para campos relacionales (ej: `ForeignKey`), debes especificar el campo subyacente:  
  
  ```python
  MyModel.objects.values_list("related_model__id", flat=True)
  ```

- **Orden y duplicados**:  
  Si necesitas eliminar duplicados, agrega `.distinct()`:  
  
  ```python
  MyModel.objects.values_list("id", flat=True).distinct()
  ```

---

### 6. **Ejemplo completo**

```python
# Supongamos un modelo:
class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

# Obtener IDs de libros de un autor específico
book_ids = Book.objects.filter(author__name="Gabriel García Márquez").values_list("id", flat=True)

# Usar esos IDs para eliminar los libros
Book.objects.filter(id__in=book_ids).delete()
```

---

### Resumen

- **`values_list("id", flat=True)`** es ideal para obtener una lista simple de valores de un campo (como `"id"`).
- Mejora el rendimiento al evitar cargar objetos completos.
- Úsalo para filtrados (`__in`), verificaciones o serialización rápida.
- Solo funciona con un único campo; para múltiples campos, omite `flat=True`.

----------

# Coalesce

El `Coalesce` en Django se utiliza para **manejar valores nulos (`NULL`) en consultas SQL** dentro del ORM. Su propósito es **devolver el primer valor no nulo de una lista de expresiones**, lo que permite definir valores por defecto cuando un campo o expresión es `NULL`.

---

### ¿Cómo se usa en `annotate()`?

Se utiliza dentro de `annotate()` para crear un campo calculado que **sustituya valores `NULL` por un valor alternativo**. Es común en consultas que combinan campos relacionados (ej: `ForeignKey`, `OneToOneField`) o expresiones complejas.

#### Ejemplo:

```python
from django.db.models import F, Value, CharField
from django.db.models.functions import Coalesce

queryset = MyModel.objects.annotate(
    author_name=Coalesce(
        F("created_by__username"),  # Primer valor a evaluar
        Value(str(_("Automático")), output_field=CharField())  # Valor por defecto si es NULL
    )
)
```

Este código hace lo siguiente:

1. **Evalúa `F("created_by__username")**: Si el campo `created_by` (una relación) no es `NULL`, usa su `username`.
2. **Si es `NULL`, usa el valor `"Automático"`** (traducido según el idioma del usuario).

---

### ¿Qué hace exactamente `Coalesce`?

Es equivalente a la función SQL `COALESCE`, que devuelve el primer valor no nulo en una lista. En la consulta anterior, se traduce a SQL como:

```sql
SELECT COALESCE("created_by"."username", 'Automático') AS "author_name"
FROM "myapp_mymodel"
LEFT JOIN "auth_user" ON ("myapp_mymodel"."created_by_id" = "auth_user"."id")
```

---

### Componentes clave:

1. **`F()`**:  
   Permite referenciar campos del modelo directamente en la base de datos (en lugar de traerlos a memoria).  
   
   - `F("created_by__username")`: Accede al campo `username` del modelo relacionado `created_by`.

2. **`Value()`**:  
   Define un valor literal (como un string o número) para usar en la consulta.  
   
   - `Value(str(_("Automático")))`: Inserta el texto `"Automático"` traducido según el idioma del usuario.

3. **`output_field=CharField()`**:  
   Especifica el tipo de dato esperado para el valor por defecto. Es necesario cuando el valor no es un campo del modelo (ej: un `Value`).

---

### Casos de uso comunes:

1. **Mostrar un valor por defecto cuando un campo es `NULL`**:
   
   ```python
   queryset = Product.objects.annotate(
       price_with_default=Coalesce(F("price"), Value(0.0))
   )
   ```
   
   Si `price` es `NULL`, muestra `0.0`.

2. **Combinar múltiples campos relacionados**:
   
   ```python
   queryset = Order.objects.annotate(
       customer_name=Coalesce(
           F("customer__name"),
           F("guest_customer__name"),
           Value("Cliente anónimo")
       )
   )
   ```
   
   Usa el nombre del cliente registrado, o el de un cliente invitado, o "Cliente anónimo" si ambos son `NULL`.

3. **Personalizar resultados en serializadores o APIs**:
   Para evitar `null` en respuestas JSON, sustituyendo por valores legibles.

---

### Ventajas:

- **Eficiencia**: La lógica se ejecuta en la base de datos, no en Python, lo que reduce la carga en la aplicación.
- **Claridad**: Permite definir reglas de negocio directamente en la consulta.
- **Flexibilidad**: Puedes usar múltiples argumentos (ej: `Coalesce(field1, field2, field3, default)`).

---

### Consideraciones:

1. **Base de datos compatible**:  
   `Coalesce` está disponible en la mayoría de las bases de datos compatibles con Django (PostgreSQL, MySQL, SQLite, etc.).

2. **Tipos de campo**:  
   Asegúrate de especificar `output_field` cuando combines tipos de datos distintos (ej: un campo `TextField` con un `Value("default")`).

3. **Múltiples argumentos**:  
   Puedes pasar tantas expresiones como necesites:
   
   ```python
   Coalesce(F("field1"), F("field2"), F("field3"), Value("default"))
   ```

---

### Ejemplo completo:

```python
from django.db.models import F, Value, CharField
from django.db.models.functions import Coalesce
from myapp.models import Post

# Anotar cada Post con el autor (username) o "Automático" si no tiene autor
posts = Post.objects.annotate(
    author_name=Coalesce(
        F("author__username"),  # Intenta obtener el username del autor
        Value(str(_("Automático")), output_field=CharField())  # Valor por defecto
    )
).values("title", "author_name")

# Resultado:
# [{"title": "Mi primer post", "author_name": "juan"}, 
#  {"title": "Post sin autor", "author_name": "Automático"}]
```

---

### Resumen:

- **`Coalesce`** es ideal para manejar valores `NULL` en consultas.
- Úsalo con `F()` para campos y `Value()` para valores literales.
- Es útil para evitar `null` en respuestas API, generar reportes o mostrar datos legibles.
- Se evalúa directamente en la base de datos, lo que mejora el rendimiento.

# django-daterangefilter

## PastDateRangeFilter

El import `from daterangefilter.filters import PastDateRangeFilter` y su uso en `list_filter` en Django Admin permite **filtrar registros basados en un rango de fechas pasado** (fechas anteriores a la actual). Es parte del paquete de terceros [`django-daterangefilter`](https://pypi.org/project/django-daterangefilter/), que extiende las capacidades de filtrado en la interfaz de administración de Django.

---

### ¿Para qué sirve?

Permite agregar un filtro visual en el panel de Django Admin para seleccionar **rangos de fechas pasadas** (como "Hoy", "Ayer", "Esta semana", "Este mes", etc.) en lugar de elegir una fecha específica. Es útil cuando necesitas analizar datos históricos sin teclear fechas manualmente.

---

### Ejemplo de uso en Django Admin:

```python
from daterangefilter.filters import PastDateRangeFilter

class MyModelAdmin(admin.ModelAdmin):
    list_filter = (
        "allow_referrals",
        "currency",
        "enabled",
        "status",
        ("created", PastDateRangeFilter),  # Filtrar por fechas pasadas en el campo "created"
    )
```

Este código añade un filtro de rango de fechas pasado para el campo `created` en la lista de registros del modelo `MyModel`.

---

### ¿Qué hace exactamente `PastDateRangeFilter`?

1. **Muestra un widget de selección de rango de fechas** en el panel de filtros de Django Admin.
2. **Permite elegir opciones predefinidas** como:
   - Hoy
   - Ayer
   - Esta semana
   - Este mes
   - Últimos 7 días
   - Últimos 30 días
   - Personalizado (para seleccionar fechas específicas)
3. **Filtra los registros** cuyo campo de fecha (en este caso, `created`) esté dentro del rango seleccionado.

---

### Comparación con filtros nativos de Django:

- **Filtro nativo `DateField` en Django Admin**:  
  Solo permite filtrar por una fecha exacta (hoy, ayer, etc.) pero no por rangos personalizados.
- **`PastDateRangeFilter`**:  
  Ofrece más flexibilidad con rangos dinámicos y una interfaz amigable.

---

### Requisitos:

1. El campo a filtrar debe ser de tipo `DateField` o `DateTimeField`.

2. Instalar el paquete:
   
   ```bash
   pip install django-daterangefilter
   ```

---

### Ventajas:

- **Facilidad de uso**: No requiere teclear fechas manualmente.
- **Análisis rápido**: Permite filtrar datos históricos con un clic.
- **Personalización**: Se puede combinar con otros filtros (como `status`, `currency`, etc.).

---

### Ejemplo de resultado:

Si tienes un modelo con un campo `created` (fecha de creación), al aplicar `PastDateRangeFilter`, verás en Django Admin:

```
Filtrar por "created":
- Hoy
- Ayer
- Esta semana
- Este mes
- Últimos 7 días
- Últimos 30 días
- Personalizado...
```

Seleccionar "Esta semana" mostrará solo los registros creados en los últimos 7 días.

---

### Alternativas:

- **`DateRangeFilter`**: Similar, pero permite seleccionar rangos que incluyan fechas futuras.
- **Filtros personalizados con `SimpleListFilter`**: Para lógicas más complejas o específicas.

---

### Resumen:

- **`PastDateRangeFilter`** es una herramienta de `django-daterangefilter` para filtrar registros en Django Admin por fechas pasadas.
- Ideal para análisis histórico de datos (ventas, usuarios, logs, etc.).
- Mejora la experiencia de usuario con opciones predefinidas y un selector visual.

# get_actions -  Admin

La sobrescritura del método `get_actions(self, request)` en un `ModelAdmin` de Django se utiliza para **personalizar las acciones disponibles en la interfaz de administración** (por ejemplo, en la lista de registros de un modelo). Estas acciones son operaciones que se pueden aplicar a múltiples registros seleccionados (como "Eliminar seleccionados", "Publicar seleccionados", etc.).

---

### ¿Qué hace `get_actions`?

Por defecto, Django incluye acciones como `"delete_selected"` (eliminar seleccionados). El método `get_actions` devuelve un diccionario con las acciones disponibles, donde cada clave es el nombre de la acción y el valor es una tupla con:

1. La función que ejecuta la acción.
2. Un nombre legible (opcional).
3. Una descripción (opcional).

Ejemplo de salida por defecto:

```python
{
    "delete_selected": (delete_selected, "Eliminar seleccionados", "Eliminar los elementos seleccionados")
}
```

---

### ¿Por qué sobrescribirlo?

1. **Agregar acciones personalizadas**  
   Ejemplo: Agregar una acción para "Archivar seleccionados" o "Enviar correo a usuarios seleccionados".

2. **Eliminar acciones predeterminadas**  
   Ejemplo: Quitar la acción `"delete_selected"` para evitar eliminaciones accidentales.

3. **Modificar condiciones dinámicas**  
   Ejemplo: Mostrar ciertas acciones solo si el usuario tiene permisos específicos o cumple ciertas condiciones.

---

### Ejemplo completo de uso:

#### 1. Definir una acción personalizada

Primero define la función que realizará la acción:

```python
from django.contrib import messages

def marcar_como_activo(modeladmin, request, queryset):
    queryset.update(estado="activo")
    messages.success(request, "Los elementos seleccionados fueron marcados como activos.")
marcar_como_activo.short_description = "Marcar como activo"
```

#### 2. Sobrescribir `get_actions` en el `ModelAdmin`

```python
class MiModeloAdmin(admin.ModelAdmin):
    def get_actions(self, request):
        actions = super().get_actions(request)

        # Añadir acción personalizada
        actions["marcar_como_activo"] = (marcar_como_activo, "marcar_como_activo", "Marcar como activo")

        # Eliminar acción predeterminada (ej: delete_selected)
        if "delete_selected" in actions:
            del actions["delete_selected"]

        return actions
```

---

### Casos de uso comunes

#### 1. **Acciones condicionales basadas en permisos**

```python
def get_actions(self, request):
    actions = super().get_actions(request)

    # Solo mostrar ciertas acciones si el usuario tiene permiso
    if not request.user.has_perm("miapp.puede_archivar"):
        if "archivar" in actions:
            del actions["archivar"]

    return actions
```

#### 2. **Acciones para usuarios específicos**

```python
def get_actions(self, request):
    actions = super().get_actions(request)

    # Solo mostrar acción si el usuario es staff
    if not request.user.is_staff:
        if "enviar_notificacion" in actions:
            del actions["enviar_notificacion"]

    return actions
```

---

### Buenas prácticas

1. **Usa `super()` para preservar acciones existentes**  
   Siempre llama a `super().get_actions(request)` para mantener las acciones predeterminadas (ej: `"delete_selected"`).

2. **Documenta las acciones**  
   Usa `short_description` para dar nombres legibles a las acciones.

3. **Maneja errores con `messages`**  
   Usa `messages.success`, `messages.warning`, etc., para informar al usuario sobre el resultado de la acción.

4. **Valida permisos**  
   Verifica que el usuario tenga permisos antes de permitir acciones sensibles.

---

### Ejemplo avanzado: Acción que filtra registros

```python
def desactivar_usuarios_inactivos(modeladmin, request, queryset):
    # Filtrar usuarios inactivos (ej: sin actividad en 30 días)
    hace_30_dias = timezone.now() - timedelta(days=30)
    inactivos = queryset.filter(ultimo_login__lt=hace_30_dias)
    inactivos.update(is_active=False)
    messages.info(request, f"{inactivos.count()} usuarios inactivos desactivados.")
desactivar_usuarios_inactivos.short_description = "Desactivar usuarios inactivos"
```

---

### Resumen

- **`get_actions`** controla qué acciones aparecen en la interfaz de administración.
- Úsalo para **agregar, eliminar o modificar acciones** según necesidades.
- Ideal para operaciones en lote (ej: actualizar estados, enviar correos, archivar).
- Combínalo con validaciones de permisos y mensajes para una experiencia segura y clara.

-----------------

# get_urls(self) - Admin

La sobrescritura del método `get_urls(self)` en Django Admin se utiliza para **definir o extender las URLs personalizadas** que estarán disponibles en la interfaz de administración de un modelo. Esto permite agregar vistas personalizadas, acciones especiales o integrar funcionalidades adicionales en el panel de administración.

---

### ¿Por qué se usa?

Por defecto, Django Admin genera automáticamente URLs para operaciones básicas como:

- Listado de registros (`/admin/app/model/`)
- Creación de registros (`/admin/app/model/add/`)
- Edición de registros (`/admin/app/model/123/change/`)
- Eliminación de registros (`/admin/app/model/123/delete/`)

Pero al sobrescribir `get_urls`, puedes:

1. **Agregar vistas personalizadas** (ej: exportar datos a CSV, generar reportes, integrar herramientas externas).
2. **Personalizar la navegación** entre modelos.
3. **Controlar el acceso a ciertas vistas** basado en permisos.
4. **Integrar APIs internas** solo accesibles desde el admin.

---

### Ejemplo básico: Agregar una vista personalizada

Supongamos que quieres crear una acción en el admin que exporte datos a CSV.

#### 1. Define la vista personalizada:

```python
from django.http import HttpResponse
import csv
from django.contrib import admin

class MiModeloAdmin(admin.ModelAdmin):
    def get_urls(self):
        urls = super().get_urls()
        # Añade una URL personalizada
        custom_urls = [
            path('exportar-csv/', self.admin_site.admin_view(self.exportar_csv), name='miapp-mimodelo-exportar'),
        ]
        return custom_urls + urls

    def exportar_csv(self, request):
        # Lógica para exportar datos a CSV
        response = HttpResponse(content_type='text/csv')
        response['Content-Disposition'] = 'attachment; filename="datos.csv"'

        writer = csv.writer(response)
        writer.writerow(['ID', 'Nombre'])

        for obj in MiModelo.objects.all():
            writer.writerow([obj.id, obj.nombre])

        return response
```

#### 2. Acceso desde el admin:

Visita `/admin/miapp/mimodelo/exportar-csv/` y obtendrás el archivo CSV.

---

### Componentes clave:

1. **`path()` o `re_path()`**:  
   Define la URL y la función que la maneja.  
   
   - `path('exportar-csv/', ...)` crea la ruta `/admin/miapp/mimodelo/exportar-csv/`.

2. **`admin_view()`**:  
   Envuelve la vista personalizada para asegurar que solo usuarios autenticados con permisos de admin puedan acceder.  
   
   - `self.admin_site.admin_view(self.exportar_csv)`

3. **`name`**:  
   Asigna un nombre único a la URL para usarlo en plantillas, redirecciones o enlaces.

---

### Casos de uso comunes:

1. **Exportar datos** (CSV, Excel, JSON).
2. **Importar datos** desde archivos.
3. **Vistas de reportes** (ej: estadísticas, gráficos).
4. **Acciones masivas** (ej: enviar correos a usuarios seleccionados).
5. **Integración con herramientas externas** (ej: conectar con una API de terceros).

---

### Ejemplo avanzado: Botón en la lista de registros

Puedes combinar `get_urls` con una columna personalizada en `list_display` para crear botones que ejecuten acciones personalizadas.

#### 1. Define el botón en la lista:

```python
class MiModeloAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'accion_personalizada')

    def accion_personalizada(self, obj):
        return format_html('<a class="button" href="{}">Exportar</a>', reverse('admin:miapp-mimodelo-exportar'))

    accion_personalizada.short_description = "Acción"
```

#### 2. Define la URL en `get_urls`:

```python
def get_urls(self):
    urls = super().get_urls()
    custom_urls = [
        path('<path:object_id>/exportar/', self.admin_site.admin_view(self.exportar_registro), name='miapp-mimodelo-exportar'),
    ]
    return custom_urls + urls
```

#### 3. Vista personalizada:

```python
def exportar_registro(self, request, object_id):
    obj = MiModelo.objects.get(pk=object_id)
    # Lógica para exportar solo este objeto
    return HttpResponse(f"Exportando {obj.nombre}")
```

---

### Consideraciones importantes:

1. **Orden de las URLs**:  
   Django busca coincidencias desde la primera URL hacia la última. Si defines una URL muy general (ej: `path('<str:slug>/', ...)`), asegúrate de que no bloquee otras rutas.

2. **Seguridad**:  
   Usa `admin_view()` para proteger vistas contra acceso no autorizado.  
   
   - También puedes validar permisos dentro de la vista:
     
     ```python
     if not request.user.has_perm('miapp.puede_exportar'):
         raise PermissionDenied
     ```

3. **CSRF y POST**:  
   Si tu vista recibe formularios o solicitudes POST, añade protección CSRF:
   
   ```python
   from django.views.decorators.csrf import csrf_protect
   
   @csrf_protect
   def mi_vista(self, request):
       # ...
   ```

4. **Plantillas personalizadas**:  
   Si tu vista renderiza una plantilla, asegúrate de usar el contexto adecuado:
   
   ```python
   from django.shortcuts import render
   
   def mi_vista(self, request):
       context = dict(self.admin_site.each_context(request))
       return render(request, "miapp/mi_vista.html", context)
   ```

---

### Buenas prácticas:

- **Usa `reverse()`** para generar URLs dinámicamente.
- **Mantén URLs limpias y descriptivas** (ej: `/admin/miapp/mimodelo/reportes/ventas/`).
- **Documenta tus vistas personalizadas** para otros desarrolladores.
- **Prueba todas las rutas** para evitar errores 404 o conflictos.

---

### Resumen:

- **`get_urls`** permite extender las URLs disponibles en Django Admin.
- Úsalo para crear vistas personalizadas como exportaciones, reportes o acciones masivas.
- Combínalo con `admin_view()` para garantizar seguridad.
- Es ideal para integrar funcionalidades avanzadas sin modificar la lógica principal del admin.

-------------------------

# @transaction.atomic

El decorador `@transaction.atomic` en Django se utiliza para **garantizar la atomicidad de operaciones de base de datos**, asegurando que **todas las operaciones dentro de un bloque de código se ejecuten completamente o ninguna**. Si ocurre un error en alguna parte, todas las operaciones se deshacen (rollback), manteniendo la base de datos en un estado consistente.

---

### **¿Qué hace `@transaction.atomic`?**

- **Inicia una transacción** cuando se ejecuta la función decorada.
- **Confirma (commit)** los cambios si todo funciona correctamente.
- **Deshace (rollback)** los cambios si ocurre una excepción (ej: `IntegrityError`, `ValidationError`, etc.).

---

### **Ejemplo básico:**

```python
from django.db import transaction

@transaction.atomic
def transferir_dinero(origen_id, destino_id, monto):
    cuenta_origen = Cuenta.objects.select_for_update().get(id=origen_id)
    cuenta_destino = Cuenta.objects.select_for_update().get(id=destino_id)

    if cuenta_origen.saldo < monto:
        raise ValueError("Saldo insuficiente")

    cuenta_origen.saldo -= monto
    cuenta_destino.saldo += monto

    cuenta_origen.save()
    cuenta_destino.save()
```

Si ocurre un error (ej: saldo insuficiente), **ninguna de las dos cuentas se actualiza**.

---

### **¿Cuándo usarlo como buena práctica?**

#### 1. **Operaciones que afectan múltiples tablas/registros**

- Ejemplo: Crear un pedido y actualizar el inventario.
  
  ```python
  @transaction.atomic
  def crear_pedido(usuario, productos):
    pedido = Pedido.objects.create(usuario=usuario)
    for producto in productos:
        if producto.stock <= 0:
            raise ValueError("Producto sin stock")
        producto.stock -= 1
        producto.save()
        PedidoProducto.objects.create(pedido=pedido, producto=producto)
    return pedido
  ```

#### 2. **Operaciones críticas donde la consistencia es vital**

- Transferencias bancarias, compras, reservas, etc.

- Ejemplo: Reservar una habitación y generar una factura.
  
  ```python
  @transaction.atomic
  def reservar_habitacion(habitacion_id, cliente_id):
    habitacion = Habitacion.objects.select_for_update().get(id=habitacion_id)
    if habitacion.ocupada:
        raise ValueError("Habitación no disponible")
    habitacion.ocupada = True
    habitacion.save()
    Factura.objects.create(cliente_id=cliente_id, habitacion=habitacion)
  ```

#### 3. **Cuando usas señales (`signals`) que modifican la base de datos**

- Si una señal lanza una excepción, la transacción se deshace.
  
  ```python
  @receiver(post_save, sender=Pedido)
  def generar_factura(sender, instance, created, **kwargs):
    if created:
        Factura.objects.create(pedido=instance)
  ```

#### 4. **En vistas que realizan múltiples operaciones**

- Ejemplo: Una vista que crea un usuario y su perfil.
  
  ```python
  @transaction.atomic
  def crear_usuario_y_perfil(request):
    user = User.objects.create(username=request.POST["username"])
    Perfil.objects.create(usuario=user, bio=request.POST["bio"])
  ```

#### 5. **Cuando usas `select_for_update()` para evitar condiciones de carrera**

- Bloquea temporalmente filas para evitar modificaciones concurrentes.
  
  ```python
  @transaction.atomic
  def actualizar_stock(producto_id):
    producto = Producto.objects.select_for_update().get(id=producto_id)
    if producto.stock > 0:
        producto.stock -= 1
        producto.save()
  ```

---

### **Ventajas de usar `@transaction.atomic`:**

- **Consistencia**: Evita estados inconsistentes en la base de datos.
- **Seguridad**: Deshace cambios si ocurre un error inesperado.
- **Control**: Permite manejar transacciones explícitamente.
- **Rendimiento**: Reduce el número de commits al agrupar operaciones.

---

### **Consideraciones y buenas prácticas:**

#### 1. **Evitar operaciones largas dentro de transacciones**

- Las transacciones largas bloquean filas y pueden causar problemas de concurrencia.

- Ejemplo incorrecto:
  
  ```python
  @transaction.atomic
  def proceso_largo(datos):
      for dato in datos:
          time.sleep(1)  # Operación lenta
          Procesar(dato).save()
  ```

#### 2. **Usar `select_for_update()` o `select_related()` cuando sea necesario**

- Para evitar condiciones de carrera o optimizar consultas.

#### 3. **Manejar excepciones dentro de la transacción**

- Capturar errores y reintentar o loguear.
  
  ```python
  @transaction.atomic
  def crear_pedido(usuario, productos):
    try:
        # Operaciones...
    except IntegrityError:
        logger.error("Error de integridad")
        raise
  ```

#### 4. **No usarlo en operaciones de solo lectura**

- No es necesario si solo estás consultando datos.
  
  ```python
  def obtener_datos():
    return Modelo.objects.all()  # Sin transacción
  ```

#### 5. **Usar contexto (`with`) para bloques específicos**

- En lugar de decorar toda una función.
  
  ```python
  def actualizar_saldos():
    with transaction.atomic():
        cuenta1.saldo -= 100
        cuenta2.saldo += 100
        cuenta1.save()
        cuenta2.save()
  ```

---

### **Resumen**

| **Caso de uso**                                   | **Recomendado usar `@transaction.atomic`** |
| ------------------------------------------------- | ------------------------------------------ |
| Transferencias bancarias                          | ✅ Sí                                       |
| Actualización de múltiples registros relacionados | ✅ Sí                                       |
| Operaciones con señales que modifican datos       | ✅ Sí                                       |
| Consultas de solo lectura                         | ❌ No                                       |
| Operaciones largas o con bloqueos                 | ⚠️ Solo si es estrictamente necesario      |

---

### **Conclusión**

Usa `@transaction.atomic` **cuando la consistencia de datos es crítica** y varias operaciones deben ejecutarse juntas. Es ideal para casos como transferencias, reservas o actualizaciones relacionadas. Sin embargo, evita usarlo en operaciones simples o largas para no afectar el rendimiento del sistema.

### **1. ¿Es necesario alguna configuración para usar `@transaction.atomic`?**

No, **no requiere configuración adicional** en la mayoría de los casos. Django ya está preparado para usar transacciones con cualquier base de datos compatible. Solo necesitas importar `transaction` y usar el decorador o el contexto:

```python
from django.db import transaction

@transaction.atomic
def mi_funcion():
    # Operaciones de base de datos
```

Sin embargo, hay **algunas consideraciones específicas según la base de datos** que debes tener en cuenta.

---

### **2. ¿Es compatible con todas las bases de datos?**

No exactamente. `@transaction.atomic` **funciona en todas las bases de datos compatibles con Django**, pero su comportamiento puede variar ligeramente según el motor:

| Base de datos  | Soporte   | Notas                                                                                  |
| -------------- | --------- | -------------------------------------------------------------------------------------- |
| **PostgreSQL** | ✅ Total   | Soporta transacciones completas, bloqueos (`select_for_update`), y savepoints.         |
| **MySQL**      | ✅ Total   | Soporta transacciones con motores como InnoDB. MyISAM no soporta transacciones.        |
| **SQLite**     | ✅ Parcial | Soporta transacciones básicas, pero tiene limitaciones con concurrencia y anidamiento. |
| **Oracle**     | ✅ Total   | Soporta transacciones avanzadas.                                                       |
| **MariaDB**    | ✅ Total   | Similar a MySQL.                                                                       |

---

### **3. Consideraciones específicas para PostgreSQL**

PostgreSQL es el motor más robusto para transacciones. Algunas cosas a tener en cuenta:

#### a. **Bloqueos con `select_for_update()`**

Si usas `select_for_update()` dentro de una transacción, asegúrate de que la transacción esté bien definida. PostgreSQL requiere que estas operaciones se realicen dentro de un bloque `atomic`.

```python
@transaction.atomic
def actualizar_stock(producto_id):
    producto = Producto.objects.select_for_update().get(id=producto_id)
    if producto.stock > 0:
        producto.stock -= 1
        producto.save()
```

#### b. **Transacciones largas**

Evita transacciones muy largas (ej: bucles con miles de registros). Pueden causar:

- **Problemas de rendimiento**: PostgreSQL usa MVCC (Multiversion Concurrency Control), y las transacciones largas pueden causar "bloating" en tablas.
- **Bloqueos innecesarios**: Si bloqueas filas con `select_for_update`, otras operaciones tendrán que esperar.

#### c. **Savepoints anidados**

PostgreSQL permite savepoints anidados, pero Django abstrae esto. Puedes usar `atomic` dentro de otro `atomic` sin problemas.

```python
@transaction.atomic
def outer():
    with transaction.atomic():  # Savepoint
        # Operaciones
```

---

### **4. Consideraciones específicas para SQLite**

SQLite es útil para desarrollo o prototipos, pero tiene limitaciones:

#### a. **Bloqueo de la base de datos**

SQLite solo permite **una escritura simultánea**. Si usas transacciones largas o múltiples conexiones concurrentes, puedes enfrentar errores de bloqueo (`database locked`).

#### b. **No soporta transacciones anidadas nativas**

Django simula el anidamiento con **savepoints**, pero no es tan robusto como en PostgreSQL.

```python
@transaction.atomic
def outer():
    with transaction.atomic():  # Simulado con savepoints
        # Operaciones
```

#### c. **Limitaciones en concurrencia**

En entornos de alta concurrencia (ej: servidores web con múltiples usuarios), SQLite no es recomendado. Usa PostgreSQL o MySQL en producción.

---

### **5. Buenas prácticas generales**

- **Evita operaciones largas dentro de transacciones**:  
  Reduce el tiempo que una transacción mantiene filas bloqueadas.

- **Usa `select_for_update()` con cuidado**:  
  Solo cuando sea necesario para evitar condiciones de carrera.

- **Maneja excepciones**:  
  Captura errores para evitar que la transacción falle silenciosamente.
  
  ```python
  @transaction.atomic
  def transferir(origen, destino, monto):
      try:
          # Operaciones
      except Exception as e:
          logger.error(f"Error en transacción: {e}")
          raise
  ```

- **No uses `atomic` para consultas de solo lectura**:  
  No es necesario y puede afectar el rendimiento.

- **Prueba en producción con tu motor real**:  
  Si desarrollas con SQLite pero usas PostgreSQL en producción, prueba en PostgreSQL antes de desplegar.

---

### **6. Ejemplo combinando PostgreSQL y SQLite**

Si usas ambos motores (ej: desarrollo con SQLite y producción con PostgreSQL), asegúrate de que tu código no dependa de funcionalidades específicas de uno.

#### Código válido para ambos:

```python
@transaction.atomic
def crear_usuario_y_perfil(datos):
    user = User.objects.create(**datos)
    Perfil.objects.create(usuario=user, bio=datos["bio"])
```

#### Código problemático (solo válido en PostgreSQL):

```python
@transaction.atomic
def usar_jsonb():
    # JSONField solo está disponible en PostgreSQL
    MiModelo.objects.create(json_data={"clave": "valor"})
```

---

### **7. Resumen por motor**

| Motor          | Transacciones | Savepoints anidados | Bloqueos                               | Recomendado para              |
| -------------- | ------------- | ------------------- | -------------------------------------- | ----------------------------- |
| **PostgreSQL** | ✅ Sí          | ✅ Sí                | ⚠️ Controlados con `select_for_update` | Producción, alta concurrencia |
| **MySQL**      | ✅ Sí          | ✅ Sí                | ⚠️ Requiere InnoDB                     | Producción, alta concurrencia |
| **SQLite**     | ✅ Sí          | ⚠️ Simulados        | ❌ Limitado (bloqueo global)            | Desarrollo, prototipos        |

---

### **Conclusión**

- **No necesitas configuración adicional** para usar `@transaction.atomic`.
- **PostgreSQL** es el motor más robusto para transacciones.
- **SQLite** tiene limitaciones en concurrencia y anidamiento.
- **Sigue buenas prácticas**: Evita transacciones largas, maneja excepciones y prueba en tu motor de producción.

Si usas principalmente PostgreSQL, estás en una de las mejores bases de datos para transacciones. Si usas SQLite, sé cuidadoso con escenarios de alta concurrencia.

# transaction.on_commit

### **¿Qué es `transaction.on_commit` y cómo se usa con `@transaction.atomic`?**

`transaction.on_commit` es una función de Django que **registra una acción (callback) que se ejecutará solo después de que una transacción de base de datos se haya confirmado (`commit`) con éxito**. Se usa dentro de bloques decorados con `@transaction.atomic` para garantizar que ciertas operaciones (como tareas en segundo plano, actualizaciones de índices de búsqueda o notificaciones) ocurran **solo si la transacción no falla**.

---

### **¿Por qué usarlo?**

Dentro de un bloque `@transaction.atomic`, todas las operaciones de base de datos se agrupan en una transacción. Si cualquier operación falla, la transacción se deshace (`rollback`). Sin embargo, **algunas acciones no deben ejecutarse si la transacción no se confirma**, como:

- Enviar correos electrónicos.
- Iniciar tareas asíncronas (ej: Celery).
- Actualizar índices de búsqueda (ej: Elasticsearch).
- Registrar logs en sistemas externos.

`transaction.on_commit` garantiza que estas acciones se ejecuten **solo si la transacción se confirma**.

---

### **Ejemplo práctico con Celery**

```python
from django.db import transaction
from myapp.tasks import seller_assignation_by_reservations

@transaction.atomic(using="default")
def create(self, request, *args, **kwargs):
    # Crear una instancia dentro de la transacción
    instance = super().create(request, *args, **kwargs)

    # Programar una tarea Celery solo si la transacción se confirma
    transaction.on_commit(lambda: seller_assignation_by_reservations.delay([instance.reservation_id]))
```

#### ¿Qué sucede aquí?

1. **`@transaction.atomic(using="default")`**:
   
   - Asegura que todas las operaciones dentro de `create()` sean atómicas.
   - Usa la base de datos `"default"` (útil en entornos con múltiples bases de datos).

2. **`instance = super().create(...)`**:
   
   - Crea un objeto en la base de datos, pero **todavía no se ha confirmado la transacción** (está en estado pendiente).

3. **`transaction.on_commit(...)`**:
   
   - Registra la tarea Celery (`seller_assignation_by_reservations.delay`) para que se ejecute **después de que la transacción se confirme**.
   - Si la transacción falla (ej: excepción), la tarea no se ejecuta.

---

### **¿Cómo funciona internamente?**

- **Dentro de `@transaction.atomic`**:
  
  - Las operaciones se ejecutan en una transacción temporal.
  - Si todo funciona, Django llama a `commit()` y ejecuta los callbacks registrados con `on_commit`.
  - Si hay un error, Django llama a `rollback()` y **omite los callbacks**.

- **Con `lambda`**:
  
  - `transaction.on_commit` requiere una función sin argumentos.
  - Usar `lambda` permite pasar argumentos a la función objetivo (ej: `instance.reservation_id`).

---

### **Casos de uso comunes**

1. **Tareas asíncronas con Celery**:
   
   ```python
   transaction.on_commit(lambda: send_email.delay(user_id))
   ```

2. **Actualizar índices de búsqueda**:
   
   ```python
   transaction.on_commit(lambda: update_search_index.delay(instance.pk))
   ```

3. **Registrar logs en sistemas externos**:
   
   ```python
   transaction.on_commit(lambda: log_to_external_system(instance))
   ```

4. **Enviar señales post-guardado**:
   
   ```python
   transaction.on_commit(lambda: post_save.send(sender=MyModel, instance=instance))
   ```

---

### **Ventajas**

- **Evita efectos secundarios no deseados**:  
  Si la transacción falla, las acciones externas no se ejecutan.

- **Seguridad en operaciones críticas**:  
  Ideal para tareas que dependen de datos confirmados en la base de datos.

- **Control de concurrencia**:  
  Asegura que las acciones solo se ejecuten después de que la transacción se haya completado.

---

### **Consideraciones y buenas prácticas**

1. **Solo funciona dentro de transacciones**:
   
   - Si no estás dentro de `@transaction.atomic`, `on_commit` se ejecuta inmediatamente.

2. **Evita operaciones costosas en el callback**:
   
   - `on_commit` se ejecuta en el hilo principal, por lo que operaciones bloqueantes pueden afectar el rendimiento.

3. **Manejo de errores en el callback**:
   
   - Si el callback lanza una excepción, **no se deshará la transacción**. Debes manejar errores dentro del callback.

4. **Compatibilidad con bases de datos**:
   
   - Funciona en PostgreSQL, MySQL (con InnoDB), Oracle y SQLite (con limitaciones).

5. **No usar con consultas de solo lectura**:
   
   - No es necesario si no hay modificaciones de datos.

6. **Transacciones anidadas**:
   
   - Si usas transacciones anidadas, `on_commit` se ejecuta cuando la transacción externa se confirma.

---

### **Ejemplo avanzado: Múltiples callbacks**

```python
@transaction.atomic
def procesar_pedido(usuario, productos):
    pedido = Pedido.objects.create(usuario=usuario)

    def registrar_log():
        logger.info(f"Pedido {pedido.id} creado")

    def enviar_notificacion():
        send_notification.delay(pedido.id)

    transaction.on_commit(registrar_log)
    transaction.on_commit(enviar_notificacion)
```

---

### **Resumen**

| Característica     | Detalle                                                        |
| ------------------ | -------------------------------------------------------------- |
| **Propósito**      | Ejecutar acciones solo si una transacción se confirma.         |
| **Uso común**      | Tareas asíncronas, actualizaciones de índices, notificaciones. |
| **Requiere**       | Estar dentro de `@transaction.atomic`.                         |
| **Riesgos**        | Operaciones bloqueantes en el callback, errores no manejados.  |
| **Compatibilidad** | PostgreSQL, MySQL (InnoDB), SQLite (limitado), Oracle.         |

---

### **Conclusión**

Usa `transaction.on_commit` **cuando necesites ejecutar acciones externas solo si una transacción se confirma con éxito**. Es ideal para integrar Django con sistemas externos (ej: Celery, Elasticsearch) o para evitar efectos secundarios en operaciones fallidas. Combínalo con `@transaction.atomic` para garantizar consistencia y seguridad en tus datos.

---------------

# nested_admin

`nested_admin` es una **librería de Django** que permite crear **inlines anidados** en la interfaz de administración de Django. Esto significa que puedes editar modelos relacionados en una sola página, incluso si esos modelos tienen relaciones jerárquicas (por ejemplo, un modelo padre con hijos, y cada hijo con nietos).

---

### **¿Por qué usar `nested_admin`?**

1. **Editar relaciones jerárquicas en una sola pantalla**:  
   Permite gestionar objetos relacionados con múltiples niveles de profundidad sin tener que navegar entre diferentes páginas del admin.

2. **Flujo de trabajo simplificado**:  
   Ideal para formularios complejos donde necesitas crear/editar varios modelos relacionados en un solo paso (por ejemplo, órdenes con productos, productos con características específicas, etc.).

3. **Mejora la experiencia del usuario**:  
   Reduce el número de clics y evita la fragmentación de datos en múltiples pantallas.

---

### **Ejemplo práctico: Ordenes con productos y características**

Vamos a crear un ejemplo donde:

- Una `Order` (orden) tiene múltiples `OrderItem` (items de la orden).
- Cada `OrderItem` tiene múltiples `OrderItemDetail` (detalles del item).

#### **1. Instalación**

```bash
pip install django-nested-admin
```

Agrega `nested_admin` a `INSTALLED_APPS` en `settings.py`:

```python
INSTALLED_APPS = [
    ...
    'nested_admin',
]
```

#### **2. Modelos**

En `myapp/models.py`:

```python
from django.db import models

class Order(models.Model):
    customer_name = models.CharField(max_length=100)
    date = models.DateField()

    def __str__(self):
        return self.customer_name

class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE)
    product_name = models.CharField(max_length=100)
    quantity = models.PositiveIntegerField()

    def __str__(self):
        return f"{self.product_name} ({self.quantity})"

class OrderItemDetail(models.Model):
    item = models.ForeignKey(OrderItem, on_delete=models.CASCADE)
    key = models.CharField(max_length=50)
    value = models.CharField(max_length=100)

    def __str__(self):
        return f"{self.key}: {self.value}"
```

Ejecuta las migraciones:

```bash
python manage.py makemigrations
python manage.py migrate
```

#### **3. Admin con inlines anidados**

En `myapp/admin.py`:

```python
import nested_admin
from django.contrib import admin
from .models import Order, OrderItem, OrderItemDetail

class OrderItemDetailInline(nested_admin.NestedTabularInline):
    model = OrderItemDetail
    extra = 1  # Número de campos adicionales para agregar detalles

class OrderItemInline(nested_admin.NestedTabularInline):
    model = OrderItem
    inlines = [OrderItemDetailInline]  # Inline anidado
    extra = 1

@admin.register(Order)
class OrderAdmin(nested_admin.NestedModelAdmin):
    inlines = [OrderItemInline]  # Primer nivel de inline
```

#### **4. Resultado**

Al acceder a la página de edición de una `Order` en el admin:

- Podrás agregar/editar `OrderItem` directamente.
- Dentro de cada `OrderItem`, podrás agregar/editar `OrderItemDetail`.

---

### **Ventajas clave**

- **Jerarquía visual clara**: Los inlines anidados se muestran como bloques dentro de otros bloques.
- **Edición en tiempo real**: No necesitas guardar la orden principal antes de agregar items o detalles.
- **Compatibilidad con Django Admin**: Funciona con las herramientas estándar de Django (filtros, acciones, etc.).

---

### **Casos de uso típicos**

- **E-commerce**: Órdenes con productos y atributos personalizados.
- **CMS**: Páginas con secciones y subsecciones.
- **Encuestas**: Preguntas con opciones y respuestas anidadas.

---

### **Notas importantes**

- **Limitaciones**: No es ideal para relaciones muy profundas (más de 2-3 niveles) por cuestiones de usabilidad.
- **Dependencias**: Asegúrate de incluir los archivos estáticos de `nested_admin` (generalmente se cargan automáticamente).
- **Compatibilidad**: Verifica que funcione con tu versión de Django (revisa la [documentación oficial](https://django-nested-admin.readthedocs.io/)).

Este ejemplo muestra cómo `nested_admin` simplifica la gestión de datos jerárquicos. ¡Adapta los modelos según tus necesidades!

------------------

# django-dynamic-db-router

Voy a crearte un tutorial completo para entender y usar `django-dynamic-db-router`. Este ejemplo incluirá:

1. Configuración básica del proyecto
2. Ejemplo de modelos
3. Configuración de múltiples bases de datos
4. Uso del router dinámico
5. Ventajas de esta librería
6. Seed con django-seed

## 1. Instalación

Primero instala la librería:

```bash
pip install django-dynamic-db-router
```

## 2. Configuración del proyecto

En `settings.py`:

```python
# settings.py

INSTALLED_APPS = [
    ...
    'dynamic_db_router',  # Añade esto
    'mi_app',
]

# Configuración de múltiples bases de datos
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db_default.sqlite3',
    },
    'clientes_db': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db_clientes.sqlite3',
    },
    'productos_db': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db_productos.sqlite3',
    }
}

# Configura el router dinámico
DATABASE_ROUTERS = ['dynamic_db_router.DynamicDbRouter']
```

## 3. Modelos de ejemplo

En `mi_app/models.py`:

```python
from django.db import models

class Cliente(models.Model):
    nombre = models.CharField(max_length=100)
    email = models.EmailField()

    class Meta:
        db_table = 'clientes'  # Tabla en clientes_db

class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    precio = models.DecimalField(max_digits=10, decimal_places=2)

    class Meta:
        db_table = 'productos'  # Tabla en productos_db
```

## 4. Configuración del router

Crea un archivo `router_config.py`:

```python
# router_config.py
from dynamic_db_router import DynamicDbRouter

class MiRouter(DynamicDbRouter):
    def db_for_read(self, model, **hints):
        if model._meta.model_name == 'cliente':
            return 'clientes_db'
        elif model._meta.model_name == 'producto':
            return 'productos_db'
        return None

    def db_for_write(self, model, **hints):
        if model._meta.model_name == 'cliente':
            return 'clientes_db'
        elif model._meta.model_name == 'producto':
            return 'productos_db'
        return None
```

Y actualiza `settings.py`:

```python
# settings.py
DATABASE_ROUTERS = ['mi_app.router_config.MiRouter']
```

## 5. Migraciones y sincronización

Ejecuta estos comandos:

```bash
# Crea migraciones para todos las bases de datos
python manage.py makemigrations --database=default
python manage.py makemigrations --database=clientes_db
python manage.py makemigrations --database=productos_db

# Aplica migraciones a todas las bases de datos
python manage.py migrate --database=default
python manage.py migrate --database=clientes_db
python manage.py migrate --database=productos_db

# O usa el comando sync_all_dbs (proporcionado por la librería)
python manage.py sync_all_dbs
```

## 6. Ejemplo de uso

```python
# Crear registros
cliente = Cliente.objects.create(nombre='Juan', email='juan@example.com')
producto = Producto.objects.create(nombre='Laptop', precio=999.99)

# Consultar registros (cada uno en su base de datos)
print(Cliente.objects.all())     # Usará clientes_db
print(Producto.objects.all())    # Usará productos_db
```

## 7. Seed con django-seed

Instala django-seed si no lo tienes:

```bash
pip install django-seed
```

En `views.py` o un management command:

```python
from django_seed import Seed
from mi_app.models import Cliente, Producto

def seed_data():
    seeder = Seed.seeder()

    # Seed para clientes_db
    seeder.add_entity(Cliente, 5, {
        'nombre': lambda x: seeder.faker.name(),
        'email': lambda x: seeder.faker.email()
    })

    # Seed para productos_db
    seeder.add_entity(Producto, 3, {
        'nombre': lambda x: seeder.faker.word(),
        'precio': lambda x: seeder.faker.random.uniform(10, 1000)
    })

    # Ejecutar seed en las bases de datos correctas
    inserted_pks = seeder.execute(db='clientes_db')  # Para clientes
    inserted_pks = seeder.execute(db='productos_db')  # Para productos
```

Ejecuta el seed:

```bash
python manage.py shell -c "from mi_app.views import seed_data; seed_data()"
```

## 8. Beneficios de usar django-dynamic-db-router

1. **Arquitectura multitenant**: Permite tener datos de diferentes clientes en bases de datos separadas.

2. **Escalabilidad**: Puedes escalar horizontalmente distribuyendo carga entre diferentes bases de datos.

3. **Aislamiento de datos**: Datos críticos pueden almacenarse en bases de datos separadas con diferentes políticas de seguridad.

4. **Optimización de rendimiento**: Consultas pesadas pueden dirigirse a bases de datos dedicadas.

5. **Flexibilidad**: Soporta múltiples tipos de bases de datos (PostgreSQL, MySQL, SQLite, etc.)

6. **Fácil de mantener**: Evita el uso de múltiples routers estáticos, centralizando la lógica de routing.

7. **Soporte para transacciones distribuidas**: Permite trabajar con múltiples bases de datos en una sola aplicación.

8. **Compatibilidad con Django ORM**: Todo el poder del ORM de Django sigue funcionando normalmente.

## 9. Comandos útiles

```bash
# Ver todas las bases de datos configuradas
python manage.py show_all_dbs

# Copiar datos entre bases de datos
python manage.py copy_db_content --source=clientes_db --target=clientes_backup

# Backup de una base de datos específica
python manage.py dbbackup --database=clientes_db
```

## 10. Consideraciones importantes

1. **Transacciones**: Ten cuidado al usar transacciones entre bases de datos (Django no soporta transacciones multi-db nativamente)

2. **Relaciones cruzadas**: Evita relaciones ForeignKey entre modelos que están en diferentes bases de datos

3. **Caché**: Considera implementar una capa de caché cuando accedas a múltiples bases de datos

4. **Testing**: Usa `TransactionTestCase` en lugar de `TestCase` cuando pruebes operaciones entre múltiples bases de datos

Este ejemplo te permite crear una arquitectura flexible donde puedes escalar fácilmente añadiendo nuevas bases de datos y actualizando la configuración del router. La principal ventaja de esta librería es que te permite manejar múltiples bases de datos de forma dinámica y flexible sin tener que escribir código repetitivo para cada base de datos.

## Pq usarlo?

La diferencia entre los **routers de base de datos estándar de Django** y los que ofrece **`django-dynamic-db-router`** es clave para entender por qué esta librería puede ser útil en ciertos escenarios. Vamos a desglosarlo con un ejemplo práctico y un caso de uso real: **una aplicación multi-tenant (multi-inquilino)**.

---

### **1. Routers estándar de Django: ¿qué hacen?**

En Django, puedes definir routers personalizados para asignar modelos a bases de datos específicas. Por ejemplo:

```python
# settings.py
DATABASES = {
    'default': {...},
    'clientes_db': {...},
    'productos_db': {...}
}

DATABASE_ROUTERS = ['mi_app.routers.AppRouter']
```

```python
# mi_app/routers.py
class AppRouter:
    def db_for_read(self, model, **hints):
        if model._meta.app_label == 'clientes':
            return 'clientes_db'
        elif model._meta.app_label == 'productos':
            return 'productos_db'
        return None

    def db_for_write(self, model, **hints):
        # Lógica similar
        ...
```

#### **Limitaciones de los routers estándar:**

- **Asignación estática**: La lógica del router se basa en el modelo o la app, no en el contexto de la petición (request).
- **No dinámico**: No puedes cambiar la base de datos en tiempo de ejecución basándote en parámetros como subdominios, headers, o usuarios.
- **Complejidad en multi-tenant**: Si cada cliente (tenant) tiene su propia base de datos, tendrías que crear un router por cada cliente, lo cual es inviable.

---

### **2. ¿Qué aporta `django-dynamic-db-router`?**

Esta librería introduce **ruteo dinámico basado en el contexto de la petición**. Es ideal para escenarios donde la base de datos depende de factores externos, como:

- **Aplicaciones multi-tenant**: Cada cliente (tenant) tiene su propia base de datos.
- **A/B testing**: Usar diferentes bases de datos para grupos de usuarios.
- **Read replicas**: Distribuir consultas a bases de datos secundarias.

#### **Ejemplo: Aplicación multi-tenant**

Imagina que cada cliente accede a tu aplicación con un subdominio único:  
`cliente1.miapp.com`, `cliente2.miapp.com`, etc.

Con `django-dynamic-db-router`, puedes asignar dinámicamente la base de datos según el subdominio:

```python
# mi_app/router_config.py
from dynamic_db_router import DynamicDbRouter

class TenantRouter(DynamicDbRouter):
    def get_db_name(self, request=None, **kwargs):
        # Extraer el subdominio del request
        host = request.get_host().split('.')[0]  # Ej: "cliente1"
        return f"tenant_{host}"  # Base de datos: tenant_cliente1
```

```python
# settings.py
DATABASE_ROUTERS = ['mi_app.router_config.TenantRouter']
```

#### **Ventajas clave:**

1. **Ruteo contextual**: Usa el objeto `request` para decidir qué base de datos usar.

2. **Bases de datos dinámicas**: No necesitas definirlas previamente en `DATABASES`. La librería las crea automáticamente si no existen.

3. **Gestión simplificada**: Evita tener que escribir múltiples routers estáticos.

4. **Migraciones por tenant**: Puedes aplicar migraciones a todas las bases de datos con un solo comando:
   
   ```bash
   python manage.py sync_all_dbs
   ```

---

### **3. Comparativa directa**

| Característica                   | Routers estándar de Django                  | `django-dynamic-db-router`         |
| -------------------------------- | ------------------------------------------- | ---------------------------------- |
| **Contexto de ruteo**            | Solo modelo/app                             | Basado en `request`, headers, etc. |
| **Creación de bases de datos**   | Manual                                      | Automática (si no existen)         |
| **Configuración de `DATABASES`** | Requiere todas las bases de datos definidas | No requiere definirlas previamente |
| **Escalabilidad**                | Limitada (máximo ~5-10 bases)               | Ideal para cientos de bases        |
| **Uso en multi-tenant**          | No viable                                   | Optimizado                         |

---

### **4. ¿Cuándo usar cada uno?**

- **Usa routers estándar** si:
  
  - Tienes un número fijo de bases de datos (ej: una para logs, otra para usuarios).
  - No necesitas cambiar la base de datos en tiempo de ejecución.

- **Usa `django-dynamic-db-router` si**:
  
  - Tienes un sistema multi-tenant.
  - Necesitas bases de datos dinámicas (ej: por usuario, región, o entorno).
  - Quieres evitar configurar manualmente cada base de datos en `settings.py`.

---

### **5. Ejemplo práctico: Multi-tenant con subdominios**

#### **Paso 1: Configura el router**

```python
# mi_app/router_config.py
from dynamic_db_router import DynamicDbRouter

class TenantRouter(DynamicDbRouter):
    def get_db_name(self, request=None, **hints):
        if request:
            subdomain = request.get_host().split('.')[0]
            return f"tenant_{subdomain}"
        return "default"
```

#### **Paso 2: Define modelos comunes**

```python
# mi_app/models.py
from django.db import models

class Cliente(models.Model):
    nombre = models.CharField(max_length=100)

class Pedido(models.Model):
    cliente = models.ForeignKey(Cliente, on_delete=models.CASCADE)
    total = models.DecimalField(max_digits=10, decimal_places=2)
```

#### **Paso 3: Accede a datos según el subdominio**

```python
# Vista ejemplo
def dashboard(request):
    # Django usará automáticamente la base de datos del tenant actual
    clientes = Cliente.objects.all()  # SELECT * FROM clientes (en tenant_cliente1)
    return render(request, 'dashboard.html', {'clientes': clientes})
```

#### **Resultado:**

- `cliente1.miapp.com`: Datos en `tenant_cliente1`.
- `cliente2.miapp.com`: Datos en `tenant_cliente2`.

---

### **6. ¿Por qué esta librería es útil?**

- **Abstrae complejidad**: Evita tener que gestionar conexiones manuales a múltiples bases de datos.
- **Dinamismo**: Permite que la lógica de ruteo dependa de cualquier factor externo (ej: API keys, headers, usuarios).
- **Escalabilidad**: Ideal para SaaS con miles de clientes.

---

### **Conclusión**

- **Django's routers** son perfectos para casos simples con bases de datos estáticas.
- **`django-dynamic-db-router`** es esencial para arquitecturas avanzadas como multi-tenant, donde la base de datos depende del contexto de la petición.

Si estás construyendo una aplicación donde cada cliente necesita su propia base de datos (como en SaaS), esta librería te ahorrará horas de código personalizado y problemas de gestión.

# Cache

### Diferencias entre **Memcached** y **Redis** en Django (y en general):

Ambos son sistemas de caché en memoria, pero tienen diferencias clave en funcionalidad, flexibilidad y uso. Aquí te explico las diferencias más relevantes:

---

### 1. **Modelo de datos**

- **Memcached**:
  
  - Solo almacena **pares clave-valor simples** (strings).
  - No admite estructuras de datos complejas (listas, hashes, sets, etc.).
  - Es ideal para casos de uso sencillos: cachear resultados de consultas, sesiones, etc.

- **Redis**:
  
  - Almacena **tipos de datos avanzados**: strings, hashes, lists, sets, sorted sets, bitmaps, hyperloglogs, etc.
  - Permite operaciones complejas (ej.: incrementar un contador, agregar elementos a una lista, etc.).
  - Es más versátil para casos de uso avanzados (ej.: colas de mensajes, contadores en tiempo real, pub/sub).

---

### 2. **Persistencia**

- **Memcached**:
  
  - **No persiste datos en disco**. Todo se almacena en memoria RAM, por lo que se pierde al reiniciar.
  - Ideal para cachés temporales.

- **Redis**:
  
  - **Puede persistir datos en disco** (opcional).
  - Soporta dos modos de persistencia:
    - **RDB**: Volcado periódico de datos en memoria a disco.
    - **AOF**: Registro de todas las operaciones en un log (más seguro, pero consume más espacio).

---

### 3. **Límites en las claves (keys)**

Aquí entra tu pregunta sobre el error de formato de la clave. Ambos tienen límites, pero Redis es más flexible:

#### **Memcached**

- **Longitud máxima de la clave**: 250 bytes (aproximadamente 250 caracteres).

- **Caracteres permitidos**: No permite espacios, newlines, carriage returns ni otros caracteres de control.

- **Ejemplo de clave inválida**:
  
  ```python
  cache.get("usuario:123 perfil")  # Error por el espacio
  ```

#### **Redis**

- **Longitud máxima de la clave**: 512 MB (prácticamente ilimitada).

- **Caracteres permitidos**: Es **binario-safe**, acepta cualquier carácter (incluyendo espacios, emojis, etc.).

- **Ejemplo de clave válida**:
  
  ```python
  cache.get("usuario:123 perfil")  # Funciona en Redis
  ```

---

### 4. **Django y el formato de las claves**

En Django, aunque uses Redis, el **framework aplica validaciones adicionales** a las claves de caché para evitar problemas con backends menos flexibles (como Memcached). Por ejemplo:

- Django **sanitiza automáticamente las claves** usando `make_key`.
- Si una clave excede el límite de Memcached (250 bytes), Django la **trunca o lanza un error** (según configuración).

#### Ejemplo:

```python
from django.core.cache import cache

# Si usas Memcached, esto podría fallar por el espacio
cache.get("usuario:123 perfil")

# Django lo transforma internamente a algo como:
# "usuario:123_perfil" (reemplaza espacios por "_")
```

---

### 5. **Errores comunes en claves**

El error que mencionas probablemente sea causado por:

- Usar **caracteres no permitidos** (ej.: control characters).
- Sobrepasar el límite de longitud (especialmente en Memcached).
- Usar objetos no serializables como claves (ej.: `cache.get(objeto)` sin convertirlo a string).

#### Solución:

- Usa `make_key` para sanitizar claves:
  
  ```python
  from django.utils.cache import get_cache_key
  from django.http import HttpRequest
  
  request = HttpRequest()
  cache_key = get_cache_key(request)  # Genera una clave segura
  ```

- Valida manualmente la longitud:
  
  ```python
  if len(cache_key) > 250:
      raise ValueError("Clave demasiado larga para Memcached")
  ```

---

### 6. **Rendimiento y escalabilidad**

- **Memcached**:
  
  - Más rápido para operaciones simples.
  - Escala horizontalmente mediante múltiples nodos (no comparte estado entre ellos).
  - Ideal para cachés distribuidos en grandes aplicaciones web.

- **Redis**:
  
  - Ligeramente más lento en operaciones simples, pero compensa con funcionalidad avanzada.
  - Soporta replicación y clústeres para alta disponibilidad.
  - Ideal para casos donde necesitas atomicidad, transacciones o estructuras de datos complejas.

---

### 7. **Uso en Django**

En Django, ambos se configuran como backends de caché en `settings.py`:

#### **Memcached**:

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',
    }
}
```

#### **Redis**:

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/0',
    }
}
```

---

### ¿Cuál elegir?

- **Memcached**: Si necesitas velocidad extrema y solo usas caché simple.
- **Redis**: Si necesitas persistencia, estructuras de datos avanzadas o funcionalidades como pub/sub, locks, etc.

---

# 

# through

El atributo `through` en un campo `ManyToManyField` de Django **se utiliza para especificar un modelo intermedio personalizado** que gestiona la relación muchos-a-muchos, en lugar del modelo intermedio automático que crea Django por defecto. Esto permite agregar **campos adicionales** o **lógica personalizada** a la relación entre dos modelos.

---

### **¿Cuándo usar `through`?**

Usas `through` cuando necesitas **almacenar información extra sobre la relación** entre los objetos relacionados. Por ejemplo:

- Estado de la relación (activa, pendiente, etc.).
- Fechas de inicio/fin de la relación.
- Metadatos específicos de la conexión (por ejemplo, comisiones, prioridad, etc.).

---

### **Ejemplo práctico**

En tu código:

```python
programs = models.ManyToManyField(
    AffiliateProgram, 
    verbose_name=_("Affiliate programs"), 
    through="AffiliateMembership"
)
```

Esto indica que la relación entre tu modelo y `AffiliateProgram` se gestionará a través del modelo `AffiliateMembership`, que tú defines. 

---

### **Definición del modelo `AffiliateMembership`**

El modelo intermedio debe tener **claves foráneas (ForeignKey)** a ambos modelos involucrados en la relación. Por ejemplo:

```python
class AffiliateMembership(models.Model):
    main_model = models.ForeignKey("TuModeloPrincipal", on_delete=models.CASCADE)
    affiliate_program = models.ForeignKey("AffiliateProgram", on_delete=models.CASCADE)

    # Campos adicionales
    joined_at = models.DateTimeField(auto_now_add=True)
    status = models.CharField(max_length=20, choices=[("active", "Active"), ("inactive", "Inactive")])
    commission_rate = models.DecimalField(max_digits=5, decimal_places=2)
```

---

### **Ventajas de usar `through`**

1. **Datos adicionales en la relación**:
   - Puedes almacenar información específica de cada conexión entre objetos.
2. **Control completo**:
   - Puedes validar, filtrar o modificar la relación según los campos extras.
3. **Flexibilidad**:
   - Permite relaciones más complejas (ej: relaciones con metadatos dinámicos).

---

### **Cómo trabajar con `through`**

- **Crear una relación**:
  
  ```python
  membership = AffiliateMembership.objects.create(
      main_model=mi_objeto,
      affiliate_program=programa,
      status="active",
      commission_rate=5.5
  )
  ```

- **Acceder a las relaciones**:
  
  ```python
  for membership in mi_objeto.affiliate_membership_set.all():
      print(membership.affiliate_program, membership.status)
  ```

- **Filtrar relaciones**:
  
  ```python
  active_programs = mi_objeto.programs.filter(affiliate_membership__status="active")
  ```

---

### **Notas importantes**

- **No uses `add()` o `remove()` directamente**:
  Si usas un modelo intermedio personalizado, debes crear y eliminar instancias de `AffiliateMembership` manualmente. Métodos como `add()` o `set()` no funcionan automáticamente.

- **Especifica `through_fields` si hay ambigüedad**:
  Si el modelo intermedio tiene múltiples claves foráneas a los mismos modelos, debes indicar explícitamente cuáles usar:
  
  ```python
  programs = models.ManyToManyField(
      AffiliateProgram, 
      through="AffiliateMembership",
      through_fields=("main_model", "affiliate_program")
  )
  ```

---

### **Resumen**

- **`through`**: Define un modelo personalizado para la relación muchos-a-muchos.
- **Usa `through` cuando necesitas campos extra o lógica específica en la relación**.
- **El modelo intermedio debe tener ForeignKey a ambos modelos relacionados**.
- **Evita métodos como `add()` y trabaja directamente con el modelo intermedio**.

Con esto, puedes gestionar relaciones complejas en Django de manera flexible y con metadatos adicionales. 🚀

---------

Cuando usas un modelo intermedio personalizado (`through`) en una relación `ManyToManyField`, **Django no permite usar `filter_horizontal` o `filter_vertical`** directamente porque estos widgets están diseñados para relaciones simples sin campos adicionales. Sin embargo, hay alternativas que puedes implementar para mantener una interfaz de administración usable.

---

### **¿Por qué no funciona `filter_horizontal` con `through`?**

- `filter_horizontal` genera un widget de selección múltiple con búsqueda y filtros, pero **solo funciona para relaciones `ManyToManyField` sin modelo intermedio**.
- Al usar `through`, Django requiere que manejes la relación a través del modelo intermedio, lo que inhabilita automáticamente los widgets predeterminados.

---

### **Alternativa 1: Usar `TabularInline` o `StackedInline`**

La solución más común es **mostrar el modelo intermedio como un inline en el admin**. Esto permite editar las relaciones junto con sus campos adicionales.

#### **Paso 1: Define el inline**

```python
# admin.py
from django.contrib import admin
from .models import TuModeloPrincipal, AffiliateMembership

class AffiliateMembershipInline(admin.TabularInline):  # Puedes usar StackedInline si prefieres
    model = AffiliateMembership
    extra = 1  # Número de filas vacías para agregar nuevas relaciones

@admin.register(TuModeloPrincipal)
class TuModeloAdmin(admin.ModelAdmin):
    inlines = [AffiliateMembershipInline]
```

#### **Resultado**

- En el admin de `TuModeloPrincipal`, verás una tabla donde puedes:
  - Seleccionar `AffiliateProgram`.
  - Editar campos adicionales (ej: `status`, `commission_rate`).
  - Agregar/eliminar relaciones.

#### **Ventajas**

- Permite editar campos del modelo intermedio.
- Compatible con cualquier configuración de `through`.

#### **Desventajas**

- No ofrece el widget de búsqueda dual de `filter_horizontal`.

---

### **Alternativa 2: Crear un campo proxy temporal en el admin**

Si insistes en usar `filter_horizontal`, puedes **crear un campo temporal en el admin** que ignore el modelo intermedio, pero esto implica un manejo manual de datos.

#### **Paso 1: Agrega un campo temporal en el admin**

```python
# admin.py
from django import forms
from django.contrib import admin
from .models import TuModeloPrincipal, AffiliateProgram, AffiliateMembership

class TuModeloForm(forms.ModelForm):
    programs_proxy = forms.ModelMultipleChoiceField(
        queryset=AffiliateProgram.objects.all(),
        required=False,
        widget=admin.widgets.FilteredSelectMultiple("Affiliate Programs", is_stacked=False)
    )

    class Meta:
        model = TuModeloPrincipal
        fields = '__all__'

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        if self.instance.pk:
            self.initial['programs_proxy'] = self.instance.programs.values_list('id', flat=True)

    def save(self, commit=True):
        instance = super().save(commit=False)
        if commit:
            instance.save()
        # Actualiza las relaciones
        instance.programs.clear()
        for program in self.cleaned_data['programs_proxy']:
            AffiliateMembership.objects.get_or_create(
                main_model=instance,
                affiliate_program=program
            )
        return instance

@admin.register(TuModeloPrincipal)
class TuModeloAdmin(admin.ModelAdmin):
    form = TuModeloForm
    filter_horizontal = ['programs_proxy']  # Usa el campo proxy
```

#### **Ventajas**

- Mantiene el widget de `filter_horizontal`.
- Útil para relaciones simples donde no necesitas editar campos del modelo intermedio.

#### **Desventajas**

- Requiere manejar manualmente la sincronización con el modelo intermedio.
- Puede causar inconsistencias si otros campos del modelo intermedio (ej: `status`, `commission_rate`) no se gestionan.

---

### **Alternativa 3: Usar un widget personalizado (avanzado)**

Si necesitas un widget similar a `filter_horizontal` pero con soporte para campos adicionales, puedes crear uno personalizado con JavaScript y Django Admin Actions.

#### **Paso 1: Extender el template del admin**

1. Crea un archivo `templates/admin/change_form.html` en tu app.
2. Inyecta JavaScript para manejar la lógica de selección múltiple y campos adicionales.

```html
<!-- templates/admin/change_form.html -->
{% extends "admin/change_form.html" %}

{% block extrahead %}
{{ block.super }}
<script>
// Lógica personalizada para manejar relaciones con campos adicionales
</script>
{% endblock %}
```

#### **Paso 2: Usar Django Grappelli o第三方 widgets**

Considera usar [Django Grappelli](https://django-grappelli.readthedocs.io/) o [Django Suit](https://github.com/darklow/django-suit), que ofrecen widgets avanzados para relaciones complejas.

---

### **Conclusión**

- **Mejor opción**: Usa `TabularInline` para mostrar el modelo intermedio directamente en el admin. Es la solución más robusta y compatible.
- **Si necesitas `filter_horizontal`**: Crea un campo proxy temporal, pero ten cuidado con la sincronización de datos.
- **Para casos avanzados**: Personaliza el widget del admin con JavaScript o usa paquetes de terceros.

Con estas alternativas, podrás gestionar relaciones `ManyToManyField` con modelos intermedios sin perder usabilidad en el panel de administración. 🛠️







# readonly Forms



En un `ModelForm` de Django, si deseas marcar ciertos campos como solo de lectura (readonly), puedes hacerlo de varias maneras, ya que Django no tiene un atributo `readonly` directamente en los campos del formulario. Sin embargo, una práctica común es ajustar el widget de esos campos para que sean solo de lectura o manipular su estado en el método `__init__` del formulario.

Aquí te muestro algunas formas de hacerlo:

### 1. Usar `attrs` en el widget para marcar los campos como readonly en el HTML

En el método `__init__` del formulario, puedes modificar los widgets de los campos que quieres que sean solo lectura, agregando el atributo `readonly` en sus atributos HTML:

```python
from django import forms
from .models import MiModelo

class MiModelForm(forms.ModelForm):
    class Meta:
        model = MiModelo
        fields = ['campo1', 'campo2', 'campo3']

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Lista de campos que quieres que sean solo lectura
        readonly_fields = ['campo2', 'campo3']
        for field_name in readonly_fields:
            if field_name in self.fields:
                self.fields[field_name].widget.attrs['readonly'] = True
                # Opcional: evitar que el campo se edite mediante JavaScript
                self.fields[field_name].widget.attrs['disabled'] = True
```

**Nota:** Si usas `disabled=True`, el campo no será enviado en la petición POST, por lo que si necesitas que el valor sea enviado, usa solo `readonly`.

### 2. Para que el usuario no pueda editar el campo, pero el valor siga siendo enviado

- Añade el atributo `readonly` en la widget.
- No pongas `disabled`, ya que esto impide que el valor sea enviado en el formulario.

```python
self.fields[field_name].widget.attrs['readonly'] = True
```

### 3. Alternativa: Sobrescribir el método `save()` para ignorar cambios en esos campos

Otra opción es mantener los campos en modo editable, pero en `save()` restaurar sus valores al momento de guardar, evitando que el usuario los modifique:

```python
class MiModelForm(forms.ModelForm):
    class Meta:
        model = MiModelo
        fields = ['campo1', 'campo2', 'campo3']

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._initial_campo2 = self.instance.campo2
        self._initial_campo3 = self.instance.campo3

    def save(self, commit=True):
        # Restaurar los valores originales en los campos readonly
        self.instance.campo2 = self._initial_campo2
        self.instance.campo3 = self._initial_campo3
        return super().save(commit=commit)
```

---

### Resumen

- Para marcar campos como solo lectura en el formulario, modifica los widgets en `__init__` agregando `attrs['readonly'] = True`.
- Si quieres que el campo no sea editable y no se envíe su valor, usa `disabled=True` (aunque esto puede tener implicaciones en la recepción del valor en `POST`).
- La opción más común y sencilla es modificar los atributos del widget en `__init__`.
