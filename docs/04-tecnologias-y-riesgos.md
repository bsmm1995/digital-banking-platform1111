# 04 · Tecnologías elegidas: por qué y riesgos

> Para cada tecnología: **por qué se eligió frente a las alternativas**, **qué riesgo trae** y **cómo se mitiga**. Complementa las [justificaciones](01-justificaciones.md) y los [ADR](adr/README.md). Los términos están en el [glosario](03-glosario.md).

## Versiones de referencia (noviembre 2026)

Criterio: **la última versión LTS** cuando el proyecto la publica; si no publica LTS, la **última versión estable** soportada.

| Tecnología | Versión propuesta | Política de soporte |
|---|---|---|
| Java (JDK) | **25 LTS** | LTS de Oracle/OpenJDK; la próxima LTS (29) se espera en septiembre de 2027. |
| Spring Boot (WebFlux) | **4.1** (Spring Framework 7) | Sin LTS comunitario: cada versión menor recibe soporte OSS unos 12 meses. Pasar a 4.2 cuando salga su GA (prevista para noviembre de 2026). |
| Quarkus | **3.40 LTS** | Una LTS cada 6 meses, con 12 meses de parches. |
| GraalVM / Mandrel | **25** (para Java 25) | Sigue la versión de Java. |
| Angular | **22** | Desde la v22, una versión mayor por año con 12 meses activos y 12 de LTS (hasta junio de 2028). |
| Flutter (Dart 3) | **3.47** o la estable más reciente | Sin LTS: versión estable trimestral; solo la última recibe correcciones. |
| Keycloak | **26.8** | Sin LTS: solo la última versión menor recibe parches (4 al año). Para soporte extendido, Red Hat build of Keycloak. |

> Las versiones se confirman al iniciar el proyecto; la regla de actualización (no quedarse más de una versión atrás) importa más que el número exacto.

## Riesgos transversales

| Riesgo | Mitigación |
|---|---|
| **Habilidades del equipo:** WebFlux, Kubernetes y FAPI tienen curva de aprendizaje alta. | Arquetipos de proyecto y lineamientos comunes, capacitación inicial y un equipo de plataforma pequeño que soporte a los equipos de producto. |
| **Dependencia de AWS (lock-in).** | Arquitectura hexagonal (la nube vive en adaptadores), PostgreSQL y Valkey compatibles con otras nubes, IaC versionada y respaldos búnker fuera de AWS. |
| **Costo creciente (FinOps).** | Presupuestos y alertas por cuenta, etiquetas por servicio, Graviton y Karpenter, y revisión mensual de costos. |
| **Datos personales fuera de Ecuador (LOPDP).** | Cifrado con llaves controladas por el banco, consentimiento explícito, cláusulas contractuales con AWS y validación previa con la Superintendencia de Bancos. |
| **Latencia Ecuador → us-east-1** (decenas de milisegundos). | CloudFront en el borde, caché para lecturas frecuentes y llamadas agregadas en los BFF. |

## Backend

| Tecnología | Por qué (frente a alternativas) | Riesgos | Mitigación |
|---|---|---|---|
| **Java 25 LTS** | Lenguaje estándar en la banca; soporte largo, talento disponible y librerías maduras de seguridad. Un solo lenguaje para todo el backend. | Consumo de memoria alto en servicios pequeños si corren sobre la JVM. | Quarkus nativo en los servicios livianos; Graviton para bajar costo. |
| **Spring Boot 4.1 WebFlux** (BFF y dominio) | Reactivo: más solicitudes con menos hilos, ideal para BFF que llaman a varios servicios a la vez. Ecosistema más grande que Quarkus o Micronaut (seguridad, Resilience4j, OpenTelemetry). | Curva alta; depuración más difícil; una llamada bloqueante (p. ej. JDBC) degrada todo el servicio. | R2DBC y clientes no bloqueantes, BlockHound en pruebas, revisiones de código. Los hilos virtuales de Java 25 quedan como opción para servicios simples. |
| **Quarkus 3.40 LTS nativo** (Notificaciones, Auditoría) | Arranca en milisegundos y usa poca memoria, manteniendo el mismo lenguaje, librerías y análisis de seguridad que el resto del backend. | Compilación nativa lenta; la reflexión debe declararse; algunas librerías no son compatibles. | Usarlo solo en servicios simples; pruebas del binario nativo en CI; modo JVM como respaldo. |

## Integración y eventos

| Tecnología | Por qué (frente a alternativas) | Riesgos | Mitigación |
|---|---|---|---|
| **Amazon API Gateway** (público y privado) | Administrado: throttling, cuotas, validación de JWT y WAF sin operar servidores. Frente a Kong o Apigee: menos operación y costo inicial. | Límites de tiempo de espera y tamaño de payload; costo por millón de llamadas a gran volumen; lock-in. | Respuestas rápidas y operaciones largas asíncronas; la lógica vive en los BFF, así que el gateway se puede reemplazar. |
| **EventBridge + SQS** | Serverless: sin clúster que dimensionar. Para el volumen de un banco mediano, Kafka (MSK) es sobredimensionado. | Sin orden garantizado en EventBridge; entrega al menos una vez (duplicados); límite de tamaño de evento; cuotas por región. | Idempotencia en cada consumidor, SQS FIFO para órdenes de pago, eventos pequeños con referencia a S3, monitoreo de cuotas. |
| **AWS DMS (CDC del core)** | Captura cambios del core sin modificarlo. Frente a Debezium: servicio administrado, sin operar Kafka Connect. | Requiere acceso a los logs de la base del core (permiso del proveedor); retraso de replicación; tareas que se detienen. | Validarlo con el proveedor del core antes de diseñar; alarmas de latencia; reanudación desde el checkpoint; Debezium como alternativa. |

## Datos

| Tecnología | Por qué (frente a alternativas) | Riesgos | Mitigación |
|---|---|---|---|
| **Aurora PostgreSQL Global** | PostgreSQL estándar (portable) con réplica entre regiones en menos de un segundo y failover administrado. Frente a Oracle o SQL Server: sin licencias. | Costo mayor que RDS simple; RPO casi cero pero no cero. | Instancias Graviton, escalar lectores según uso; outbox para no perder eventos en el failover. |
| **ElastiCache Valkey** | Compatible con Redis, de código abierto y más barato en AWS que Redis OSS. | Datos desactualizados frente al core. | TTL corto, invalidación por eventos del CDC y el saldo **nunca** se guarda en caché. |
| **DynamoDB** (notificaciones) | Escala sin administración y replica entre regiones con Global Tables. | Modelo de consultas rígido; particiones calientes. | Diseñar las claves según las consultas conocidas; datos relacionales en Aurora. |
| **S3 Object Lock (WORM)** | Evidencia inmutable para auditoría y KYC, a bajo costo. QLDB fue descartado (AWS dejó de darle soporte). | Retención mal configurada imposible de corregir. | Definir plazos con Cumplimiento antes de activar el modo *compliance*. |

## Identidad y seguridad

| Tecnología | Por qué (frente a alternativas) | Riesgos | Mitigación |
|---|---|---|---|
| **Keycloak 26.8** (clientes) | Código abierto, soporta FAPI 2.0 y passkeys, se despliega en la nube del banco. Frente a Cognito: más control sobre perfiles de seguridad; frente a Auth0: sin costo por usuario. | Lo opera el banco: actualizaciones frecuentes, alta disponibilidad y parches de seguridad son responsabilidad propia. | Operador de Keycloak en EKS, clúster multi-AZ, actualizaciones planificadas; soporte comercial (Red Hat build of Keycloak) si se requiere. Reemplazable por el IdP vigente. |
| **Microsoft Entra ID** (empleados) | Estándar corporativo en la banca ecuatoriana: SSO, MFA y acceso condicional ya existentes. | Costo de licencias (acceso condicional requiere planes superiores); dependencia del proveedor. | Reutilizar el tenant existente del banco; integración estándar OIDC, reemplazable. |
| **Passkeys + biometría del dispositivo** | Resistentes a phishing; la biometría nunca sale del teléfono. | Pérdida o cambio de dispositivo; usuarios poco habituados. | Recuperación con nueva prueba de vida (KYC); OTP como respaldo temporal. |
| **Proveedor KYC** (FaceTec, Facephi, iProov) | Prueba de vida certificada (ISO 30107-3) y detección de deepfakes; construirla internamente no es viable. | Costo por verificación; dependencia del proveedor; SDK disponible para Flutter. | Adaptador hexagonal para cambiar de proveedor; verificar plugin Flutter en la evaluación. |

## Frontend

| Tecnología | Por qué (frente a alternativas) | Riesgos | Mitigación |
|---|---|---|---|
| **Angular 22** (web) | Estructura definida, TypeScript obligatorio y seguridad integrada (protección XSS). Muy usado en banca. Frente a React: menos decisiones por equipo y más consistencia. | **Migraciones obligatorias:** desde la v22 sale una versión mayor por año con unos 24 meses de soporte; antes eran cada 6 meses. Si se acumulan versiones, la actualización se vuelve un proyecto. Curva alta. Cambios de paradigma en curso (signals, sin zone.js). | Actualizar cada versión mayor con `ng update` dentro del ciclo normal, como máximo una versión atrás. Lineamientos comunes y librería de componentes propia. |
| **Flutter 3.47** (móvil) | Un solo código para Android e iOS con rendimiento casi nativo; fuerte en fintech. Frente a React Native: UI más consistente entre plataformas. | No tiene LTS: hay que seguir las versiones estables trimestrales. Talento en Dart más escaso; dependencia de Google; los SDK nativos (KYC, atestación) necesitan plugins; apps más pesadas. | Contract-first para generar clientes; *platform channels* para SDK nativos; capacitación del equipo. |

## Infraestructura

| Tecnología | Por qué (frente a alternativas) | Riesgos | Mitigación |
|---|---|---|---|
| **EKS** (Kubernetes) | Estándar portable, service mesh con mTLS, políticas de red y escalado fino con Karpenter. Frente a ECS: más portable y con mayor ecosistema. | Complejidad operativa: requiere equipo de plataforma; actualizaciones de versión de Kubernetes frecuentes. | Add-ons administrados, IaC, actualizaciones planificadas; ECS Fargate como alternativa si el equipo es pequeño. |
| **Respaldos búnker en Oracle Cloud (Bogotá)** | Copia inmutable fuera de AWS ante una falla total o un ataque a la cuenta; región cercana a Ecuador. | Costo de salida de datos (egress); otro proveedor que administrar. | Solo respaldos (no operación activa); restauración probada al menos una vez al año. |
