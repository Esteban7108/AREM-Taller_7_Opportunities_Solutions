# 📄 Informe Técnico del Taller

## 🔖 Nombre del Taller
Taller 7 - Opportunities & Solutions

## 👥 Integrantes del equipo
- Esteban Díaz 
- Juliana Moreno 

**Fecha:** 27/09/2025

## 🧠 Descripción general del trabajo

El objetivo de este taller es proponer la arquitectura objetivo (TO-BE) de RedExpress, retomando el AS-IS ya construido en los Talleres 3 (C1/C2) y 4 (mapa de infraestructura y diagnóstico), y siguiendo la metodología en 4 partes de la guía paso a paso: Diagnóstico inicial, Propuesta de mejoras, Visualización TO-BE, y Análisis de beneficios y riesgos.

## 🔧 Proceso de desarrollo

Se partió del diagnóstico ya documentado en el Taller 4 para identificar tres brechas técnicas: el balanceador de carga de instancia única, la base de datos con escritura centralizada en Bogotá, y la dependencia total de Medellín respecto del módulo de rutas de Bogotá. A partir de estas brechas se hizo una lluvia de 8 ideas de mejora sin censura inicial (técnicas, de proceso y de comunicación), priorizando las 3 mejoras técnicas por ser las únicas con brecha diagnosticada formalmente en el curso.

Para la brecha del balanceador de carga, que admitía tres soluciones distintas (activo-pasivo, activo-activo, o mantener el balanceador único reforzando el monitoreo), se aplicó la matriz de decisión ponderada de 8 pasos: se definieron los criterios (disponibilidad, costo, complejidad, tiempo) con sus pesos validados desde la perspectiva del negocio, se generaron las tres opciones, se consultó a los responsables técnicos y operativos, y se puntuó cada opción justificando cada celda. El resultado, tras aplicar un criterio eliminatorio de disponibilidad, fue elegir el esquema activo-pasivo.

Con esa decisión tomada, se construyó el TO-BE extendiendo el C2 del Taller 3 (agregando el Motor de Rutas - Medellín) y el mapa de infraestructura del Taller 4 (agregando el Balanceador de Carga - Pasivo y la BD Medellín particionada por región), representados en la versión visual interactiva del taller con el interruptor AS-IS/TO-BE.

## 🧩 Análisis del modelo propuesto

El TO-BE resultante conserva la misma estructura de zonas de RedExpress (Clientes, Borde/Global, Región Bogotá, Región Medellín) trabajada en talleres anteriores, pero elimina el punto único de falla del balanceador y descentraliza tanto la escritura de datos como el procesamiento de rutas hacia la región Medellín. Cada elemento agregado —el balanceador pasivo, la BD de Medellín y el módulo de rutas de Medellín— corresponde directamente a una brecha específica del diagnóstico del Taller 4, manteniendo la trazabilidad AS-IS → TO-BE exigida por el taller.

### Supuestos

Se asumieron como válidos los datos de costo, tiempo y capacidad utilizados en la matriz de decisión ponderada del balanceador de carga (cotizaciones del proveedor cloud y estimaciones del líder de infraestructura), documentados como supuestos del ejercicio.


## 📋 Tabla de actores, entidades o componentes

| Nombre del elemento | Tipo | Descripción |
|---|---|---|
| Balanceador de Carga - Activo | Infraestructura (existente) | Distribuye el tráfico entrante hacia los API Gateway regionales |
| Balanceador de Carga - Pasivo | Infraestructura (nuevo) | Segunda instancia que toma el tráfico ante conmutación por falla |
| BD Bogotá | Base de datos (existente) | Partición regional de la base de datos distribuida |
| BD Medellín | Base de datos (nuevo) | Partición regional que reduce la dependencia de escritura centralizada en Bogotá |
| Motor/Módulo de Rutas - Bogotá | Contenedor (existente) | Calcula rutas óptimas para la región Bogotá |
| Motor/Módulo de Rutas - Medellín | Contenedor (nuevo) | Réplica que permite a Medellín procesar rutas sin depender de Bogotá |
| Servicio de Monitoreo y Alertas | Servicio (existente) | Recibe reportes de ambos gateways regionales |

## 🔍 Investigación complementaria

### Tema investigado:
Metodología de matrices de decisión ponderada para el análisis de brechas de arquitectura, y patrones de alta disponibilidad (activo-pasivo vs. activo-activo) para balanceadores de carga.

### Resumen:
La literatura de arquitectura empresarial coincide en que una propuesta TO-BE solo es defendible ante un comité cuando cada elemento nuevo se traza a una brecha concreta del AS-IS, evitando que el diseño se convierta en una lista de deseos desconectada del diagnóstico previo. En esa línea, la matriz de decisión ponderada —con criterios definidos por el negocio, pesos que no se delegan al equipo técnico, y un análisis de sensibilidad frente a escenarios alternativos de esos pesos— es el mecanismo estándar para elegir entre opciones técnicamente viables sin caer en la preferencia subjetiva ("me gusta más B"). Aplicado al caso del balanceador de carga de RedExpress, el análisis confirmó que un esquema activo-pasivo ofrece el mejor balance entre disponibilidad, costo y tiempo de implementación frente a la campaña de fin de año, mientras que un esquema activo-activo, aunque elimina por completo la ventana de conmutación, resulta más costoso y lento de implementar de lo que el cronograma permite.

## 📚 Referencias

Las referencias utilizadas y la información de investigación complementaria se encuentran registradas en `referencias.md`.

---

_Este documento hace parte de la entrega del Taller 7 del curso AREM (Arquitectura Empresarial) - Universidad de La Sabana._
