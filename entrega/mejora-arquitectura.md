# Mejora de Arquitectura (TO-BE) — Identificación y Priorización de Mejoras

## Cliente
Oasis Atelier Floral (floristería personalizada, Sopó, Cundinamarca)

## Integrantes del equipo
- Esteban Díaz Vargas
- Katherin Juliana Moreno Carvajal

---

## 1. Diagnóstico inicial

A diferencia de RedExpress (un sistema en producción), Oasis todavía no tiene ningún sistema desplegado: opera 100% por Instagram y WhatsApp manual. El "AS-IS" de este diagnóstico combina, por tanto, la fricción del proceso manual actual (Taller de BPMN) con los riesgos ya diagnosticados sobre la arquitectura objetivo (Talleres 3-6).

- **¿Cuáles son los procesos o tecnologías que generan mayor fricción en la operación?** La gestión de pedidos por Instagram y WhatsApp sin registro centralizado obliga al Propietario y a la Encargada de Atención a reconstruir manualmente el estado de cada solicitud, sin trazabilidad ni respaldo de la información.
- **¿Qué problemas recurrentes señalaron los usuarios o el cliente?** No existen usuarios de un sistema todavía; el problema recurrente documentado es la pérdida de información entre canales dispersos y la dificultad para hacer seguimiento a solicitudes, cotizaciones y pedidos (Taller de BPMN).
- **¿Qué vulnerabilidades de seguridad o riesgos quedaron evidenciados en el análisis previo?** Consolidando los Talleres 3 a 6: infraestructura de bajo costo sin redundancia ni backups (Taller 4, riesgo Alto), ausencia de cifrado en reposo confirmado con exposición de datos personales (Taller 5, amenaza T4, riesgo Alto), y 12 de 13 ítems del checklist normativo en estado Parcial (Taller 6), destacando la falta de consentimiento informado y de mecanismo de derechos ARCO.

**Resumen del problema actual (foto del AS-IS):** Oasis está en fase de diseño, no de operación. El sistema objetivo ya definido (App Web + API REST + Base de Datos Oasis, más la integración con WhatsApp Business API) resuelve la fricción del proceso manual, pero hereda tres focos de riesgo que tres talleres distintos diagnosticaron de forma independiente: la base de datos no tiene garantía de cifrado en reposo ni backups automáticos, no existe ningún mecanismo de verificación de identidad del cliente, y el diseño no contempla todavía el consentimiento informado que exige la Ley 1581 de 2012.

| Riesgo / Brecha | Taller de origen | Tipo |
|---|---|---|
| Base de Datos Oasis sin cifrado en reposo confirmado ni backups automáticos | Taller 4, Taller 5 (T4), Taller 6 (ítems 5-6) | Técnica / Seguridad / Cumplimiento |
| Sin verificación de identidad del cliente al consultar o modificar una solicitud | Taller 5 (T1) | Seguridad |
| Sin autorización informada de tratamiento de datos ni mecanismo ARCO | Taller 6 (ítems 1-2) | Cumplimiento |

---

## 2. Propuesta de mejoras

### 2.1 Lluvia de ideas (sin censura inicial)

| # | Idea de mejora | Tipo |
|---|---|---|
| 1 | Migrar la base de datos a un proveedor con cifrado en reposo y backups automáticos | Técnica |
| 2 | Módulo de autenticación ligera (código de verificación por WhatsApp) para consultar/modificar un pedido | Técnica / Seguridad |
| 3 | Forzar HTTPS/HSTS y validar la integridad del payload del pedido en el servidor | Técnica |
| 4 | Registro de auditoría (usuario + timestamp) en cada cambio de estado del pedido | Técnica |
| 5 | Definir roles y permisos diferenciados (RBAC básico) antes de sumar un tercer usuario | Proceso / Técnica |
| 6 | Casilla de autorización de tratamiento de datos y política de privacidad publicada | Proceso / Cumplimiento |
| 7 | Rate limiting en la API y cola de reintentos hacia WhatsApp Business API | Técnica (mediano plazo) |
| 8 | Recordatorios automáticos de estado del pedido al cliente por WhatsApp | Proceso / Comunicación |
| 9 | Encuesta corta de satisfacción tras cada entrega | Proceso / Comunicación |

### 2.2 Priorización (2-3 ideas seleccionadas, con justificación)

De las 9 ideas anteriores, el equipo prioriza 3, por ser las que cierran los tres riesgos de mayor nivel diagnosticados de forma independiente en los Talleres 4, 5 y 6:

| Solución priorizada | Esfuerzo | Impacto | Quick win / Largo plazo | Justificación |
|---|---|---|---|---|
| Migración a Supabase Pro (cifrado en reposo + backup automático) | Bajo | Alto | Quick win | Cierra el riesgo Alto diagnosticado de forma independiente en 3 talleres; se decidió mediante matriz de decisión ponderada (ver anexo `matriz-decision-backup-oasis.md`) |
| Módulo de Autenticación Ligera | Medio | Medio | Quick win | Cierra la amenaza de Spoofing (T1) del Taller 5, sin requerir infraestructura nueva de terceros más allá de WhatsApp Business API, ya prevista en el diseño |
| Casilla de autorización + política de tratamiento de datos | Bajo | Medio | Quick win | Cierra 2 de las 12 brechas normativas del Taller 6 con el menor esfuerzo técnico de las tres, siendo principalmente un requisito de contenido y UI |

Las ideas #3, #4, #5 y #7 (integridad en tránsito, auditoría, RBAC, rate limiting) son válidas y responden a brechas reales de los Talleres 5 y 6, pero quedan como backlog de la siguiente iteración: no comprometen datos personales de forma directa como las tres priorizadas, y el equipo — sin perfil técnico dedicado — no puede abordar las seis a la vez. Las ideas #8 y #9 son quick wins de proceso sin una brecha formal que las respalde todavía en este curso.

Para la brecha de mayor riesgo (cifrado en reposo y backup), que admite tres soluciones reales, se aplicó la matriz de decisión ponderada de 8 pasos documentada en `matriz-decision-backup-oasis.md`: la decisión fue migrar a **Supabase plan Pro**, descartando mantener un script propio de backup (no supera el criterio eliminatorio de confiabilidad) y descartando aceptar el riesgo sin cambios (tampoco lo supera, a pesar de tener el total ponderado más alto en bruto).

---

## 3. Visualización TO-BE

### 3.1 Proceso mejorado
El proceso de registro y gestión de una solicitud de pedido deja de depender exclusivamente de la memoria y disponibilidad del Propietario o la Encargada de Atención: al incorporar el Módulo de Autenticación Ligera, el propio cliente puede consultar el estado de su solicitud sin intervención manual, y al migrar la base de datos, la información del pedido queda respaldada automáticamente en vez de depender de que alguien recuerde hacer una copia manual.

### 3.2 Cambios en aplicaciones, infraestructura y flujos de información

**TO-BE de Aplicaciones** (extiende el C2 del Taller 3, `entrega/to-be-aplicaciones-final.drawio`): se agrega un **Módulo de Autenticación Ligera**, que verifica un código enviado por WhatsApp antes de permitir consultar o modificar una solicitud existente, y se mejora la App Web Oasis con una casilla de autorización de tratamiento de datos y enlace a la política de privacidad. La API REST y la Base de Datos Oasis no cambian como contenedores; el resto de la arquitectura del Taller 3 se conserva sin modificaciones.

**TO-BE de Tecnología** (extiende el mapa del Taller 4, `entrega/to-be-tecnologia-final.drawio`): la Base de Datos Oasis migra de un plan gratuito sin garantías a **Supabase Pro**, con cifrado en reposo nativo y backup automático diario. WhatsApp Business API pasa a usarse también como canal del código de verificación, no solo de notificaciones de estado. La capa de Aplicación (App Web + API REST) se mantiene como instancia única en esta iteración — ese riesgo de disponibilidad queda como backlog explícito para la siguiente.

### 3.3 Controles de seguridad integrados
El Módulo de Autenticación Ligera integra directamente la mitigación propuesta en el Taller 5 para la amenaza T1 (Spoofing): verificación mínima antes de exponer o modificar los datos de una solicitud. La migración a Supabase Pro integra la mitigación de la amenaza T4 (Information Disclosure): cifrado en reposo y control de acceso reforzado sobre los datos personales de los clientes.

---

## 4. Análisis de beneficios y riesgos

> El detalle completo de esta sección (brechas, riesgos, capacidades y paquetes de trabajo) está consolidado en `entrega/matriz-brechas.xlsx`, con una hoja por tabla.

| Mejora / Solución | Beneficio de negocio | Beneficio tecnológico/seguridad | Riesgo, limitación o dependencia de implementación |
|---|---|---|---|
| Migración a Supabase Pro | Continuidad del negocio ante pérdida accidental de datos; reduce exposición legal ante la SIC | Cifrado en reposo nativo y backup automático diario | Depende de la aprobación del costo adicional (≈USD 25/mes) por el Propietario y de confirmar por escrito el alcance del cifrado con el proveedor |
| Módulo de Autenticación Ligera | El cliente puede confiar en que solo él puede ver o cambiar su propio pedido | Cierra la amenaza de Spoofing (T1) del Taller 5 | Depende de que la integración con WhatsApp Business API (Fase 3) esté disponible antes de poder enviar el código |
| Casilla de autorización + política de datos | Cumplimiento formal desde el lanzamiento, reduciendo exposición ante la SIC | Cierra 2 brechas normativas del Taller 6 | Depende de que el Propietario revise y apruebe el texto legal antes de publicarlo |

### Capacidades de negocio y paquetes de trabajo

| Capacidad | Madurez AS-IS | Madurez TO-BE | Qué la explica |
|---|---|---|---|
| Recibir y gestionar solicitudes de pedido | 2 | 3 | Proceso manual sin registro centralizado (Taller 1); mejora parcial, el sistema de gestión en sí no cambia en esta iteración |
| Elaborar arreglos florales | 3 | 3 | Sin brechas diagnosticadas; no se toca en esta iteración |
| Proteger los datos personales de los clientes | 1 | 4 | Riesgo Alto repetido en Talleres 4, 5 y 6; cierra con la migración a Supabase Pro |
| Verificar la identidad de quien gestiona un pedido | 1 | 3 | Sin mecanismo hoy (Taller 5, T1); cierra parcialmente con el Módulo de Autenticación Ligera |
| Cumplir la normativa de protección de datos | 1 | 3 | 12 de 13 ítems en Parcial (Taller 6); esta iteración cierra los 2 más críticos |

| Paquete de trabajo | Brechas que incluye | Capacidad que mejora | Tipo |
|---|---|---|---|
| WP1 · Protección de datos del cliente | Migración a Supabase Pro (decisión en `matriz-decision-backup-oasis.md`) | Proteger los datos personales de los clientes (1 → 4) | Quick win · 1-2 semanas |
| WP2 · Confianza e identidad en el pedido | Módulo de Autenticación Ligera + casilla de autorización y política de datos | Verificar identidad (1 → 3) y Cumplir normativa (1 → 3) | Quick win · 2-3 semanas |

---

## Anexos
- Diagrama TO-BE de Aplicaciones (extiende el C2 del Taller 3): `entrega/to-be-aplicaciones-final.drawio`
- Diagrama TO-BE de Tecnología (extiende el mapa del Taller 4): `entrega/to-be-tecnologia-final.drawio`
- Matriz de brechas (Gap Analysis), con beneficios, riesgos, capacidades y paquetes de trabajo: `entrega/matriz-brechas.xlsx`
- Matriz de decisión ponderada (cifrado en reposo y backup de la base de datos): `entrega/matriz-decision-backup-oasis.md`

---

**Formato de entrega:** documento único de máximo 6 páginas + anexos, entregado como PDF con nombre `EquipoX_Mejora_Arquitectura.pdf` para la entrega formal ante el docente, si lo pide aparte.

---

_Este documento hace parte de la entrega del Taller 7 (Opportunities & Solutions) del curso AREM - Universidad de La Sabana._
