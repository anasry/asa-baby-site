# CLAUDE.md : standards de design front-end

Synthèse opérationnelle de trois référentiels open source, à appliquer à tout composant, page ou style de ce projet :

- **Taste Skill** (Leonxlnx, MIT) : design front-end sans tics d'IA. https://github.com/Leonxlnx/taste-skill
- **Emil Kowalski, skills** (MIT) : animation et finition d'interface. https://github.com/emilkowalski/skills
- **Impeccable** (Paul Bakaus, Apache-2.0) : typographie, hiérarchie, distillation. https://github.com/pbakaus/impeccable

Ce fichier est une reformulation condensée, pas une copie. En cas de conflit entre deux sources, la règle la plus stricte l'emporte. Un brief explicite de l'utilisateur (charte, polices, couleurs imposées) prime sur tout ce qui suit.

## 0. Avant de produire quoi que ce soit

1. Lire le brief et déclarer en une ligne : « Je lis ceci comme : <type de page> pour <public>, avec un langage <ambiance>. » Poser au plus **une** question, seulement si la lecture diverge vraiment.
2. Identifier l'objectif unique de la page (une action principale, quelques secondaires, le reste en retrait).
3. Réutiliser l'existant : logo, couleurs, polices, photos et voix de la marque sont du matériau de départ, pas des options. Ne jamais modifier en silence les URLs, libellés de navigation principaux, noms de champs de formulaire, logo ni textes légaux.
4. Si le projet repose sur un système existant (thème WordPress, Elementor, bibliothèque de composants), s'y conformer : un seul système par projet, ne pas réécrire à la main ce que le système fournit. Ne jamais livrer un composant à son état par défaut non personnalisé.
5. Vérifier qu'une dépendance existe avant de l'importer. Ne jamais supposer qu'une bibliothèque est installée.

## 1. Design sans tics d'IA (Taste)

### Typographie
- Éviter Inter, Arial et les polices système comme choix par défaut. Choisir une famille adaptée à la marque (ex. Geist, Outfit, Satoshi, Cabinet Grotesk), une seule famille principale, deux au maximum.
- Pas de serif « par défaut » sous prétexte de premium ou de créatif. Serif seulement si la marque l'impose ou si l'univers est vraiment éditorial. Bannies par défaut : Fraunces, Instrument Serif.
- Emphase dans un titre : italique ou gras de la **même** famille, jamais un mot en police différente.
- Italique en display : interligne 1.1 minimum et réserve basse pour ne pas rogner les descendantes (y g j p q).
- Pas de H1 démesurés : la hiérarchie vient du poids et de la couleur, pas seulement de la taille.
- Polices chargées en local ou via le mécanisme du framework, avec `font-display: swap`.

### Couleur
- Un seul accent, saturation modérée, utilisé à l'identique sur toute la page. Une seule palette de neutres (pas de mélange chaud/froid).
- Pas de noir pur (#000) ni de blanc pur (#fff) : neutres teintés.
- Pas de violet/bleu « lueur IA », pas de néon, pas de halos colorés, pas de texte en dégradé.
- Palette « beige + laiton + espresso » interdite par réflexe pour les marques premium ou artisanales, sauf si la marque la demande explicitement.
- Un thème par page (clair ou sombre), pas de section qui s'inverse en cours de route. Prévoir le mode sombre si la page est publique, et tester les deux.
- Contraste WCAG AA : 4,5:1 pour le texte courant et les placeholders, 3:1 pour le grand texte. Vérifier chaque bouton, champ de formulaire et focus.

### Mise en page
- Hero qui tient dans le premier écran : titre sur 2 lignes maximum, sous-titre de 20 mots maximum, CTA visible sans défilement. Quatre éléments de texte au plus (accroche, titre, sous-titre, CTA).
- Navigation sur une seule ligne en desktop, 80 px de haut au maximum.
- Hauteur plein écran : `min-height: 100dvh`, jamais `100vh`.
- Grille CSS plutôt que calculs de pourcentages en flexbox.
- Pas de rangée de trois cartes identiques. Varier les familles de sections (au moins 4 familles différentes pour 8 sections), pas plus de 2 sections consécutives en zigzag image/texte.
- Cartes uniquement quand l'élévation exprime une vraie hiérarchie. Sinon : espace, `border-top`, séparateurs.
- Un seul système de rayons de bordure sur toute la page.
- Pas de bloc « split header » (gros titre à gauche, petit paragraphe flottant à droite) : empiler titre puis texte, 65ch maximum.
- Mobile : toute mise en page asymétrique se replie en une colonne sous 768 px, déclaré explicitement.
- Pas de superposition de z-index arbitraires : une échelle documentée.

### Contenu et accessoires
- Pas de faux écrans bricolés avec des `div`, pas de SVG décoratifs dessinés à la main, pas de grain sur des conteneurs défilants. Vraies photos ou emplacements clairement marqués à remplir.
- Pas de tiret cadratin (—) ni de demi-cadratin (–) comme séparateur : virgule, point, deux-points ou trait d'union simple.
- Eyebrows (petits libellés en capitales au-dessus d'un titre), numéros de section (01/02/03), indicateurs « Scroll », pastilles de statut décoratives, bandeaux de décoration en bas de hero, légendes de crédit photo décoratives : **supprimés**.
- Un libellé unique par intention de CTA sur toute la page (pas « Nous contacter » + « Écrivons-nous » + « Prendre contact »). Libellé de bouton sur une seule ligne.
- Pas de noms génériques (« Jane Doe », « Acme »), pas de verbes creux (« révolutionner », « sublimer »), pas de chiffres inventés présentés comme réels.
- Relire chaque texte visible avant livraison : une phrase bancale ou « poétique pour rien » est remplacée par une phrase fonctionnelle.
- Icônes : une seule bibliothèque (Phosphor, Tabler, Radix, Hugeicons), épaisseur de trait uniforme. Pas d'emoji dans l'interface sauf demande explicite.

### États et formulaires
- Prévoir chargement (squelette à la forme du contenu), vide, erreur, désactivé, focus clavier visible.
- Libellé au-dessus du champ, message d'erreur dessous, jamais de placeholder en guise de libellé.
- Retour tactile : `:active` avec léger enfoncement (voir §2).

## 2. Animation (Emil Kowalski)

### Faut-il animer ?
| Fréquence de l'action | Décision |
|---|---|
| Des centaines de fois par jour, ou déclenchée au clavier | Aucune animation |
| Des dizaines de fois par jour (survols, listes) | Réduire fortement ou retirer |
| Occasionnelle (modales, tiroirs, toasts) | Animation standard |
| Rare (premier usage, célébration) | Peut être plus expressive |

Toute animation doit répondre à « qu'est-ce que ça communique ? » : continuité spatiale, état, explication, retour d'action, ou éviter un changement brutal. « C'est joli » ne suffit pas pour une interaction fréquente.

### Courbes
- **Entrées et sorties : ease-out.** Jamais `ease-in` sur une interface (démarrage mou, sensation de lenteur).
- Déplacement à l'écran : ease-in-out. Survol et couleur : `ease`. Mouvement continu (progression, défilement) : `linear`.
- Les courbes natives sont trop faibles, utiliser des courbes personnalisées :
  - `--ease-out: cubic-bezier(0.23, 1, 0.32, 1)`
  - `--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1)`
  - `--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1)`
- Pas d'effet rebond ou élastique en interface (daté). Un léger ressort (bounce 0.1 à 0.3) reste réservé au glisser-déposer et aux interactions ludiques.

### Durées
| Élément | Durée |
|---|---|
| Retour de pression d'un bouton | 100 à 160 ms |
| Infobulles, petits popovers | 125 à 200 ms |
| Menus déroulants, sélecteurs | 150 à 250 ms |
| Modales, tiroirs | 200 à 500 ms |
| Sortie | plus rapide que l'entrée |

**Règle : une animation d'interface reste sous 300 ms.** Les durées plus longues sont réservées à un unique moment mis en scène (explicatif ou marketing), pas à chaque section.

### Règles de fabrication
- Animer seulement `transform` et `opacity`. Jamais `top`, `left`, `width`, `height`, `margin`, `padding`. Blur (≤ 20 px) uniquement pour masquer un fondu imparfait.
- Jamais `transition: all`. Lister les propriétés.
- Jamais d'entrée depuis `scale(0)` : partir de `scale(0.95)` avec `opacity: 0`.
- Boutons et éléments pressables : `:active { transform: scale(0.97) }` (entre 0.95 et 0.98).
- Popovers : `transform-origin` sur le déclencheur. Les modales restent centrées.
- Transitions CSS plutôt que keyframes pour tout ce qui peut être déclenché rapidement (interruptibles). `@starting-style` pour les entrées quand c'est supporté.
- Stagger : 30 à 80 ms entre éléments, décoratif, ne bloque jamais l'interaction.
- Survols derrière `@media (hover: hover) and (pointer: fine)`.
- **`prefers-reduced-motion`** : respecter systématiquement. Garder les fondus qui aident à comprendre, supprimer les déplacements.
- Pas d'écouteur `scroll` brut ni de boucle `requestAnimationFrame` qui touche l'état : `IntersectionObserver`, CSS scroll-driven animations ou la bibliothèque d'animation du projet.
- Pas d'animation en boucle infinie sur du contenu informatif. Pas de marquee multiple. Pas de curseur personnalisé.
- Tester au ralenti (×2 à ×5) et relire le lendemain.

## 3. Typographie, hiérarchie, distillation (Impeccable)

### Typographie
- Corps de texte : 1rem / 16 px minimum.
- Longueur de ligne : 45 à 75 ch (65 à 75 ch pour la prose longue). Plus la ligne est longue, plus l'interlignage augmente.
- Display : 6rem au maximum. Crénage jamais en dessous de -0,04em (souvent -0,02 à -0,03em se lit mieux). Titres équilibrés (`text-wrap: balance`).
- Hiérarchie par **rôles** nommés selon la fonction (titre, corps, libellé, métadonnée), pas par valeur. Un minimum de rôles et de familles. Combiner taille, poids, espace et ton plutôt que de tout confier à la taille. Un rôle répété reste identique d'un écran à l'autre.
- Texte clair sur fond sombre : interligne un peu plus haut, espacement des lettres un peu plus large.
- Paragraphes : espacement **ou** retrait de première ligne, jamais les deux.
- Monospace réservé au code, aux données et aux mesures, pas comme décor « technique ».
- Chiffres tabulaires pour les données alignées. Polices de repli à métriques compatibles pour éviter les décalages.
- Vérifier : titres, corps, libellés et métadonnées se distinguent sans lire le texte. Zoom, taille de texte système et petits écrans restent utilisables.

### Hiérarchie et mise en page
- La hiérarchie suit la priorité du contenu, pas les réglages par défaut du framework. Test du plissement d'yeux : en floutant, on identifie l'élément principal, le secondaire, puis les groupes, dans cet ordre.
- La proximité organise d'abord, les conteneurs ensuite. Groupes serrés, séparations généreuses, plus d'espace au-dessus d'un titre qu'en dessous.
- Échelle d'espacement documentée sur base 4, `gap` pour le rythme entre frères, aucune valeur ponctuelle. Alterner intervalles serrés et généreux plutôt qu'une valeur unique partout.
- Élévation déclarée une seule fois, par bordure **ou** par ombre (pas de « ghost card »). Ombres avec décalage et flou doux, teintées. Rayons de cartes 12 à 16 px, pilules réservées aux petits contrôles.
- Pas de cartes imbriquées, pas de cartes identiques « icône + titre + texte » comme structure unique.
- Responsive structurel : réordonner, replier ou révéler selon l'importance, pas seulement rétrécir. Ordre du DOM et du focus cohérents avec l'ordre visuel. Tester étroit, large, zoomé, contenus longs, états vides.
- Sur surface colorée, teinter le texte secondaire à partir de la teinte, jamais en gris. Pas de bordure latérale colorée épaisse (> 1 px) sur cartes ou alertes.

### Distillation
- Retirer tout ce qui ne mérite pas sa place ; chaque élément doit se justifier, sinon il est retiré, masqué ou fusionné.
- Un objectif principal par page ; une action principale, peu d'actions secondaires.
- Visuel : 1 à 2 couleurs plus des neutres, une famille typographique, 3 à 4 tailles, 2 à 3 graisses, un seul système d'espacement ; retirer bordures, ombres et fonds décoratifs.
- Contenu : phrases courtes, voix active, langage simple, aucun jargon ni répétition.
- Interaction : moins de choix, bonnes valeurs par défaut, édition en ligne plutôt qu'en modale, une seule étape suivante évidente.
- Code : supprimer CSS mort, composants orphelins, variantes superflues.
- **Limites** : ne jamais supprimer une fonctionnalité nécessaire, ne jamais sacrifier l'accessibilité (libellés, ARIA), ne pas retirer une information utile à la décision, conserver assez de hiérarchie.
- Quand tout est vert, préférer une direction engagée à une neutralité tiède.

## 4. Contrôle avant livraison

Passer en revue sur le rendu réel (bureau et mobile en un seul lot, corriger en une fois, un seul passage de confirmation) :

- [ ] Lecture du brief déclarée, un seul accent, une seule palette, un seul thème, un seul système de rayons
- [ ] Contrastes AA vérifiés (texte, boutons, formulaires, focus)
- [ ] Aucun tic de §1 : eyebrow, numéro de section, tiret cadratin, dégradé de texte, trois cartes identiques, faux écran, indicateur « Scroll »
- [ ] Hero dans le premier écran, navigation sur une ligne, CTA sans doublon d'intention
- [ ] Toutes les animations justifiées, ≤ 300 ms, ease-out, `transform`/`opacity`, `prefers-reduced-motion` géré, pas de `transition: all`
- [ ] États vide, chargement, erreur et focus présents
- [ ] Mesure de texte 45 à 75 ch, corps ≥ 16 px, rôles typographiques cohérents
- [ ] Chaque texte visible relu, aucune phrase creuse ni chiffre inventé
- [ ] Mobile : replis explicites, cibles tactiles utilisables, pas de défilement horizontal

## 5. Notes de mise en œuvre pour ce dépôt

- Le dépôt est un site de micro-crèche : public parental, besoin de confiance et de clarté. Régler l'ambiance vers le calme, la lisibilité et l'accessibilité plutôt que vers l'expérimental : peu de mouvement, forte lisibilité, contraste solide.
- Aucun fichier front-end n'est encore versionné ici. Dès qu'il y en aura (thème, CSS personnalisé, composants), appliquer ce fichier avant de livrer, et lister dans la description de la PR les écarts assumés.
- Si des styles passent par Elementor ou un CSS additionnel, y déclarer les courbes, durées et échelles d'espacement comme variables CSS partagées (`--ease-out`, `--space-*`, etc.) plutôt que de répéter des valeurs en dur.
