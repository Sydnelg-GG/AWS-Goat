Reto: Sensitive Data Exposure

Endpoint legítimo: https://xcflyck1ye.execute-api.us-east-1.amazonaws.com/v1/list-posts

Endpoint oculto descubierto: https://xcflyck1ye.execute-api.us-east-1.amazonaws.com/v1/dump

Herramienta: Burp Suite Intruder (alternativa a ZAP Fuzzer)

Wordlist: /usr/share/wordlists/dirb/common.txt

Metodología:

1. Se interceptó la request a list-posts con Burp Proxy.
2. Se envió a Intruder, marcando "list-posts" como punto de inyección.
3. Se cargó el wordlist de dirb como payload.
4. Se lanzó el ataque - la mayoría de rutas devolvieron 403 Forbidden.
5. Se identificó la ruta "dump" devolviendo 200 OK.
6. Al acceder directamente vía GET (navegador) a /v1/dump, se obtuvo un JSON completo con TODOS los usuarios registrados, sin ningún tipo de autenticación.

Datos expuestos: email, password (hash bcrypt), secretQuestion, secretAnswer, address, phone, country, authLevel, username, id, userStatus, creationDate.

Clasificación OWASP: A01:2021 - Broken Access Control 
(endpoint sin protección de autenticación/autorización)

Impacto: CRÍTICO. Cualquier persona sin autenticar puede obtener la base de 
datos completa de usuarios, incluyendo hashes de contraseñas y preguntas de 
seguridad, habilitando ataques de fuerza bruta offline, ingeniería social, 
y compromiso de cuentas mediante recuperación de contraseña.
