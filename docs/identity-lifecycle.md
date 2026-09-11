# Identity Lifecycle Management

## 1. Objetivo

Este documento define cómo AccessLab solicita, aprueba, provisiona, modifica,
suspende, deshabilita y conserva las identidades y sus accesos.

El proceso aplica los principios de:

- `Default Deny`
- `Least Privilege`
- `Need to Know`
- `Separation of Duties`
- `Traceability`
- Revocación oportuna
- Revisiones periódicas de acceso

La creación de una identidad no concede permisos automáticamente. Cada acceso
debe provenir de una asignación de rol explícita, aprobada y auditable.

## 2. Alcance

El ciclo de vida comprende:

- `Workforce identities`
- `External identities`
- `Workload identities`
- `Agent identities`
- `Device identities`

También comprende:

- cuentas de usuario;
- membresías de grupos;
- asignaciones de roles;
- sesiones activas;
- tokens y credenciales;
- secretos de aplicación;
- dispositivos registrados;
- evidencia de auditoría.

## 3. Fuente autoritativa

La `authoritative source` es el sistema considerado confiable para determinar
si una identidad debe existir y qué relación mantiene con la organización.

Para AccessLab:

| Tipo de identidad | Fuente autoritativa |
|---|---|
| `Workforce identity` | Sistema ficticio de Recursos Humanos |
| `External identity` | Solicitud y patrocinador interno aprobados |
| `Workload identity` | Registro de aplicaciones y servicios autorizados |
| `Agent identity` | Registro de agentes y propietario responsable |
| `Device identity` | Sistema de registro y administración de dispositivos |

AccessLab no debe permitir que una cuenta permanezca activa cuando su fuente
autoritativa indique que la relación finalizó.

## 4. Estados de una identidad

| Estado | Significado | Autenticación permitida |
|---|---|---|
| `Requested` | La identidad fue solicitada, pero todavía no fue aprobada. | `No` |
| `Approved` | La solicitud fue aprobada, pero la cuenta todavía no fue creada. | `No` |
| `Provisioned` | La cuenta fue creada y configurada, pero todavía no está habilitada para operar. | `No` |
| `Active` | La identidad está habilitada y puede utilizar sus accesos autorizados. | `Yes` |
| `Suspended` | El acceso fue bloqueado temporalmente por una investigación o condición administrativa. | `No` |
| `Disabled` | La identidad ya no puede autenticarse y sus accesos fueron revocados. | `No` |
| `Archived` | La cuenta permanece únicamente como registro histórico y evidencia de auditoría. | `No` |

## 5. Actores y responsabilidades

Los procesos de identidad deben separar las siguientes funciones:

- `Requester`: solicita el acceso y aporta la justificación.
- `Approver`: determina si el acceso corresponde.
- `Implementer`: ejecuta técnicamente el cambio aprobado.
- `Reviewer`: comprueba posteriormente que el proceso y las evidencias sean correctos.

Para accesos sensibles o privilegiados, una misma identidad no debe controlar
todas las etapas del proceso.

| Actor | Función dentro del ciclo de vida | Lo que no debe hacer |
|---|---|---|
| `HR System` | Fuente autoritativa para ingresos, cambios de puesto y desvinculaciones de empleados. | Asignar directamente permisos de aplicación. |
| `Line Manager` | Solicitar y justificar accesos de los integrantes de su equipo. | Implementar técnicamente el acceso que solicita o aprobarse privilegios propios. |
| `Resource Owner` | Aprobar el acceso a recursos sensibles y definir sus condiciones. | Crear cuentas o asignar técnicamente los roles. |
| `IAM Administrator` | Crear, modificar, deshabilitar cuentas y ejecutar asignaciones aprobadas. | Aprobar la misma solicitud privilegiada que implementa. |
| `Security Analyst` | Contener incidentes, revocar sesiones comprometidas y solicitar bloqueos de seguridad. | Modificar roles o cerrar una investigación sin los controles requeridos. |
| `Auditor` | Verificar evidencias, tiempos de revocación, cambios de roles y cumplimiento de JML. | Solicitar, aprobar o implementar los cambios que audita. |
| `External Sponsor` | Justificar y supervisar el acceso de una identidad externa. | Provisionar directamente la cuenta externa o conceder permisos sin aprobación. |
| `Workload Owner` | Justificar la existencia, permisos y continuidad de una identidad no humana. | Asignarse credenciales o ampliar directamente los permisos de la workload. |
| `Identity Holder` | Completar la activación de su cuenta, registrar MFA y proteger sus credenciales. | Compartir credenciales, aprobar sus propios accesos o asignarse roles. |
| `AccessLab System` | Aplicar estados, permisos y condiciones; rechazar accesos y generar auditoría. | Tomar decisiones de negocio que requieren aprobación humana. |

### Ejercicio: asignación de responsabilidades

| Situación | Actor principal |
|---|---|
| Registrar la fecha de ingreso de una nueva empleada. | `HR System` |
| Solicitar el rol inicial necesario para una analista de soporte. | `Line Manager` |
| Aprobar acceso a información clasificada como `Restricted`. | `Resource Owner` |
| Crear una cuenta después de recibir las aprobaciones. | `IAM Administrator` |
| Revocar una sesión potencialmente comprometida. | `Security Analyst` |
| Verificar que una cuenta fue deshabilitada dentro del plazo establecido. | `Auditor` |
| Justificar la continuidad del acceso de un técnico externo. | `External Sponsor` |
| Solicitar la desactivación de `support_automation` porque el servicio fue retirado. | `Workload Owner` |
| Impedir el login de una identidad cuyo estado es `Disabled`. | `AccessLab System` |

## 6. Proceso Joiner

El proceso `Joiner` administra la incorporación de una nueva identidad desde
que se recibe el evento de ingreso hasta que la cuenta queda activa.

Ninguna cuenta debe activarse únicamente porque fue creada. Antes de permitir
la autenticación, AccessLab debe verificar la fuente autoritativa, las
aprobaciones, la fecha de inicio, el estado de la cuenta y los controles de
autenticación requeridos.

### 6.1 Principios del proceso Joiner

- La identidad debe tener un identificador interno único e inmutable.
- El correo electrónico o nombre visible no debe utilizarse como identificador principal.
- La cuenta debe crearse inicialmente en estado `Provisioned`.
- La autenticación no debe habilitarse antes de la fecha de inicio.
- El acceso inicial debe aplicar `Least Privilege`.
- Los roles sensibles o privilegiados requieren aprobación explícita.
- La persona debe registrar MFA antes de operar normalmente.
- Las credenciales temporales deben ser de uso único y corta duración.
- Las contraseñas nunca deben enviarse o registrarse en texto plano.
- Cada etapa debe generar evidencia de auditoría.

### 6.2 Datos requeridos para un Joiner

| Campo | Propósito |
|---|---|
| `subject_id` | Identificador interno, único e inmutable de la identidad. |
| `full_name` | Nombre visible de la persona. |
| `start_date` | Fecha desde la cual puede activarse la cuenta. |
| `employment_type` | Indica si es empleada, contratista, pasante u otro tipo de relación. |
| `department` | Área organizacional a la que pertenece. |
| `team` | Equipo utilizado para determinar alcances y asignaciones. |
| `job_title` | Función laboral informada por la fuente autoritativa. |
| `manager_id` | Identificador de la persona responsable del equipo. |
| `requested_role` | Rol de AccessLab solicitado. |
| `business_justification` | Motivo por el cual necesita ese acceso. |
| `approvals` | Evidencia de las aprobaciones requeridas. |
| `end_date` | Fecha de finalización cuando la relación es temporal. |

### 6.3 Flujo Joiner

| Paso | Evento o acción | Actor principal | Estado resultante |
|---|---|---|---|
| 1 | Registrar el ingreso desde la fuente autoritativa. | `HR System` | `Requested` |
| 2 | Solicitar el rol inicial y aportar la justificación laboral. | `Line Manager` | `Requested` |
| 3 | Validar los campos obligatorios, la fecha de inicio y que no exista una identidad duplicada. | `AccessLab System` | `Requested` |
| 4 | Aprobar el acceso sensible solicitado para la nueva analista. | `Resource Owner` | `Approved` |
| 5 | Crear la cuenta y asignar únicamente los grupos y roles aprobados. | `IAM Administrator` | `Provisioned` |
| 6 | Completar la activación inicial y registrar MFA. | `Identity Holder` | `Provisioned` |
| 7 | Verificar la fecha de inicio, MFA, aprobaciones y estado antes de habilitar el acceso. | `AccessLab System` | `Active` |
| 8 | Revisar posteriormente la evidencia del proceso de incorporación. | `Auditor` | `Active` |