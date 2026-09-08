# AccessLab

Portal interno de soporte técnico e Identity and Access Management para una empresa financiera ficticia.

## Descripción

AccessLab es un proyecto educativo de ciberseguridad orientado al diseño, implementación y prueba de controles de identidad y acceso.

La aplicación simulará el portal interno de soporte técnico de una empresa financiera. Permitirá administrar usuarios, tickets, roles, sesiones y eventos de auditoría dentro de un entorno local y autorizado.

El proyecto está diseñado para demostrar tanto conocimientos de desarrollo seguro como la capacidad de analizar políticas IAM, riesgos de autorización y controles de seguridad.

## Objetivos de aprendizaje

- Diseñar políticas de acceso antes de programar.
- Aplicar los principios de least privilege y deny by default.
- Implementar Role-Based Access Control (RBAC).
- Diseñar procesos Joiner–Mover–Leaver.
- Detectar conflictos de Separation of Duties (SoD).
- Diferenciar authentication de authorization.
- Proteger sesiones, cookies y credenciales.
- Implementar logging y auditing de eventos de seguridad.
- Probar controles de acceso mediante Pytest y Postman.
- Analizar vulnerabilidades como IDOR y privilege escalation.
- Integrar posteriormente OAuth 2.0 y OpenID Connect.
- Trabajar con human, external, device y workload identities.

## Contexto ficticio

AccessLab representa una aplicación utilizada por una empresa financiera para gestionar solicitudes internas de soporte técnico.

La aplicación no procesa dinero ni utiliza datos financieros reales. Todos los usuarios, tickets, registros y credenciales utilizados durante el desarrollo serán ficticios.

Algunos tickets podrán clasificarse como sensibles para practicar controles de acceso propios de un entorno financiero.

## Identidades previstas

- Employee
- Support Analyst L1
- Support Analyst L2
- Support Manager
- IAM Administrator
- Security Analyst
- Auditor
- External Technician
- Support Automation service account
- AccessLab API workload identity

## Funcionalidades iniciales

- Creación administrativa de usuarios.
- Inicio y cierre de sesión.
- Desactivación de cuentas.
- Administración de roles y permisos.
- Creación y gestión de tickets.
- Restricción de tickets por propietario y asignación.
- Paneles diferentes según el rol.
- Registro de accesos permitidos y denegados.
- Historial de cambios de roles.
- Vencimiento y revocación de sesiones.
- Consulta de eventos por parte de auditoría.

## Principios de seguridad

### Deny by default

Toda acción que no esté permitida explícitamente será denegada.

### Least privilege

Cada identidad recibirá solamente los permisos necesarios para cumplir su función.

### Separation of Duties

Las operaciones sensibles serán separadas entre diferentes roles cuando sea necesario.

### Server-side authorization

Los permisos serán verificados por el backend en cada endpoint protegido. Ocultar elementos del Front-End no será considerado un control de autorización.

### Accountability

Las acciones importantes deberán quedar asociadas a una identidad y registradas en el historial de auditoría.

### Secure by design

Las políticas, amenazas y requisitos de seguridad serán definidos antes de implementar las funcionalidades.

## Stack previsto

- Front-End: HTML5, CSS3 y JavaScript.
- Backend: Python y FastAPI.
- Database: SQLite durante el MVP; PostgreSQL en una etapa posterior.
- Testing: Pytest y Postman.
- Version control: Git y GitHub.
- Identity Provider: Keycloak y posteriormente Microsoft Entra ID.
- Containerization: Docker en una etapa posterior.

## Documentación

- `docs/access-control-matrix.md`: roles, recursos, permisos y conflictos de acceso.
- `docs/identity-lifecycle.md`: procesos Joiner–Mover–Leaver.
- `docs/threat-model.md`: activos, amenazas, controles y riesgo residual.

## Roadmap

1. Diseño de políticas IAM.
2. Laboratorio IAM en Linux.
3. Desarrollo de la aplicación base.
4. Implementación y pruebas de RBAC.
5. Vulnerabilidades controladas y correcciones.
6. Autenticación y session management.
7. OAuth 2.0, OpenID Connect y SSO.
8. Microsoft Entra ID y workload identities.
9. Auditoría y detección de eventos.
10. Docker, CI/CD y documentación final.

## Alcance ético

Todas las pruebas de seguridad se realizarán exclusivamente sobre AccessLab o laboratorios expresamente autorizados.

Las versiones vulnerables serán utilizadas únicamente de forma local y no serán publicadas como servicios accesibles desde Internet.

Nunca se almacenarán credenciales reales, tokens válidos ni información personal en el repositorio.

## Estado actual

Fase 0: inicialización del repositorio y preparación de la documentación de seguridad.