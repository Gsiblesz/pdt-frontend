# Calculadora de Procesos de Panadería — Guía rápida

Esta guía resume cómo usar la app (frontend), cómo enviar resultados a la base de datos, y cómo resolver los problemas más comunes.

## Qué incluye la app

- Medición
  - Cronometrado de 3 procesos por amasadora (Esponja, Masa, Mantequilla)
  - Variables generales por día: Fecha, TA (temp. ambiente), H (humedad), N° personal
  - Campos por máquina: Tipo de masa (con "Otro"), Hielo, Temperatura de la masa, Lote, Observaciones
  - Cálculo de tiempos muertos (TM1, TM2) y total de la máquina
- Resultados
  - Subpestaña Locales: ver/filtrar lo guardado localmente, enviar por tarjeta o “Enviar todo a la BD” por fecha, borrar local
  - Subpestaña Base de datos: listar registros remotos, refrescar, borrar por ítem y “Borrar todo BD”
- Exportar
  - PDF: exporta la vista de resultados sin botones
  - CSV: filas por proceso con factores diarios y de máquina compactados

## Flujo de uso recomendado

1) En Medición, selecciona la Fecha y completa TA, H, N° personal (arriba).
2) En cada amasadora, registra Tipo de masa, Hielo, Lote, Temp. de masa y Observaciones.
3) Usa Iniciar/Detener/Reset para los 3 procesos; se calculan TM1, TM2 y Total.
4) Pulsa “Guardar resultados” en cada tarjeta completada.
5) Ve a Resultados → Locales para revisar, exportar, o “Enviar todo a la BD”.
6) Ve a Resultados → Base de datos para refrescar, borrar por ítem o borrar todo.

## Enviar a la Base de datos

- Por tarjeta: botón “Enviar a BD” en cada tarjeta en Resultados → Locales.
- Por fecha: botón “Enviar todo a la BD” (usa la Fecha seleccionada en Medición y agrupa todas las amasadoras de ese día).

La API usada por defecto es:

- https://pdt-backend-1.onrender.com

Puedes cambiarla temporalmente desde Resultados → Base de datos → “Más opciones” → “API override”.

## Vista “Base de datos”

- Refrescar: trae la lista desde el backend (GET /registros)
- Borrar: elimina un registro por ID (DELETE /registros/:id)
- Borrar todo BD: elimina todos los registros (DELETE /registros)

Notas:
- Si borras “todo BD” y luego refrescas, la lista debe quedar vacía.
- Si intentas borrar un ID inexistente, el backend responde 404 con { error: "Registro no encontrado" }.

## Exportar

- PDF: “Exportar Resultados (PDF)” captura la vista (sin botones) para informe rápido.
- CSV: “Exportar Resultados (CSV)” genera filas por proceso con campos principales (incluye TM1, TM2 y total).

## Payload que envía el frontend (ejemplo mínimo)

```json
{
  "fecha": "2025-09-22",
  "tempAmbiente": "25",
  "humedad": "48",
  "personal": "6",
  "amasadoras": [
    {
      "nombre": "Amasadora 3",
      "tipoMasa": "Tradicional",
      "hielo": "No",
      "lote": "5",
      "tempMasa": "25",
      "observaciones": "bien",
      "procesos": [
        { "id": 1, "minutos": 10, "segundos": 5, "startTime": 1695390000000, "endTime": 1695390605000 },
        { "id": 2, "minutos": 8,  "segundos": 0,  "startTime": 1695390610000, "endTime": 1695391090000 },
        { "id": 3, "minutos": 5,  "segundos": 30, "startTime": 1695391100000, "endTime": 1695391430000 }
      ]
    }
  ]
}
```

## Troubleshooting

- No veo nada en “Base de datos”
  - Pulsa “Refrescar”. Si sigue vacío, puede que no hayas enviado nada aún o el backend esté vacío.
- “Enviar a BD” falla (mensaje rojo)
  - Revisa la conexión. Verifica que https://pdt-backend-1.onrender.com/registros responde (usa pruebas más abajo).
  - Si usas una API override, que sea la URL base (sin terminar en "/registros").
- Al borrar por ID aparece 404
  - Ocurre si el registro ya no existe. Es un estado normal; refresca la lista.
- Limpiar datos locales
  - En Resultados → Locales, usa “Borrar todo” para limpiar el almacenamiento local.
- Persistencia de campos temporales
  - La app guarda inputs por máquina en localStorage (tempAmasadoras) para evitar perder datos al cambiar de pestaña.

## Pruebas rápidas de API (PowerShell en Windows)

- GET (listar):

```powershell
(Invoke-RestMethod -Uri https://pdt-backend-1.onrender.com/registros -Method GET) | ConvertTo-Json -Depth 4
```

- POST (crear):

```powershell
$body = @{ fecha = '2025-09-22'; tempAmbiente = '25'; humedad = '48'; personal = '6'; amasadoras = @(@{ nombre='Amasadora 3'; tipoMasa='Tradicional'; hielo='No'; tempMasa='25'; observaciones='ok'; procesos=@() }) } | ConvertTo-Json
Invoke-RestMethod -Uri https://pdt-backend-1.onrender.com/registros -Method POST -ContentType 'application/json' -Body $body
```

- DELETE (todo):

```powershell
Invoke-RestMethod -Uri https://pdt-backend-1.onrender.com/registros -Method DELETE
```

- DELETE (por id):

```powershell
Invoke-WebRequest -Uri https://pdt-backend-1.onrender.com/registros/99999 -Method DELETE | Select-Object -ExpandProperty StatusCode
```

## Compatibilidad y ejecución

- El frontend es puro HTML/JS/CSS: abre `index.html` directamente en tu navegador (Chrome/Edge recomendados).
- No requiere servidor local.
- Si usas un servidor estático, asegúrate de servir el archivo tal cual (no hace falta rutas especiales).
