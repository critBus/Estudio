# invalidate_model

El método (en realidad, una función) `invalidate_model` del paquete **`cacheops`** en Django se utiliza para **invalidar manualmente todas las cachés asociadas a un modelo específico** cuando hay cambios en sus datos que *no son detectados automáticamente* por el sistema de caché. Aquí te explico su propósito y uso:

---

### **¿Qué hace `invalidate_model`?**

- **Limpia todas las cachés** relacionadas con un modelo de Django que hayan sido creadas mediante `cacheops` (por ejemplo, al usar `MyModel.objects.cache().filter(...)`).
- Es útil cuando los datos del modelo son modificados **fuera del flujo normal del ORM de Django**, como en:
  - Operaciones masivas con `.update()` o `.delete()`.
  - Consultas SQL crudas (`raw()` o `connection.cursor()`).
  - Cambios desde otro proceso o servicio (ej: una tarea asíncrona con Celery).
  - Actualizaciones en la base de datos directamente (sin pasar por Django).

---

### **¿Por qué es necesario?**

`cacheops` invalida automáticamente la caché cuando usas métodos como `.save()` o `.delete()` en instancias de un modelo, gracias a las señales de Django. **Pero no detecta cambios en operaciones masivas o SQL crudo**, lo que puede causar que la caché muestre datos desactualizados. `invalidate_model` soluciona esto.

---

### **Ejemplo de uso**

```python
from cacheops import invalidate_model
from myapp.models import Producto

# Supongamos una actualización masiva que no activa señales de Django:
Producto.objects.filter(precio__lt=10).update(descuento=0.1)

# ¡La caché de Producto aún tiene datos antiguos!
# Forzamos la invalidación manual:
invalidate_model(Producto)  # Limpia TODAS las cachés del modelo Producto
```

---

### **Casos comunes donde se usa**

1. **Operaciones masivas**:
   
   ```python
   User.objects.all().update(es_activo=False)
   invalidate_model(User)
   ```

2. **SQL crudo**:
   
   ```python
   from django.db import connection
   with connection.cursor() as cursor:
       cursor.execute("UPDATE tabla SET campo = 'valor' WHERE condicion")
   invalidate_model(MiModelo)
   ```

3. **Tareas asíncronas** (Celery, RQ):
   
   ```python
   @task
   def actualizar_datos():
       # ... lógica que modifica datos sin usar el ORM ...
       invalidate_model(ModeloCritico)
   ```

---

### **Alternativas en `cacheops`**

- `invalidate_obj(obj)`: Invalida solo la caché de **una instancia específica** (útil para cambios puntuales).
- `invalidate_all()`: Invalida **toda la caché de la aplicación** (usar con cuidado, es muy agresivo).

---

### **¡Importante!**

- **No abuses de `invalidate_model`**: Invalidar cachés constantemente reduce los beneficios del caché (mayor carga en la BD). Úsalo solo cuando sea estrictamente necesario.
- **Prefiere el flujo automático**: Siempre que sea posible, usa métodos del ORM que activen señales (`.save()`, `.delete()`) para que `cacheops` invalide automáticamente.

---

### **Cómo funciona internamente**

`cacheops` almacena las cachés usando claves basadas en:

- El modelo afectado.
- Los campos consultados.
- Los filtros aplicados.

Cuando llamas a `invalidate_model(MiModelo)`, elimina todas las entradas de caché cuya clave esté asociada a ese modelo, asegurando que las próximas consultas obtengan datos frescos.

---

### **Ejemplo en un proyecto real**

```python
# views.py
from cacheops import invalidate_model
from .models import Reporte

def generar_reporte(request):
    # Lógica que actualiza datos masivamente
    Reporte.objects.raw("CALL generar_reporte_mensual()")  # Procedimiento almacenado
    invalidate_model(Reporte)  # ¡Sin esto, la caché seguiría mostrando datos viejos!
    return render(request, "exito.html")
```

---

### **Conclusión**

`invalidate_model` es una **herramienta crítica para mantener la coherencia de datos** cuando trabajas con cachés en Django y realizas operaciones que escapan al ORM. Si ves este método en tu proyecto, es probable que el equipo esté manejando escenarios de alto rendimiento donde se prioriza la consistencia frente a optimizaciones agresivas de caché. 

¿Trabajas con operaciones masivas o SQL crudo en tu proyecto? Ese es el contexto típico donde aparece esta función. 🚀



# throttle_classes

En Django REST Framework (DRF), el atributo `throttle_classes` en un **ViewSet** se utiliza para aplicar **mecanismos de limitación de tasa (throttling)** a las solicitudes entrantes. Específicamente, `throttle_classes = (AnonRateThrottle,)` configura una política de throttling que **limita la cantidad de solicitudes que pueden hacer los usuarios no autenticados (anónimos)** en un período de tiempo determinado.

---

### ¿Qué hace `AnonRateThrottle`?

- **Objetivo**: Proteger tu API contra abusos de usuarios no autenticados (ej: bots, scrapers, ataques de fuerza bruta).
- **Cómo funciona**:
  1. **Identifica usuarios anónimos** usando su dirección IP.
  2. **Cuenta las solicitudes** por IP dentro de un intervalo de tiempo (ej: 100 solicitudes por hora).
  3. Si se excede el límite, devuelve un error `429 Too Many Requests`.

---

### Configuración necesaria en `settings.py`

Para que `AnonRateThrottle` funcione, debes definir el límite de tasa en la configuración de DRF:

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/hour',  # Límite para usuarios anónimos
        # 'user': '1000/hour', (Opcional: para usuarios autenticados con UserRateThrottle)
    }
}
```

- **`anon`**: Es la clave que DRF usa para asociar el límite con `AnonRateThrottle`.
- Formatos válidos: `'10/second'`, `'100/minute'`, `'1000/hour'`, `'5000/day'`.

---

### Ejemplo de uso en un ViewSet

```python
from rest_framework.throttling import AnonRateThrottle
from rest_framework import viewsets

class MiViewSet(viewsets.ModelViewSet):
    throttle_classes = [AnonRateThrottle]  # Lista o tupla de clases de throttling
    # ... otros atributos (queryset, serializer_class, etc.) ...
```

---

### Comportamiento en la práctica

1. **Usuario anónimo**:
   - Si hace más de 100 solicitudes/hora (según el ejemplo de configuración), recibirá:
     
     ```http
     HTTP 429 Too Many Requests
     {
         "detail": "Demasiadas solicitudes. Inténtalo de nuevo más tarde."
     }
     ```
2. **Usuario autenticado**:
   - **No se aplica este throttling** (a menos que también agregues `UserRateThrottle`).

---

### Casos de uso comunes

- Proteger endpoints públicos como **registro de usuarios**, **login**, o **listas de recursos públicas**.
- Evitar que scrapers descarguen grandes volúmenes de datos no autenticados.
- Mitigar ataques de denegación de servicio (DoS) básicos.

---

### Diferencias clave con otras clases de throttling

| Clase                | Aplica a                       | Criterio de identificación    |
| -------------------- | ------------------------------ | ----------------------------- |
| `AnonRateThrottle`   | Solo usuarios **anónimos**     | Dirección IP                  |
| `UserRateThrottle`   | Solo usuarios **autenticados** | ID del usuario (token/cookie) |
| `ScopedRateThrottle` | Endpoints específicos          | Scope definido en el View     |

---

### Notas importantes

- **Prioridad**: Si defines `throttle_classes` en el ViewSet, **anula** la configuración global (`DEFAULT_THROTTLE_CLASSES` en `settings.py`).
- **Múltiples throttles**: Puedes combinar clases:
  
  ```python
  throttle_classes = [AnonRateThrottle, UserRateThrottle]
  ```
- **Personalización**: Para lógica específica (ej: límites por endpoint), usa `ScopedRateThrottle`:
  
  ```python
  throttle_scope = 'registro'  # Debe coincidir con una clave en DEFAULT_THROTTLE_RATES
  ```

---

### Documentación oficial

- [Throttling en DRF](https://www.django-rest-framework.org/api-guide/throttling/)
- [`AnonRateThrottle`](https://www.django-rest-framework.org/api-guide/throttling/#anonratethrottle)
