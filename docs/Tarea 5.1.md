Tabla Cuentas.  
Guarda información sobre el usuario que realiza movimientos en la App

| Columna | Tipo de dato | Llave | Nulo (Null) | Descripción |
| :---- | :---- | ----- | :---- | :---- |
| ID | INT | Primaria | No | Identificador único para cada usuario |
| numero\_cuenta | varchar(18) | \- | No | Numero de cuenta bancaria asociado al usuario |
| saldo | decimal(15,2) | \- | No | Saldo disponible de cada usuario para realizar transferencias o retiros |
| estado | ENUM('ACTIVA', 'BLOQUEADA', 'CERRADA') | \- | No | Estado de la cuenta del usuario |
| fecha\_creacion | timestamp | \- | No | Fecha de creación de la cuenta del usuario |

Tabla Historial\_transferencias  
Aquí se guardan las transferencias realizadas por los usuarios.

| Columna | Tipo de dato | Llave | Nulo (Null) | Descripción |
| :---- | :---- | ----- | :---- | :---- |
| Id | Char(36) | Primaria | No | Clave única que identifica cada transferencia |
| cuenta\_origen\_id | int | foranea | No | id de la cuenta que realiza la transferencia |
| cuenta\_destino\_id | int | Foranea | No | id de la cuenta que recibe el dinero de la transferencia |
| monto | decimal(15,2) | \- | No | Cantidad de dinero transferida |
| estado | ENUM('PENDIENTE', 'EXITOSA', 'FALLIDA')  | \- | No | Estado actual de la transferencia |
| motivo\_fallo | varchar(255) | \- | DEFAULT | Motivo por el cual la transferencia no se realizo con exito |
| fecha\_transaccion | timestamp | \- | No | Fecha en la que se realizó la transferencia |

