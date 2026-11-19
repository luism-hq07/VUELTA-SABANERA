# Vuelta Sabanera · pantallas basadas en la API Sitram

Versión 1.1.0. Conserva los colores, tarjetas, formularios y navegación de la entrega anterior, adaptando el contenido al Swagger del profesor:

https://api.sitram.cecar.cloud/swagger/index.html

## Abrir y ejecutar

Descomprime el archivo y abre en VS Code la carpeta **`vuelta_sabanera`**, que contiene directamente `pubspec.yaml`, `lib`, `android` y `web`. En su terminal:

```powershell
flutter pub get
flutter run -d chrome
```

Si tu terminal está en la carpeta que contiene el proyecto, primero ejecuta:

```powershell
cd .\vuelta_sabanera
```

Ejecuta los comandos de uno en uno. No copies los símbolos `PS ...>` ni `>>`. Si PowerShell muestra `>>`, cancela con Ctrl+C y escribe de nuevo el comando.

## Elegir el origen de los datos

La pantalla de acceso tiene el selector **“Usar datos de muestra”**:

| Selector | Acceso | Datos |
|---|---|---|
| Activado | Usuario `demo`, contraseña `Vuelta123!` | Registros ficticios con la misma estructura JSON de Sitram |
| Desactivado | Usuario y contraseña del profesor | Peticiones reales a `https://api.sitram.cecar.cloud/api` |

También puedes iniciar directamente en modo API:

```powershell
flutter run -d chrome --dart-define=DEMO=false
```

La sesión real utiliza el rol y la empresa devueltos por Sitram. Los datos de muestra nunca se envían al servidor. El token solo se mantiene en memoria; al cerrar o recargar la app se inicia sesión nuevamente. La contraseña no se guarda.

No se han proporcionado credenciales del profesor, por lo que se comprobó el contrato público y se probaron respuestas simuladas; no se afirma haber iniciado una sesión real en su servidor.

## Qué cambia en las pantallas

| Pantalla / recurso | Datos utilizados |
|---|---|
| Acceso y perfil | Username, identificación, rol real y empresa (razón social, NIT) |
| Asociados | Identificación, nombres, estado activo, celular, correo, nacimiento, ingreso y credencial |
| Conductores | Identificación, nombres, nacimiento, activo, número/categoría/vigencia de licencia, celular y correo |
| Vehículos | Placa, conductor, tipo, marca, modelo, pasajeros, tarjeta de propiedad, tarifa, activo, SOAT, revisión técnico-mecánica e ingreso |
| Afiliaciones | ID, identificación del asociado, placa, fechas de afiliación/fin y estado |
| Despachos | ID, ruta, placa, hora, fecha, pasajeros, estado, autor y cancelación |
| Planillas | Número, conductor, placa, tipo, estado, tarifa y datos del pago |
| Tarifas | Gastos varios, reposición, aporte social, total y activo |
| Pagos | Planilla, valor pagado, fecha y usuario que lo registró |
| Sanciones | Conductor, motivo, inicio, fin y estado |
| Mantenimientos | Vehículo, ingreso, motivo, observaciones, salida y estado |

Las fichas completas incluyen fechas de registro y actualización cuando el DTO las proporciona. Los valores nulos se muestran como **“No registrado”**, no como métricas o importes calculados sin fuente. Los contadores del inicio cuentan los registros consultados. No se presupone que un asociado activo esté al día en cartera.

## Dónde encontrar los módulos nuevos

Como administrador de muestra: **Inicio → Todos los módulos** o **Mi perfil → Módulos de la cooperativa**.

El menú operativo contiene diez módulos: Asociados, Conductores, Vehículos, Afiliaciones, Despachos, Tarifas, Planillas, Pagos, Sanciones y Mantenimientos. Tienen consulta, filtros publicados por la API, paginación y fichas completas. Los botones de registro/edición/cancelación se muestran según las operaciones disponibles de cada recurso. Sitram sigue autorizando cada petición; un 403 se muestra al usuario.

Con rol **ASOCIADO**, se usan **Mis vehículos**, **Mis despachos** y **Mis planillas y viajes**, a través de `/mi-cuenta/*`. Estas pantallas son de consulta y no ofrecen operaciones de gestión.

Para revisar las 15 pantallas originales: administrador de muestra → Mi perfil → Ver las 15 pantallas. Sus rutas de archivos se conservan; varias fichas reutilizan ahora un componente de detalle para presentar todos los campos del DTO.

## Formularios y operaciones

- Asociados y conductores conservan el registro en dos pasos con los campos de sus Request DTO.
- Los formularios comunes usan fechas, horas, enteros, decimales, booleanos y enums del contrato.
- Los campos marcados con asterisco se validan en el frontend. El backend puede aplicar reglas adicionales que Swagger no documenta; sus mensajes se presentan en pantalla.
- Editar no envía campos de solo lectura ni identificadores que no formen parte del Request.
- Cancelar/finalizar/anular pide la confirmación dentro del formulario antes de enviar la petición.
- El pago recibe número de planilla y fecha de pago: **no solicita un valor inventado**. El importe lo devuelve la API.
- Registrar salida de mantenimiento utiliza `PUT /mantenimientos/{id}` con fecha y observaciones de salida.

Bruno es independiente del frontend: puedes seguir usando la colección que te dio el profesor para probar las mismas peticiones. Este proyecto no crea otra API ni otra base de datos.

## Organización del código

- `docs/PANTALLAS_Y_CARPETAS.md`: cada pantalla, carpeta y componente.
- `docs/API_DEL_PROFESOR.md`: endpoints y operaciones implementadas.
- `docs/swagger_sitram_2026-11-17.json`: copia del contrato público consultado.
- `docs/ejemplos_respuesta.json`: datos ficticios en formato DTO para estudiar el código.
- `docs/VALIDACION.md`: comprobaciones ejecutadas y sus límites.

## Funciones del diseño que no publica Sitram

Las pantallas OTP siguen disponibles para revisar el diseño en modo muestra (código `48219`). En acceso real no se ofrece recuperación por OTP: el Swagger no publica esos endpoints. El cambio autenticado de contraseña usa `/auth/cambiar-password`, incluyendo el caso `requiereCambioPassword` del login.

El rol `CONDUCTOR` no existe en el enum del Swagger. La vista Conductor es una vista previa de diseño con datos de muestra; los roles reales son `ADMIN`, `GERENTE`, `ASISTENTE` y `ASOCIADO`.

Esta entrega cubre la gestión operativa de una empresa y los endpoints personales. No incluye la administración global `/plataforma/empresas`, el registro de usuarios `/auth/registrar`, las credenciales temporales ni la generación masiva de planillas. Tampoco inventa GPS, RUNT, biometría, cartera, calificación o puntualidad. El logo y las fotografías originales se pueden reemplazar en `assets/images/`.

## Requisitos y solución de problemas

Se comprueba con Flutter 3.38.1 y Dart 3.10.0. La compatibilidad declarada empieza en Flutter 3.24/Dart 3.5, pero no se han probado todas las versiones intermedias.

```powershell
flutter --version
flutter doctor
flutter pub get
flutter analyze
flutter test
```

Si aparece “Expected to find project root”, entra en `vuelta_sabanera`, donde está `pubspec.yaml`. Si no encuentra `package:flutter/...`, revisa el SDK y completa `flutter pub get`. La salida sobre versiones más nuevas disponibles no equivale por sí sola a un error.

Un error de CORS en Chrome requiere permitir el origen desde el servidor. Para Android se usa la misma URL HTTPS, con permiso INTERNET ya declarado. Se incluyen las carpetas Android/web; no se incluye una APK ni el SDK de Android.
