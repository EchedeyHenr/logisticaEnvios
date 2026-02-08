# 🛠️ Troubleshooting y Solución de Problemas

## 🚨 Errores Comunes

### Error 1: Módulo no encontrado
`ModuleNotFoundError: No module named 'logistica'`

#### Causas Posibles:
1. **Directorio incorrecto**: No estás en el directorio correcto para ejecutar
2. **PYTHONPATH no configurado**: Python no encuentra el módulo
3. **Estructura de carpetas alterada**: Falta algún `__init__.py`

#### Soluciones:
```bash
# Verificar estructura
ls -la
# Debe mostrar: logistica/ (carpeta)

# Navegar al directorio correcto
cd /ruta/completa/al/proyecto  # Un nivel arriba de logistica/

# Ejecutar correctamente
python -m logistica.presentation.menu

# Alternativa: configurar PYTHONPATH
export PYTHONPATH="/ruta/al/proyecto:$PYTHONPATH"
python -c "import logistica; print('OK')"
```

### Error 2: Atributo no encontrado
`AttributeError: 'Shipment' object has no attribute 'x'`

#### Causas Posibles:
1. **Typo en nombre de atributo**: shipment.priority vs shipment.prioridad 
2. **Atributo privado usado externamente**: _priority en lugar de priority 
3. **Versión incorrecta del código**: Código compilado vs fuente

#### Soluciones:
```bash
# Verificar nombres correctos (consultar documentación)
print(dir(shipment))  # Ver todos los atributos disponibles

# Usar propiedades públicas, no atributos privados
# ✅ CORRECTO
priority = shipment.priority  # Propiedad pública
# ❌ INCORRECTO  
priority = shipment._priority  # Atributo privado

# Verificar imports correctos
from logistica.domain.shipment import Shipment  # ✅
# from some_other_module import Shipment  # ❌
```

### Error 3: Error de valor (ValueError)
`ValueError: [mensaje específico]`

#### Mensajes comunes y soluciones:

| Mensaje de Error | Causa | Solución |
| :--- | :--- | :--- |
| **"Ya existe un envío con el código 'X'"** | Código duplicado | Usar código diferente o eliminar el existente |
| **"Transición no permitida: de X a Y"** | Secuencia de estados incorrecta | Seguir: REGISTERED → IN_TRANSIT → DELIVERED |
| **"La prioridad debe ser 1, 2 o 3"** | Prioridad fuera de rango | Usar 1, 2 o 3 únicamente |
| **"Un envío frágil no puede tener prioridad inferior a 2"** | Regla de negocio para frágiles | Usar prioridad 2 o 3 para envíos frágiles |
| **"El centro de origen y destino no pueden ser el mismo"** | Validación de coherencia | Usar centros diferentes para origen y destino |
| **"La ruta 'X' no está activa"** | Ruta completada o no existe | Usar ruta activa o crear nueva |
| **"No existe un centro con el identificador 'X'"** | Centro no registrado | Verificar ID o registrar centro primero |

### Error 4: Tipo incorrecto (TypeError)
`TypeError: [mensaje sobre tipos]`

#### Causas Comunes:
1. **Parámetro de tipo incorrecto**: Pasar string donde se espera número
2. **Método llamado con argumentos incorrectos**: Faltan o sobran argumentos
3. **Operación entre tipos incompatibles**: String + número sin conversión

#### Soluciones:
```python
# Verificar tipos esperados
help(Shipment.__init__)  # Mostrar firma del método

# Conversión explícita cuando sea necesario
priority = int(input("Prioridad: "))  # Convertir string a int

# Usar isinstance para verificar tipos
if not isinstance(shipment, Shipment):
    raise TypeError("Se esperaba un objeto Shipment")
```

---

## 🔧 Problemas de Ejecución

### Problema 1: La aplicación se cierra inesperadamente

#### Síntomas:
- La aplicación termina sin mensaje de error
- Menú desaparece después de cierta operación
- No hay stack trace visible

#### Soluciones:
```python
# Agregar manejo global de excepciones
if __name__ == "__main__":
    try:
        main()
    except Exception as e:
        print(f"ERROR NO MANEJADO: {e}")
        import traceback
        traceback.print_exc()
        input("Presiona Enter para salir...")

# O ejecutar con depuración
python -m pdb -m logistica.presentation.menu
```

### Problema 2: Comportamiento inconsistente entre ejecuciones

#### Síntomas:
- Datos diferentes cada vez que se ejecuta
- Estado que no persiste entre ejecuciones
- Operaciones que a veces funcionan y a veces no

#### Causa y Solución:
**Causa**: Los repositorios en memoria se reinician en cada ejecución.

```python
# Comportamiento esperado: datos se pierden al cerrar
# Si necesitas persistencia, implementa repositorios persistentes

# Solución temporal para desarrollo:
def save_state(repositories, filename="state.pkl"):
    import pickle
    with open(filename, "wb") as f:
        pickle.dump(repositories, f)

def load_state(filename="state.pkl"):
    import pickle
    with open(filename, "rb") as f:
        return pickle.load(f)
```

### Problema 3: Performance lenta con muchos datos

#### Síntomas:
- Operaciones lentas con cientos de envíos
- Tiempo de respuesta alto en listados
- Consumo alto de memoria

#### Soluciones:
```python
# Optimizar repositorios en memoria
class OptimizedShipmentRepositoryMemory(ShipmentRepository):
    def __init__(self):
        self._by_tracking_code = {}
        self._by_status = {}  # Índice secundario por estado
        self._by_priority = {}  # Índice secundario por prioridad
    
    def list_by_status(self, status):
        # Más rápido usando índice
        return list(self._by_status.get(status, {}).values())
    
    def add(self, shipment):
        key = shipment.tracking_code.lower()
        self._by_tracking_code[key] = shipment
        # Mantener índices actualizados
        self._by_status.setdefault(shipment.current_status, {})[key] = shipment
        self._by_priority.setdefault(shipment.priority, {})[key] = shipment
```

---

## 🐛 Problemas de Lógica de Negocio

### Problema 1: Estado inconsistente después de error

#### Síntomas:
- Operación falla pero deja datos en estado intermedio
- Relaciones bidireccionales desincronizadas
- Invariantes del dominio violadas

#### Ejemplo:
```python
# ❌ PROBLEMA: Si add_shipment falla después de assign_route
shipment.assign_route(route_id)  # Se ejecuta
route.add_shipment(shipment)     # Falla, pero shipment ya tiene ruta asignada

# ✅ SOLUCIÓN: Patrón de rollback o transacción
try:
    route.add_shipment(shipment)
    shipment.assign_route(route_id)
except Exception:
    # Rollback si algo falla
    if shipment.is_assigned_to_route():
        shipment.remove_route()
    raise
```

### Problema 2: Validaciones que no se ejecutan

#### Síntomas:
- Datos inválidos pasan las validaciones
- Estados imposibles en el sistema
- Invariantes violadas silenciosamente

#### Solución:
```python
# Agregar validaciones defensivas
class Shipment:
    def __init__(self, tracking_code, sender, recipient, priority):
        # Validar inmediatamente
        self._validate_attributes(tracking_code, sender, recipient, priority)
        
        self.__tracking_code = tracking_code
        self.__sender = sender
        self.__recipient = recipient
        self._priority = priority
        self._validate_invariants()  # Validar invariantes después de inicializar
    
    def _validate_invariants(self):
        """Verificar que todas las invariantes se cumplen."""
        assert self.priority in (1, 2, 3), f"Prioridad inválida: {self.priority}"
        assert self.current_status in ("REGISTERED", "IN_TRANSIT", "DELIVERED")
        # ... más validaciones
```

### Problema 3: Problemas con herencia y polimorfismo

#### Síntomas:
- Métodos que no se comportan como se espera en subclases
- Atributos privados no accesibles en clases hijas
- LSP (Liskov Substitution Principle) violado

#### Solución:
```python
# Diseñar jerarquía correctamente
class Shipment:
    def __init__(self, tracking_code, sender, recipient, priority):
        # Atributos privados con __
        self.__tracking_code = tracking_code
        
    @property
    def tracking_code(self):
        return self.__tracking_code

class FragileShipment(Shipment):
    def __init__(self, tracking_code, sender, recipient, priority=2):
        # Llamar al constructor padre
        super().__init__(tracking_code, sender, recipient, priority)
        # Validaciones específicas
        if priority < 2:
            raise ValueError("Prioridad mínima 2 para frágiles")
        
    # Sobrescribir/complementar comportamientos
    def decrease_priority(self):
        if self._priority <= 2:  # _priority es protected (accesible en subclases)
            raise ValueError("No puede bajar de 2")
        self._priority -= 1
```

---

## ✅ Checklist de Troubleshooting

### Antes de Reportar un Problema
- Ejecuté run_diagnostics()
- Revisé la documentación relevante 
- Verifiqué que no es error de usuario 
- Probé con datos mínimos para reproducir 
- Revisé logs/errores completos

### Para Problemas de Desarrollo
- Tests pasan localmente
- No hay imports circulares
- Dependencias inyectadas correctamente
- Atributos privados/protegidos usados correctamente

### Para Problemas de Producción (futuros)
- Logs disponibles y revisados
- Estado de la base de datos verificado
- Backup reciente disponible
- Rollback planificado si es necesario

