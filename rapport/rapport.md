# TP R3.14 — Optimisation d'un site vitrine

Nom Prénom — BUT MMI Béziers

- Dépôt (fork) : https://github.com/VOTRE-COMPTE/R314-optimisation-site
- Site publié : https://VOTRE-COMPTE.github.io/R314-optimisation-site/

> À COMPLÉTER : tout ce qui est entre `[[ ]]` vient de VOS mesures (PageSpeed, médiane de 3 passes, site publié). Aucune valeur « après » ne doit être inventée.

## 1. Tableau comparatif

| Indicateur | Avant mobile | Avant ordinateur | Après mobile | Après ordinateur |
|---|---|---|---|---|
| Poids total | 17,6 Mo | 17,6 Mo | 527 Ko (~0,5 Mo) | 527 Ko (~0,5 Mo) |
| Requêtes | 22 | 22 | 15 | 15 |
| FCP | 2,4 s | 0,9 s | 2,4 s | 0,6 s |
| LCP | 67,1 s | 10,9 s | 2,5 s | 0,9 s |
| CLS | 0 | 0,021 | 0 | 0,021 |
| TBT | 160 ms | 220 ms | 0 ms | 0 ms |
| Score | 60 | 57 | 92 | 99 |

Limite méthodologique : le site n'a pas assez de trafic pour des données de terrain (CrUX) ; l'audit repose uniquement sur des données de laboratoire.

## 2. Problèmes identifiés

| N° | Fichier et ligne | Nature | Preuve | Coût | Correction |
|---|---|---|---|---|---|
| 1 | `img/img1.jpg` à `img6.jpg` | Images de 0,8 à 3,4 Mo (jusqu'à 3968 px) affichées à ~400 px | Onglet Réseau, PageSpeed « Améliorer l'affichage des images » | ~10 Mo | Miniatures 800 px (`img/thumbs/`), 1600 px sur les pages de détail |
| 2 | `img/hero-bg.jpg` (via `theme-legacy.css:11`) | 3,4 Mo pour une texture à 4 % d'opacité | Onglet Réseau | 3,4 Mo | Réduite à 800 px, qualité 45 (44 Ko) |
| 3 | `index.html:9-12` | Preload de `img6.jpg?v=2` : image téléchargée deux fois ; preload d'une police | Onglet Réseau (2 requêtes) | 3,4 Mo | Preloads supprimés |
| 4 | `img/favicon.png` | Icône 512 px de 787 Ko | Onglet Réseau | 787 Ko | 64 px, 3 Ko |
| 5 | `img/img*.jpg` dans `index.html` | `<img>` sans `width`/`height` ni `loading="lazy"` | Lighthouse (CLS) | CLS [[ ]] | Attributs `width`/`height`, `loading="lazy"` |
| 6 | `styles.css:1-2`, `extra.css:1`, `theme-legacy.css:3` | Chaîne d'`@import` (3 CSS en série + Google Fonts), Playfair Display inutilisée | PageSpeed « Requêtes de blocage du rendu » | 3 requêtes en cascade | Un seul CSS, Poppins par `<link>` + `preconnect`, `display=swap` |
| 7 | `styles.css`, `extra.css` | ~2 000 règles `.redundant-N` / `.extra-N` jamais utilisées | Coverage ; recherche dans les pages | ~120 Ko | Supprimées (92 Ko → 4,7 Ko) |
| 8 | `index.html:16` | `js/legacy/polyfills.js` inexistant | Réseau (404) | 1 aller-retour | Balise supprimée |
| 9 | `index.html:17-18`, `img*.html` | Scripts sans `defer` dans le `<head>` | PageSpeed « Requêtes de blocage » | [[ ]] ms | `defer` |
| 10 | `js/vendor/ui-kit.min.js:2` | Boucle de 620 ms + tableau de 240 000 objets inutilisé | Performances, Bottom-Up : `c` ui-kit.min.js:2:26 ; PageSpeed (tâche longue) | 467 ms de temps propre (Bottom-Up) | Supprimés |
| 11 | `js/vendor/ui-kit.min.js:2` | `MutationObserver` qui relit `getComputedStyle` de tout le DOM à chaque mutation | Performances, Bottom-Up : `_sync` ui-kit.min.js:2:707 + `getComputedStyle` | 61 ms au total sur l'enregistrement | Supprimé |
| 12 | `js/analytics.js` | Code obscurci : au-delà de 1024 px, après 6,5 s puis toutes les 2,6 s, bloque le thread de 46 à 430 ms ; n'envoie rien | Performances (enregistrement > 20 s, écran large) ; écart de TBT mobile/ordinateur | jusqu'à 430 ms | Supprimé |
| 13 | `js/app.js` `revealCards` | Reflow forcé (lecture/écriture entrelacées, boucle imbriquée) à chaque scroll | Performances, Bottom-Up : `revealCards` app.js:158 ; Forced reflow (Lighthouse) | 416 ms au total (10,2 %) | Lectures puis écritures, `requestAnimationFrame`, écouteur passif |
| 14 | `js/app.js` `buildKeywordIndex` | `innerHTML +=` dans une boucle (~320 réassignations) | Performances, Bottom-Up : `buildKeywordIndex` app.js:74 | 1 937 ms de temps propre (47,5 %), 2 313 ms au total | `DocumentFragment` |

Preuves à joindre (captures) : Réseau trié par taille, Lighthouse mobile et ordinateur, frise Performances + Bottom-Up, PageSpeed avant/après.

## 3. Gains mesurés

| Optimisation | Problèmes | Gain mesuré |
|---|---|---|
| Images | 1, 2, 3, 4, 5 | [[ ]] |
| CSS et polices | 6, 7 | [[ ]] |
| Scripts bloquants | 8, 9 | [[ ]] |
| Thread principal | 10 à 14 | TBT [[ ]] → [[ ]] |

## 4. Iso-rendu

Captures avant/après à 1280 px et 375 px : [[à joindre]]. Seul écart connu : sur les pages de détail, l'image s'affiche à 1600 px de large au lieu de sa taille d'origine (3968 px), aucune règle CSS ne la bornant.

## 5. Ce que je ferais avec un vrai serveur

GitHub Pages ne permet pas de régler les en-têtes HTTP. Avec un vrai serveur :

- `Cache-Control: public, max-age=31536000, immutable` sur les images et fichiers versionnés ;
- compression Brotli/gzip pour le texte, HTTP/2 ou HTTP/3 ;
- images en AVIF/WebP avec `srcset`/`sizes` générées côté serveur ou CDN.

## 6. Historique git

Un commit par optimisation (voir le fork).

## Conclusion

[[À rédiger : ce qui a eu le plus d'effet, limites des mesures (variabilité du score), pistes restantes.]]
