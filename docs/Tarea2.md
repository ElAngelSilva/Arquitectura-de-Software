1. Decisión.  
   1. Estilo Macro.  
      Microservicios, pues el área de Transferencias será un servicio totalmente independiente a otros apartados como lo serían los préstamos o la gestión de usuarios.  
   2. Estilo interno.  
      Arquitectura Hexagonal y Arquitectura por Eventos  
2. Boceto

![Diagrama informal](/img/Diagrama.png)

3. Justificación  
   1. Se decidió usar microservicios pues si algún otro apartado de la aplicación colapsa, los usuarios pueden seguir usando los demás para todos.  
   2. Se implementará Arquitectura por Eventos pues esto evitará los cuellos de botella. El servicio de transferencias solo se encarga de mover el dinero.  
   3. La arquitectura hexagonal ayudará a que algunas “reglas financieras” se mezclen  con el código de la base de datos o la API. También brinda facilidad a la hora de realizar pruebas unitarias y flexibilidad si se quisiera cambiar el modelo de la base de datos.
 
