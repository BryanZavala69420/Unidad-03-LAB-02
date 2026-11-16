
# Actividad 2 — El incidente sin perfilado

## 1. ¿Puedes demostrar qué cambió? ¿Y quién se conectó a esa hora?

No, con la evidencia obtenida no es posible demostrar qué dato cambió ni identificar con certeza quién realizó el cambio.

Esto se debe a que:

- `db.getProfilingStatus()` devuelve `was: 0`, lo que indica que el perfilador está desactivado.
- `db.system.profile.countDocuments()` devuelve `0`, por lo que no existen registros de operaciones en la colección de perfilado.
- La búsqueda del valor `GAMA780912XXX` en la bitácora devuelve `0`, por lo que no se encontró evidencia de ese cambio.
- La búsqueda de `app-ventas-web` inicialmente devuelve un arreglo vacío, pero posteriormente aparecen eventos de conexión.

En los últimos resultados se observan eventos `NETWORK` y `ACCESS`, incluyendo mensajes como `client metadata` y `Successfully authenticated`.

Por lo tanto, sí se puede demostrar que hubo conexiones autenticadas a esa hora, pero la evidencia mostrada no permite determinar qué usuario de la aplicación realizó un cambio específico ni qué dato modificó.

## 2. ¿Por qué `mongosh` abre varias conexiones (`conn8`, `conn9`, …) para una sola instrucción? ¿Cuál es la que se autentica?

`mongosh` puede abrir varias conexiones porque utiliza diferentes canales para comunicarse con MongoDB y realizar tareas auxiliares, como obtener metadatos del servidor, ejecutar operaciones y mantener conexiones internas.

Cada conexión recibe un identificador diferente, por ejemplo, `conn8`, `conn9`, `conn10`, etc.

Según la evidencia obtenida, las conexiones que muestran el mensaje `Successfully authenticated` son:

- `conn12`
- `conn13`
- `conn14`

En cambio, `conn11` muestra el mensaje `Connection not authenticating`, lo que significa que esa conexión no estaba realizando autenticación.

Por lo tanto, no todas las conexiones realizan autenticación. Para identificar cuál ejecutó una operación específica, sería necesario relacionar los eventos de conexión con los registros de las operaciones correspondientes.
