# 📁 Contrato de Repositorios del Sistema Logístico

## 🎯 Introducción

Los **contratos de repositorio** definen las interfaces que la capa de **Domain** espera para la persistencia de datos. La capa de **Infrastructure** proporciona implementaciones concretas de estos contratos, permitiendo cambiar el mecanismo de almacenamiento sin afectar la lógica de negocio.

---

## 🏗️ Arquitectura de Repositorios

### Diagrama de Dependencias

Domain Layer (Contratos)
├── ShipmentRepository (Interface)
├── CenterRepository (Interface)
└── RouteRepository (Interface)

Infrastructure Layer (Implementaciones)
├── ShipmentRepositoryMemory
├── CenterRepositoryMemory
└── RouteRepositoryMemory

### Principios de Diseño
1. **Dependency Inversion**: Domain define interfaces, Infrastructure las implementa
2. **Persistence Ignorance**: Domain no sabe cómo se persisten los datos
3. **Interface Segregation**: Cada repositorio tiene responsabilidad única
4. **Liskov Substitution**: Cualquier implementación puede sustituir a otra

---

## 📦 ShipmentRepository

### Contrato (`domain/shipment_repository.py`)

```python
class ShipmentRepository:
    """Contrato para la persistencia y recuperación de envíos."""
    
    def add(self, shipment: Shipment) -> None:
        """
        Almacena un nuevo envío o actualiza uno existente.
        
        Args:
            shipment: Instancia de Shipment (o subtipo) a almacenar.
            
        Raises:
            ValueError: Si el envío no es válido (depende de implementación).
        """
        raise NotImplementedError
    
    def remove(self, tracking_code: str) -> bool:
        """
        Elimina un envío del repositorio.
        
        Args:
            tracking_code: Código de seguimiento del envío a eliminar.
            
        Returns:
            True si el envío existía y fue eliminado, False en caso contrario.
        """
        raise NotImplementedError
    
    def get_by_tracking_code(self, tracking_code: str) -> Optional[Shipment]:
        """
        Recupera un envío por su código de seguimiento.
        
        Args:
            tracking_code: Código único del envío a buscar.
            
        Returns:
            El objeto Shipment si se encuentra, None si no existe.
            Nota: Puede devolver cualquier subtipo (FragileShipment, ExpressShipment).
        """
        raise NotImplementedError
    
    def list_all(self) -> List[Shipment]:
        """
        Obtiene todos los envíos almacenados en el repositorio.
        
        Returns:
            Lista con todos los objetos Shipment almacenados.
            El orden no está garantizado por el contrato.
        """
        raise NotImplementedError
```

### Implementación en Memoria (`infrastructure/memory_shipment.py`)
```python
class ShipmentRepositoryMemory(ShipmentRepository):
    """
    Implementación en memoria del repositorio de envíos.
    
    Características:
    - Almacenamiento temporal en diccionario Python
    - Búsquedas case-insensitive
    - No persiste entre ejecuciones
    - Soporta polimorfismo (todos los subtipos de Shipment)
    """
    
    def __init__(self):
        self._by_tracking_code = {}  # key: tracking_code.lower()
    
    def add(self, shipment):
        key = shipment.tracking_code.lower()
        self._by_tracking_code[key] = shipment
    
    def remove(self, tracking_code):
        key = (tracking_code or "").strip().lower()
        if key in self._by_tracking_code:
            del self._by_tracking_code[key]
            return True
        return False
    
    def get_by_tracking_code(self, tracking_code):
        key = (tracking_code or "").strip().lower()
        return self._by_tracking_code.get(key)
    
    def list_all(self):
        return list(self._by_tracking_code.values())
```

### Características de la Implementación

| Característica | Descripción | Impacto |
| :--- | :--- | :--- |
| **Case-insensitive** | Normaliza las claves (IDs) a minúsculas | Búsquedas más tolerantes y menos propensas a errores de usuario |
| **Polimorfismo** | Capacidad de almacenar cualquier subtipo de `Shipment` | Flexibilidad para manejar envíos Standard, Fragile o Express indistintamente |
| **In-memory** | Los datos residen en la memoria RAM, no se guardan en disco | Acceso extremadamente rápido, pero los datos se pierden al cerrar la aplicación |
| **Simple Dict** | Utiliza diccionarios nativos de Python como estructura base | Implementación sencilla y directa, aunque no está optimizada para acceso concurrente |

## 🏭 CenterRepository

### Contrato (`domain/center_repository.py`)
```python
class CenterRepository:
    """Contrato para la persistencia y recuperación de centros logísticos."""
    
    def add(self, center: Center) -> None:
        """
        Almacena un nuevo centro o actualiza uno existente.
        
        Args:
            center: Instancia de Center a almacenar.
        """
        raise NotImplementedError
    
    def remove(self, center_id: str) -> bool:
        """
        Elimina un centro del repositorio.
        
        Args:
            center_id: Identificador único del centro a eliminar.
            
        Returns:
            True si el centro existía y fue eliminado, False en caso contrario.
        """
        raise NotImplementedError
    
    def get_by_center_id(self, center_id: str) -> Optional[Center]:
        """
        Recupera un centro por su identificador único.
        
        Args:
            center_id: ID del centro a buscar.
            
        Returns:
            El objeto Center si se encuentra, None si no existe.
        """
        raise NotImplementedError
    
    def list_all(self) -> List[Center]:
        """
        Obtiene todos los centros almacenados en el repositorio.
        
        Returns:
            Lista con todos los objetos Center almacenados.
        """
        raise NotImplementedError
```

### Implementación en Memoria (`infrastructure/memory_center.py`)
```python
class CenterRepositoryMemory(CenterRepository):
    """
    Implementación en memoria del repositorio de centros.
    
    Características:
    - Almacenamiento en diccionario Python
    - Búsquedas case-insensitive
    - Integridad referencial básica
    """
    
    def __init__(self):
        self._by_center_id = {}  # key: center_id.lower()
    
    def add(self, center):
        key = center.center_id.lower()
        self._by_center_id[key] = center
    
    def remove(self, center_id):
        key = (center_id or "").strip().lower()
        if key in self._by_center_id:
            del self._by_center_id[key]
            return True
        return False
    
    def get_by_center_id(self, center_id):
        key = (center_id or "").strip().lower()
        return self._by_center_id.get(key)
    
    def list_all(self):
        return list(self._by_center_id.values())
```

## 🚛 RouteRepository

### Contrato (`domain/route_repository.py`)
```python
class RouteRepository:
    """Contrato para la persistencia y recuperación de rutas de transporte."""
    
    def add(self, route: Route) -> None:
        """
        Almacena una nueva ruta o actualiza una existente.
        
        Args:
            route: Instancia de Route a almacenar.
        """
        raise NotImplementedError
    
    def remove(self, route_id: str) -> bool:
        """
        Elimina una ruta del repositorio.
        
        Args:
            route_id: Identificador único de la ruta a eliminar.
            
        Returns:
            True si la ruta existía y fue eliminada, False en caso contrario.
        """
        raise NotImplementedError
    
    def get_by_route_id(self, route_id: str) -> Optional[Route]:
        """
        Recupera una ruta por su identificador único.
        
        Args:
            route_id: ID de la ruta a buscar.
            
        Returns:
            El objeto Route si se encuentra, None si no existe.
        """
        raise NotImplementedError
    
    def list_all(self) -> List[Route]:
        """
        Obtiene todas las rutas almacenadas en el repositorio.
        
        Returns:
            Lista con todos los objetos Route almacenados.
        """
        raise NotImplementedError
```

### Implementación en Memoria (`infrastructure/memory_route.py`)
```python
class RouteRepositoryMemory(RouteRepository):
    """
    Implementación en memoria del repositorio de rutas.
    
    Características:
    - Similar a otros repositorios en memoria
    - Mantiene referencias a objetos Center
    - Case-insensitive para IDs
    """
    
    def __init__(self):
        self._by_route_id = {}  # key: route_id.lower()
    
    def add(self, route):
        key = route.route_id.lower()
        self._by_route_id[key] = route
    
    def remove(self, route_id):
        key = (route_id or "").strip().lower()
        if key in self._by_route_id:
            del self._by_route_id[key]
            return True
        return False
    
    def get_by_route_id(self, route_id):
        key = (route_id or "").strip().lower()
        return self._by_route_id.get(key)
    
    def list_all(self):
        return list(self._by_route_id.values())
```

## 🔄 Patrones de Uso Comunes

### 1. Inyección de Dependencias en Servicios
```python
# Application Service recibe repositorio por constructor
class ShipmentService:
    def __init__(self, repo: ShipmentRepository):  # <- Aquí el contrato
        self._repo = repo
    
    def register_shipment(self, tracking_code, sender, recipient):
        # Usa el repositorio a través del contrato
        if self._repo.get_by_tracking_code(tracking_code):
            raise ValueError("Código duplicado")
        
        shipment = Shipment(tracking_code, sender, recipient)
        self._repo.add(shipment)  # <- Llama al método del contrato
```

### 2. Creación y Configuración
```python
# En el punto de entrada (main/menu)
from infrastructure.memory_shipment import ShipmentRepositoryMemory
from application.shipment_service import ShipmentService

# Crear implementación concreta
repo = ShipmentRepositoryMemory()

# Inyectar en servicio (que usa el contrato/interface)
service = ShipmentService(repo)

# El servicio funciona igual con cualquier implementación
```

## 🧪 Garantías del Contrato

### Garantías para Implementadores
1. **Métodos Obligatorios**: Deben implementar todos los métodos del contrato
2. **Tipos de Retorno**: Deben respetar los tipos declarados
3. **Comportamiento Esperado**: Deben seguir la semántica descrita
4. **Excepciones Documentadas**: Deben lanzar las excepciones documentadas

### Garantías para Clientes (Domain/Application)
1. **Abstracción**: No necesitan conocer detalles de implementación
2. **Sustituibilidad**: Pueden cambiar implementaciones transparentemente
3. **Consistencia**: Mismos métodos en todas las implementaciones
4. **Testabilidad**: Pueden mockear/stubear fácilmente

## ✅ Checklist para Nuevas Implementaciones

### Requisitos Mínimos
* Implementar todos los métodos del contrato
* Respetar tipos de retorno declarados
* Seguir semántica de cada método
* Manejar casos bordes (None, strings vacíos)

### Buenas Prácticas
* Documentar excepciones específicas
* Implementar búsquedas case-insensitive
* Mantener consistencia con otras implementaciones
* Proporcionar tests de integración

### Performance
* Optimizar operaciones frecuentes (get_by_id)
* Considerar caché para datos estáticos
* Manejar conexiones/pools eficientemente