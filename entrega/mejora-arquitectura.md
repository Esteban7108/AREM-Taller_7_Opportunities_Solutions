# Mejora de Arquitectura (TO-BE) — Identificación y Priorización de Mejoras

## Cliente
RedExpress (caso base de referencia, Plataforma de Logística)

## Integrantes del equipo
- Esteban Díaz
- Juliana Moreno

**Fecha:** 27/09/2025

---

## 1. Diagnóstico inicial

Este diagnóstico se apoya en lo ya evidenciado en el Taller 4 (Mapa de Infraestructura y Diagnóstico Técnico) de RedExpress, sin inventar hallazgos nuevos. El ejercicio de clase se enfoca en las brechas técnicas, porque son las que ya están completamente diagnosticadas en el curso para este caso base.

- **¿Cuáles son los procesos o tecnologías que generan mayor fricción en la operación?** El balanceador de carga de instancia única y la base de datos con escritura centralizada en Bogotá generan lentitud y riesgo de caída total ante picos de demanda, como las campañas de fin de año.
- **¿Qué problemas recurrentes señalaron los usuarios o el cliente en las entrevistas?** Los usuarios de la región Medellín reportan demoras en la asignación de rutas, porque toda solicitud de esa región depende del módulo de procesamiento de rutas alojado en Bogotá.
- **¿Qué vulnerabilidades de seguridad o riesgos quedaron evidenciados en el análisis previo?** Punto único de falla en el balanceador de carga, cuello de botella de escritura en la base de datos distribuida, y límite de escalabilidad geográfica en la región Medellín — los tres riesgos priorizados como Altos/Medio en la tabla de diagnóstico del Taller 4.

**Resumen del problema actual (foto del AS-IS):** RedExpress opera hoy con una infraestructura de cuatro zonas (Clientes, Borde/Global, Región Bogotá y Región Medellín) en la que los componentes compartidos entre regiones — el balanceador de carga y la base de datos distribuida — no tienen redundancia, y la región Medellín depende por completo de la infraestructura de Bogotá para procesar rutas. Esto concentra el riesgo de disponibilidad y de escalabilidad en un solo punto del sistema, justo en el escenario (picos de demanda regionales) donde más se necesita que la plataforma responda.

| Riesgo / Brecha | Taller de origen | Tipo |
|---|---|---|
| Balanceador de Carga (instancia única) | Taller 4 | Técnica |
| Base de Datos Distribuida (escritura única en Bogotá) | Taller 4 | Técnica |
| Región Medellín sin módulo de rutas propio | Taller 4 | Técnica / Funcional |

---

## 2. Propuesta de mejoras

### 2.1 Lluvia de ideas (sin censura inicial)

| # | Idea de mejora | Tipo |
|---|---|---|
| 1 | Balanceador de carga redundante (activo-pasivo) | Técnica |
| 2 | Base de datos particionada por región | Técnica |
| 3 | Módulo de rutas propio en Medellín | Técnica / Funcional |
| 4 | Notificaciones proactivas al cliente cuando se detecta una demora prevista en la entrega | Proceso / Comunicación |
| 5 | Checklist digital de verificación del paquete en el punto de entrega, firmado por el mensajero | Proceso |
| 6 | Encuesta corta de satisfacción integrada en la app, justo después de cada entrega | Proceso / Comunicación |
| 7 | Canal de WhatsApp o chatbot para consultar el estado de un envío sin llamar a soporte | Proceso / Comunicación |
| 8 | Panel unificado de monitoreo para operadores, con alertas por región | Técnica (mediano plazo) |

### 2.2 Priorización (2-3 ideas seleccionadas, con justificación)

De las 8 ideas anteriores, el equipo prioriza las 3 mejoras técnicas (#1, #2, #3) porque son las únicas con una brecha diagnosticada y evidenciada formalmente en el Taller 4 — es decir, tienen trazabilidad directa AS-IS → TO-BE. Las ideas de proceso y comunicación (#4 a #7) son válidas y de bajo esfuerzo (quick wins), pero quedan como backlog para una siguiente iteración porque, en este caso base, no cuentan con un hallazgo formal que las respalde.

| Solución priorizada | Esfuerzo | Impacto | Quick win / Largo plazo | Justificación |
|---|---|---|---|---|
| Balanceador redundante | Medio | Alto | Quick win | Cierra el punto único de falla de mayor prioridad (Alta) del diagnóstico del Taller 4; se decidió mediante matriz de decisión ponderada (ver anexo `matriz-decision-balanceador.md`) |
| BD particionada por región | Alto | Alto | Largo plazo | Elimina el cuello de botella de escritura centralizada en Bogotá, mejorando la latencia del rastreo en tiempo real fuera de esa región |
| Módulo de rutas en Medellín | Alto | Medio | Largo plazo | Elimina la dependencia total de Medellín respecto del módulo de rutas de Bogotá, resolviendo la demora reportada por los usuarios de esa región |

Para la brecha del balanceador de carga, que admite varias soluciones posibles (activo-pasivo, activo-activo, mantener el balanceador único con monitoreo reforzado), se aplicó la matriz de decisión ponderada de 8 pasos, documentada en `matriz-decision-balanceador.md`. La decisión resultante fue implementar un **balanceador activo-pasivo con failover automático**, descartando el esquema activo-activo por costo y tiempo de implementación, y descartando mantener el balanceador único por no superar el criterio eliminatorio de disponibilidad.

---

## 3. Visualización TO-BE

### 3.1 Proceso mejorado

El proceso de asignación de rutas para la región Medellín deja de depender de una llamada remota al módulo de Bogotá: al replicarse el módulo de procesamiento de rutas en Medellín, la solicitud se resuelve localmente, reduciendo la latencia percibida por operadores y mensajeros de esa región sin cambiar el flujo de negocio (el usuario y el mensajero siguen interactuando con la misma App Móvil y el operador con el mismo Portal Web).

### 3.2 Cambios en aplicaciones, infraestructura y flujos de información

**TO-BE de Aplicaciones** (extiende el C2 del Taller 3, `entrega/c2-contenedores-final.drawio`): se agrega un **Motor de Rutas - Medellín**, réplica del de Bogotá, conectado al Módulo de Gestión de Paquetes junto con el resto de contenedores ya existentes (App Móvil, Portal Web Operadores, Seguimiento GPS, Sistema de Alertas), sin reemplazar ninguno de ellos.

**TO-BE de Tecnología** (extiende el mapa del Taller 4, `entrega/mapa-final.drawio`): el Balanceador de Carga pasa de instancia única a un esquema activo-pasivo (se agrega un Balanceador de Carga - Pasivo que recibe el tráfico solo ante conmutación por falla), y la Base de Datos Distribuida se particiona por región, agregando una BD Medellín junto a la BD Bogotá ya existente, junto con el Módulo de Rutas - Medellín en la región correspondiente.

Ambos diagramas TO-BE, junto con el interruptor interactivo AS-IS/TO-BE, están disponibles en la versión visual del taller: `clase/visualizacion-opportunities-solutions.html`.



---

## 4. Análisis de beneficios y riesgos

> El detalle completo de esta sección (brechas, riesgos, capacidades y paquetes de trabajo) también está consolidado en `entrega/matriz-brechas.xlsx`, con una hoja por tabla.

**Brechas cerradas y beneficios esperados:**

| AS-IS | TO-BE | Brecha que cierra | Beneficio esperado |
|---|---|---|---|
| Balanceador único | Balanceador redundante (activo-pasivo) | Punto único de falla | Alta disponibilidad de toda la plataforma |
| BD con escritura única en Bogotá | BD particionada por región | Cuello de botella de latencia | Mejor rendimiento del rastreo en tiempo real fuera de Bogotá |
| Medellín sin módulo de rutas propio | Módulo de rutas replicado en Medellín | Límite de escalabilidad geográfica | La región puede crecer sin saturar Bogotá |

**Riesgos, limitaciones y dependencias de implementación:**

| Solución | Riesgo / limitación / dependencia |
|---|---|
| Balanceador redundante | Depende de la aprobación de presupuesto adicional (≈ USD 400/mes) del proveedor cloud para la segunda instancia; si no se aprueba, la mejora no puede iniciar en el plazo previsto antes de la campaña de fin de año. |
| BD particionada por región | Requiere una migración con ventana de mantenimiento; existe riesgo de downtime parcial y de inconsistencia de datos durante la sincronización inicial entre particiones. |
| Módulo de rutas en Medellín | Depende de contratar o reasignar personal técnico en la región; sin ese equipo local, el módulo replicado no tiene quién lo opere ni lo mantenga. |

### Capacidades de negocio y paquetes de trabajo

| Capacidad | Madurez AS-IS | Madurez TO-BE | Qué la explica |
|---|---|---|---|
| Recepción y registro de envíos | 4 | 4 | Sin brechas diagnosticadas; no se toca en esta iteración |
| Planeación y asignación de rutas | 2 | 4 | Medellín depende del motor de Bogotá y se demora (Taller 4) |
| Seguimiento en tiempo real | 3 | 4 | Funciona, pero la escritura centralizada en Bogotá lo hace lento fuera de allí |
| Notificación y atención al cliente | 3 | 3 | Sin brecha formal; las ideas #4 y #7 quedaron en backlog |
| Continuidad operativa de la plataforma | 2 | 4 | Punto único de falla en el balanceador |

| Paquete de trabajo | Brechas que incluye | Capacidad que mejora | Tipo |
|---|---|---|---|
| WP1 · Continuidad de la plataforma | Balanceador redundante activo-pasivo (decisión documentada en `matriz-decision-balanceador.md`) | Continuidad operativa (2 → 4) | Quick win · 4-6 semanas |
| WP2 · Rutas y datos regionales | Módulo de rutas en Medellín + BD particionada por región | Planeación y asignación de rutas (2 → 4) y Seguimiento en tiempo real (3 → 4) | Largo plazo |

---

## Anexos
- Diagrama TO-BE de Aplicaciones (extiende el C2 del Taller 3): `entrega/to-be-aplicaciones-final.drawio`
- Diagrama TO-BE de Tecnología (extiende el mapa del Taller 4): `entrega/to-be-tecnologia-final.drawio`
- Matriz de brechas (Gap Analysis), con beneficios, riesgos, capacidades y paquetes de trabajo: `entrega/matriz-brechas.xlsx`
- Versión interactiva AS-IS/TO-BE (referencia visual complementaria): `clase/visualizacion-opportunities-solutions.html`
- Matriz de decisión ponderada (balanceador de carga): `entrega/matriz-decision-balanceador.md`
- Diagramas base heredados: `c1-contexto-final.drawio`, `c2-contenedores-final.drawio` (Taller 3), `mapa-final.drawio` (Taller 4)

---

_Este documento hace parte de la entrega del Taller 7 (Opportunities & Solutions) del curso AREM - Universidad de La Sabana._
