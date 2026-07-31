# grain// — physical film grain simulator

Webapp single-file (zero build step) per simulazione di grana pellicola con controllo di:

- **distribuzione non uniforme** — intensità della grana pesata per luminanza del pixel (parabola, zero su nero/bianco puro, picco spostabile verso le ombre via `shadow bias`)
- **grana per canale RGB indipendente** — tre noise field separati (R/G/B), sliderabile tra grana mono e completamente indipendente per canale (`channel independence`)
- **clumping** — mappa di densità a bassa risoluzione (upscale bilineare) che modula localmente l'intensità del noise fine, producendo aggregazione in cluster invece di rumore isotropo

## Algoritmo

```
noise field (Box-Muller, per canale)
        │
   box blur × 3 pass (approssima gaussiana) ── controlla dimensione grano
        │
   × clump density mask (coarse noise upscalato bilineare)
        │
   × luminance weight = 4·lᵖ·(1-lᵖ)   dove p = shadow bias
        │
   + immagine originale, per canale
```

Tutto il processing è in `Float32Array` su CPU via Canvas 2D — nessuna dipendenza esterna, nessun upload server-side. Limite pratico ~2200px sul lato lungo per restare fluido in JS puro (vedi Limiti).

## File

```
index.html   — tutto: markup, stile, algoritmo, UI
README.md
```

## Uso locale

```bash
git clone <repo>
cd film-grain-studio
python3 -m http.server 8000
# apri http://localhost:8000
```

Nessuna build, nessun `npm install`. Funziona anche aprendo `index.html` direttamente da filesystem (`file://`) nella maggior parte dei browser.

## Deploy su GitHub Pages

```bash
git init
git add .
git commit -m "init"
git remote add origin <url-repo>
git push -u origin main
```

Poi: Settings → Pages → Deploy from branch → `main` / root. Live in ~1 minuto.

## Limiti noti / possibili estensioni

- **Performance**: box blur separabile è O(n), ma tre pass × tre canali su immagini grandi in JS puro ha un tetto. Per immagini >4K o video: portare il kernel su WebGL2 (fragment shader) o `OffscreenCanvas` + Worker.
- **Blur radius vs vera size grano**: qui il blur controlla la scala spaziale del noise, non la forma esatta dei granuli (Dehancer modella particelle 3D con rotazione rispetto al piano immagine — molto più costoso).
- **Clumping**: attualmente una singola mappa coarse; un modello più fedele userebbe multi-ottava (2-3 scale di clump sovrapposte) o vero Poisson-disk sampling per i centri dei grani.
- **Halation**: non implementato — è un effetto separato (glow rosso/arancio sui bright highlights dovuto a riflessione nel film base), meccanicamente indipendente dalla grana.
