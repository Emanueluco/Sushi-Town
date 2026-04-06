# Sushi-Town


# 🎓 Guía de Presentación: Proyecto UamiShop

Esta guía está estructurada para seguir el flujo de la presentación `UamiShop.pptx`, vinculando los requisitos de las **Prácticas 1 a 9** con la **implementación técnica** actual en el repositorio.

---

## 🗺️ Introducción: Estrategia de Transformación
*   **Contexto:** El proyecto representa la evolución de una aplicación tradicional (monolito) hacia un sistema distribuido escalable y resiliente.
*   **Referencia Práctica:** Prácticas 1 y 5.
*   **En el Proyecto:** 
    *   Estructura multi-proyecto con servicios independientes: `uamishop-catalogo`, `uamishop-ventas`, `uamishop-ordenes`, `uamishop-gateway`.
    *   Estandarización con **Docker Compose**.
*   **Puntos Clave:** Mencionar que no se dividió por "capas" (BD, Lógica, UI), sino por "dominios de negocio" (Catálogo vs Ventas).

---

## 🏗️ Fase 1: Fundamentos y DDD (Diseño Guiado por Dominio)
*   **Objetivo:** Establecer fronteras claras (Bounded Contexts) para evitar el acoplamiento.
*   **Referencia Práctica:** Práctica 2.
*   **Qué demostrar en el Código:**
    *   **Value Objects:** Ver `Money.java` y `ProductoId.java` en el Shared Kernel. Estos garantizan que los datos sean inmutables y válidos (ej. un precio no puede ser negativo).
    *   **Aggregate Roots:** Mostrar `Orden.java`. Resaltar que todas las operaciones sobre ítems o direcciones pasan a través de la raíz del agregado para mantener la integridad.
*   **Narrativa:** *"Definimos el lenguaje del negocio primero. En lugar de pensar en tablas de SQL, pensamos en reglas de negocio protegidas por nuestras entidades de dominio."*

---

## 📦 Fase 2: El Monolito Modular
*   **Objetivo:** Organizar el código antes de la separación física.
*   **Referencia Práctica:** Práctica 5.
*   **Qué demostrar en el Código:**
    *   **Encapsulamiento:** El uso de interfaces como `VentasApi` para interactuar entre módulos.
    *   **Shared Kernel:** Carpeta `shared` que contiene `ClienteId`, `Money`, etc., permitiendo que los módulos se entiendan sin conocer los detalles internos de otros.
*   **Narrativa:** *"Un buen microservicio nace de un modulo bien diseñado. Si el monolito es un desorden, los microservicios serán un desorden distribuido."*

---

## 🚀 Fase 3: Comunicación Basada en Eventos
*   **Objetivo:** Desacoplamiento total y consistencia eventual.
*   **Referencia Práctica:** Práctica 6 y 8.
*   **Qué demostrar en el Código:**
    *   **RabbitMQ:** Ver `RabbitConfig.java` en `uamishop-ordenes`.
    *   **Eventos:** Mostrar `ProductoCompradoEvent`.
    *   **Listeners:** Mostrar cómo `uamishop-catalogo` escucha eventos para actualizar sus estadísticas de forma asíncrona.
*   **Narrativa:** *"Usamos el patrón 'Event-Carried State Transfer'. Cuando una orden se crea, el sistema emite una señal. Los otros servicios reaccionan a esa señal sin que el servicio de Órdenes sepa si existen o no."*

---

## 🛡️ Resiliencia y Tolerancia a Fallos
*   **Objetivo:** Evitar fallos en cascada en un sistema distribuido.
*   **Referencia Práctica:** Práctica 9.
*   **Qué demostrar en el Código:**
    *   **Resilience4J:** Ver `CatalogoClient.java` en el microservicio de Ventas.
    *   **Circuit Breaker:** La anotación `@CircuitBreaker` y el método `fallback`.
*   **Narrativa:** *"En la red, todo puede fallar. Si el Catálogo no responde, el sistema de Ventas no se queda bloqueado; activa un 'cortador de circuito' y ofrece una experiencia degradada pero funcional (fallback)."*

---

## 🧪 Consistencia Distribuida: Transactional Outbox
*   **Objetivo:** Garantizar atomicidad entre la Base de Datos y el Broker de Mensajes.
*   **Referencia Práctica:** Práctica 9.
*   **Qué demostrar en el Código:**
    *   **Atomicidad:** Mostrar cómo en una misma transacción SQL se guarda la `Orden` y el `OutboxEvent`.
    *   **Worker:** La clase `OutboxPublicador` que reintenta enviar los mensajes pendientes.
*   **Narrativa:** *"Nunca publicamos directamente a RabbitMQ dentro de la transacción de negocio. Primero lo guardamos en el 'Outbox' de la base de datos para asegurar que, si el sistema se apaga, no perdamos el evento."*

---

## 🚦 Infraestructura: Gateway y Calidad
*   **Objetivo:** Punto único de entrada y validación automática.
*   **Referencia Práctica:** Práctica 7 y 9.
*   **Qué demostrar en el Código:**
    *   **Spring Cloud Gateway:** `uamishop-gateway` en el puerto 8080.
    *   **Karate DSL:** Pruebas E2E en `src/test/java/com/uamishop/gateway/karate`.
*   **Narrativa:** *"El Gateway simplifica la vida del frontend al ocultar la complejidad de los múltiples microservicios detrás de un solo puerto. Validamos la calidad del API con Karate, simulando flujos reales de usuario."*

---

## 💡 Tips para la Demo en Vivo
1.  **El 'Momento Wow' (Resiliencia):** Apaga el microservicio de Catálogo con `docker stop`. Agregue un ítem al carrito desde el frontend o Swagger del Gateway. Muestre el mensaje de "Servicio no disponible" (vía Fallback) en lugar de un error 500 feo.
2.  **Trazabilidad (Outbox):** Muestra la tabla `outbox_events` con un evento en estado `PENDING` (mientras RabbitMQ está apagado) y cómo cambia a `SENT` al encenderlo.
3.  **Docker Compose:** Muestra `docker ps` para ver los 7 servicios corriendo:
    *   3 Microservicios (Catálogo, Ventas, Órdenes).
    *   Gateway (Puerta de enlace).
    *   Base de Datos MySQL.
    *   RabbitMQ (Eventos).
    *   Frontend (Interfaz de usuario).

---
**Preparado por Antigravity para la presentación final.**
