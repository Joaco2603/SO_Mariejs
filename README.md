# PROYECTO SC-8 — Demostración de Seguridad (Single Block)

Este repositorio contiene una pequeña demostración en lenguaje de bajo nivel (ensamblador tipo "SC-8") que ilustra la separación entre modo usuario y modo kernel y una llamada al sistema (syscall) que accede a datos secretos.

## Resumen

- Archivo principal: `code.mas`
- Propósito: mostrar cómo se realiza una transición controlada al kernel para ejecutar código privilegiado (leer datos secretos) y luego regresar al contexto de usuario.
- Comportamiento esperado: el programa de usuario invoca la rutina del kernel (`Syscall_Print`), el kernel activa su bit de modo, lee `SecretData` (HEX CAFE) y lo envía a la salida, y luego vuelve al modo usuario.

## Contenido del archivo `code.mas`

Comentarios clave (resumido):

- ORG 000 — dirección de inicio.
- `Jump StartUser` — salta al inicio del programa de usuario (bootstrap simple).
- Variables del kernel (ring 0): `MMSR` (registro de modo), `KernelBit`, `UserBit`, `SecretData` (`HEX CAFE`).
- `Syscall_Print` — rutina del kernel que:
  1. Activa el bit de modo en `MMSR` (cambia a modo kernel).
  2. Lee `SecretData` y lo envía a la salida (`Output`).
  3. Restaura el bit de modo a usuario y retorna mediante `JumpI`.
- `StartUser` — programa de usuario que carga un valor, guarda en `Temp`, invoca la syscall con `JnS Syscall_Print` y finaliza con `Halt`.
- Variables de usuario: `UserVal`, `Temp`.

El código completo está en `code.mas` (comentado en español dentro del propio archivo).

## Cómo ejecutar

Este proyecto es código de ejemplo para un simulador/ensamblador de arquitectura educativa SC-8 (o similar). Los pasos generales para ejecutar son:

1. Abrir un simulador/ensamblador compatible con el dialecto de `code.mas`. Si no tienes uno, puedes usar cualquier emulador de arquitectura educativa que acepte las instrucciones mostradas.
2. Ensamblar/compilar el archivo `code.mas` según las instrucciones del simulador.
3. Cargar el binario/imagen en el emulador y ejecutar desde la dirección 0x000.
4. Observar la salida: debería imprimirse el valor `0xCAFE` (contenido de `SecretData`) cuando se ejecuta la syscall desde el modo usuario.

Notas: si tu simulador ofrece instrucciones diferentes para syscalls o manejo de bits de modo, adapta las etiquetas y las directivas (`JumpI`, `JnS`, `Output`, etc.) al conjunto de instrucciones del simulador.

## Estructura del repositorio

- `code.mas` — código fuente ensamblador con la demostración.
- `.git/` — control de versiones (si corresponde).
- `README.md` — este documento.

## Explicación técnica (breve)

- El flujo principal muestra una llamada a la rutina del kernel desde el espacio de usuario (mediante `JnS Syscall_Print`).
- Antes de acceder a `SecretData`, la rutina del kernel establece el registro `MMSR` para indicar que se está en modo kernel (privilegiado).
- Tras leer y emitir el dato secreto, la rutina restaura `MMSR` al modo usuario y retorna, evitando que el código de usuario lea directamente `SecretData` sin pasar por la interfaz controlada.

Este ejemplo observa el principio de mínima exposición: solo el kernel (rutinamente controlado) puede leer datos sensibles.

## Licencia

MIT