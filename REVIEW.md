# REVISIÓN FASE 01

## RECOMENDACIONES / COMENTARIOS

- La aplicación prometem pero debes reestructurar algunos aspectos para que siga los principios del diseño por capas que hemos visto en clase. 
- Las validaciones en los constructores mejor hacerlas con **setters**. Por ejemplo de `center_id`, `name`, `location` en la clase `Center` puedes hacer las validaciones en los **setters** de los mismos y luego usarlos en el constructor.

## ASPECTOS A CAMBIAR / AÑADIR

- La capa de validación debería simplemente orquestar y como mucho validar las entradas de los usuarios, no aplicar lógica de negocio, ni almacenar. Esas acciones se deberían hacer en el dominio. Ejemplos:
  - En la capa de aplicación se decide cómo se crea un envío según su tipo”  decide que standard → Shipment, fragile → FragileShipment, express → ExpressShipment y además fija que express no acepta priority. Esto son reglas de negocio.
  - Cuándo se considera que una ruta ya está despachada” en `route_service.py` (líneas 117-118) usa la condición `all(s.current_status == "IN_TRANSIT" ...)` para decidir si se puede volver a despachar. Esto es una regla del proceso logístico (dominio), no una mera validación de entrada.
  - Cuando impides asignar un envío ya asignado (`route_service.py` (líneas 72-73)). Aunque se apoya en métodos del dominio, la regla se debería aplicr al llamar desde el servicio al dominio en `Route` o en `Shipment` y ser estos los que lancen la excepción.
- Tienes dos fuentes de verdad para envíos y rutas: los repositorios que los usas en la capa de aplicación y listas en la capa de dominio. Deberías solo usar repositorios y hacerlo en el dominio no en la aplicación.




 