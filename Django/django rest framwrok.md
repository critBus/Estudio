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
