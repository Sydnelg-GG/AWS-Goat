# AWSGoat — Evaluación de Seguridad Ofensiva y Defensiva en AWS

**Curso:** Tópicos Especiales II — Grupo 1S3141
**Profesor:** Armando Jipsion
**Universidad Tecnológica de Panamá** — Facultad de Ingeniería en Sistemas Computacionales, Licenciatura en Ciberseguridad
**Autores:** Cleidys Guzmán, Mónica Pineda, Victor Pineda

---

## Sobre este repositorio

Este repositorio documenta el ejercicio semestral realizado sobre **AWSGoat**, un laboratorio intencionalmente vulnerable desplegado sobre Amazon Web Services. El trabajo se dividió en dos roles complementarios, tal como ocurre en un equipo de seguridad real:

- **Red Team:** identificación y explotación controlada de vulnerabilidades presentes en cada módulo, documentando el procedimiento, la evidencia técnica y el impacto de cada hallazgo.
- **Blue Team:** análisis, remediación y fortalecimiento de la infraestructura comprometida, incluyendo corrección de código Terraform, configuración de controles de detección nativos de AWS y verificación funcional de cada control implementado.

Todo el ejercicio se realizó en una cuenta de AWS dedicada exclusivamente al laboratorio, dentro de la capa gratuita (Free Tier), y la infraestructura fue destruida (`terraform destroy`) al finalizar cada fase de trabajo.

---

## Qué es AWSGoat

AWSGoat es un proyecto de código abierto mantenido por **INE Security** (repositorio original: `ine-labs/AWSGoat`), diseñado como un entorno "vulnerable por diseño" para la práctica de pentesting, auditoría de infraestructura y seguridad en la nube (CloudSec). A diferencia de un laboratorio de pentesting web tradicional, AWSGoat combina vulnerabilidades de aplicación (inyección, control de acceso roto, SSRF) con configuraciones inseguras reales de servicios administrados de AWS (IAM, S3, Lambda, API Gateway, ECS, EC2), permitiendo practicar cadenas de ataque que van desde la aplicación web hasta el compromiso total de una cuenta en la nube.

Toda la infraestructura se despliega mediante **Terraform**, lo que permite tanto reproducir el entorno vulnerable de forma consistente como codificar su remediación como parte del mismo flujo de trabajo de Infrastructure as Code.

AWSGoat está compuesto por dos módulos independientes, cada uno con una superficie de ataque distinta.

---

## Estructura del repositorio

```
.
├── Modulo-1/
│   └── evidencias/              Capturas de pantalla organizadas por vulnerabilidad
│                                 (reconocimiento, XSS, inyección SQL, IDOR,
│                                  exposición de datos sensibles)
│
├── Modulo-2/
│   └── evidencias/              Capturas de pantalla organizadas por vulnerabilidad
│                                 (reconocimiento, inyección SQL, carga de archivos/RCE,
│                                  escape de contenedor, escalación de privilegios IAM)
│
├── Reportes - Red & Blue Team/
│   ├── Informe Red Team         Informe técnico de la evaluación ofensiva (ambos módulos)
│   ├── Informe Blue Team        Informe técnico de remediación y controles aplicados
│   ├── arquitectura-modulo1     Diagrama de arquitectura recomendada / mejorada — Módulo 1
│   └── arquitectura-modulo2     Diagrama de arquitectura recomendada / mejorada — Módulo 2
│
├── Informe_Final_General        Documento consolidado del ejercicio completo
└── README.md
```

La evidencia cruda (capturas) vive dentro de cada carpeta de módulo, organizada por fase; el análisis narrativo, la clasificación de cada hallazgo y las recomendaciones viven en los informes de la carpeta de Reportes; y el documento a nivel de raíz consolida ambos módulos en una sola narrativa de cierre.

---

## Módulo 1 — Blog Serverless

**Arquitectura:** Amazon API Gateway, AWS Lambda, Amazon DynamoDB y Amazon S3.

Este módulo simula un blog corporativo construido sobre una arquitectura completamente serverless. El objetivo de esta parte del ejercicio fue demostrar que eliminar la gestión de servidores no elimina los riesgos clásicos de una aplicación web: la superficie de ataque real sigue estando en la lógica de negocio y en el control de acceso de cada endpoint expuesto.

| ID | Vulnerabilidad | Componente afectado | Clasificación OWASP |
|---|---|---|---|
| R1 | Cross-Site Scripting (XSS) reflejado | Barra de búsqueda del blog | A03:2021 — Injection |
| R2 | Inyección SQL | Endpoint `/v1/search-author` | A03:2021 — Injection |
| R3 | IDOR en cambio de contraseña | Endpoint de cambio de contraseña | A01:2021 — Broken Access Control |
| R4 | Exposición de datos sensibles vía endpoint oculto | Endpoint `/v1/dump` | A01:2021 — Broken Access Control |

El hallazgo más crítico del módulo (R4) se descubrió mediante enumeración de rutas no documentadas, y expuso el volcado completo de la base de usuarios de la aplicación — un recordatorio directo de que "seguridad por oscuridad" no es un control real.

---

## Módulo 2 — Aplicación de Recursos Humanos (ECS / ALB / RDS)

**Arquitectura:** Amazon ECS sobre EC2 (Auto Scaling Group), Application Load Balancer, Amazon RDS (MySQL), AWS Secrets Manager, y múltiples roles de AWS IAM (`ecs-instance-role`, `ecs-task-role`, `ec2Deployer-role`).

Este módulo simula una aplicación interna de nómina, y su objetivo fue evaluar vulnerabilidades a nivel de infraestructura, contenedores y gestión de identidades — un escenario mucho más cercano a un entorno empresarial real que el Módulo 1. A diferencia de las fallas aisladas del blog serverless, aquí se documentó una **cadena de explotación completa**: cada vulnerabilidad individual sirvió como llave de acceso para la siguiente, hasta alcanzar control administrativo total sobre la cuenta de AWS.

| # | Vulnerabilidad | Servicio afectado | Clasificación OWASP |
|---|---|---|---|
| 1 | Inyección SQL en el mecanismo de autenticación | Amazon ECS, Amazon RDS | A03:2021 — Injection |
| 2 | Carga insegura de archivos con ejecución remota de código | Amazon ECS, AWS IAM | A05:2021 — Security Misconfiguration |
| 3 | Escalación de privilegios local y escape de contenedor | Amazon ECS, Amazon EC2, AWS IAM, IMDS | A01:2021 — Broken Access Control |
| 4 | Escalación de privilegios IAM (`iam:PassRole`, `ec2:RunInstances`) | AWS IAM, Amazon EC2 | A01:2021 — Broken Access Control |

La cadena completa —desde un acceso anónimo hasta la creación de un usuario con `AdministratorAccess`— quedó documentada paso a paso en el Informe Red Team, y su remediación (corrección de políticas IAM, cifrado y logging de S3, CloudTrail integrado con CloudWatch Logs, alarmas y notificaciones vía SNS) en el Informe Blue Team.

---

## Metodología aplicada

En ambos módulos se siguió un flujo de trabajo equivalente al de una evaluación de seguridad ofensiva en un entorno controlado:

1. Enumeración de funcionalidades, endpoints y componentes visibles de la aplicación.
2. Interceptación y análisis de solicitudes mediante Burp Suite, OWASP ZAP y las herramientas de desarrollador del navegador.
3. Modificación de parámetros, tokens y solicitudes para validar los controles de entrada, autenticación y autorización.
4. Verificación del impacto real de cada hallazgo mediante la observación de las respuestas del servidor y el acceso efectivo a datos o funciones no autorizadas.
5. Documentación de la evidencia técnica (capturas de pantalla, respuestas HTTP, comandos ejecutados) para cada vulnerabilidad confirmada.
6. Aplicación de controles de remediación y detección, y verificación funcional de cada uno mediante pruebas dirigidas.

---

## Nota sobre el manejo de credenciales

Durante el ejercicio se expusieron credenciales temporales de AWS pertenecientes a roles de la infraestructura de laboratorio. Estas credenciales son de corta duración y corresponden exclusivamente a una infraestructura desechable, destruida al finalizar cada sesión de trabajo. Ningún dato ni credencial expuesta en este repositorio corresponde a un entorno productivo real.

---

## Alcance y propósito

Este trabajo tiene fines exclusivamente educativos, realizado dentro del marco del curso Tópicos Especiales II. AWSGoat está diseñado específicamente para este tipo de práctica controlada; ninguna de las técnicas aquí documentadas fue aplicada fuera de este entorno de laboratorio.
