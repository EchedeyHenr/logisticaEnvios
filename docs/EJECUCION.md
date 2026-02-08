## 📋 Requisitos del Sistema

### Requisitos Obligatorios
- **Python 3.10 o superior** (recomendado 3.11+)
- **Git** (para clonar el repositorio)
- **Terminal/Consola** con soporte UTF-8

### Verificar Versión de Python
```bash
python --version
```
Debe mostrar: Python 3.10.x o superior

### Requisitos Opcionales (para desarrollo)
- **IDE con soporte Python** (VSCode, PyCharm, etc.)
- **Entorno Virtual** (venv, conda, pipenv) - recomendado

## ⌨️ Ejecutar la aplicación

### 1. Clonar repositorio
```bash
git clone https://github.com/EchedeyHenr/logistica.git
cd logistica
```

### 2. Ejecutar Directamente (sin instalación)

Desde la raíz del proyecto:
```bash
python -m logistica.presentation.menu
```

### 3. Estructura de Comandos Alternativa

```bash
# Si estás dentro de la carpeta logistica/
python -m presentation.menu

# Usando el módulo principal
python -m logistica
```

## ⚡ Flujo rápido de ejemplo

### Ejemplo 1: Ciclo Completo de un Envío

1. Registrar centro (opción 9)
   - ID: VAL-01
   - Nombre: Valencia Norte
   - Ubicación: Calle Ejemplo 123

2. Registrar envío (opción 1)
   - Código: TEST001
   - Remitente: Empresa Ejemplo
   - Destinatario: Cliente Final
   - Prioridad: 2
   - Tipo: standard

3. Crear ruta (opción 12)
   - ID: RUTA-01
   - Origen: VAL-01
   - Destino: MAD-16 (ya existe en datos iniciales)

4. Asignar envío a ruta (opción 2)
   - Código: TEST001
   - Ruta: RUTA-01

5. Despachar ruta (opción 15)
   - Ruta: RUTA-01

6. Completar ruta (opción 16)
   - Ruta: RUTA-01

7. Verificar envío entregado (opción 8)
   - Código: TEST001

### Ejemplo 2: Gestión de Prioridades

1. Registrar envío frágil (opción 1)
   - Tipo: fragile
   - Prioridad: 2 (mínimo permitido)

2. Aumentar prioridad (opción 5)
   - Código: [código del envío]

3. Disminuir prioridad (opción 6)
   - Nota: No podrá bajar de 2 (regla de negocio)

## ⚠️ Errores comunes y Soluciones

### Error: "Módulo no encontrado"

``` 
ModuleNotFoundError: No module named 'logistica'
```

**Solución:** Asegúrate de estar en el directorio correcto:

```
# Estructura correcta:
# /ruta/al/proyecto/logistica/
#                           ├── presentation/
#                           ├── application/
#                           └── ...

# Ejecutar desde un nivel arriba:
cd /ruta/al/proyecto
python -m logistica.presentation.menu
```

### Error: "Ya existe un envío con el código..."

```
ValueError: Ya existe un envío con el código de seguimiento 'ABC123'
```

**Solución:** Los códigos de seguimiento deben ser únicos. Usa uno diferente o consulta los existentes con la opción 7.

### Error: "Transición no permitida"

```
ValueError: Transición no permitida: de REGISTERED a DELIVERED
```

**Solución:** Los estados deben seguir la secuencia:

1. REGISTERED → (asignar a ruta)
2. IN_TRANSIT → (despachar ruta)
3. DELIVERED → (completar ruta)

### Error: "La ruta no está activa"

```
ValueError: La ruta 'RUTA-X' no está activa.
```

**Solución:** Una ruta se marca como inactiva al completarse. Crea una nueva ruta o usa una activa.

### Error: "No se puede aumentar/disminuir la prioridad"

```
ValueError: No se puede aumentar la prioridad del envío.
```

**Solución:**

* **Envíos Express:** Prioridad fija en 3 (no se puede cambiar)
* **Envíos Estándar:** Rango 1-3
* **Envíos Frágiles:** Rango 2-3 (no pueden bajar de 2)