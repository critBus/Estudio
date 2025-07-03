# **walrus operator**

Sí, el símbolo `:=` en Python se llama **operador de asignación de expresión** (también conocido como **"walrus operator"** debido a su forma `:=`). Su propósito principal es **asignar un valor a una variable dentro de una expresión**, especialmente útil en condicionales o bucles, evitando repetir código o hacer llamadas redundantes.

---

### Ejemplo práctico:

```python
if existing_user := CustomUser.objects.filter(email=user_email).first():
    print(f"Usuario encontrado: {existing_user.name}")
```

#### ¿Qué hace?

1. **Asigna el resultado de `CustomUser.objects.filter(...).first()`** a la variable `existing_user`.
2. **Evalúa si `existing_user` es verdadero** (es decir, si no es `None` o un objeto vacío).
3. Si es verdadero, entra al bloque `if` y puedes usar `existing_user` directamente.

---

### Comparación con código anterior a Python 3.8:

Antes de Python 3.8, tenías que escribir:

```python
existing_user = CustomUser.objects.filter(email=user_email).first()
if existing_user:
    print(f"Usuario encontrado: {existing_user.name}")
```

Con el operador `:=`, reduces líneas y evitas repetir la llamada a `.first()`.

---

### Casos de uso comunes:

1. **Filtrar y usar un valor en una sola línea**:
   
   ```python
   if (match := re.search(r"\d+", text)):
       print(f"Número encontrado: {match.group()}")
   ```

2. **Bucles con condiciones dinámicas**:
   
   ```python
   while (line := input("Escribe algo (o 'salir'): ")) != "salir":
       print(f"Escribiste: {line}")
   ```

3. **Evitar llamadas redundantes**:
   
   ```python
   # Sin walrus operator
   data = get_data()
   if data:
       process(data)
   
   # Con walrus operator
   if data := get_data():
       process(data)
   ```

---

### Ventajas:

- **Menos código**: Reduce la necesidad de declarar variables antes del condicional.
- **Eficiencia**: Evita calcular el mismo valor múltiples veces.
- **Legibilidad**: En casos simples, mejora la claridad del código.

---

### Consideraciones:

- **Requiere Python 3.8+**: No está disponible en versiones anteriores.
- **No abuses de él**: Si el código se vuelve complejo, es mejor separar asignación y condición para mantener la legibilidad.
- **Solo en expresiones**: No puedes usarlo en asignaciones globales o en contextos donde no se permiten expresiones.

---

### Ejemplo avanzado con listas:

```python
# Filtrar números positivos y mostrar los que pasan el filtro
numbers = [1, -2, 3, -4, 5]
positive = [n for num in numbers if (n := num) > 0]
print(positive)  # Output: [1, 3, 5]
```

---

### Resumen:

- **`:=`** asigna y devuelve un valor en la misma línea.
- Ideal para casos donde necesitas **reutilizar el resultado de una expresión** en una condición o bucle.
- Mejora la eficiencia y reduce código repetido, pero **usa con moderación** para no sacrificar la claridad.
