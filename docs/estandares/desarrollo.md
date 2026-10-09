# Estándares de desarrollo

Criterios que aplica el agente desarrollador al implementar y que `/verificar` usa al revisar. Valen para cualquier lenguaje; los ejemplos de estructura asumen un backend.

## Principio rector

La solución correcta es la más simple que cumple los criterios de aceptación de la spec. Cada abstracción tiene que resolver un problema presente, no uno imaginado.

## Diseño orientado a objetos

- Cada clase tiene una responsabilidad clara que se puede describir en una oración.
- El estado interno se encapsula; se expone comportamiento, no datos.
- Se prefiere la composición sobre la herencia.
- Los objetos del dominio se crean en un estado válido y lo mantienen.

## SOLID

Se aplica como criterio de diseño, no como checklist:

| Principio | Señal de que se está violando |
|-----------|-------------------------------|
| Responsabilidad única | La clase cambia por motivos distintos |
| Abierto/cerrado | Agregar un caso nuevo obliga a editar un `if`/`switch` que ya existía en varios lugares |
| Sustitución de Liskov | Una subclase lanza errores o ignora métodos de la clase base |
| Segregación de interfaces | Una implementación deja métodos vacíos o sin uso |
| Inversión de dependencias | El dominio importa frameworks, ORM o clientes HTTP |

## Patrones de diseño

- Se usa un patrón solo cuando resuelve un problema que existe en la spec actual.
- Antes de introducir uno, la pregunta es: ¿qué se rompe o se duplica si no lo uso?
- Si la respuesta es "nada todavía", no se usa.

## Arquitectura

Arquitectura limpia / hexagonal:

```
entrada (API, CLI)  ──►  aplicación (casos de uso)  ──►  dominio
                                   │
                                   ▼
                     puertos (interfaces)  ◄──  adaptadores (BD, HTTP, colas)
```

- El dominio no depende de frameworks, bases de datos ni servicios externos.
- Los casos de uso orquestan el dominio y hablan con el exterior a través de puertos.
- Los adaptadores implementan los puertos y son reemplazables.
- La estructura de carpetas comunica el negocio, no el framework.

## KISS y YAGNI

- No se agregan parámetros, configuraciones ni extensiones "por si acaso".
- No se generaliza hasta tener al menos dos casos reales que lo justifiquen.
- Código que no se usa se elimina.

## Código limpio

- Nombres expresivos: el nombre explica qué hace o qué representa, sin necesidad de comentario.
- Funciones cortas, con un solo nivel de abstracción.
- Sin duplicación de lógica de negocio.
- Los comentarios explican el porqué, nunca el qué.
- Los errores se manejan de forma explícita; no se silencian excepciones.

## Tests

- Ciclo TDD: test que falla → implementación mínima → refactor.
- Los tests verifican comportamiento observable, no detalles de implementación.
- Cada test prueba una sola cosa y su nombre describe el escenario y el resultado esperado.
- Los servicios externos se reemplazan por dobles de prueba en el borde, a través de los puertos.
- Los tests son deterministas: sin dependencia de hora, orden de ejecución ni red.
