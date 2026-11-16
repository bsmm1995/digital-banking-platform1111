# ADR-002: Integración con API Gateway + BFF, Saga para transferencias y eventos serverless

- **Estado:** Propuesto
- **Fecha:** 2026-11-15

## Contexto

El reto pide un API Gateway que orqueste los servicios de cliente, movimientos y transferencias. Las transferencias interbancarias cruzan el core y el BCE sin una transacción única. Los servicios deben comunicarse sin acoplarse.

## Decisión

**Amazon API Gateway** para lo transversal y **BFF por canal** (Spring WebFlux) para orquestar. Transferencias con **Saga orquestada**, idempotencia y outbox. Eventos asíncronos con **EventBridge + SQS**; CDC del core con **AWS DMS**. Servicios en **Java 25 LTS**: Spring Boot 4.1 WebFlux en BFF y dominio; Quarkus 3.40 LTS nativo en los transversales livianos.

## Alternativas descartadas

- **Orquestar en el gateway**: Acopla la lógica de negocio a la infraestructura.
- **Go para los servicios livianos**: Agrega un segundo lenguaje, otra cadena de librerías y de análisis de seguridad; Quarkus nativo logra un consumo similar sin salir de Java.
- **Apache Kafka (MSK)**: Sobredimensionado para una cooperativa: exige operar y dimensionar un clúster.

## Consecuencias

- ✅ Gateway delgado y canales independientes.
- ✅ Ningún doble débito ante reintentos.
- ✅ Eventos serverless, pagados por uso y sin servidores que operar.
- ⚠️ EventBridge entrega *al menos una vez*: los consumidores deben ser idempotentes.
