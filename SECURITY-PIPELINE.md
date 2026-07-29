# Pipeline DevSecOps de la célula Clinic

El workflow [`Clinic DevSecOps CI`](.github/workflows/devsecops-ci.yml) se ejecuta
automáticamente con cada `push` y también en cada `pull_request`.

## Etapas

1. Clonación del repositorio.
2. SAST con Semgrep. El gate falla ante severidad `WARNING` o `ERROR`, equivalentes
   al umbral académico MEDIUM/HIGH/CRITICAL.
3. SCA con Trivy en modo filesystem, herramienta equivalente a Dependency-Check.
   El gate falla ante `MEDIUM`, `HIGH` o `CRITICAL`.
4. Construcción de la imagen Docker.
5. Image Security con Trivy. El gate falla ante `MEDIUM`, `HIGH` o `CRITICAL`.

Los tres reportes se publican como artefactos de GitHub Actions y se conservan
durante 30 días. Los scanners se ejecutan con `continue-on-error` para que una
misma corrida produzca todas las evidencias; el paso final agrega los resultados
y deja el workflow completo en estado fallido si cualquier gate fue bloqueado.

## Fallos controlados para las evidencias

Desde **Actions → Clinic DevSecOps CI → Run workflow**, seleccione uno de estos
valores en `force_control`:

- `sast`: habilita temporalmente un archivo Java con inyección de comandos.
- `sca`: analiza un POM aislado con `commons-collections:3.2.1`; Trivy detecta
  CVE-2015-7501 (CRITICAL) y CVE-2015-6420 (HIGH).
- `image`: construye la aplicación sobre una imagen base antigua.

También se puede forzar el control desde un commit incluyendo uno de los textos
`[force-sast-fail]`, `[force-sca-fail]` o `[force-image-fail]` en su mensaje.

Los fixtures solo existen con fines académicos y no se integran en la aplicación
ni en la imagen normal.

Trivy usa sus bases de vulnerabilidades y Java sin requerir secretos de terceros.
