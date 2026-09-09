# Matriz de control de acceso de AccessLab

## 1. Objetivo

Este documento define qué identidades pueden realizar acciones sobre los recursos protegidos de AccessLab.

La autorización debe ser validada por el backend en cada solicitud. La interfaz puede ocultar funciones, pero nunca será el mecanismo encargado de autorizar el acceso.

## 2. Alcance

AccessLab es un portal interno de soporte técnico para una empresa financiera ficticia.

El sistema utiliza usuarios, roles, tickets, sesiones, registros de auditoría y workload identities. No almacena dinero, credenciales reales ni información financiera perteneciente a personas reales.

## 3. Principios de seguridad

### Default deny

Todo acceso que no se encuentre expresamente permitido debe ser rechazado.

La ausencia de un permiso equivale a `DENY`.

### Least privilege

Cada identidad recibe únicamente los permisos indispensables para cumplir su función.

### Need to know

El acceso a información sensible se concede solamente cuando resulta necesario para una tarea autorizada.

### Separation of Duties

Las operaciones críticas deben dividirse entre diferentes roles para evitar que una sola persona pueda iniciar, aprobar y ocultar una acción.

### Server-side authorization

Todos los permisos deben verificarse en FastAPI. Ocultar un botón mediante JavaScript no constituye un control de autorización.

### Traceability

Los accesos denegados, cambios de roles y operaciones privilegiadas deben producir un evento de auditoría.

### No shared accounts

Cada persona debe utilizar una cuenta individual. Las service accounts y workload identities deben tener un propósito específico y no permitir inicio de sesión interactivo.

## 4. Decisión de autorización

Una solicitud recibe `ALLOW` solamente cuando se cumplen todas estas condiciones:

1. La identidad fue autenticada.
2. La cuenta se encuentra activa.
3. La sesión continúa siendo válida.
4. La identidad tiene un rol activo.
5. Existe un permiso explícito para la acción solicitada.
6. El recurso se encuentra dentro del scope permitido.
7. Se cumplen las condiciones adicionales de seguridad.

Si alguna condición falla, el resultado es `DENY`.

## 5. Clasificación de sensibilidad

- `Internal`: información disponible para personal autenticado de la empresa.
- `Confidential`: información operativa o personal limitada según rol y scope.
- `Restricted`: información de seguridad o privilegios cuyo acceso indebido podría producir un impacto grave.

## 6. Recursos protegidos

| Identificador | Recurso | Descripción | Sensibilidad |
|---|---|---|---|
| `ticket` | Ticket de soporte | Solicitud creada por un empleado y gestionada por soporte. | `Confidential` |
| `ticket_attachment` | Archivo adjunto | Capturas, documentos o evidencia incorporada a un ticket. | `Restricted` |
| `knowledge_article` | Artículo de conocimiento | Procedimientos internos utilizados por soporte. | `Internal` |
| `user_account` | Cuenta de usuario | Identidad, estado y atributos de una persona. | `Confidential` |
| `role_assignment` | Asignación de rol | Relación entre una identidad y sus privilegios. | `Restricted` |
| `session` | Sesión autenticada | Sesión activa asociada con una identidad. | `Restricted` |
| `audit_event` | Evento de auditoría | Registro de acciones permitidas, denegadas o privilegiadas. | `Restricted` |
| `security_alert` | Alerta de seguridad | Evento que requiere investigación del equipo de seguridad. | `Restricted` |
| `report` | Informe | Información operativa o de auditoría exportada. | `Confidential` |
| `application_secret` | Secreto de aplicación | Credencial utilizada por una workload identity. | `Restricted` |

## 7. Reglas de clasificación

- Un recurso puede elevar su clasificación según el contenido o el contexto.
- Un recurso derivado debe heredar la clasificación más alta de sus fuentes.
- Los archivos adjuntos se consideran `Restricted` por defecto.
- Los tickets relacionados con seguridad, fraude o acceso privilegiado se consideran `Restricted`.
- Los informes de auditoría o seguridad se consideran `Restricted`.
- Los secretos, tokens y credenciales nunca deben aparecer en logs ni respuestas de la API.
- La clasificación no concede acceso: cada operación también requiere un permiso explícito.