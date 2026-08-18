# diaz-post1-u1
Post-contenido — Refactorización SOLID y análisis de patrones GoF en Spring

## Análisis de Violaciones SOLID

| Principio | Método/Sección afectada | Descripción de la violación |
|-----------|-------------------------|-----------------------------|
| SRP | calculateTotal + applyDiscount + saveOrder + sendEmail + printReport | La clase OrderProcessor se encarga de cinco responsabilidades distintas (cálculo, descuentos, persistencia, notificación y presentación) en un mismo lugar; un cambio en cualquiera de ellas obliga a modificar la misma clase, aumentando el riesgo de introducir errores no relacionados. |
| OCP | applyDiscount (if/else sobre customerType) | Agregar un nuevo tipo de cliente requiere editar directamente el método applyDiscount en lugar de extender el comportamiento mediante nuevas clases, violando el principio de "abierto a extensión, cerrado a modificación". |
| DIP | Toda la clase (dependencias internas sin abstracciones) | OrderProcessor implementa directamente toda la lógica sin depender de abstracciones (interfaces); esto impide sustituir, por ejemplo, el mecanismo de persistencia o notificación sin reescribir la clase completa. |

## Descripción
Repositorio del post-contenido de la Unidad 1 de Patrones de Diseño de Software — Sexto Semestre. Contiene dos partes: refactorización SOLID de un God Object (`parte-1-refactorizacion-solid/`) y análisis de patrones GoF en Spring Framework (`parte-2-analisis-gof-spring/`).
 
## Parte 1 — Refactorización SOLID
Proyecto Maven que refactoriza `OrderProcessor` aplicando SRP, OCP y DIP. Ver `parte-1-refactorizacion-solid/`.
 
## Parte 2 — Análisis de Patrones GoF en Spring
 
| # | Patrón | Categoría | Clase en Spring |
|---|--------|-----------|-----------------|
| 1 | Factory Method | Creacional | FactoryBean\<T\> |
| 2 | Adapter | Estructural | HandlerAdapter |
| 3 | Observer | Comportamiento | ApplicationEventPublisher / ApplicationListener |
 
Ver `parte-2-analisis-gof-spring/documento-analisis.md`.
 
## Herramientas utilizadas
- Java 26, Apache Maven, VS Code, Git, GitHub
- Código fuente de Spring Framework (investigación)
## Conclusiones

La actividad de post-contenido sirvió mucho para entender como tomar buenas decisiones desde el principio ahorra mucho tiempo en el futuro, que construir software no es sólamente escribir codigo, sino que tambien hay muchas técnicas y planeación detrás para que todo funcione correctamente, al final debemos saber escoger el patrón que nos parezca más útil para resolver el problema teniendo en cuenta los requisitos del cliente y nuestro repertorio de herramientas. 

También fue un ejercicio de troubleshooting, ya que una parte imporante del tiempo invertido en esta actividad fué aprender como Maven, Git y VSCode trabajan de manera cooperativa. El control de las versiones entre estos software y la compatibilidad fue algo que me llevó a bastantes búsquedas de google, pero al final todo encajó y salí aprendiendo muchas cosas.