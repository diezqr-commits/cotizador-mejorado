# Cotizador Diez — Instrucciones de proyecto

**Repositorio:** https://github.com/diezqr-commits/cotizador-mejorado  
**Producción:** https://cotizador-mejorado.pages.dev  
**Directorio:** /Users/sebastian/Documents/pryecto

## Regla obligatoria

Después de CADA cambio a cualquier archivo de este proyecto, siempre ejecutar:

```bash
cd /Users/sebastian/Documents/pryecto
git add -A
git commit -m "descripción del cambio"
git push
```

Cloudflare Pages detecta el push automáticamente y redesplega en ~1 minuto. No se necesita hacer nada más en Cloudflare.

## Archivos principales

- `cotizador-mejorado.html` — app principal (serigrafía, bordado, promocionales, DTF UV)
- `index.html` — redirección para Cloudflare Pages
- `logo-diez.png`, `icono whats app.png`, `icono telefono fijo.png` — assets del cotizador

## Git config

- Usuario: diezqr-commits
- Email: diez.qr@gmail.com
- Remote: https://github.com/diezqr-commits/cotizador-mejorado.git (autenticado via token en la URL)
