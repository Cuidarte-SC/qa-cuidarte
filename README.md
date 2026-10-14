# qa-cuidarte

Pruebas y evidencias de la **EP02 (ISY1102 · Seguridad y Calidad en el Desarrollo de Software)** sobre **Cuidarte+ v1.1.0**, es decir, el tag `v1.1.0` de `api-cuidarte`, `frontend-cuidarte` e `infra-cuidarte`.

Equipo: Valentina Pino Norambuena · Wilfred Vinet. Sección 003V.

Este repo está separado del código del sistema para mantener la independencia entre quien desarrolla y quien prueba.

## Estructura

```
qa-cuidarte/
├── postman/      # colección Postman con los casos de API y sus asserts
├── evidencias/   # capturas: Anexo_X.Y_CP-XXX-NN_descripcion.png
└── reportes/     # ESLint, npm audit, ZAP, JMeter, Lighthouse (se agregan al ejecutar)
```

## Casos de prueba de la EP02 

★ = caso heredado de la EP01.

| Caso | Tipo | Requerimientos | Herramienta |
|---|---|---|---|
| ★CP-FUN-01 | Funcional | RF-1.1, RF-1.2, NFR-SEG-2, NFR-SEG-9 | Postman |
| ★CP-FUN-02 | Funcional | RF-2.2, NFR-SEG-5 | Postman |
| CP-FUN-03 | Funcional | RF-2.1 | Postman + navegador |
| CP-FUN-04 | Funcional | RF-2.3, RF-2.4, RB-1 | Postman |
| CP-FUN-05 | Funcional | RF-2.5, RF-2.6, RF-2.7, RB-2 | Postman + navegador |
| CP-FUN-06 | Funcional | RF-3.2, RF-3.3, RF-5.1 | Postman + navegador |
| ★CP-INT-01 | Integración | RF-3.1, RF-5.1, RB-3 | Postman |
| CP-INT-02 | Integración | RF-4.2, RF-4.3, RF-4.4, RF-3.2 | Navegador |
| CP-INT-03 | Integración | RF-4.1, RF-4.2, RB-1, NFR-USAB-4 | Navegador |
| ★CP-SEG-01 | Seguridad | NFR-SEG-4, RB-1 | Postman |
| CP-SEG-02 | Seguridad | NFR-SEG-5 | OWASP ZAP |
| CP-SEG-03 | Seguridad | NFR-SEG-1, NFR-SEG-2, NFR-SEG-3 | Navegador, terminal |
| CP-SEG-04 | Seguridad | NFR-SEG-6, NFR-SEG-7, NFR-SEG-8 | Terminal, scripts de respaldo |
| CP-SAST-01 | Análisis estático | NFR-SEG-5, NFR-SEG-6, ISO/IEC 25010 | ESLint, npm audit |
| ★CP-PERF-01 | Rendimiento | NFR-PERF-1, NFR-PERF-3 | Apache JMeter |
| CP-PERF-02 | Rendimiento | NFR-PERF-2 | Lighthouse |
| ★CP-USA-01 | Usabilidad | NFR-USAB-1, NFR-USAB-2, NFR-USAB-4, NFR-COMPAT-2 | axe DevTools, Lighthouse |
| CP-USA-02 | Usabilidad | NFR-USAB-3 | Navegador |
| CP-COMPAT-01 | Compatibilidad | NFR-COMPAT-1 | Chrome, Edge, Firefox |
| CP-DIS-01 | Disponibilidad | NFR-DIS-1, NFR-DIS-2 | Terminal, Docker |

## Colección Postman

`postman/Cuidarte_EP02.postman_collection.json`: 46 peticiones y 77 verificaciones automáticas. Cada carpeta lleva el ID del caso que respalda.

| Carpeta | Qué verifica |
|---|---|
| 00 · Ambiente | API, BD y especificación OpenAPI disponibles |
| CP-FUN-01 | Login 200 + JWT de 15 min; 401 genérico; token revocado tras logout |
| CP-FUN-02 | 400 con RUT de DV incorrecto; 201 con RUT válido; 409 duplicado |
| CP-INT-01 | Examen creado (201) y registro CREATE en auditoría |
| CP-SEG-01 | 403 al crear; 403 a ficha ajena; BD intacta; intento auditado |
| CP-FUN-03 | Autoregistro 201 con rol paciente aunque se envíe `rol: admin` |
| CP-FUN-01 · Parte 2 | Refresh token: renovación con rotación; reutilización 401 |
| CP-FUN-05 | Gestión de usuarios: listar, vincular ficha, rol inválido, autobloqueo, eliminar |
| CP-FUN-04 | Editar paciente 200; RUT inválido 400; paciente no edita 403 |
| CP-FUN-06 | Eliminar examen: paciente 403; médico 204; luego 404 y auditado |

### Ejecución

1. Deja la BD en su estado inicial. La colección espera datos limpios:
   ```powershell
   cd ..\infra-cuidarte
   docker compose --profile app down -v
   docker compose --profile app up -d --build
   ```
2. En Postman: **Import** → `postman/Cuidarte_EP02.postman_collection.json`.
3. Ejecuta las carpetas **en orden**, porque comparten tokens: **Run collection**, o *Send* petición por petición.
4. Si aparece **429**, se superaron los 10 logins en 15 minutos. Reinicia la API (`docker compose --profile app restart api`) y continúa.

## Nomenclatura de evidencias

`Anexo_<sección>.<n>_<ID-caso>_<descripcion>.png`. Por ejemplo, `Anexo_1.1_CP-FUN-01_login-200-jwt.png`.

| Anexo | Contenido |
|---|---|
| 0.x | Ambiente QA, organización y trazabilidad en Git |
| 1.x | Pruebas funcionales |
| 2.x | Pruebas de integración |
| 3.x | Seguridad y análisis estático |
| 4.x | Rendimiento |
| 5.x | Usabilidad y compatibilidad |
| 6.x | Disponibilidad |

Cada captura debe mostrar el dato que valida el caso (código HTTP, Test Results, registro de auditoría, alerta, puntaje), con fecha y hora visibles.

## Ajustes respecto de la EP01

- **Datos de prueba**: el RUT `18.765.432-K` de CP-FUN-02 tiene un dígito verificador incorrecto; el correcto es **7**. La colección conserva el dato original, que se espera rechazado con 400, y usa el corregido para el registro exitoso. El RUT `19.123.456-L` de CP-SEG-01 tampoco es válido, porque ningún RUT termina en L; no altera el resultado, ya que el control de acceso responde 403 antes de validar los datos.
- **Trazabilidad**: CP-FUN-02 se trazaba a RF-2.3 (editar paciente); se corrige a RF-2.2. CP-INT-01 citaba RB-2 para el médico; se corrige a RB-3.
- **Cobertura**: de 6 casos en la EP01 a 20 en la EP02, con cobertura completa del ERS.
