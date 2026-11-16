
#  Actividad 4 — Reducir el volumen con un filtro

### 1. ¿Qué perdiste de evidencia al aplicar el filtro? ¿Qué ganaste?

**Perdí:** evidencia de operaciones que no fueran `insert`, `update` o `remove`, como las consultas de lectura (`query`). Esto significa que ya no puedo observar todas las actividades realizadas sobre la base de datos.

**Gané:** una bitácora más específica, enfocada en las modificaciones de información. Esto reduce el volumen de registros almacenados y facilita identificar cambios realizados en los documentos.

### 2. ¿Qué filtro usarías si solo interesaran operaciones sobre la colección `clientes`?

Utilizaría un filtro sobre el campo `ns` (namespace), especificando la colección `comercial.clientes`.

El comando sería:

```javascript
db.setProfilingLevel(1, {
  filter: {
    ns: "comercial.clientes"
  }
})
```

De esta manera, el perfilador registraría las operaciones realizadas sobre la colección `clientes` de la base de datos `comercial`, independientemente del tipo de operación.
