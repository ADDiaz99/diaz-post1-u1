# diaz-post1-u1
Post-contenido — Refactorización SOLID y análisis de patrones GoF en Spring

## Análisis de Violaciones SOLID

| Principio | Método/Sección afectada | Descripción de la violación |
|-----------|-------------------------|-----------------------------|
| SRP | calculateTotal + applyDiscount + saveOrder + sendEmail + printReport | La clase OrderProcessor se encarga de cinco responsabilidades distintas (cálculo, descuentos, persistencia, notificación y presentación) en un mismo lugar; un cambio en cualquiera de ellas obliga a modificar la misma clase, aumentando el riesgo de introducir errores no relacionados. |
| OCP | applyDiscount (if/else sobre customerType) | Agregar un nuevo tipo de cliente requiere editar directamente el método applyDiscount en lugar de extender el comportamiento mediante nuevas clases, violando el principio de "abierto a extensión, cerrado a modificación". |
| DIP | Toda la clase (dependencias internas sin abstracciones) | OrderProcessor implementa directamente toda la lógica sin depender de abstracciones (interfaces); esto impide sustituir, por ejemplo, el mecanismo de persistencia o notificación sin reescribir la clase completa. |