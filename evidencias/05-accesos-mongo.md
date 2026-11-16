
# Actividad 5 — Accesos en la bitácora de mongod

### 1. ¿Qué campo distingue un acceso fallido de uno exitoso?

El campo `msg` permite distinguir ambos resultados:

- **Acceso exitoso:** aparece el mensaje `Successfully authenticated`.
- **Acceso fallido:** aparece el mensaje `Failed to authenticate`.

También se puede utilizar el campo `result`:

- `result: 0` indica autenticación exitosa.
- Un valor distinto de `0` indica un error de autenticación.

### 2. ¿Qué diferencia hay entre el error de contraseña incorrecta y el de usuario inexistente? ¿Por qué el cliente recibe el mismo mensaje en ambos casos?

La diferencia está en la causa del error:

- **Contraseña incorrecta:** el usuario existe, pero las credenciales proporcionadas no coinciden. La evidencia muestra `storedKey mismatch`, con código `18`.
- **Usuario inexistente:** MongoDB no encuentra el usuario solicitado en la base de datos de autenticación. La evidencia muestra `UserNotFound`, con código `11`.

El cliente recibe un mensaje de autenticación fallida en ambos casos porque MongoDB utiliza una respuesta general para evitar revelar si un nombre de usuario existe o no. Esto ayuda a prevenir ataques de enumeración de usuarios.

### 3. ¿Por qué el nombre de la aplicación del intento fallido es `mongosh 2.10.0` y no `app-ventas-web`?

Porque el intento fallido se realizó directamente desde la terminal utilizando `mongosh`, en lugar de utilizar la aplicación web.

MongoDB registra el nombre de la aplicación que establece la conexión, según los metadatos enviados por el cliente.

Por eso:

- `app-ventas-web` identifica las conexiones realizadas desde la aplicación web.
- `mongosh 2.14.2` identifica las conexiones realizadas desde la consola de MongoDB, según la evidencia proporcionada.

Esto permite diferenciar los accesos de la aplicación de los intentos realizados manualmente desde la terminal.
