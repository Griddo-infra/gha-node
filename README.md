<h1 align="center">
  <img src=".github/banner.png" alt="gha-node · Infraestructura de Griddo" width="100%">
</h1>

<p align="center">
  <a href="https://github.com/Griddo-infra/gha-node/actions/workflows/prueba.yml"><img src="https://github.com/Griddo-infra/gha-node/actions/workflows/prueba.yml/badge.svg" alt="Estado de las pruebas de la action"></a>
  <img src="https://img.shields.io/badge/tipo-action-5057ff?labelColor=001b3c" alt="Tipo: action">
  <img src="https://img.shields.io/badge/entorno-todos-199ab3?labelColor=001b3c" alt="Entorno: todos">
  <img src="https://img.shields.io/badge/equipo-infra-5057ff?labelColor=001b3c" alt="Equipo: infra">
</p>

Action que instala **la versión de Node acordada para Griddo**.

> [!NOTE]
> **Versión congelada y sin adoptar.** La actualización automática está
> desactivada y ningún repo usa todavía esta action: su adopción está pendiente
> de acordarse con el equipo de desarrollo. Este documento describe cómo está
> hecha y cómo funcionará cuando se active.

---

## 🎯 Qué es y para qué sirve ##

Sirve para que todos los repos que la usen corran exactamente la misma versión
de Node, y para que esa versión se cambie en un solo sitio: este repo.

- La versión exacta está en el fichero `.node-version` (hoy, la 24.22.3).
- La versión mayor está fijada en `.node-major` (hoy, la 24).
- Cambiar de versión mayor es una decisión: se hace a mano, en un PR.
- Dentro de esa mayor, la minor y el parche están pensados para actualizarse
  solos con Renovate. Hoy esa actualización está desactivada (ver «Despliegue»).

---

## 📋 Requisitos ##

- Un runner Linux, de GitHub o propio. Las pruebas cubren x64 y arm64.
- En los runners propios de Griddo la versión ya viene en la imagen
  (`Griddo-infra/gha-griddo-runners`), así que no se descarga nada.

---

## 🚀 Uso ##

> [!IMPORTANT]
> No la añadas todavía a ningún workflow: la adopción se acordará antes con el
> equipo de desarrollo. Lo que sigue es cómo se usará.

```yaml
steps:
  - uses: actions/checkout@v7
  - uses: Griddo-infra/gha-node@latest
  - run: npm ci
```

Con caché de dependencias:

```yaml
  - uses: Griddo-infra/gha-node@latest
    with:
      cache: npm
```

La referencia prevista es `@latest`: es la que avanza con cada versión, así un
repo no tiene que tocar nada para estar al día.

### Referencias disponibles ###

| Referencia | Qué es |
| --- | --- |
| `latest` | La versión vigente. Es la prevista para los workflows |
| `v24` | La última de la mayor 24. Para quedarse en la 24 cuando `latest` pase a la siguiente |
| `v24.22.3` | Una versión concreta. No se mueve |

---

## ⚙️ Configuración: variables y secretos ##

Este repo no tiene variables ni secretos. La action se configura con sus
entradas:

| Entrada | Para qué | Por defecto |
| --- | --- | --- |
| `cache` | Gestor cuya caché se usa: `npm`, `yarn` o `pnpm` | sin caché |
| `cache-dependency-path` | Ruta del fichero de dependencias si no está en la raíz | – |
| `registry-url` | Registro de paquetes para autenticarse | – |
| `scope` | Ámbito del registro (por ejemplo, `@griddo`) | – |

| Salida | Qué da |
| --- | --- |
| `node-version` | Versión instalada |
| `cache-hit` | Si se ha recuperado la caché |

Por dentro llama a `actions/setup-node` con la versión de `.node-version` y le
pasa las entradas tal cual.

---

## 🚢 Despliegue ##

Cuando la actualización automática esté activa, una versión nueva llegará a los
repos en tres pasos:

1. Renovate actualiza `.node-version` en `main`: abre el PR cuando una versión
   de la mayor fijada lleva 7 días publicada y lo fusiona si pasan las pruebas.
2. La construcción semanal de `Griddo-infra/gha-griddo-runners` lee ese fichero
   y añade la versión a la imagen de los runners.
3. Cuando esa imagen se promueve a `estable`, el mismo proceso crea la etiqueta
   `vX.Y.Z` y mueve `v<mayor>` y `latest`.

`latest` solo avanza cuando la imagen ya lleva la versión: los jobs de nuestros
runners nunca la descargan.

> [!NOTE]
> Hoy el paso 1 está parado: `renovate.json` desactiva el gestor de
> `.node-version`. Las etiquetas `latest`, `v24` y `v24.22.3` apuntan a la
> versión 24.22.3.

---

## 🗂️ Estructura del repositorio ##

- `action.yml` → la action.
- `.node-version` → la versión exacta de Node.
- `.node-major` → la versión mayor fijada.
- `.github/workflows/prueba.yml` → pruebas en x64 y arm64.
- `.github/CODEOWNERS` → quién revisa los cambios.
- `.github/banner.png` → cabecera de este documento.
- `renovate.json` → alta en Renovate, con la configuración común y la regla que congela la versión.

---

## 🛠️ Operación y resolución de problemas ##

### Reactivar la actualización automática ###

En un PR, sustituye en `renovate.json` la regla que desactiva el gestor `nodenv`
por la de fusión automática, que está en el historial de ese fichero. Solo
después de acordarlo con el equipo de desarrollo.

### Cambiar de versión mayor ###

En un PR, cambia `.node-major` y `.node-version` a la nueva. Antes, comprueba
que la imagen de los runners admite esa versión (`NODE_MINIMA` en su
`Dockerfile`).

### Mover las etiquetas a mano ###

Hace falta si la construcción de la imagen está parada. Solo pueden hacerlo los
propietarios de la organización:

```bash
git fetch origin main
git tag v24.23.0 origin/main && git push origin v24.23.0
git tag -f v24 origin/main && git push -f origin v24
git tag -f latest origin/main && git push -f origin latest
```

### Un repo necesita quedarse en una versión ###

Que use `@v24.22.3` en lugar de `@latest`, y que lo anote, porque dejará de
actualizarse.

### Las pruebas fallan en un PR ###

`prueba.yml` comprueba, en x64 y en arm64, que `.node-version` es una versión
completa de la mayor de `.node-major` y que la action instala justo esa. Si
falla, revisa que los dos ficheros coincidan.

---

## 🔒 Consideraciones de seguridad ##

`latest` decide qué Node ejecutan todos los workflows que usen esta action. Por
eso:

- `main` solo admite cambios por PR, con una aprobación del equipo de
  infraestructura (`CODEOWNERS`) y la comprobación `prueba` pasada. No se puede
  borrar ni reescribir.
- Las etiquetas solo las pueden crear, mover o borrar los propietarios de la
  organización y la automatización.
- El repo es público para que puedan usarlo las tres organizaciones; no
  contiene nada más que un número de versión.
- El análisis de secretos y la protección al hacer push están activos.

> [!WARNING]
> `latest` es una etiqueta que se mueve: quien la use no fija la action por
> SHA. Se acepta porque es propia y este repo está protegido: no relajes sus
> reglas.

---

## 📬 Contacto y responsables ##

- **Responsable:** Equipo de Infra de Griddo.
- **Versión de Node y adopción:** se acuerdan con el equipo de desarrollo.
- **Dudas o fallos:** abre una issue en este repo.

---

## 📝 Historial de cambios ##

Este repo no lleva un `CHANGELOG.md`: cada cambio entra por un PR con el título
en el formato `tipo(ámbito): descripción` y se fusiona como un solo commit.

- [PR fusionados](https://github.com/Griddo-infra/gha-node/pulls?q=is%3Apr+is%3Amerged)
- [Commits de `main`](https://github.com/Griddo-infra/gha-node/commits/main)
- [Etiquetas](https://github.com/Griddo-infra/gha-node/tags), una por versión de Node publicada.

---

## 👤 Autor ##

- ✍️ Equipo de Infra Griddo (OJRR) - Con asistencia de Claude
- 📅 Última actualización: Noviembre 2026
