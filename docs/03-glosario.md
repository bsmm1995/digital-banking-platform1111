# 03 · Glosario

Términos, siglas y herramientas usados en la propuesta, explicados en una línea.

## Negocio y regulación (Ecuador)

| Término | Significado |
|---|---|
| **BCE** | Banco Central del Ecuador. Administra el sistema de pagos entre entidades financieras. |
| **SPI** | Sistema de Pagos Interbancarios del BCE. Transfiere dinero entre bancos por **cortes** (lotes) durante el día; sin límite de monto. |
| **SPI1** | Formato del archivo con el que cada banco envía sus transferencias al SPI. |
| **RPI** | Red de Pagos Instantáneos del BCE. Transferencias entre bancos **en segundos, 24/7**, con llaves o QR y monto máximo regulado. |
| **ISO 20022** | Estándar internacional de mensajería financiera. *pacs.008* = orden de pago; *pacs.002* = confirmación o rechazo. |
| **Red interbancaria privada** | Red de pagos operada por un privado (p. ej. Banred) que conecta a sus entidades asociadas en tiempo real. |
| **Core bancario** | Sistema central del banco: clientes, cuentas, saldos y movimientos. Es la fuente de verdad contable. |
| **Prospecto** | Persona que aún **no es cliente** y se está registrando (onboarding). Al terminar se convierte en cliente. |
| **Onboarding** | Proceso de alta de un nuevo cliente. |
| **KYC** | *Know Your Customer*: verificar la identidad del cliente antes de abrirle una cuenta. |
| **Registro Civil** | Fuente oficial para validar la cédula y obtener la foto del ciudadano. |
| **LAFT** | Prevención de Lavado de Activos y Financiamiento del Terrorismo. |
| **UAFE** | Unidad de Análisis Financiero y Económico del Ecuador; emite listas de control LAFT. |
| **OFAC / ONU** | Listas internacionales de personas y entidades sancionadas. |
| **PEP** | Persona Expuesta Políticamente; requiere debida diligencia reforzada. |
| **LOPDP** | Ley Orgánica de Protección de Datos Personales del Ecuador. |
| **SPDP** | Superintendencia de Protección de Datos Personales (autoridad de la LOPDP). |
| **SB** | Superintendencia de Bancos; supervisa riesgo operativo, ciberseguridad y continuidad. |
| **PCI-DSS** | Estándar de seguridad para datos de tarjetas de pago. |

## Arquitectura y patrones

| Término | Significado |
|---|---|
| **Spring Boot WebFlux** | Framework Java para servicios **reactivos**: atiende muchas solicitudes con pocos hilos sin bloquearse. |
| **Quarkus** | Framework Java orientado a la nube; puede compilarse a imagen nativa para arrancar en milisegundos con poca memoria. |
| **Imagen nativa (GraalVM / Mandrel)** | Compilar la aplicación Java a un ejecutable del sistema operativo, sin JVM: arranque instantáneo y menos memoria, a cambio de compilaciones más lentas. |
| **C4** | Modelo para diagramar software en niveles: Contexto, Contenedores, Componentes (y Código). |
| **ADR** | *Architecture Decision Record*: documento corto que registra una decisión y sus alternativas. |
| **Arquitectura hexagonal** | El dominio (reglas de negocio) se aísla de la tecnología mediante **puertos** (interfaces) y **adaptadores** (implementaciones). |
| **BFF** | *Backend for Frontend*: un backend dedicado a cada canal (web, móvil) que orquesta y adapta las respuestas. |
| **ACL** | *Anti-Corruption Layer*: capa que traduce el modelo del sistema legado al modelo nuevo, protegiéndolo de sus particularidades. |
| **Saga** | Secuencia de pasos con compensación: si un paso falla, se deshacen los anteriores (p. ej. reverso del débito). |
| **Transactional Outbox** | Guardar el evento en la misma transacción que el cambio de datos y publicarlo después, para no perder eventos. |
| **Idempotencia** | Repetir una operación produce el mismo resultado. La *Idempotency-Key* evita debitar dos veces ante un reintento. |
| **Router de rieles** | Componente que elige por dónde enviar una transferencia (core, RPI, SPI) según destino, monto y horario. |
| **Strategy** | Patrón que permite cambiar o agregar algoritmos (rieles, canales) sin modificar el código existente. |
| **Cache-Aside** | Se busca primero en caché; si no está, se consulta el origen y se guarda en caché. |
| **LFU** | *Least Frequently Used*: la caché descarta primero lo menos consultado; lo frecuente permanece. |
| **CDC** | *Change Data Capture*: detectar los cambios de una base de datos y publicarlos como eventos, sin modificar la aplicación origen. |
| **Pub/Sub** | Un servicio publica un evento y varios interesados lo reciben sin conocerse entre sí. |
| **DLQ** | *Dead Letter Queue*: cola donde quedan los mensajes que fallaron tras varios reintentos, para analizarlos. |
| **FIFO** | *First In, First Out*: los mensajes se procesan en el orden en que llegaron. |
| **Circuit breaker** | Corta temporalmente las llamadas a un servicio que falla, para no saturarlo ni propagar el error. |
| **Contract-first** | Definir primero el contrato de la API (OpenAPI) y generar desde él el código de cliente y servidor. |
| **WORM** | *Write Once, Read Many*: almacenamiento que no permite modificar ni borrar lo escrito. |
| **Hash encadenado** | Cada registro incluye la huella (hash) del anterior; cualquier alteración rompe la cadena y se detecta. |
| **Envelope encryption** | Los datos se cifran con una llave de datos (una por cliente), y esa llave se cifra con una llave maestra de KMS. Así bastan pocas llaves en KMS para millones de clientes. |
| **Almacén de llaves (key store)** | Tabla separada de los datos donde se guarda, cifrada, la llave de cada cliente. Sus respaldos tienen retención corta para que el borrado sea efectivo. |
| **Crypto-shredding** | Borrar datos destruyendo su llave de cifrado: quedan ilegibles incluso en respaldos. |
| **RTO / RPO** | RTO: tiempo máximo para recuperar el servicio. RPO: cantidad máxima de datos que se pueden perder. |
| **Warm standby / Pilot light** | Estrategias de DR: réplica mínima siempre encendida / solo datos replicados y cómputo bajo demanda. |
| **Respaldo búnker** | Copia inmutable de los datos en otro proveedor de nube, para sobrevivir a una falla total del proveedor principal. |
| **IaC** | *Infrastructure as Code*: la infraestructura se define en archivos (p. ej. Terraform) y se crea idéntica con un comando. |
| **Multi-AZ** | Desplegar en varias zonas de disponibilidad (data centers separados) de una misma región. |

## Seguridad e identidad

| Término | Significado |
|---|---|
| **IdP** | *Identity Provider*: sistema que autentica usuarios y emite tokens. |
| **OAuth 2.0 / OIDC** | Estándares de autorización (OAuth) e identidad (OpenID Connect) usados para el login. |
| **Authorization Code + PKCE** | Flujo OAuth recomendado para apps web y móviles; PKCE impide el robo del código de autorización. |
| **FAPI 2.0** | *Financial-grade API*: perfil de seguridad de OAuth para banca (Open Banking / Open Finance). |
| **PAR** | *Pushed Authorization Request*: los parámetros del login se envían por un canal seguro, no por el navegador. |
| **DPoP** | Liga el token a una llave del dispositivo; un token robado no sirve en otro equipo. |
| **RAR** | *Rich Authorization Request*: la autorización incluye el detalle de la operación (monto, beneficiario). |
| **Step-up** | Pedir una autenticación más fuerte para operaciones sensibles, como una transferencia. |
| **CIBA** | Flujo OAuth que permite aprobar una operación desde otro dispositivo (útil para agentes autorizados). |
| **Passkey / FIDO2 / WebAuthn** | Credencial sin contraseña, desbloqueada con huella o Face ID; resistente a phishing. |
| **SSO / MFA** | Inicio de sesión único / autenticación con más de un factor. |
| **Liveness (prueba de vida)** | Verificar que la selfie corresponde a una persona real presente, no a una foto o video. |
| **Face match 1:1** | Comparar la selfie contra una foto de referencia (cédula o Registro Civil). |
| **ISO/IEC 30107-3 (PAD)** | Norma que certifica la detección de ataques de **presentación** (fotos, máscaras, pantallas). |
| **CEN/TS 18099 (IAD)** | Norma que certifica la detección de ataques de **inyección** (deepfakes enviados directo al sistema, sin cámara). |
| **Atestación de app** | *Play Integrity* (Android) / *App Attest* (iOS): confirman que la app es la original y corre en un dispositivo real. |
| **Certificate pinning** | La app solo confía en el certificado del banco; evita interceptar la comunicación. |
| **Namespace** | División lógica dentro de Kubernetes para separar grupos de servicios y aplicarles políticas de red y permisos. |
| **Service mesh** | Capa (p. ej. Istio) que gestiona automáticamente mTLS, reintentos y observabilidad entre servicios, sin tocar el código. |
| **mTLS** | TLS mutuo: cliente y servidor se autentican con certificados. |
| **Keycloak** | IdP open source; en la propuesta autentica a los clientes. |
| **Microsoft Entra ID** | IdP corporativo de Microsoft; en la propuesta autentica a los empleados. |
| **OWASP** | Comunidad que publica estándares de seguridad de aplicaciones (Top 10, ASVS). |

## Servicios AWS

| Servicio | Para qué se usa |
|---|---|
| **EKS** (+ Karpenter, Graviton) | Kubernetes administrado donde corren los microservicios; Karpenter escala nodos y Graviton reduce costo. |
| **API Gateway** | Puerta de entrada a las APIs: límites de uso, validación y enlace privado (**VPC Link**) hacia los servicios. |
| **ALB** | Balanceador de aplicaciones; en la propuesta expone solo el login de Keycloak. |
| **VPC / Subred** | Red privada del banco en AWS, dividida en subredes pública (salida/entrada controlada), de aplicación y de datos. |
| **NLB** | Balanceador de red privado entre API Gateway y EKS. |
| **CloudFront** | CDN: sirve la web desde ubicaciones cercanas al usuario. |
| **WAF / Shield** | Firewall de aplicaciones web / protección contra ataques DDoS. |
| **Route 53 + ARC** | DNS y control de conmutación entre regiones ante desastres. |
| **Aurora (Global Database)** | Base de datos PostgreSQL administrada, replicada a otra región. |
| **ElastiCache (Valkey)** | Caché en memoria administrada. |
| **DynamoDB** | Base de datos NoSQL administrada. |
| **S3 (Object Lock)** | Almacenamiento de archivos; *Object Lock* lo hace inmodificable (WORM). |
| **EventBridge (+ Pipes)** | Bus de eventos con reglas de enrutamiento; *Pipes* conecta fuentes (Kinesis) con el bus. |
| **SQS** | Colas de mensajes administradas, con reintentos y DLQ. |
| **DMS** | *Database Migration Service*: captura cambios (CDC) de la base del core. |
| **Kinesis Data Streams** | Flujo de datos en tiempo real que transporta los cambios capturados por DMS. |
| **KMS / Secrets Manager** | Gestión de llaves de cifrado / gestión de contraseñas y secretos. |
| **SES / End User Messaging** | Envío de correos / envío de SMS. |
| **Direct Connect / Transit Gateway / DX Gateway** | Enlace dedicado entre el data center del banco y AWS / concentrador de redes / une el enlace con varias regiones (por eso DR usa el mismo Direct Connect). |
| **NAT Gateway** | Salida controlada a internet desde redes privadas. |
| **CloudWatch, GuardDuty, Security Hub, CloudTrail, Config** | Monitoreo, detección de amenazas, postura de seguridad, registro de acciones y cumplimiento de configuración. |
| **AWS Backup** | Respaldos centralizados e inmutables (*vault lock*). |

## Frontend

| Término | Significado |
|---|---|
| **SPA** | *Single Page Application*: aplicación web que carga una vez y se actualiza sin recargar la página. |
| **Angular** | Framework web de Google, estructurado y con TypeScript; usado para la banca web. |
| **Flutter** | Framework de Google para apps iOS y Android con un solo código; usado para la app móvil. |
