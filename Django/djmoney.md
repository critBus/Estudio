El método `convert_money` de `djmoney.contrib.exchange.models` en **django-money** **sí convierte entre dos monedas**, pero **depende de tasas de cambio predefinidas en tu base de datos**, no consulta tasas del mercado en tiempo real por sí mismo. Aquí está el detalle:

---

### **1. ¿Cómo funciona `convert_money`?**

```python
from djmoney.contrib.exchange.models import convert_money
from djmoney.money import Money

# Ejemplo: Convertir 10 USD a EUR
resultado = convert_money(Money(10, "USD"), "EUR")
```

- **Requiere tasas de cambio almacenadas**: El método busca en tu base de datos las tasas de conversión entre USD y EUR (u otras monedas) que hayas registrado previamente.
- **Usa un backend de conversión**: Por defecto, utiliza el backend configurado en `settings.py` (ej: `'djmoney.contrib.exchange.backends.FixerBackend'`), pero puedes especificar otro.

---

### **2. ¿Necesita tasas de mercado actualizadas?**

**¡Sí!** Pero **no las obtiene automáticamente**. Tú debes garantizar que las tasas en tu base de datos estén actualizadas. Por ejemplo:

- Si almacenas una tasa **1 USD = 0.93 EUR** (fija o desactualizada), la conversión usará ese valor, aunque en el mercado real sea diferente.
- **No consulta APIs externas por defecto**: A diferencia de servicios como Open Exchange Rates, `convert_money` no hace llamadas en tiempo real a menos que configures un backend personalizado.

---

### **3. ¿Cómo mantener las tasas actualizadas?**

Debes **integrar una fuente externa** y actualizar las tasas en tu base de datos periódicamente. Ejemplo:

```python
# Ejemplo con Fixer.io (requiere django-money汇率)
from djmoney.contrib.exchange.backends import FixerBackend

backend = FixerBackend()
backend.update_rates()  # Esto descarga tasas actuales y las guarda en la BD
```

Luego, `convert_money` usará las tasas recién actualizadas.

---

### **4. Configuración mínima necesaria**

En `settings.py`:

```python
# Habilitar el soporte para conversiones
INSTALLED_APPS += ["djmoney", "djmoney.contrib.exchange"]

# Configurar el backend (ej: Fixer.io)
EXCHANGE_BACKEND = "djmoney.contrib.exchange.backends.FixerBackend"
FIXER_ACCESS_KEY = "tu_api_key"  # Necesario para Fixer.io
```

---

### **Conclusión**

- **Sí convierte entre monedas**, pero **depende de tasas almacenadas en tu BD**.
- **No usa tasas de mercado en tiempo real** a menos que implementes un backend que las actualice (ej: llamando a una API externa periódicamente).
- **Tú eres responsable** de mantener las tasas actualizadas para que las conversiones sean precisas.


