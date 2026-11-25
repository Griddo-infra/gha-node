# 🟢 GHA Node #

Action que instala **la versión de Node acordada para Griddo**. Todos los repos
que la usan corren exactamente la misma versión, y esa versión se cambia en un
solo sitio: este repo.

> [!NOTE]
> **Versión congelada.** La actualización automática está desactivada y ningún repo usa todavía esta action: está pendiente de acordarse con el equipo de desarrollo. Lo que sigue describe cómo funcionará.

---

## 📦 Qué es ##

- La versión exacta está en el fichero `.node-version`.
- La versión mayor está fijada en `.node-major` (hoy, la 24).
- Dentro de esa mayor, la minor y el parche **se actualizan solos**: Renovate
  abre el PR cuando una versión lleva 7 días publicada y lo fusiona si pasan
  las pruebas.
- Cambiar de versión mayor es una decisión: se hace a mano, en un PR.

---

## 🛠️ Requisitos ##

- Un runner Linux, de GitHub o propio.
- En los runners propios de Griddo la versión ya viene en la imagen
  (`Griddo-infra/gha-griddo-runners`), así que no se descarga nada.

---

## 🚀 Uso ##

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

Usa siempre `@latest`: es la referencia que avanza con cada versión. Así un
repo no tiene que tocar nada para estar al día.

---

## 🔧 Configuración ##

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

Referencias disponibles:

| Referencia | Qué es |
| --- | --- |
| `latest` | La versión vigente. Es la que hay que usar |
| `v24` | La última de la mayor 24. Para quedarse en la 24 cuando `latest` pase a la siguiente |
| `v24.22.3` | Una versión concreta. No se mueve |

---

## 🚢 Despliegue ##

Una versión nueva llega a los repos en tres pasos:

1. Renovate actualiza `.node-version` en `main`.
2. La construcción semanal de `Griddo-infra/gha-griddo-runners` lee ese fichero
   y añade la versión a la imagen de los runners.
3. Cuando esa imagen se promueve a `estable`, el mismo proceso crea la etiqueta
   `vX.Y.Z` y mueve `v24` y `latest`.

`latest` solo avanza cuando la imagen ya lleva la versión: los jobs de nuestros
runners nunca la descargan.

---

## 🗂️ Estructura del repositorio ##

- `action.yml` → la action.
- `.node-version` → la versión exacta de Node.
- `.node-major` → la versión mayor fijada.
- `.github/workflows/prueba.yml` → pruebas en x64 y arm64.
- `renovate.json` → actualización automática de la versión.

---

## 🧰 Operación ##

- **Cambiar de versión mayor:** en un PR, cambia `.node-major` y `.node-version`
  a la nueva. Antes, comprueba que la imagen de los runners admite esa versión
  (`NODE_MINIMA` en su `Dockerfile`).
- **Mover las etiquetas a mano** (si la construcción de la imagen está parada).
  Solo pueden hacerlo los propietarios de la organización:

  ```bash
  git fetch origin main
  git tag v24.23.0 origin/main && git push origin v24.23.0
  git tag -f v24 origin/main && git push -f origin v24
  git tag -f latest origin/main && git push -f origin latest
  ```

- **Un repo necesita quedarse en una versión:** que use `@v24.22.3` en lugar de
  `@latest`, y que lo anote, porque dejará de actualizarse.

---

## 🔒 Seguridad ##

`latest` decide qué Node ejecutan todos los workflows que usan esta action. Por
eso:

- `main` solo admite cambios por PR, con las pruebas pasadas.
- Las etiquetas solo las pueden crear o mover los propietarios de la
  organización y la automatización.
- El repo es público para que puedan usarlo las tres organizaciones; no
  contiene nada más que un número de versión.

---

## 👤 Autor ##

- ✍️ Equipo de Infra Griddo (OJRR) - Con asistencia de Claude
- 📅 Última actualización: Noviembre 2026
