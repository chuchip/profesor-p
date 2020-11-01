---
title: Securizando aplicación de SpringBoot con KeyCloack
pre: "<b>o </b>"
author: El Profe
type: post
date: 2020-11-01T00:00:00+00:00
url: /springboot/seguridad-con-keycloack
categories:
  - java
  - oauth2
  - keycloack
  - jwt
  - rest
  - seguridad
  - spring boot
tags:
  - java
  - oauth2
  - keycloack
  - jwt
  - rest
  - seguridad
  - spring boot

---

### <u>_Borrador_</u>. Pendiente de refactorizar y documentar en español.

Programa demostrando como crear una aplicación en Spring Boot cuya seguridad es gestionada por [KeyCloak](https://www.keycloak.org/).

El programa esta basado en este articulo  https://medium.com/@ddezoysa/securing-spring-boot-rest-apis-with-keycloak-1d760b2004e

<!--more-->

El código fuente del programa esta en https://github.com/chuchip/keycloack-springboot

### 

En la clase `KeycloakSecurityConfig`se ha incluido la función accessToken . Con este **bean** podremos comprobar dentro de nuestro código como los recursos del token JWT mandado a la aplicación.

```java
 @Bean
 @Scope(scopeName = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
 public AccessToken accessToken() {
     HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
     return ((KeycloakSecurityContext) ((KeycloakAuthenticationToken) request.getUserPrincipal()).getCredentials()).getToken();
 }
```



