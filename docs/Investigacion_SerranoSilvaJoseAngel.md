Fase 1\.  
1\. Métodos HTTP Semánticos en REST

| Método | Uso semántico | Operación CRUD |
| :---- | :---- | :---- |
| **GET** | Obtener/leer información del servidor | Read |
| **POST** | Crear un nuevo recurso | Create |
| **PUT** | Reemplazar/actualizar un recurso completo | Update |
| **DELETE** | Eliminar un recurso | Delete |

2\. Estructura JSON  
Por sus siglas JavaScript Object Notation (Notación de Objetos de JavaScript), desplazó a XML por su simplicidad y eficiencia en el intercambio de datos.

| Característica | JSON | XML |
| :---- | :---- | :---- |
| Peso | Ligero, menos sobrecarga | Más pesado, tags redundantes |
| Legibilidad | Fácil para humanos y programadores | Más complejo |
| Parsing | Más rápido y sencillo | Requiere parser más complejo |
| Basado en | pares clave-valor `{}` | tags anidados |

3\. Códigos de Error HTTP.

| Familia | Significado | Rango |
| :---- | :---- | :---- |
| **4XX** | Error del **cliente** (Frontend) | 400-499 |
| **5XX** | Error del **servidor** (Backend) | 500-599 |

Diferencias.

| Código | Nombre | Explicación |
| :---- | :---- | :---- |
| **400** | Bad Request | La solicitud está **mal formada** o contiene datos inválidos |
| **404** | Not Found | El recurso **no existe** en el servidor |
| **503** | Service Unavailable | El servidor **no puede manejar** la solicitud (sobrecarga, mantenimiento) |

4\. Especificación OpenAPI (Swagger).  
OpenAPI es un estándar para describir APIs REST de forma estandarizada. Swagger es la herramienta más popular para implementarlo.

Objetivo principal: Entregar documentación interactiva y automática de la API al equipo de desarrollo, funcionando como un "contrato" que define:

* Endpoints disponibles  
* Parámetros aceptados  
* Respuestas esperadas  
* Esquemas de datos

Esto permite que front-end y back-end trabajen en paralelo sin ambigüedades.

5\. Contratos de Interfaz.  
Una Interface es un contrato que define qué métodos debe implementar una clase, sin definir su implementación interna.

¿Por qué es un "contrato inmutable"?

* Obliga a las clases que la implementan a cumplir con los métodos definidos  
* Crea una capa de abstracción entre lógica de negocio y base de datos  
* Permite cambiar la implementación sin romper el código que usa la interfaz  
* Garantiza que siempre existan ciertos métodos disponibles, independientemente de cómo se implementen

Esto desacopla el código y facilita pruebas, mantenimiento y escalabilidad  
6\. Manejo de Excepciones y Resiliencia.  
Stacktrace \= Lista detallada de errores en texto que muestra:

* La secuencia de llamadas a funciones que llevaron al error  
* Nombres de archivos y números de línea  
* Tipos de excepciones y mensajes

¿Por qué NUNCA mostrarlo crudo al usuario?

| Razón | Explicación |
| :---- | :---- |
| **Seguridad** | Revela estructura interna del sistema, rutas de archivos, nombres de tablas |
| **Experiencia de usuario** | Confunde y asusta al usuario no técnico |
| **Información sensible** | Puede exponer credenciales, queries SQL, o lógica empresarial |
| **Responsabilidad** | Debe mostrarse un mensaje amigable \+ loguear el stacktrace internamente |

Fase 2\.  
Mi diagrama C4 se asemeja mas al escenario 1 por las siguientes razones:

1\. Separación Física y Redes (Frontend vs. Backend)  
En mi arquitectura, la App Móvil (Contenedor 1\) se ejecuta en el teléfono del usuario, mientras que la lógica (Contenedor 3\) y la base de datos (Contenedor 4\) viven en un servidor remoto (VPS). No comparten memoria ni sistema operativo. La única forma en la que el celular puede pedirle al servidor que mueva dinero es a través de la red, enviando un "Request Body" en formato JSON (monto, cuenta destino) al API Gateway (Contenedor 2).

2\. El Manejo de Errores (El Código 503\)  
En el Escenario 1 se menciona que si hay un error de "Caja Rota", se devuelve un HTTP 503 Service Unavailable. Esto es exactamente lo que hace el patrón Circuit Breaker que se propuso. Si el contenedor de MySQL pierde conexión, el backend en Python no lanza una excepción que cierre el sistema; la atrapa y le responde al API Gateway con un código HTTP 503 para que la app móvil muestre el mensaje amigable de "demoras técnicas".

3\. Independencia de Despliegue  
El backend de Python en el puerto 8000 se comunica con MySQL en el puerto 3306 a través de protocolos de red internos, no mediante invocaciones directas a la memoria interna del motor de base de datos como pasaría en el Escenario 2\.