Trackium Data Model

Este documento describe el modelo de datos preliminar para el MVP de Trackium.  Las tablas definidas a continuación proporcionan una estructura inicial para almacenar la información de empresas, usuarios, dispositivos (trackers), lecturas de los dispositivos, misiones y alertas.

Entidades principales

Empresas

* id (INT, PK, auto‑incremento): Identificador único de la empresa.
* nombre (VARCHAR): Nombre legal o comercial de la empresa.
* dirección (VARCHAR, opcional): Dirección física o sede.
* teléfono (VARCHAR, opcional): Teléfono de contacto.
* email (VARCHAR, opcional): Email de contacto o soporte.
* estado (ENUM: activa/inactiva): Permite habilitar o deshabilitar la empresa.
* fecha_creación (DATETIME): Fecha de alta en la plataforma.
* fecha_actualización (DATETIME): Fecha de la última modificación.

Usuarios

* id (INT, PK, auto‑incremento).
* empresa_id (INT, FK → Empresas.id): Empresa a la que pertenece el usuario.
* nombre (VARCHAR).
* email (VARCHAR, único).
* rol (ENUM: admin, operador, viewer): Nivel de permisos.
* password_hash (VARCHAR): Hash de la contraseña.
* estado (ENUM: activo/inactivo).
* fecha_creación (DATETIME).
* fecha_actualización (DATETIME).

Trackers

* id (INT, PK, auto‑incremento).
* empresa_id (INT, FK → Empresas.id): Empresa propietaria del dispositivo.
* nombre (VARCHAR): Nombre interno o alias del dispositivo.
* imei (VARCHAR, único): Identificador del dispositivo.
* iccid (VARCHAR, único, opcional): Identificador de la SIM.
* modelo (VARCHAR, opcional): Modelo de hardware del tracker.
* firmware (VARCHAR, opcional): Versión de firmware.
* estado_batería (INT): Porcentaje de batería reportado.
* última_conexión (DATETIME): Fecha y hora de la última lectura recibida.
* fecha_creación (DATETIME).
* fecha_actualización (DATETIME).

Lecturas

* id (BIGINT, PK, auto‑incremento).
* tracker_id (INT, FK → Trackers.id).
* timestamp (DATETIME): Fecha y hora de la lectura.
* latitud (DECIMAL).
* longitud (DECIMAL).
* velocidad (FLOAT, opcional).
* dirección (FLOAT, opcional): Rumbo o orientación.
* altitud (FLOAT, opcional).
* batería (INT, opcional).
* temperatura (FLOAT, opcional).
* otros_datos (JSON, opcional): Campo flexible para sensorística extra.
* fecha_creación (DATETIME).
* Índice (tracker_id, timestamp) para acelerar consultas temporales.

Misiones (opcional)

Si vuestra operativa necesita asociar un tracker a una tarea o misión concreta, definid:

* id (INT, PK, auto‑incremento).
* empresa_id (INT, FK → Empresas.id).
* tracker_id (INT, FK → Trackers.id).
* nombre (VARCHAR).
* descripcion (TEXT, opcional).
* fecha_inicio (DATETIME).
* fecha_fin (DATETIME, opcional).
* estado (ENUM: activa, finalizada, cancelada).
* fecha_creación (DATETIME).
* fecha_actualización (DATETIME).

Alertas

* id (INT, PK, auto‑incremento).
* empresa_id (INT, FK → Empresas.id).
* tracker_id (INT, FK → Trackers.id).
* lectura_id (BIGINT, FK → Lecturas.id, opcional): Lectura asociada que disparó la alerta.
* tipo (ENUM: bateria_baja, fuera_zona, parado, temperatura, otro).
* mensaje (VARCHAR, opcional): Mensaje o descripción.
* nivel (ENUM: info, aviso, crítico).
* fecha_creación (DATETIME).
* Índice (tracker_id, fecha_creación) para notificaciones rápidas.

Relaciones clave

* Una empresa tiene muchos usuarios y muchos trackers.
* Un usuario pertenece a una sola empresa.
* Un tracker pertenece a una sola empresa.
* Un tracker tiene muchas lecturas.
* Una lectura pertenece a un tracker.
* Una alerta pertenece a un tracker y opcionalmente a una lectura.
* Una misión puede asignarse a un tracker y, por extensión, a una empresa.

Consideraciones adicionales

1. Autenticación y roles: almacena sólo hashes de contraseñas; añade campos de recuperación o tokens JWT en tablas auxiliares si es necesario.
2. Normalización vs. rendimiento: este diseño inicial está normalizado; valora desnormalizar lecturas si necesitas consultas agregadas rápidas.
3. Histórico de lecturas: si la tabla de lecturas crece mucho, considera particionarla por fecha o crear tablas de lectura históricas archivadas.
4. Indices: añade índices en timestamp, tracker_id y otras columnas que se utilicen en filtros y ordenaciones frecuentes.
5. Geodatos: según la base de datos que utilices (PostgreSQL con PostGIS, MySQL con Spatial), crea tipos de datos geográficos para latitud/longitud y aprovecha funciones espaciales.
