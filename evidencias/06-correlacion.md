
# Actividad 6 — Correlacionar un cambio de configuración con su autor

### 1. ¿Qué usuario cambió la configuración del perfilador? ¿Qué dos eventos lo demuestran?

La evidencia proporcionada muestra los cambios de configuración del perfilador, pero no permite identificar directamente qué usuario los realizó.

Los eventos disponibles demuestran que:

- A las `15:04:10`, la configuración del perfilador de la base `comercial` cambió de nivel `0` a nivel `2`.
- A las `15:28:38`, se configuró el nivel `1` con un filtro para registrar operaciones `insert`, `remove` y `update`.

Para identificar al responsable sería necesario relacionar estos eventos con los registros de autenticación y la sesión correspondiente. La evidencia mostrada no incluye esa relación.

### 2. ¿Qué condición debe cumplirse para que esta correlación sea válida? (Piensa en qué pasa si el servidor se reinicia y los números de conexión vuelven a empezar.)

Es necesario que los eventos pertenezcan al mismo periodo de ejecución del servidor y que no haya ocurrido un reinicio entre ellos.

Esto es importante porque los identificadores de conexión, como `conn9` o `conn31`, pueden reutilizarse después de reiniciar MongoDB.

Por lo tanto, no basta con comparar únicamente el número de conexión: también se deben considerar las fechas y horas de los eventos y el periodo de ejecución del servidor.

### 3. ¿Por qué es relevante que quede registrado un cambio de configuración del propio mecanismo de registro?

Porque permite identificar cuándo se modifica el mecanismo encargado de recopilar la evidencia y qué configuración tenía antes y después del cambio.

Esto es importante para detectar posibles intentos de reducir o desactivar el registro de operaciones, conocer los periodos en los que determinadas actividades no quedaron registradas y mantener la trazabilidad de las modificaciones realizadas al sistema de auditoría.

Además, permite comprobar si la configuración del perfilador era adecuada durante el momento en que ocurrió un incidente.
