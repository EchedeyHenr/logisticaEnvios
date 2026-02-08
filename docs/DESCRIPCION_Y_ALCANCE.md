# Descripción y Alcance

## 📖 Descripción funcional

El **Sistema de Gestión de Expediciones y Envíos** es una aplicación de consola que simula la operación de una red logística completa. Permite a los operadores gestionar el ciclo de vida completo de los envíos, desde su registro hasta la entrega final, pasando por centros logísticos y rutas de transporte.

Los usuarios pueden:
- Registrar tres tipos de envíos (Estándar, Frágil y Express) con diferentes reglas de prioridad
- Gestionar centros logísticos y su inventario
- Crear y administrar rutas de transporte entre centros
- Asignar envíos a rutas y seguir su estado en tiempo real
- Consultar la trazabilidad completa de cada envío

## 🎯 Objetivos de la fase

### Objetivo General
Implementar un sistema modular y extensible para la gestión coherente del flujo logístico, con separación clara de responsabilidades mediante arquitectura por capas.

### Objetivos Específicos
1. **Modelado del Dominio**: Crear entidades que representen fielmente los conceptos logísticos (Envíos, Centros, Rutas)
2. **Polimorfismo en Envíos**: Implementar jerarquía de clases para manejar diferentes tipos de envíos con comportamientos específicos
3. **Gestión de Estados**: Controlar el ciclo de vida de los envíos con transiciones válidas y trazabilidad
4. **Separación de Capas**: Aplicar arquitectura limpia (Presentation, Application, Domain, Infrastructure)
5. **Validación de Negocio**: Implementar reglas que prevengan operaciones incoherentes

## 🔭 Alcance

### ✅ Incluye

| Módulo | Funcionalidades |
|--------|----------------|
| **Gestión de Envíos** | Registro, consulta, actualización de estado, gestión de prioridades |
| **Tipos de Envíos** | Estándar, Frágil (prioridad ≥2), Express (prioridad fija 3) |
| **Centros Logísticos** | Registro, inventario, recepción y despacho de paquetes |
| **Rutas de Transporte** | Creación, asignación de envíos, despacho, finalización |
| **Estado y Trazabilidad** | Historial de estados, validación de transiciones |
| **Persistencia** | Repositorios en memoria con datos iniciales |
| **Interfaz** | Menú de consola con todas las operaciones principales |

### ❌ No Incluye

| Ámbito | Limitación |
|--------|------------|
| **Persistencia Permanente** | No hay base de datos real (solo memoria) |
| **Interfaz Gráfica** | Solo interfaz de consola |
| **APIs Externas** | Sin integración con mapas, GPS o sistemas de pago |
| **Gestión de Flota** | No se modelan vehículos, conductores o combustible |
| **Usuarios y Permisos** | No hay sistema de autenticación o roles |
| **Reportes Avanzados** | Solo listados básicos, sin gráficos o estadísticas |

## 🚧 Supuestos y límites de la fase

### Supuestos Técnicos
- **Python 3.10+**: El proyecto utiliza características de Python 3.10 o superior
- **Sin Dependencias Externas**: Solo biblioteca estándar de Python
- **Case-Insensitive**: Los identificadores (códigos, IDs) no distinguen mayúsculas/minúsculas
- **Memoria Volátil**: Los datos se pierden al cerrar la aplicación

### Límites Operacionales
1. **Prioridades**: Sistema de 3 niveles (1=baja, 2=media, 3=alta)
2. **Estados de Envío**: Solo 3 estados posibles (REGISTERED → IN_TRANSIT → DELIVERED)
3. **Capacidad de Centros**: Sin límite de capacidad física
4. **Rutas Simples**: Cada ruta tiene un único origen y destino
5. **Tiempo Real**: No hay conceptos de tiempo, fechas o plazos

### Decisiones de Diseño
1. **Inmutabilidad**: Algunas propiedades (ID, nombre) son de solo lectura después de la creación
2. **Validación Temprana**: Las reglas de negocio se validan al momento de la operación
3. **Separación Estricta**: Las capas solo se comunican mediante interfaces definidas
4. **Polimorfismo Controlado**: Los tipos de envío especializados heredan de la clase base Shipment

---

**Nota**: Este alcance está diseñado para mantener la complejidad manejable mientras se cubren los conceptos principales de diseño por capas y dominio rico.