**Reto: SQL Injection**

URL objetivo: https://856k42qfe7.execute-api.us-east-1.amazonaws.com/prod/react#/home

Endpoint: https://xcflyck1ye.execute-api.us-east-1.amazonaws.com/v1/search-author

Herramienta: Burp Suite (Proxy + Repeater)

Payload usado:
"value":"hello' or '1'='1"
"authlevel":"hello' or '1'='1"

<img width="589" height="537" alt="image" src="https://github.com/user-attachments/assets/a375f18b-f188-4164-925e-dc2dc861d6d1" />


Resultado: Se obtuvieron los datos de TODOS los usuarios registrados 
(no solo los que coincidían con la búsqueda "h"), incluyendo campos 
sensibles como teléfono, dirección e IDs.

<img width="589" height="324" alt="image" src="https://github.com/user-attachments/assets/08c4bb5d-8644-4ce9-87f5-84889cb76cbb" />


Clasificación OWASP: A03:2021 - Injection (SQL Injection)
Impacto: Bypass de autenticación/autorización en la consulta, exposición 
masiva de datos sensibles de usuarios sin validación de input en el backend.
