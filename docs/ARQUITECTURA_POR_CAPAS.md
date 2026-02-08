<!-- En construcción
<p align="center">
  <strong>Choose language / Elige idioma</strong><br><br>
  <a href="#english" style="text-decoration: none; margin-right: 30px;">
    <img src="docs/images/uk_flag.png" alt="English" width="24" height="18" style="margin-right: 6px; vertical-align: middle;"> 
    <span style="vertical-align: middle;">English</span>
  </a>
  <a href="#español" style="text-decoration: none;">
    <img src="docs/images/spain_flag.png" alt="Español" width="24" height="18" style="margin-right: 6px; vertical-align: middle;"> 
    <span style="vertical-align: middle;">Español</span>
  </a>
</p>

<details open id="english">
<summary>🇬🇧 English</summary>


###  🏗️ Layers and responsibilities

* **Presentation**: Console-based user interface.
* **Application**: Use cases and orchestration of business logic.
* **Domain**: Business models and rules.
* **Infrastructure**: Technical persistence implementations.

<br />

### 🔗 Allowed Dependencies

* **Presentation $\rightarrow$ Application $\rightarrow$ Domain**
* **Infrastructure $\rightarrow$ Domain**

**Dependency Rules**

* The domain and application layers must not depend on the presentation layer. 
* Direct dependencies from domain or application to presentation are strictly prohibited

### 🗺️ Map files

<hr>

</details>
-->
<details open id="español">
<summary>🇪🇸 Español</summary>

### 🧱 Capas y responsabilidades

El proyecto sigue una arquitectura en capas:

* **Presentation**: Interfaz de usuario por consola.
* **Application**: Casos de uso y orquestación de la lógica.
* **Domain**: Modelos y reglas del negocio.
* **Infrastructure**: Implementaciones técnicas de persistencia.

Esta separación facilita la mantenibilidad, la escalabilidad y la evolución del sistema.

### 🔗 Dependencias permitidas

* **Presentation $\rightarrow$ Application $\rightarrow$ Domain**
* **infrastructure $\rightarrow$ domain**

No se debe depender de presentation desde domain ni application.

### Reglas Estrictas
1. **Presentation solo depende de Application** (nunca de Domain o Infrastructure directamente)
2. **Application depende de Domain** (entidades y contratos/interfaces)
3. **Domain NO depende de nadie** (es el núcleo independiente)
4. **Infrastructure depende de Domain** (implementa sus interfaces)

### Ejemplos Correctos
```python
# ✅ CORRECTO: Presentation → Application
from logistica.application.shipment_service import ShipmentService

# ✅ CORRECTO: Application → Domain
from logistica.domain.shipment import Shipment

# ✅ CORRECTO: Infrastructure → Domain
from logistica.domain.shipment_repository import ShipmentRepository

# ❌ INCORRECTO: Domain → Application (prohibido)
# ❌ INCORRECTO: Domain → Infrastructure (prohibido)
# ❌ INCORRECTO: Presentation → Domain (prohibido)
```

## 🗺️ Mapa de archivos

### 1. Capa Presentation (presentation/)

| Archivo | Responsabilidad | Dependencias |
| :--- | :--- | :--- |
| `menu.py` | Interfaz de usuario por consola | Application services |

### 2. Capa Application (application/)

| Archivo | Responsabilidad | Dependencias |
| :--- | :--- | :--- |
| `shipment_service.py` | Casos de uso de envíos | Domain entities, repositories |
| `route_service.py` | Gestión de rutas | Domain entities, repositories |
| `center_service.py` | Gestión de centros | Domain entities, repositories |

### 3. Capa Domain (domain/)

| Archivo | Responsabilidad | Tipo |
| :--- | :--- | :--- |
| `shipment.py` | Entidad base de envío | Entity |
| `fragile_shipment.py` | Envío frágil (prioridad $\ge 2$) | Entity |
| `express_shipment.py` | Envío express (prioridad fija 3) | Entity |
| `center.py` | Centro logístico y su inventario | Entity |
| `route.py` | Ruta entre centros | Entity |
| `shipment_repository.py` | Contrato para repositorios de envíos | Interface |
| `center_repository.py` | Contrato para repositorios de centros | Interface |
| `route_repository.py` | Contrato para repositorios de rutas | Interface |

### 4. Capa Infrastructure (infrastructure/)

| Archivo | Responsabilidad | Implementa |
| :--- | :--- | :--- |
| `memory_shipment.py` | Repositorio en memoria de envíos | ShipmentRepository |
| `memory_center.py` | Repositorio en memoria de centros | CenterRepository |
| `memory_route.py` | Repositorio en memoria de rutas | RouteRepository |
| `seed_data.py` | Datos iniciales para pruebas | - |

## 🎯 Responsabilidades por Capa

### Capa Presentation

* **Objetivo:** Comunicarse con el usuario final
* **Responsabilidades:**
  - Mostrar menús y opciones
  - Capturar entrada del usuario 
  - Formatear y mostrar resultados 
  - Manejar errores de interfaz
* **No debe:**
  - Contener lógica de negocio 
  - Acceder directamente a entidades del dominio
  - Realizar validaciones complejas

### Capa Application

* **Objetivo:** Orquestar casos de uso
* **Responsabilidades:**
  - Coordinar flujos de trabajo complejos 
  - Validar datos de entrada a nivel de aplicación 
  - Gestionar transacciones (si las hubiera)
  - Adaptar datos entre capas
* **No debe:**
  - Contener reglas de negocio complejas 
  - Almacenar estado permanente

### Capa Domain

* **Objetivo:** Contener la lógica central del negocio
* **Responsabilidades:**
  - Definir entidades y sus relaciones
  - Implementar reglas de negocio
  - Validar invariantes
  - Definir contratos/interfaces para infraestructura
* **Características:**
  - Independiente de frameworks
  - Testeable en aislamiento
  - Sin dependencias externas

### Capa Infrastructure

* Objetivo: Proporcionar implementaciones técnicas
* Responsabilidades:
  - Implementar repositorios (persistencia)
  - Proporcionar datos iniciales
  - Manejar aspectos técnicos (red, archivos, etc.)
* Puede ser reemplazada:
  - Cambiar de memoria a base de datos
  - Cambiar de consola a API web
  - Cambiar de archivos a servicios en la nube

## 🔄 Patrones Utilizados

### 1. Repository Pattern

```
# Contrato en Domain
class ShipmentRepository:
    def add(self, shipment): ...

# Implementación en Infrastructure
class ShipmentRepositoryMemory(ShipmentRepository):
    def add(self, shipment):
        self._storage[shipment.tracking_code] = shipment
```

### 2. Dependency Injection

```
# Los servicios reciben repositorios por inyección
class ShipmentService:
    def __init__(self, repo):  # Inyección de dependencia
        self._repo = repo
```

### 3. Polymorphism

```
# Diferentes tipos de envíos con comportamientos específicos
class Shipment: ...  # Base
class FragileShipment(Shipment): ...  # Prioridad especial
class ExpressShipment(Shipment): ...  # Prioridad fija
```

## 📊 Ventajas de Esta Arquitectura

| Ventaja | Beneficio |
| :--- | :--- |
| **Separación de preocupaciones** | Cada capa tiene una responsabilidad clara |
| **Testabilidad** | El dominio puede probarse sin infraestructura |
| **Mantenibilidad** | Cambios en una capa no afectan a las otras |
| **Escalabilidad** | Fácil agregar nuevas interfaces (web, API) |
| **Flexibilidad** | Cambiar infraestructura sin tocar el dominio |

<hr>

</details>