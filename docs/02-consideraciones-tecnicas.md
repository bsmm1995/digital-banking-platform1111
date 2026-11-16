# 02 · Consideraciones técnicas

![Infraestructura AWS](../infra/infraestructura-aws.png)

## 1. Normativa

| Norma | Cómo se atiende |
|---|---|
| **LOPDP** y su Reglamento | Hay consentimientos por finalidad, minimización de datos y atención de derechos. AWS actúa como **encargado del tratamiento** mediante contrato y cláusulas tipo. La SPDP aclaró que un encargado fuera del país no convierte, por sí solo, el tratamiento en transferencia internacional. |
| Derecho de supresión | **Cifrado con una llave por cliente** (AWS KMS). Destruir la llave deja sus datos ilegibles, incluso en respaldos que no se pueden modificar. |
| **Superintendencia de Bancos**: riesgo operativo, ciberseguridad y continuidad | Plan de continuidad con objetivos de recuperación (RTO/RPO) por nivel, pruebas periódicas de DR, gestión de proveedores críticos y monitoreo 24/7. |
| **Banco Central del Ecuador**: SPI y Red de Pagos Instantáneos | Conector dedicado con mensajería **ISO 20022** y conciliación diaria. |
| **LAFT / UAFE** | Screening en el onboarding y monitoreo antifraude de las transacciones. |
| **PCI-DSS v4.0.1** | Aplica solo si se manejan datos de tarjeta. Se reduce el alcance con **tokenización**: el número de tarjeta nunca llega a la plataforma. |
| ISO/IEC 27001 | Marco del sistema de gestión de seguridad de la información (SGSI). |

## 2. Alta disponibilidad y tolerancia a fallos

- **Multi-AZ** en todos los componentes: EKS, Aurora y ElastiCache se reparten en 3 zonas de disponibilidad. EventBridge y SQS son regionales por diseño.
- **Resiliencia en el código:** circuit breaker, timeouts y reintentos con Resilience4j.
- **Degradación controlada:** si el core no responde, las consultas muestran el último dato conocido y las transferencias quedan **en cola sin perderse**.
- **Desacoplamiento por eventos:** si Notificaciones o Auditoría fallan, los mensajes esperan en SQS y se procesan al volver.
- **Conexión al core redundante:** 2 enlaces Direct Connect y una VPN de respaldo.

## 3. Recuperación ante desastres

| Nivel | Servicios | RPO | RTO | Estrategia |
|---|---|---|---|---|
| **T0** | Identidad, transferencias, auditoría | ≈ 0 | ≤ 15 min | *Warm standby* en us-east-2 (Aurora Global Database) |
| **T1** | Consultas y notificaciones | ≤ 1 min | ≤ 30 min | *Warm standby* |
| **T2** | Onboarding y backoffice | ≤ 15 min | ≤ 4 h | *Pilot light* (datos replicados, cómputo bajo demanda) |
| **T3** | Reportes | ≤ 24 h | ≤ 24 h | Backup & restore |

- **Por qué niveles:** perder una transferencia confirmada es un riesgo financiero y regulatorio, mientras que un reporte tolera más tiempo. Así el costo del DR es **proporcional a la criticidad**.
- **Respaldos inmutables** con AWS Backup en otra región y otra cuenta, como protección frente a ransomware.
- **Conmutación controlada** con Route 53 ARC.
- **Simulacros de DR** semestrales.

## 4. Seguridad y monitoreo

- **En tránsito:** TLS 1.3, mTLS entre servicios y con el BCE, y *certificate pinning* en la app.
- **En reposo:** todo cifrado con **AWS KMS** usando llaves del banco. Los datos personales sensibles se cifran además a nivel de campo.
- **Accesos:** mínimo privilegio con IAM, cuentas AWS separadas por entorno y acceso de empleados con **Entra ID + MFA**.
- **Desarrollo seguro (OWASP):** ASVS y API Security Top 10, más análisis de código y dependencias (Checkmarx, Snyk) en CI/CD.
- **Monitoreo continuo:** CloudWatch con OpenTelemetry, GuardDuty, Security Hub, CloudTrail y AWS Config, con alertas por objetivos de nivel de servicio (SLO).

## 5. Escalabilidad y desacoplamiento

- **Un servicio nuevo no toca a los existentes:** se suscribe a eventos en EventBridge o se publica detrás de los BFF.
- **Contract-first** (OpenAPI) con versionado y pruebas de contrato.
- **Arquitectura hexagonal:** cambiar el core o un proveedor (KYC, mensajería, IdP) solo cambia un adaptador.
- **Escalado automático e independiente** por servicio en EKS. EventBridge y SQS escalan solos.
- **Reactivo** (Spring WebFlux): más throughput con menos recursos.

## 6. Infraestructura cloud y baja latencia

**AWS** con región principal **us-east-1** (N. Virginia) y DR en **us-east-2** (Ohio).

| Opción | Decisión |
|---|---|
| **us-east-1** | ✅ Principal: catálogo completo de servicios, menor costo y buena conectividad desde Ecuador. |
| us-east-2 | ✅ DR: mismo precio y otra zona geográfica. |
| São Paulo | ❌ De las regiones más caras de AWS; la ruta desde Ecuador no suele ser más corta. |
| Chile | 🔜 Evolución: se re-evalúa cuando tenga disponibilidad general y catálogo completo. |

**Baja latencia:**
- **CloudFront** sirve la web desde el borde, cerca del usuario.
- **Caché** para clientes frecuentes y **composición en paralelo** en los BFF.
- **Direct Connect** dedicado hacia el core.
- *La latencia real desde Quito y Guayaquil se mide antes de la puesta en producción.*
