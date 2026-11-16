# ADR-003: OAuth 2.0 con perfil FAPI 2.0, Keycloak para clientes, Entra ID para empleados y biometría en dos momentos

- **Estado:** Propuesto
- **Fecha:** 2026-11-15

## Contexto

Clientes web y móvil con riesgo creciente de robo de tokens, automatización con IA y deepfakes. Los empleados necesitan acceso corporativo con SSO.

## Decisión

**Authorization Code + PKCE** bajo **FAPI 2.0** (PAR, DPoP, step-up). **Keycloak 26.8** para clientes y **Microsoft Entra ID** para empleados, por su amplia adopción corporativa en la banca ecuatoriana. Ambos son reemplazables por el IdP vigente de la organización, previa evaluación de crecimiento, soporte de estándares, costo y continuidad del proveedor. **Onboarding:** KYC con prueba de vida certificada (ISO 30107-3 + CEN/TS 18099). **Login:** passkeys con la biometría del dispositivo.

## Alternativas descartadas

- **Amazon Cognito**: Soporte limitado de FAPI 2.0 y baja adopción en banca.
- **Contraseña + SMS OTP**: No es resistente a phishing.

## Consecuencias

- ✅ Un token robado no sirve fuera del dispositivo.
- ✅ La biometría nunca sale del teléfono.
- ✅ La arquitectura depende de estándares (OIDC), no de un producto.
- ⚠️ Hay que operar Keycloak en alta disponibilidad.
