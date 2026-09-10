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

## 8. Modelo de identidades, roles y grupos

Una identidad representa a una persona, dispositivo o proceso. Un rol agrupa permisos y un grupo permite asignar roles a múltiples identidades de manera administrable.

Las identidades humanas reciben roles mediante grupos de seguridad. Las workload identities reciben permisos directamente y no deben permitir inicio de sesión interactivo.

| Actor | Tipo de identidad | Rol de aplicación | Mecanismo de asignación | Inicio interactivo |
|---|---|---|---|---|
| `employee_user` | `Workforce identity` | `Employee` | `AccessLab-Employees` | `Yes` |
| `support_l1_user` | `Workforce identity` | `Support Analyst L1` | `AccessLab-Support-L1` | `Yes` |
| `support_l2_user` | `Workforce identity` | `Support Analyst L2` | `AccessLab-Support-L2` | `Yes` |
| `support_manager_user` | `Workforce identity` | `Support Manager` | `AccessLab-Support-Managers` | `Yes` |
| `iam_administrator_user` | `Workforce identity` | `IAM Administrator` | `AccessLab-IAM-Admins` | `Yes` |
| `security_analyst_user` | `Workforce identity` | `Security Analyst` | `AccessLab-Security-Analysts` | `Yes` |
| `auditor_user` | `Workforce identity` | `Auditor` | `AccessLab-Auditors` | `Yes` |
| `external_technician_user` | `External identity` | `External Technician` | `AccessLab-External-Technicians` | `Yes` |
| `support_automation` | `Workload identity (service account)` | `Support Automation` | Asignación directa | `No` |
| `accesslab_api` | `Workload identity` | Permisos de infraestructura | Asignación directa | `No` |
| `registered_device` | `Device identity` | No corresponde | Señal de acceso contextual | `No` |
| `log_analysis_agent` | `Agent identity` | `Log Analysis Agent` | Asignación directa | `No` |

## 9. Reglas de asignación

- Las personas deben utilizar cuentas individuales.
- Los roles humanos deben asignarse mediante grupos de seguridad.
- Las workload identities no deben pertenecer a grupos humanos.
- Las identidades no humanas no deben permitir inicio de sesión interactivo.
- Los accesos externos deben ser temporales y revisados periódicamente.
- Los roles privilegiados deben requerir MFA.
- Una device identity puede funcionar como condición de acceso, pero no reemplaza la identidad del usuario.
- La pertenencia simultánea a roles incompatibles debe ser rechazada.

## 10. Clasificación de riesgo de los roles

La clasificación de un rol depende del impacto que tendría su uso indebido, del scope autorizado y de las acciones que puede ejecutar.

### Niveles

- `Standard`: acceso rutinario y limitado principalmente a recursos propios.
- `Sensitive`: acceso a datos de otras personas o recursos sensibles, sin capacidad para administrar privilegios.
- `Privileged`: capacidad para modificar identidades, accesos, configuraciones o ejecutar operaciones de alto impacto.

El acceso a datos propios no convierte automáticamente un rol en `Sensitive`. El riesgo aumenta cuando permite acceder o actuar sobre recursos pertenecientes a otras identidades.

| Rol | Categoría | Justificación |
|---|---|---|
| `Employee` | `Standard` | Administra únicamente sus propios tickets, archivos y sesiones, sin acceso a recursos de terceros. |
| `Support Analyst L1` | `Sensitive` | Consulta y actualiza tickets asignados pertenecientes a otras personas, con un scope limitado. |
| `Support Analyst L2` | `Sensitive` | Accede a tickets escalados y evidencia técnica sensible, pero no administra identidades ni privilegios. |
| `Support Manager` | `Privileged` | Tiene acceso amplio a los tickets del equipo y puede asignarlos, cambiar prioridades y aprobar cierres. |
| `IAM Administrator` | `Privileged` | Puede crear o desactivar cuentas y asignar o revocar roles, por lo que su uso indebido permitiría una privilege escalation. |
| `Security Analyst` | `Privileged` | Investiga alertas y eventos restringidos y puede revocar sesiones comprometidas durante un incidente. |
| `Auditor` | `Sensitive` | Puede leer y exportar evidencia restringida, pero no modificar usuarios, permisos ni registros. |
| `External Technician` | `Sensitive` | Como tercero, accede temporalmente a tickets y archivos de otras personas dentro de un scope estrictamente limitado. |
| `Support Automation` | `Sensitive` | Procesa tickets de diferentes usuarios mediante reglas automáticas, pero no administra roles ni realiza acciones críticas. |
| `AccessLab API Runtime` | `Privileged` | Accede a la base de datos, escribe auditorías y utiliza permisos de infraestructura necesarios para ejecutar la aplicación. |
| `Log Analysis Agent` | `Sensitive` | Lee registros restringidos y crea alertas, pero no puede modificar evidencia ni ejecutar medidas de contención. |

## 11. Modelo de permisos

Los permisos de AccessLab utilizan el siguiente formato:

`resource:action:scope`

- `resource`: objeto protegido.
- `action`: operación solicitada.
- `scope`: conjunto de recursos sobre el que puede realizarse.

Un permiso no produce automáticamente un `ALLOW`. También deben cumplirse las condiciones de autenticación, estado de cuenta, sesión, sensibilidad y contexto.

## 12. Scopes

| Scope | Descripción |
|---|---|
| `self` | La propia cuenta o sesión de la identidad. |
| `own` | Recursos cuyo propietario es la identidad. |
| `assigned` | Recursos asignados expresamente a la identidad. |
| `team` | Recursos pertenecientes al equipo de la identidad. |
| `queue` | Recursos pendientes dentro de una cola operativa autorizada. |
| `all` | Todos los recursos del tipo indicado, sujetos a condiciones adicionales. |
| `service` | Recursos técnicos estrictamente necesarios para una workload o agent identity. |

El scope `all` debe utilizarse excepcionalmente porque aumenta el impacto de una cuenta comprometida.

## 13. Catálogo de acciones

| Recurso | Acciones disponibles |
|---|---|
| `ticket` | `create`, `read`, `update`, `comment`, `assign`, `change_priority`, `escalate`, `close`, `reopen` |
| `ticket_attachment` | `upload`, `read`, `delete` |
| `knowledge_article` | `read`, `create`, `update`, `publish`, `archive` |
| `user_account` | `read`, `create`, `update`, `disable`, `reactivate` |
| `role_assignment` | `read`, `request`, `approve`, `assign`, `revoke` |
| `session` | `read`, `revoke` |
| `audit_event` | `create`, `read`, `export` |
| `security_alert` | `create`, `read`, `investigate`, `update_status`, `close` |
| `report` | `create`, `read`, `export` |
| `application_secret` | `read`, `rotate` |

## 14. Acciones no permitidas

AccessLab no implementará los siguientes permisos:

- `ticket:delete`: los tickets se conservan como evidencia operativa.
- `audit_event:update`: los eventos de auditoría deben ser inmutables.
- `audit_event:delete`: ninguna identidad puede borrar evidencia.
- `user_account:delete`: las cuentas deben desactivarse, no eliminarse.
- Mostrar secretos mediante la interfaz o incluirlos en logs.
- Permitir que una identidad asigne o aumente sus propios privilegios.

## 15. Condiciones contextuales

Además del permiso, AccessLab podrá exigir condiciones como:

- Cuenta activa.
- Sesión válida.
- MFA completado.
- Dispositivo registrado o compliant.
- Acceso externo no vencido.
- Archivo externo aprobado.
- Ticket dentro de un estado modificable.
- Clasificación compatible con el rol.

Estas condiciones complementan RBAC y se aproximan a un modelo `ABAC`.

## 16. Ejercicio de traducción de permisos

| Caso | Permiso |
|---|---|
| Un empleado consulta su propio ticket. | `ticket:read:own` |
| Un analista L1 actualiza un ticket asignado. | `ticket:update:assigned` |
| Un Support Manager asigna un ticket de su equipo. | `ticket:assign:team` |
| Un IAM Administrator desactiva una cuenta. | `user_account:disable:all` |
| Un Auditor exporta eventos de auditoría. | `audit_event:export:all` |
| Un External Technician consulta un archivo autorizado de un ticket asignado. | `ticket_attachment:read:assigned`; condición: `external_access_active AND attachment_external_approved` |
| Support Automation asigna un ticket de la cola. | `ticket:assign:queue` |
| AccessLab API Runtime lee un secreto necesario para operar. | `application_secret:read:service` |
| Log Analysis Agent lee logs autorizados y crea una alerta. | `audit_event:read:service` y `security_alert:create:service` |
| Security Analyst revoca una sesión comprometida. | `session:revoke:all` |

## 17. Matriz RBAC

La matriz RBAC relaciona roles con permisos explícitos. Cada celda autorizada contiene el scope correspondiente.

El símbolo `—` representa la ausencia de un permiso y produce `DENY` por defecto.

### 17.1 Permisos operativos sobre tickets

| Permiso base | Employee | Support L1 | Support L2 | Support Manager | External Technician |
|---|---|---|---|---|---|
| `ticket:create` | `own` | `—` | `—` | `—` | `—` |
| `ticket:read` | `own` | `assigned` | `assigned` | `team` | `assigned` |
| `ticket:update` | `own` | `assigned` | `assigned` | `—` | `—` |
| `ticket:comment` | `own` | `assigned` | `assigned` | `team` | `assigned` |
| `ticket:assign` | `—` | `—` | `—` | `team` | `—` |
| `ticket:change_priority` | `—` | `—` | `—` | `team` | `—` |
| `ticket:escalate` | `—` | `assigned` | `assigned` | `team` | `—` |
| `ticket:close` | `own` | `assigned` | `assigned` | `team` | `—` |
| `ticket:reopen` | `own` | `assigned` | `assigned` | `team` | `—` |

### Condiciones de los tickets

- Los empleados solo pueden modificar tickets propios que continúen en un estado modificable.
- Los analistas solo pueden actuar sobre tickets que tengan asignados.
- Los tickets escalados a L2 deben ser asignados expresamente a un analista L2.
- El Support Manager administra tickets de su equipo, no todos los tickets de la organización.
- Los tickets de seguridad o fraude quedan fuera del scope general del equipo de soporte.
- El External Technician requiere una asignación activa y acceso externo no vencido.
- Ningún rol puede eliminar definitivamente un ticket.

### 17.2 Permisos sobre archivos adjuntos

Un archivo adjunto solamente puede consultarse cuando la identidad tiene acceso tanto al archivo como al ticket relacionado.

Esto evita que una persona modifique el ID del archivo y acceda a evidencia perteneciente a otro ticket.

| Permiso base | Employee | Support L1 | Support L2 | Support Manager | External Technician |
|---|---|---|---|---|---|
| `ticket_attachment:upload` | `own` | `assigned` | `assigned` | `—` | `—` |
| `ticket_attachment:read` | `own` | `assigned` | `assigned` | `team` | `assigned` |
| `ticket_attachment:delete` | `—` | `—` | `—` | `—` | `—` |

#### Condiciones para archivos adjuntos

- La identidad también debe poder leer el ticket relacionado.
- Los archivos heredan como mínimo la clasificación del ticket.
- Los nombres de archivo deben generarse de forma segura.
- El sistema debe validar tamaño, extensión y tipo MIME.
- Los archivos ejecutables deben ser rechazados.
- Los archivos deben almacenarse fuera del directorio público de la aplicación.
- Un archivo potencialmente malicioso debe ser bloqueado o puesto en cuarentena.
- Un External Technician solo puede consultar archivos aprobados expresamente.
- Ningún rol operativo puede eliminar definitivamente un archivo incorporado como evidencia.

### 17.3 Permisos sobre artículos de conocimiento

Los artículos utilizan los estados `draft`, `published` y `archived`.

| Permiso base | Employee | Support L1 | Support L2 | Support Manager | External Technician |
|---|---|---|---|---|---|
| `knowledge_article:read` | `all` | `all` | `all` | `all` | `—` |
| `knowledge_article:create` | `—` | `—` | `own` | `own` | `—` |
| `knowledge_article:update` | `—` | `—` | `own` | `team` | `—` |
| `knowledge_article:publish` | `—` | `—` | `—` | `team` | `—` |
| `knowledge_article:archive` | `—` | `—` | `—` | `team` | `—` |

#### Condiciones para artículos de conocimiento

- Employee y Support L1 solo pueden consultar artículos publicados.
- Support L2 puede crear y modificar sus propios borradores.
- Support Manager revisa y publica artículos del equipo.
- La persona que redacta un artículo no debe publicarlo si requiere aprobación.
- Los artículos archivados conservan su historial.
- Los artículos sensibles pueden requerir controles adicionales.
- External Technician no accede a la base interna de conocimiento.

#### Controles de archivado

- Archivar un artículo no elimina su contenido ni su historial.
- Solo Support Manager puede archivar artículos de su equipo.
- La acción requiere una justificación obligatoria.
- Cada archivado genera un evento de auditoría.
- Las versiones anteriores deben permanecer inmutables.
- Los artículos archivados no aparecen como procedimientos vigentes.
- Auditor puede consultar artículos archivados y su historial, pero no modificar su estado.
- Los artículos no pueden eliminarse permanentemente mediante la aplicación.