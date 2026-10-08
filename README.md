# qa-cuidarte

Pruebas y evidencias de la **EP02 (ISY1102)** sobre **Cuidarte+ v1.0.0** (`api-cuidarte` e `infra-cuidarte`, tag `v1.0.0`).

Este repo está separado del código del sistema para mantener la independencia entre quien desarrolla y quien prueba.

## Estructura

```
qa-cuidarte/
├── postman/      # colección con los casos de la EP01 y sus asserts
├── evidencias/   # capturas: Anexo_X.Y_CP-XXX-NN_descripcion.png
└── reportes/     # ESLint, npm audit, ZAP, JMeter, Lighthouse (se agregan al ejecutar)
```

## Casos automatizados en Postman

| Carpeta | Caso EP01 | Requerimientos | Resultado esperado |
|---|---|---|---|
| 00 · Ambiente | — | NFR-DIS | API y BD operativas |
| CP-FUN-01 | Autenticación | RF-1.1, NFR-SEG-2, NFR-SEG-9 | 200 + JWT de 15 min; 401 genérico; token revocado tras logout |
| CP-FUN-02 | Registro de paciente | RF-2.2, RF-2.3, RB-2 | 400 con DV incorrecto; 201 con rol paciente; 409 duplicado |
| CP-INT-01 | Examen + auditoría | RF-3.1, RF-5.1, NFR-SEG-6 | 201; registro CREATE en auditoría |
| CP-SEG-01 | RBAC | NFR-SEG-4, RB-1 | 403 al crear; 403 a ficha ajena; BD intacta; intento auditado |

## Ejecución

1. Reinicia la BD a su estado inicial. La colección espera datos limpios: el RUT 18.765.432-7 no debe existir.
   ```powershell
   cd ..\infra-cuidarte
   docker compose --profile app down -v
   docker compose --profile app up -d api
   ```
2. En Postman: **Import** → `postman/Cuidarte_EP02.postman_collection.json`.
3. Ejecuta las carpetas **en orden**, porque los tokens se comparten entre peticiones. Puedes usar **Run collection** o *Send* petición por petición.

## Nomenclatura de evidencias

`Anexo_<sección>.<n>_<ID-caso>_<descripcion>.png`. Por ejemplo, `Anexo_1.1_CP-FUN-01_login-200-jwt.png`.

Cada captura debe mostrar el método, la URL, el código HTTP y la pestaña **Test Results**.

## Ajuste respecto de la EP01

El dato de prueba de CP-FUN-02 (`18.765.432-K`) tiene un dígito verificador incorrecto; el correcto es **7**. La colección conserva el dato original, que se espera rechazado con 400, y usa el corregido para el registro exitoso. El RUT `19.123.456-L` de CP-SEG-01 tampoco es válido: ningún RUT termina en L. En ese caso no altera el resultado, porque el control de acceso responde 403 antes de validar los datos.
