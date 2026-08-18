# Análisis de Patrones de Diseño GoF en Spring Framework

## Portada

**Nombre:** Andrés David Diaz Rincón
**Código:** 02240131056
**Curso:** Patrones de Diseño de Software - A1
**Unidad:** 1 — Fundamentos de Patrones de Diseño y Buenas Prácticas
**Fecha:** 18 de agosto de 2026

---

## Introducción

El presente documento examina tres patrones de diseño pertenecientes al catálogo del Gang of Four (GoF), tomando como base el código fuente de Spring Framework, uno de los frameworks de desarrollo Java con mayor adopción en el ámbito profesional. Se busca evidenciar cómo un framework consolidado y de uso extendido incorpora, de forma sistemática y no accidental, patrones de diseño reconocidos para dar solución a problemas de arquitectura de software que se repiten con frecuencia, estableciendo además una conexión entre estas decisiones técnicas y los principios SOLID (Gamma et al., 1994).

Spring Boot fue seleccionado como caso de estudio por tratarse de una de las plataformas más comunes para construir aplicaciones empresariales en Java. El corazón de este framework —que incluye el contenedor de inversión de control, la administración de beans, el soporte para programación orientada a aspectos y el acceso a datos— se apoya en patrones de diseño tradicionales, lo cual lo convierte en un escenario apropiado para analizar estos patrones dentro de un entorno real de producción, en contraposición a ejemplos académicos simplificados (Spring Framework, 2026).

---

## Análisis de Patrón 1: Factory Method

El patrón Factory Method pertenece a la categoría Creacional del catálogo GoF (Gamma et al., 1994). Su propósito general es definir una interfaz para crear un objeto, mientras se delega a una implementación concreta la decisión de qué clase específica instanciar, evitando que el código cliente dependa directamente de constructores concretos.

En Spring Framework, este patrón aparece en la interfaz FactoryBean<T>, ubicada en el paquete org.springframework.beans.factory, dentro del módulo spring-beans (Spring Framework, 2026). Cualquier bean que implemente esta interfaz actúa como una fábrica de otro objeto dentro del contenedor IoC.

El problema que resuelve en este contexto es permitir la creación de objetos complejos —que no pueden construirse simplemente con new porque requieren lógica adicional, dependen de recursos externos (JNDI, sesiones de Hibernate) o necesitan configuración previa— sin que el contenedor de Spring tenga que conocer los detalles de esa construcción. En lugar de que cada componente cliente sepa cómo ensamblar un objeto complejo, delega esa responsabilidad a una fábrica dedicada.

Como evidencia de código, se identificó la definición central de la interfaz:

```java
public interface FactoryBean<T> {

    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

    @Nullable
    T getObject() throws Exception;

    @Nullable
    Class<?> getObjectType();

    default boolean isSingleton() {
        return true;
    }
}
```

Cuando el contenedor de Spring necesita satisfacer una dependencia asociada a un FactoryBean, no retorna la instancia del FactoryBean en sí, sino que invoca internamente su método getObject() y entrega el resultado de esa invocación (Spring Projects, 2026). Esto queda documentado explícitamente en el propio framework, que aclara que esta interfaz es utilizada intensivamente para casos como ProxyFactoryBean o JndiObjectFactoryBean (Spring Projects, 2026).

Este patrón refuerza principalmente el principio de Inversión de Dependencias (DIP), ya que el código cliente depende únicamente de la abstracción FactoryBean<T> y no de las clases concretas que construyen los objetos reales, permitiendo cambiar la lógica de creación sin tocar el código que consume esos objetos.
---

## Análisis de Patrón 2: Adapter

El patrón Adapter pertenece a la categoría Estructural del catálogo GoF (Gamma et al., 1994). Su propósito general es convertir la interfaz de una clase en otra interfaz que el cliente espera, permitiendo que clases con interfaces incompatibles trabajen juntas sin modificar su código original.

En Spring Framework, este patrón aparece en la interfaz HandlerAdapter, ubicada en el paquete org.springframework.web.servlet, dentro del módulo spring-webmvc (Spring Framework, 2026). Esta interfaz es utilizada por el DispatcherServlet, el componente central de Spring MVC.

El problema que resuelve en este contexto es que, a lo largo de los años, Spring MVC ha soportado distintos tipos de "manejadores" de peticiones: controladores anotados con @RequestMapping, controladores que implementan la interfaz Controller, manejadores tipo HttpRequestHandler, entre otros. El DispatcherServlet no puede invocar cada tipo de manejador de forma distinta sin volverse una clase enorme llena de condicionales. En lugar de eso, delega la invocación a un HandlerAdapter, que actúa como intermediario y traduce la petición HTTP genérica hacia la forma específica en que cada tipo de manejador espera ser llamado.

Como evidencia de código, se identificó la definición de la interfaz (Spring Projects, 2026):

```java
public interface HandlerAdapter {

    boolean supports(Object handler);

    @Nullable
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response,
            Object handler) throws Exception;

    long getLastModified(HttpServletRequest request, Object handler);
}
```

El método supports(Object handler) permite que cada implementación decida si sabe "traducir" ese tipo de manejador, mientras que handle(...) ejecuta la adaptación real: recibe la petición y respuesta HTTP en su forma estándar de Servlet y las convierte en la llamada específica que el manejador concreto necesita. Gracias a esto, el DispatcherServlet puede seguir siendo extensible indefinidamente sin contener código específico para cada tipo de manejador (Spring Projects, 2026).

Este patrón refuerza principalmente el principio de Abierto/Cerrado (OCP), ya que es posible añadir soporte para un nuevo tipo de controlador simplemente creando una nueva implementación de HandlerAdapter y registrándola, sin modificar el código existente del DispatcherServlet.
---

## Análisis de Patrón 3: Observer

El patrón Observer pertenece a la categoría de Comportamiento del catálogo GoF (Gamma et al., 1994). Su propósito general es definir una dependencia de uno a muchos entre objetos, de forma que cuando un objeto (el sujeto) cambia de estado o produce un evento, todos sus dependientes (los observadores) sean notificados automáticamente sin que el sujeto necesite conocer los detalles de cada uno.

En Spring Framework, este patrón aparece en la pareja de interfaces ApplicationEventPublisher y ApplicationListener, ubicadas en el paquete org.springframework.context, dentro del módulo spring-context (Spring Framework, 2026). Todo ApplicationContext de Spring implementa la interfaz ApplicationEventPublisher.

El problema que resuelve en este contexto es permitir que distintas partes de una aplicación reaccionen a un mismo suceso (por ejemplo, el registro de un nuevo usuario) sin que el componente que originó el suceso tenga que llamar manualmente y de forma acoplada a cada uno de los servicios interesados (envío de correo, auditoría, actualización de contadores). En lugar de eso, el componente productor simplemente publica un evento, y cualquier número de "escuchas" (listeners) puede suscribirse a él sin que el productor sepa que existen.

Como evidencia de código, se identificó el método central de publicación (Spring Projects, 2026):

```java
@FunctionalInterface
public interface ApplicationEventPublisher {

    default void publishEvent(ApplicationEvent event) {
        publishEvent((Object) event);
    }

    void publishEvent(Object event);
}
```

Internamente, cuando se invoca publishEvent(...), el contenedor delega la difusión del evento a un ApplicationEventMulticaster (por defecto, SimpleApplicationEventMulticaster), el cual mantiene el registro de todos los ApplicationListener interesados y los notifica uno por uno (Spring Framework, 2026). Un componente puede suscribirse implementando ApplicationListener<MiEvento> o usando la anotación @EventListener, sin que el publicador conozca su existencia.

Este patrón refuerza principalmente el principio de Responsabilidad Única (SRP) combinado con bajo acoplamiento, ya que el componente que genera el evento se responsabiliza únicamente de notificar que algo ocurrió, mientras que cada listener asume su propia responsabilidad (enviar un correo, auditar, actualizar una caché) de forma completamente independiente y desacoplada del productor del evento.
---

## Conclusiones

El análisis de estos tres patrones en Spring Framework confirma que su presencia no es incidental, sino el resultado de decisiones de diseño deliberadas orientadas a resolver problemas recurrentes de creación flexible de objetos, integración de interfaces incompatibles y comunicación desacoplada entre componentes (Gamma et al., 1994). Cada patrón identificado —Factory Method, Adapter y Observer— está directamente conectado con uno o más principios SOLID, lo que confirma que los patrones de diseño y los principios SOLID son herramientas complementarias: los principios orientan el "por qué" de una decisión de diseño, mientras que los patrones ofrecen el "cómo" concreto de implementarla. Esta relación es una lección directa para el diseño de software propio: reconocer el principio que se busca reforzar ayuda a identificar qué patrón de diseño resulta más adecuado para el problema en cuestión.

---

## Referencias

Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1994). Design patterns: Elements of reusable object-oriented software. Addison-Wesley.

Spring Framework. (2026). Spring Framework documentation. VMware. https://docs.spring.io/spring-framework/reference/

Spring Projects. (2026). spring-framework [Código fuente]. GitHub. https://github.com/spring-projects/spring-framework