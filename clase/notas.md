# Registro de Trabajo en Clase - Taller 7

## Fecha de la sesión
27/09/2025

## Integrantes presentes
- Esteban Díaz
- Juliana Moreno

## Actividades realizadas en clase

Se retomó el AS-IS de RedExpress ya construido en los Talleres 3 (C1/C2) y 4 (mapa de infraestructura y diagnóstico) para proponer, sobre esa misma base, la arquitectura objetivo (TO-BE), siguiendo la metodología en 4 partes de la guía paso a paso: Diagnóstico inicial → Propuesta de mejoras → Visualización TO-BE → Análisis de beneficios y riesgos.

- **Diagnóstico inicial:** se respondieron las tres preguntas orientadoras apoyándose únicamente en los hallazgos ya documentados en el Taller 4: el balanceador de carga de instancia única, la base de datos con escritura centralizada en Bogotá y la dependencia de Medellín del motor de rutas de Bogotá.
- **Propuesta de mejoras:** se hizo una lluvia de 8 ideas sin censura (técnicas, de proceso y de comunicación) y se priorizaron las 3 mejoras técnicas, por ser las únicas con brecha diagnosticada y evidenciada en el Taller 4. Para la brecha del balanceador de carga, que admite varias soluciones, se decidió con la matriz de decisión ponderada (8 pasos), comparando activo-pasivo, activo-activo y mantener el balanceador único reforzando el monitoreo.
- **Visualización TO-BE:** se extendió el C2 del Taller 3 agregando el Motor de Rutas - Medellín, y se extendió el mapa de infraestructura del Taller 4 agregando el balanceador pasivo y la BD de Medellín (partición regional).
- **Análisis de beneficios y riesgos:** se construyó la matriz de brechas cerradas frente a los beneficios esperados, se documentaron los riesgos/dependencias de implementación de cada solución, y se agruparon las brechas por capacidad de negocio en dos paquetes de trabajo (WP1 y WP2).
- Herramientas usadas: draw.io (heredado de Talleres 3 y 4) y Markdown para el documento de entrega.

## Boceto inicial del modelo

El boceto TO-BE parte directamente de los diagramas finales de los Talleres 3 y 4 (`c2-contenedores-final.drawio` y `mapa-final.drawio`), agregando los elementos nuevos marcados como "NUEVO" en la versión visual del taller.

## Tareas definidas para complementar el taller

| Tarea asignada | Responsable | Fecha estimada |
|----------------|-------------|----------------|
| Modelado final en draw.io (TO-BE de Aplicaciones y de Tecnología) | Juliana Moreno | 27/09/2025 |
| Investigación y referencias | Juliana Moreno | 27/09/2025 |
| Redacción del informe | Esteban Díaz | 27/09/2025 |

---

_Este documento resume el trabajo colaborativo realizado durante la sesión del Taller 7 en el curso AREM - Universidad de La Sabana._
