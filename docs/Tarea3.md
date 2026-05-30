![Diagrama de Contexto](/docs/img/Diagramadecontexto.jpeg)

![Diagrama de Cpntenedores](/docs/img/Diagramadecontenedores.jpeg)


### **Análisis de la caja rota**

### **Si el contenedor de la Base de Datos se cae por completo o pierde conexión durante 10 minutos…**

###  **1\. ¿Qué ocurre con el resto de los contenedores?**

* **El Frontend (App Móvil):** No colapsa ni se cierra inesperadamente porque se ejecuta en el teléfono del usuario, totalmente ajeno a la infraestructura. Sin embargo, cualquier acción que requiera datos (ver saldo, transferir) se quedará esperando una respuesta.  
* **El API Gateway:** Sigue activo y enrutando tráfico.  
* **El Backend (Servicio de Transferencias en Python):** El contenedor sigue ejecutándose, pero comienza a fallar internamente. Si no está configurado correctamente, las peticiones entrantes se acumularán esperando a que la base de datos responda, lo que puede agotar la memoria o los hilos de ejecución del servidor. El backend colapsaría por saturación, no por un error de código.

### **2\. ¿Cómo se entera el usuario? (Feedback Visual)**

El usuario **nunca** debe ver un mensaje técnico como "Error 500: Connection Refused" o una pantalla en blanco con un icono de carga infinito.

La respuesta visual debe ser controlada y dar tranquilidad financiera:

* **Manejo del Código HTTP:** El backend debe capturar el error de conexión a la base de datos y devolver de inmediato un código HTTP 503 Service Unavailable al API Gateway, y de ahí al Frontend.  
* **La Interfaz de Usuario (UI):** La app móvil intercepta ese código 503 y muestra una pantalla de error amigable y clara:  
  *"Estamos experimentando demoras técnicas en el sistema de transferencias. No te preocupes, **tu dinero y tus saldos están seguros**. Por favor, intenta tu operación de nuevo en unos minutos."*  
* **Bloqueo de Interfaz:** Además del mensaje, el botón de "Transferir" debe deshabilitarse temporalmente para evitar que el usuario, en su desesperación, presione el botón 20 veces por segundo, lo que empeoraría la saturación del sistema.

### **3\. Estrategia de manejo de errores para evitar el colapso**

Para que el sistema sea resiliente y se recupere con gracia de esos 10 minutos de caída, se implementaran estos tres mecanismos:

* **Timeouts Estrictos (Tiempos de espera):** El backend no debe esperar a la base de datos indefinidamente. Configurar un timeout agresivo. Si MySQL no responde en ese tiempo, el backend aborta el intento y asume que está caída.  
    
* **Patrón Circuit Breaker (Cortocircuito):**  
  * **Cerrado (Normal):** El tráfico fluye a la base de datos.  
  * **Abierto (Fallo):** Si el sistema detecta, por ejemplo, 5 fallos consecutivos por timeout, el circuito "salta" a estado Abierto. Durante los siguientes minutos, el backend rechaza *todas* las peticiones de los usuarios instantáneamente con un error 503, **sin intentar tocar la base de datos**.   
  * **Semi-abierto (Prueba):** Después de un tiempo, el circuito deja pasar una o dos peticiones para ver si la base de datos ya despertó. Si responden bien, el circuito se cierra y todo vuelve a la normalidad.  
* **Health Checks y Auto-reinicio:** A nivel de infraestructura, debes tener configurado un HEALTHCHECK. Si se detecta que el contenedor de MySQL dejó de responder a un "ping" básico, automáticamente intentará matarlo y levantar una nueva instancia limpia de la base de datos.

### **Diccionario de contenedores**

### **Contenedor: Mobile Application (Frontend)**

* **Tecnología:** React Native (Móvil).  
* **Responsabilidad:** Renderizar la interfaz de usuario, capturar el monto a transferir, validar formatos iniciales (ej. montos positivos), solicitar el NIP de confirmación e interceptar códigos HTTP para mostrar mensajes amigables (ej. en caso de caída del backend).  
* **Despliegue:** Tiendas de aplicaciones (App Store / Play Store) o `localhost:3000` vía `npm start` para desarrollo.

### **Contenedor: API Gateway**

* **Tecnología:** Nginx o Kong.  
* **Responsabilidad:** Ser la única puerta de entrada pública al sistema. Recibe las peticiones del frontend, valida los tokens de sesión (autenticación) y enruta el tráfico hacia el microservicio interno correspondiente.  
* **Despliegue:** Contenedor Docker exponiendo los puertos `80` (HTTP) y `443` (HTTPS) hacia el exterior.

### **Contenedor: Transfer Service (Backend Core)**

* **Tecnología:** Python 3.10+ / FastAPI (Estructurado con Arquitectura Hexagonal).  
* **Responsabilidad:** Contener la lógica de negocio pura. Validar que el saldo origen sea suficiente, ejecutar el patrón *Circuit Breaker* en caso de fallos, coordinar la transacción con la base de datos y emitir eventos de éxito.  
* **Despliegue:** Contenedor Docker en el puerto local `8000` (ejecutado vía Uvicorn/Gunicorn), operando en una red interna aislada del exterior.

### **Contenedor: Database**

* **Tecnología:** MySQL 8.0.  
* **Responsabilidad:** Garantizar la transaccionalidad ACID absoluta. Almacenar de forma segura el estado de las cuentas (saldos) y el registro histórico inmutable de las transferencias.  
* **Despliegue:** Contenedor Docker en el puerto local `3306`, con un volumen persistente mapeado al servidor host para evitar pérdida de datos si el contenedor se reinicia. No expuesto a internet.

### **Contenedor: Message Broker (Bus de Eventos)**

* **Tecnología:** Redis (Pub/Sub) o RabbitMQ (Redis es ideal si buscas mantener el consumo de RAM muy bajo).  
* **Responsabilidad:** Recibir el evento `TransferenciaRealizada` emitido por el Backend y encolarlo para que otros servicios secundarios (como el envío de correos o SMS de comprobante) lo procesen en segundo plano sin ralentizar la respuesta al usuario.  
* **Despliegue:** Contenedor Docker en el puerto local `6379` (Redis) o `5672` (RabbitMQ), únicamente accesible para la red de contenedores internos.

