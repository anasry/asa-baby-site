# Plan CRO et acquisition ASA BABY

20 octobre 2026 · Alexandre Nasry

> Copie du document de travail [Plan CRO et acquisition ASA BABY](https://claude.ai/code/artifact/bd56353c-f35a-4407-8d6a-b1567f3c8f70). C'est un plan : rien n'est publié sur le site, GTM ou Metricool sans validation d'Alexandre. Agents mobilisés : audit UX, social media, juriste (voir `AI-RULES.md` et `.ai-agents/`).

## Synthèse

Aujourd'hui, ASA BABY ne mesure ni les clics vers Kidola, ni la barre d'urgence, ni la provenance Instagram : il faut 3 semaines de mesure avant tout test A/B. Ce plan n'est pas en production : rien ne sera posé sur le site, GTM ou Metricool sans validation d'Alexandre.

**Trois décisions à prendre**

1. Valider l'installation d'un conteneur GTM unique, piloté par le consentement CookieYes, et d'un petit script `dataLayer` (section Tracking). Le pixel Meta reste hors périmètre tant qu'il n'y a pas de publicité payante.
2. Valider la nomenclature UTM et le lien en bio à boutons (un bouton « post du moment » par publication), seule façon d'attribuer un parent à un post Instagram précis.
3. Valider les deux tests A/B : « reste à charge » contre « prix brut » sur le bloc coût de l'accueil, et Reel « Expertise PMI » contre Reel « Une journée chez nous ».

**Ce qui a été vérifié le 20 octobre 2026 (lecture seule)**

| Outil | État constaté | Source |
| --- | --- | --- |
| Site Kit by Google 1.189.0 | Extension active. Le module connecté (GA4 ou non) n'a pas pu être lu : à vérifier dans Site Kit > Réglages | état système WordPress |
| Google Tag Manager, pixel Meta | Aucune trace dans le contenu de l'accueil, aucun extrait Angie ne les charge | lecture de la page et des 19 extraits |
| CookieYes 3.5.7 | Extension active ; mode de consentement Google non vérifié | état système WordPress |
| Search Console (via Rank Math) | Connectée : sur 30 jours, « asa baby » = 9 clics pour 180 impressions, « crèche poissy » = 5 impressions, 0 clic | Rank Math Analytics |
| Traceur web Metricool | 0 visite sur 30 jours : non installé ou inactif | Metricool, marque 7274663 |
| Instagram @asa\_baby78 | 93 à 96 abonnés ; portée des posts du 6 au 17 octobre de 9 à 104 comptes ; 4 Reels : 37 à 115 comptes, durée de visionnage moyenne 3,0 à 5,7 s | Metricool |
| Barre d'urgence | Barre collante mobile (≤ 768 px), extrait « Encart pré-inscription Kidola » ; elle affiche « Dernières places » seulement si chaque jour ouvert a 2 places libres ou moins. Le 20 octobre, elle affiche « Places libres : du lun. au ven. » | code de l'extrait 4847 et page d'accueil |

**Ce qu'il faut mesurer avant de conclure quoi que ce soit** : sessions mobiles par semaine, taux de consentement au bandeau, nombre d'affichages de la barre, clics Kidola par bouton, et pré-inscriptions réellement reçues dans Kidola par semaine. Aucun taux de conversion n'est donné dans ce plan : il n'existe pas encore.

## 1. Architecture de tracking

Tout ce qui est mesurable se passe sur asa-baby.com, jusqu'au clic vers Kidola. Kidola est un site tiers : on n'y pose aucun tag, et la pré-inscription réelle se compte par déclaration et par export Kidola (1.6).

### 1.1 Chaîne technique

1. **Un seul extrait Angie « ASA BABY - dataLayer (mesure) »** pousse les événements dans `window.dataLayer` (code en 1.3). Il ne dépose aucun cookie et ne modifie aucun widget Elementor.
2. **Un conteneur GTM** chargé sur toutes les pages (par Site Kit, module Tag Manager, ou par le même extrait). Si Site Kit place déjà une balise GA4, la retirer de Site Kit pour éviter le double comptage.
3. **CookieYes en mode de consentement Google v2**, valeurs par défaut « refusé » (`analytics_storage`, `ad_storage`, `ad_user_data`, `ad_personalization`). Mode « basique » recommandé : aucune balise Google ne part avant le choix du parent (section 4).
4. **GA4** reçoit les événements seulement si `analytics_storage = granted`.
5. **Pixel Meta : non installé à ce stade.** Il ne sert qu'à la publicité payante. Si des campagnes Meta sont lancées, l'activer dans GTM sous `ad_storage = granted`, avec le seul événement `Lead` mappé sur `kidola_click`.
6. **Option sans cookie (à valider par le juriste)** : un compteur interne `asa/v1/mesure` qui additionne par jour les affichages et clics, sans identifiant ni adresse IP. Il corrige l'angle mort des parents qui refusent les cookies.

### 1.2 Plan de taggage : événements

| Événement GA4 | Déclencheur exact | Paramètres | Événement clé |
| --- | --- | --- | --- |
| `kidola_click` | clic sur un lien dont l'URL contient `kidola.app/registration/741` | `cta_id`, `cta_label`, `bar_state`, `ab_variant`, `ms_since_load` | oui |
| `asa_bar_show` | la barre `#asi-bar` reçoit la classe `is-on` pour la première fois dans la page | `bar_state`, `ms_since_load` | non |
| `asa_cta_view` | un bouton Kidola est visible à 50 % ou plus, première fois dans la page | `cta_id` | non |
| `asa_cta_click` | clic vers Tarifs, simulateur, téléphone, e-mail ou itinéraire | `cta_id`, `destination` (`tarifs`, `simulateur`, `tel`, `email`, `itineraire`) | `tel` et `itineraire` oui |
| `asa_section_view` | un conteneur principal de l'accueil est visible à 30 % ou plus | `section_id` | non |
| `asa_scroll` | 25, 50, 75 et 90 % de la page atteints | `percent` | non |
| `asa_page_exit` | la page passe en arrière-plan (`visibilitychange`), une fois par page | `max_scroll`, `last_section`, `kidola_clicked` (oui/non), `seconds_on_page` | non |
| `asa_simulateur_start` | premier clic dans `#asa-sim` (page Tarifs) | aucun | non |
| `asa_simulateur_result` | le bloc résultat `#asa-res` se remplit | `jours` (1 à 5) uniquement | non |

Aucun paramètre ne contient le revenu, la tranche CAF, la situation familiale, l'âge ou le prénom de l'enfant.

**Dimensions personnalisées GA4 (portée événement)** : `cta_id`, `cta_label`, `bar_state`, `ab_variant`, `section_id`, `destination`, `last_section`. Statistique personnalisée : `max_scroll`, `seconds_on_page`.

### 1.3 Identifiants des boutons (accueil, page 95)

| `cta_id` | Texte affiché le 20 octobre | Repère dans le DOM | Destination |
| --- | --- | --- | --- |
| `menu_preinscription` | Pré-inscription | lien dans l'en-tête (menu en double desktop / mobile) | Kidola |
| `hero_preinscription` | Faire ma pré-inscription | widget `54a0dee` | Kidola |
| `hero_tarifs` | Voir les tarifs | widget `715db13` | Tarifs |
| `encart_preinscription` | Commencer ma pré-inscription | `.asi` (shortcode `8f46ec5`, section parcours) | Kidola |
| `bloc_cout_simulateur` | Calculer pour ma situation | shortcode `8b30dfe` | simulateur |
| `bloc_cout_devis` | Obtenir mon devis | shortcode `8b30dfe` | Kidola |
| `barre_urgence` | Pré-inscrire mon enfant | `#asi-bar` (mobile seulement) | Kidola |
| `planning_jours` | Demander le … | `.asp-root` (page Tarifs) | Kidola |
| `simulateur_resultat` | bouton du résultat | `#asa-cta` (page Tarifs) | Kidola |
| `acces_tel` / `acces_itineraire` | Nous appeler / Itinéraire | widgets `d6103c8` / `b0e1605` | téléphone / Maps |

Valeurs de `bar_state` : `dernieres_places`, `places_libres`, `complet`, `texte_libre`, `absente`. Elles sont lues dans le texte réel de la barre, pour savoir si « Dernières places » fait cliquer plus que « Places libres ».

### 1.4 Code de l'extrait dataLayer (à publier seulement après validation)

```javascript
(function(){
  var KID = /kidola\.app\/registration\/741/, t0 = Date.now();
  var dl = window.dataLayer = window.dataLayer || [];
  var maxScroll = 0, lastSection = 'hero', kidola = false, sent = {};
  var SECTIONS = {'1dec47e':'hero','bb32b0d':'reperes','ed3073e':'avis','57b425c':'cout_equipe',
    'cad987c':'pedagogie','39cdd06':'vie_creche','3bdedf2':'lieux','bfb1b42':'bibliotheque',
    'c8d44fe':'parcours_inscription','06fc0dc':'cout_aides','381f74f':'entreprises','891d413':'acces'};
  function push(e, p){ p = p || {}; p.event = e; p.ms_since_load = Date.now() - t0;
    p.ab_variant = document.documentElement.getAttribute('data-asa-ab') || 'aucun'; dl.push(p); }
  function once(k){ if (sent[k]) return false; sent[k] = 1; return true; }
  function barState(){ var s = document.querySelector('#asi-bar .asi-bar-t span');
    var t = s ? s.textContent.trim() : '';
    return !t ? 'absente' : /^Derni/.test(t) ? 'dernieres_places' : /^Places/.test(t) ? 'places_libres'
      : /^Complet/.test(t) ? 'complet' : 'texte_libre'; }
  function ctaId(a){
    if (a.closest('#asi-bar')) return 'barre_urgence';
    if (a.closest('#asa-sim')) return 'simulateur_resultat';
    if (a.closest('.asi')) return 'encart_preinscription';
    if (a.closest('.asp-root')) return 'planning_jours';
    if (a.closest('[data-id="54a0dee"]')) return 'hero_preinscription';
    if (a.closest('[data-id="715db13"]')) return 'hero_tarifs';
    if (a.closest('[data-id="8b30dfe"]')) return /kidola/.test(a.href) ? 'bloc_cout_devis' : 'bloc_cout_simulateur';
    if (a.closest('[data-id="d6103c8"]')) return 'acces_tel';
    if (a.closest('[data-id="b0e1605"]')) return 'acces_itineraire';
    if (a.closest('header, .elementor-location-header, nav')) return 'menu_preinscription';
    return 'autre'; }
  function dest(h){ return KID.test(h) ? 'kidola' : /^tel:/.test(h) ? 'tel' : /^mailto:/.test(h) ? 'email'
    : /google\.[a-z.]+\/maps|maps\.app\.goo/.test(h) ? 'itineraire' : /simulateur/.test(h) ? 'simulateur'
    : /tarification/.test(h) ? 'tarifs' : null; }
  document.addEventListener('click', function(e){
    var a = e.target.closest && e.target.closest('a[href]'); if (!a) return;
    var d = dest(a.href); if (!d) return;
    var p = {cta_id: ctaId(a), cta_label: (a.textContent || '').trim().slice(0, 60), bar_state: barState()};
    if (d === 'kidola'){ kidola = true; push('kidola_click', p); } else { p.destination = d; push('asa_cta_click', p); }
  }, true);
  var bar = document.getElementById('asi-bar');
  if (bar && window.MutationObserver) new MutationObserver(function(){
    if (bar.classList.contains('is-on') && once('bar')) push('asa_bar_show', {bar_state: barState()});
  }).observe(bar, {attributes: true, attributeFilter: ['class']});
  if ('IntersectionObserver' in window){
    var io = new IntersectionObserver(function(es){ es.forEach(function(x){ if (!x.isIntersecting) return;
      var el = x.target, id = el.getAttribute('data-id');
      if (SECTIONS[id]){ lastSection = SECTIONS[id]; if (once('s' + id)) push('asa_section_view', {section_id: SECTIONS[id]}); }
      else { var c = ctaId(el); if (once('v' + c)) push('asa_cta_view', {cta_id: c}); } }); }, {threshold: [0.3, 0.5]});
    Object.keys(SECTIONS).forEach(function(id){ var s = document.querySelector('[data-id="' + id + '"]'); if (s) io.observe(s); });
    [].forEach.call(document.querySelectorAll('a[href*="kidola.app/registration/741"]'), function(a){ io.observe(a); });
  }
  addEventListener('scroll', function(){
    var h = document.documentElement, p = Math.round(100 * (scrollY + innerHeight) / h.scrollHeight);
    if (p > maxScroll) maxScroll = p;
    [25, 50, 75, 90].forEach(function(s){ if (p >= s && once('sc' + s)) push('asa_scroll', {percent: s}); });
  }, {passive: true});
  document.addEventListener('visibilitychange', function(){
    if (document.visibilityState === 'hidden' && once('exit')) push('asa_page_exit', {max_scroll: maxScroll,
      last_section: lastSection, kidola_clicked: kidola ? 'oui' : 'non', seconds_on_page: Math.round((Date.now() - t0) / 1000)});
  });
  var sim = document.getElementById('asa-sim');
  if (sim){ sim.addEventListener('click', function(){ if (once('sim')) push('asa_simulateur_start'); }, true);
    var res = document.getElementById('asa-res');
    if (res && window.MutationObserver) new MutationObserver(function(){
      var n = sim.querySelectorAll('[aria-pressed="true"]').length;
      if (res.textContent.trim() && once('res')) push('asa_simulateur_result', {jours: n || 'inconnu'});
    }).observe(res, {childList: true, subtree: true, characterData: true}); }
})();
```

Le sélecteur `[aria-pressed="true"]` du simulateur est une hypothèse à confirmer en prévisualisation GTM ; les identifiants de sections viennent de la structure Elementor lue le 20 octobre.

### 1.5 Configuration GTM

| Élément GTM | Réglage |
| --- | --- |
| Variables de couche de données | `cta_id`, `cta_label`, `bar_state`, `ab_variant`, `section_id`, `destination`, `percent`, `max_scroll`, `last_section`, `kidola_clicked`, `seconds_on_page`, `jours`, `ms_since_load` |
| Déclencheur | un déclencheur « Événement personnalisé » par nom d'événement de 1.2 (regex possible : `^(kidola_click\|asa_.*)$`) |
| Balise Google (GA4) | ID de mesure `G-…` (à récupérer), consentement requis : `analytics_storage` ; signaux Google désactivés ; personnalisation publicitaire désactivée |
| Balise Événement GA4 | une seule balise, nom = `{{Event}}`, paramètres = variables ci-dessus, déclenchée par la regex |
| Mesure améliorée GA4 | désactiver « Défilements » et « Clics sortants » (remplacés par `asa_scroll` et `kidola_click`, pour éviter les doublons) |
| Conservation des données GA4 | 14 mois (plafond CNIL recommandé : 25 mois) |
| Contrôle avant publication | Prévisualisation GTM + DebugView GA4 sur iPhone (390 px), avec et sans consentement |

### 1.6 Indicateurs exacts

| Indicateur | Formule | Lecture |
| --- | --- | --- |
| Taux de clic réel de la barre | pages vues avec `kidola_click` (`cta_id = barre_urgence`) ÷ pages vues avec `asa_bar_show`, ventilé par `bar_state` | répond à « « Dernières places » fait-il cliquer ? » |
| Abandon avant Kidola | 1 − (sessions avec `kidola_click` ÷ sessions mobiles) | la fuite globale du site |
| Où l'abandon se produit | répartition des `asa_page_exit` sans clic par `last_section` et `max_scroll` ; entonnoir GA4 ouvert : `session_start` → `asa_section_view(avis)` → `asa_section_view(cout_aides)` → `asa_cta_view` → `kidola_click` | la section où le parent part |
| Délai avant décision | médiane de `ms_since_load` sur `kidola_click`, par `cta_id` | parents décidés (clic rapide) contre parents qui lisent |
| Clic vers pré-inscription réelle | pré-inscriptions Kidola de la semaine ÷ sessions avec `kidola_click` de la semaine | la fuite dans Kidola, invisible autrement |
| Contact alternatif | `asa_cta_click` `tel` + `itineraire` | un appel n'est pas une fuite |

GA4 ne voit que les parents qui acceptent les cookies. Le taux de consentement (statistiques CookieYes) doit être relevé chaque semaine pour corriger les volumes.

### 1.7 Attribution : nomenclature UTM

Règles : minuscules, sans accent, mots séparés par des tirets, dates `aaaammjj`, jamais de donnée personnelle.

| Paramètre | Valeurs autorisées |
| --- | --- |
| `utm_source` | `instagram`, `facebook`, `google`, `linkedin`, `flyer`, `newsletter` |
| `utm_medium` | `bio`, `story`, `dm`, `post`, `organic-local` (fiche Google), `qr` |
| `utm_campaign` | typologie du post : `pedagogie`, `conseils`, `vie-creche`, `respiration`, `conversion` ; `bio` pour le lien permanent ; `gbp` pour la fiche Google |
| `utm_content` | `{aaaammjj}-{format}-{sujet}`, format = `post`, `carrousel`, `reel`, `story` ; ex. `20261021-post-winnicott-doudou` |

**Éducatif contre direct** : un groupe de canaux personnalisé GA4 « Social éducatif » = `utm_campaign` dans (`pedagogie`, `conseils`) ; « Social direct » = `conversion` ; « Social sans UTM » = référents `instagram.com`, `l.instagram.com`, `facebook.com`, `lm.facebook.com`, `m.facebook.com`.

**Pourquoi un lien par post** : une légende Instagram n'est pas cliquable, et les légendes actuelles renvoient à « lien en bio ou www.asa-baby.com ». Sans dispositif, tout arrive en « Instagram » sans savoir quel post a convaincu. Trois sources déterministes :

1. **Lien en bio à boutons** (SmartLinks Metricool ou les 5 liens natifs d'Instagram) : bouton fixe « Places et tarifs » (`utm_campaign=bio`), bouton « Pré-inscription », et bouton « Le post du moment » dont l'URL change à chaque publication (`utm_campaign` = typologie, `utm_content` = le post). Le lien actuel de la bio est à relever.
2. **Stories avec sticker lien** : `utm_medium=story`, même `utm_content` que le post relayé.
3. **Mot-clé en message privé** : chaque post de conversion se termine par un mot-clé (ex. « Écrivez PLACE ») ; la réponse automatique envoie un lien `utm_medium=dm`. Outil de réponse automatique par mot-clé : à vérifier (Meta Business Suite ou outil tiers).

Sur Facebook, le lien est cliquable dans le post : chaque post porte directement ses UTM. La fiche Google reçoit `?utm_source=google&utm_medium=organic-local&utm_campaign=gbp`.

### 1.8 Ce qui se passe après le clic Kidola

- **Décoration du lien** : ajouter `?utm_source=asa-baby&utm_medium=site&utm_content={cta_id}` aux liens Kidola, après avoir testé que le formulaire s'ouvre toujours. Utile seulement si Kidola expose un jour la source des dossiers.
- **Questions à poser à Kidola** (réponses inconnues) : la crèche voit-elle le référent ou les UTM d'un dossier ? Peut-on ajouter une question « Comment nous avez-vous connus ? » ? Peut-on définir une URL de retour après envoi (ex. `/merci-pre-inscription/`) ? Si oui, cette page déclenche `generate_lead`, la seule vraie conversion mesurable.
- **Registre de rapprochement** (sans retour possible) : une ligne par dossier Kidola reçu, avec date, numéro de dossier Kidola, source déclarée au premier appel (liste fermée calquée sur `utm_campaign`), formule demandée et étape (Cocon, Chenille, Chrysalide, Papillon). Aucun nom, aucune donnée sur l'enfant. Relevé chaque lundi, face aux `kidola_click` de la semaine.

## 2. Analyse comportementale du parcours

La fuite la plus probable se situe avant le site : depuis Instagram, un parent doit faire 2 gestes de plus (profil, lien en bio) puis attendre une page mobile qui mettait 7,2 s à s'afficher le 6 octobre. Ces pertes sont invisibles pour GA4, qui ne se charge qu'après. Chaque constat ci-dessous est observé et daté, ou marqué « hypothèse à vérifier » ; tous deviennent mesurables avec le plan de la section 1.

Priorité = impact × fréquence × facilité de correction (chacun de 1 à 3) : 18 et plus = haute, 9 à 17 = moyenne, moins de 9 = basse.

Parcours (étapes encadrées = mesurables par GA4 après consentement) : Post social → Profil, lien en bio (F1) → Chargement mobile (F2) → **Accueil, 1er écran (F3, F7)** → **Bloc coût (F4, F5)** → **Clic vers Kidola (F6, F8, F9)** → Formulaire Kidola (site tiers, aucun tag) → Dossier reçu (registre du lundi).

Seules les trois étapes encadrées se mesurent sur le site ; le reste se lit dans Metricool (avant) et dans le registre des dossiers Kidola (après).

| # | Étape | Constat | Effet sur le parent | Preuve | Priorité | Objectif fonctionnel | Indicateur |
| --- | --- | --- | --- | --- | --- | --- | --- |
| F1 | Post → site | Toutes les légendes du 6 au 17 octobre renvoient à « lien en bio ou www.asa-baby.com », non cliquable | 2 gestes et un changement d'écran avant la première page ; le parent intéressé par un post précis atterrit sur une page générique | légendes lues dans Metricool, 20 octobre | haute (3×3×3 = 27) | chaque post propose un chemin en un geste vers la page qui prolonge son sujet | clics sur le bouton « post du moment » ÷ portée du post |
| F2 | Chargement | LCP mobile 7,2 s, score 65 (PageSpeed, accueil, 6 octobre) ; mesure à refaire depuis les correctifs | le parent quitte avant l'affichage, souvent dans le navigateur intégré d'Instagram | rapport PageSpeed du 6 octobre | haute (3×3×2 = 18) | le premier écran s'affiche en moins de 2,5 s sur mobile | écart entre clics sur le lien en bio (Metricool) et sessions `utm_campaign=bio` (GA4) |
| F3 | Accueil, premier écran | Un même objectif, 5 libellés : « Pré-inscription », « Faire ma pré-inscription », « Commencer ma pré-inscription », « Obtenir mon devis », « Pré-inscrire mon enfant » | le parent se demande si ce sont des démarches différentes (charge de rappel) | page d'accueil lue le 20 octobre | haute (2×3×3 = 18) | un seul nom pour la pré-inscription, partout | part des `kidola_click` par `cta_id` |
| F4 | Bloc coût | « Obtenir mon devis » mène au formulaire de pré-inscription Kidola, alors que le devis est l'étape Chenille, après la pré-inscription | promesse non tenue au clic : le parent attend un prix, il trouve un dossier à remplir | lien du shortcode `8b30dfe` | haute (3×2×3 = 18) | le libellé annonce exactement ce que le parent trouve après le clic | `kidola_click` du bloc coût face aux pré-inscriptions déclarées « venues pour un devis » |
| F5 | Accueil, coût | Deux blocs pour la même question : « Combien coûte une place ? » (vers `#tarif-accueil`) puis « Combien ça coûte vraiment ? » (vers le simulateur de la page Tarifs) | deux chemins pour la question n° 2 du parent (loi de Hick) ; le chiffre utile arrive tard | ordre des sections, 20 octobre | moyenne (2×2×3 = 12) | une seule réponse au coût, un seul chemin vers l'estimation personnalisée | `asa_section_view(cout_aides)` puis `asa_cta_click(simulateur)` |
| F6 | Clic → Kidola | Le bouton du haut de page et la barre envoient directement vers Kidola, sans l'encart « Avant de commencer, munissez-vous de » (n° allocataire CAF, coordonnées, besoins) ; Kidola s'ouvre dans un nouvel onglet | le parent découvre dans Kidola qu'il lui manque son n° CAF et abandonne : une fuite que le site ne voit pas | code des extraits 4847 et de la page, 20 octobre ; abandon dans Kidola = hypothèse à vérifier | moyenne (3×2×2 = 12) | tout parent qui part vers Kidola sait avant de partir ce qu'il lui faudra | pré-inscriptions reçues ÷ sessions avec `kidola_click` (registre) |
| F7 | Parents « pas encore prêts » | Seuls choix : pré-inscription (engagement fort) ou téléphone ; la visite (Chrysalide) n'arrive qu'en 3e étape | le parent venu d'un post pédagogique n'a pas de petit pas possible et part | parcours en 4 étapes de l'accueil ; comportement = hypothèse à vérifier | moyenne (3×2×2 = 12) | un engagement léger existe pour qui n'est pas prêt à remplir un dossier | `asa_page_exit` sans clic après `asa_section_view(pedagogie)` |
| F8 | Barre d'urgence (loi de Fitts) | Elle est dans la zone du pouce et suit le parent après 500 px de défilement : bon placement. Son bouton mesure environ 40 px de haut (padding 11 px × 2 + une ligne de 14 px), sous les 44 px recommandés | quelques ratés de toucher d'une main, rien de bloquant | CSS de l'extrait 4847 (calcul, pas mesure) | moyenne (1×3×3 = 9) | la cible tactile de la barre atteint au moins 44 × 44 px | taux de clic de la barre (1.6) |
| F9 | Barre, « Dernières places » | Le texte bascule seul selon les places saisies par l'équipe dans l'écran « Places disponibles » ; s'il n'est pas à jour, la rareté affichée devient fausse | perte de confiance si le parent apprend que c'était faux ; risque juridique (section 4) | code des extraits 4653 et 4847 | basse (2×1×3 = 6) | les places affichées datent toujours de moins de 7 jours | date de mise à jour de l'écran « Places disponibles », relevée chaque lundi |

### Le discours social crée-t-il une friction avec le site ?

Oui, pour le trafic éducatif. Les posts et Reels partent d'une inquiétude du parent (doudou, adaptation, écrans), répondent longuement et citent leurs sources (CAF, Légifrance, service-public.fr). Le premier écran du site, lui, parle d'une démarche : une promesse et deux boutons, dont la pré-inscription. La preuve pédagogique (Montessori, Lóczy, Pikler) n'arrive qu'aux 12e et 13e blocs de l'accueil, et le vocabulaire des neurosciences affectives n'y apparaît pas.

Le parent change donc de registre au moment de changer d'écran : de « on vous explique » à « inscrivez-vous ». La barre « Dernières places », quand elle s'affiche, ajoute une pression juste après un contenu qui disait « pas de culpabilité ».

**Hypothèse testable H-friction** : les sessions « Social éducatif » ont un taux de `kidola_click` plus bas et un taux de `asa_section_view(pedagogie)` plus haut que les sessions « Social direct ». Si c'est vrai après 6 semaines de mesure, le bouton « post du moment » doit pointer vers le contenu qui prolonge le post (FAQ, fiche Parents, Notre modèle d'accueil), pas vers l'accueil. Ce point est une recommandation fonctionnelle : sa traduction visuelle revient à l'agent web designer.

## 3. Deux tests A/B

Avec le trafic actuel, un test classique sur la pré-inscription elle-même est impossible : on teste donc une micro-conversion fréquente, avec un effet attendu important, un horizon fixe et une règle de décision écrite avant le lancement. Les deux tests démarrent après 3 semaines de mesure (phase 0, section 5).

### Test 1 (site) : reste à charge contre prix brut

| Élément | Définition |
| --- | --- |
| Hypothèse | Si le bloc « Combien ça coûte vraiment ? » affiche d'abord un reste à charge estimé pour un cas de référence explicite, plus de parents mobiles passent du bloc à une action, parce que le prix brut (415 à 1 590 € par mois) fait partir le parent avant qu'il voie les aides. |
| Variante A (contrôle) | Bloc actuel, prix brut en premier |
| Variante B | Reste à charge estimé en premier (« à partir de X € par mois après CMG et crédit d'impôt, pour \[cas de référence\] »), prix brut en second. X est calculé par l'agent comptable avec les formules du simulateur : **à vérifier**. Mise en forme : agent web designer |
| Emplacement | shortcode `8b30dfe` (conteneur `06fc0dc`), accueil, page 95 |
| Répartition | 50/50 par session, tirée au premier affichage, attribut `data-asa-ab` = `cout-a` ou `cout-b` sur `<html>`, gardée en `sessionStorage` (effacée à la fermeture de l'onglet) ; les deux variantes sont dans le HTML et la variante inactive est masquée avant affichage, compatible avec WP Fastest Cache |
| Métrique principale | sessions avec un clic dans le bloc (`bloc_cout_simulateur` ou `bloc_cout_devis`) ÷ sessions avec `asa_section_view(cout_aides)`, mobile uniquement |
| Métriques secondaires | `kidola_click` par session ; `asa_simulateur_result` par session |
| Garde-fous | les appels (`asa_cta_click` `tel`) et les pré-inscriptions du registre ne baissent pas pendant le test |
| Horizon | 2 semaines minimum, 8 semaines maximum, pas d'arrêt anticipé sur un bon départ |
| Décision | Si l'échantillon prévu est atteint : test de deux proportions, B adoptée si p < 0,05 et garde-fous respectés. Si 8 semaines passent sans l'atteindre : B adoptée seulement si la probabilité que B soit meilleure dépasse 90 % (loi bêta, a priori neutre) et que les garde-fous tiennent ; sinon on garde A et on note « non concluant » |

**Taille d'échantillon** (risque d'erreur 5 %, puissance 80 %, test bilatéral) : sessions exposées nécessaires **par variante**, selon le taux de départ que la phase 0 mesurera.

| Taux mesuré en A | Effet recherché en B | Sessions par variante |
| --- | --- | --- |
| 5 % | 10 % | 435 |
| 10 % | 20 % | 199 |
| 15 % | 25 % | 250 |
| 20 % | 30 % | 294 |
| 20 % | 35 % | 138 |
| 30 % | 45 % | 163 |

**Durée** = 2 × sessions par variante ÷ sessions exposées par semaine. Exemple pour 10 % → 20 % (398 sessions au total) : 16 semaines à 25 sessions exposées par semaine, 8 semaines à 50, 4 semaines à 100. En dessous de 25 sessions exposées par semaine, ne pas lancer de test A/B : comparer 4 semaines avant et 4 semaines après, et le présenter comme une indication, pas une preuve.

L'alternative « libellé du bouton » (ex. « Faire ma pré-inscription » contre « Vérifier les places pour mon enfant ») est gardée pour plus tard : son effet attendu est plus faible, donc son échantillon plus grand. Corriger d'abord les libellés F3 et F4, qui ne demandent pas de test.

### Test 2 (social, Metricool) : Reel « Expertise PMI » contre Reel « Une journée chez nous »

| Élément | Définition |
| --- | --- |
| Hypothèse | Un Reel qui raconte un moment précis de la journée retient mieux qu'un Reel d'expertise réglementaire, mais le Reel d'expertise fait plus cliquer vers le site parce qu'il répond à « puis-je leur faire confiance ? ». Les deux effets sont mesurés séparément. |
| Variante A | « Expertise PMI » : ce que la PMI vérifie dans une micro-crèche, expliqué par la direction (4 sujets, un par semaine). Contenu réglementaire : **Brouillon**, envoyé pour revue dans Metricool, validé par la direction |
| Variante B | « Une journée chez nous » : accueil du matin, repas, sieste, retrouvailles (4 épisodes), filmés sans visage d'enfant (mains, dos, objets, portes peintes) |
| Plan | 4 paires, soit 8 Reels sur 4 semaines : chaque semaine un A et un B, sur deux créneaux fixes libres dans le planning (ex. mardi 20 h 30 et vendredi 10 h, à confronter aux meilleurs créneaux Metricool), en alternant chaque semaine quelle variante prend le premier |
| Contrôles | même durée (20 à 30 s, à 2 s près), texte d'accroche à l'écran dans les 2 premières secondes, légende de longueur proche, même appel à l'action, aucune promotion payante, aucun autre Reel ces semaines-là |
| Métrique principale | rétention moyenne (part moyenne du Reel regardée, Metricool `IGRE27`) ; si Metricool la laisse vide, durée de visionnage moyenne ÷ durée du Reel (`IGRE24` ÷ durée) |
| Métriques secondaires | taux de vues de plus de 3 s (`IGRE28`), partages, enregistrements, clics sur le bouton « post du moment » (`utm_content` du Reel), sessions GA4 de ce `utm_content` |
| Point de départ mesuré | 4 Reels du 13 au 19 octobre : portée de 37 à 115 comptes, durée de visionnage moyenne de 3,0 à 5,7 s. La rétention (`IGRE27`) est revenue vide : à vérifier dans Metricool avant le test |
| Échantillon | l'unité est le Reel : 4 par variante. Aucune significativité statistique n'est possible à ce volume ; on lit la constance de l'écart |
| Décision | B (ou A) gagne si elle fait mieux dans les 4 paires (test du signe, p = 0,0625 unilatéral), ou dans 3 paires sur 4 avec un écart moyen d'au moins 5 points de rétention. Sinon : égalité, on garde les deux formats dans la ligne éditoriale. Le format gagnant prend 2 Reels sur 3 le mois suivant |
| Quota Metricool | offre gratuite limitée à 20 posts par mois : novembre compte déjà 8 posts programmés ; 8 Reels de plus portent le mois à 16. Option : remplacer le post fixe « journée type » du 14 novembre par l'épisode B1 (à décider par Alexandre) |

Instagram propose aussi les « essais de Reels », montrés d'abord aux non-abonnés : disponibilité sur le compte à vérifier. Ils permettraient de tester une accroche sans toucher les abonnés, mais leur audience n'est pas forcément locale.

## 4. Cadre juridique : consentement, mineurs, droit à l'image

**BROUILLON – à valider par la direction (Smaïn Nasry / Agnès Perouchine).** Cette section n'est pas un avis juridique définitif ; pour le consentement aux traceurs, l'outil d'auto-évaluation de la CNIL doit être rempli, et un avocat ou la CNIL consultés en cas de doute.

GA4 et le pixel Meta ne peuvent fonctionner qu'après un « J'accepte » du parent : la CNIL n'exempte de consentement que la mesure d'audience produisant des statistiques anonymes pour le seul compte de l'éditeur, sans réutilisation par le fournisseur ([CNIL, page datée du 4 juillet 2025, consultée le 20 octobre 2026](https://www.cnil.fr/fr/cookies-solutions-pour-les-outils-de-mesure-daudience)). GA4 n'est pas cité sur cette page : le classer « soumis à consentement » par prudence.

| Risque | Niveau | Mesure correctrice |
| --- | --- | --- |
| Balise GA4 ou Meta déclenchée avant le choix du parent | élevé | CookieYes en mode de consentement Google v2, valeurs par défaut « refusé », mode basique (aucun signal envoyé avant consentement). Le mode avancé, qui envoie des signaux sans cookie avant le choix, n'est pas traité par la page CNIL consultée : ne pas l'utiliser |
| Bandeau où refuser est plus difficile qu'accepter | élevé | bouton « Tout refuser » au même niveau et de même forme qu'« Tout accepter », dès le premier écran ([recommandation CNIL consolidée](https://www.cnil.fr/sites/default/files/2026-01/recommandation_cookies_consolidee.pdf), à relire en détail) ; retrait du consentement possible à tout moment depuis un lien « Gérer les cookies » du pied de page |
| Bandeau partiellement en anglais (« Privacy Overview » relevé le 6 octobre) | modéré | traduire tout le bandeau ; lier la politique de confidentialité |
| Carte Google Maps en iframe déposant des cookies avant consentement (audit du 6 octobre) | modéré | bloquer l'iframe tant que la catégorie correspondante n'est pas acceptée (fonction de blocage de CookieYes) ou carte chargée au clic |
| Durées | faible | traceurs : 13 mois maximum ; conservation des données : 25 mois maximum (CNIL) ; GA4 réglé sur 14 mois |
| Transfert hors UE (Google LLC, Meta) | modéré | mentionner le transfert dans la politique ; base : cadre de protection des données UE – États-Unis de juillet 2023, **à vérifier** (statut à la date de publication) |
| Compteur interne sans cookie (option 1.1 étape 6) | faible | remplir l'outil d'auto-évaluation CNIL ; ne stocker ni IP, ni identifiant, ni empreinte du navigateur ; agrégats par jour seulement |
| Variante A/B gardée en `sessionStorage` | modéré | statut « exempt » ou « soumis à consentement » **à vérifier** avec l'outil CNIL. Option prudente : n'appliquer le tirage qu'après consentement, au prix d'un échantillon plus petit |
| Données d'enfants ou de revenus dans les mesures | élevé | interdit par conception (1.2) : ni revenu, ni tranche CAF, ni âge, ni prénom, ni date de naissance dans un événement, une URL ou un UTM. Aucun tag sur Kidola, qui traite des données de revenus et de famille |
| Registre de rapprochement des pré-inscriptions | faible | numéro de dossier Kidola, date, source déclarée et étape seulement ; l'inscrire au registre des traitements ; durée de conservation alignée sur celle des pré-inscriptions non retenues (1 an proposé le 7 octobre, non validé) |
| « Dernières places » faux ou périmé | modéré | une rareté inexacte peut constituer une pratique commerciale trompeuse (Code de la consommation, articles L121-2 et suivants, **à vérifier** sur Légifrance) ; garder le calcul automatique, mettre à jour les places chaque semaine, conserver la date de mise à jour |
| Reste à charge affiché (test 1, variante B) | modéré | présenter le montant comme une estimation, avec son cas de référence et la mention « montant réel selon votre situation CAF » ; montant validé par l'agent comptable ; obligations d'information sur les prix **à vérifier** |
| Reels du test 2 | élevé | aucun visage d'enfant sans autorisation écrite des titulaires de l'autorité parentale, par support et révocable ; le floutage n'en dispense pas ; autorisation écrite de chaque professionnelle filmée ; loi de février 2024 sur le droit à l'image des enfants **à vérifier** |
| Reels « Expertise PMI » | élevé | statut Brouillon, envoyés pour revue dans Metricool, jamais programmés directement ; aucun ratio d'encadrement, numéro d'article ou seuil sans vérification sur Légifrance le jour même ; numéro et date de l'autorisation PMI toujours manquants |

### Brouillon : texte du bandeau cookies

> **BROUILLON – à valider par la direction.** Nous utilisons des cookies pour mesurer la fréquentation du site et améliorer vos démarches. Ils ne sont déposés qu'avec votre accord. Vous pouvez accepter, refuser ou choisir, et changer d'avis à tout moment avec le lien « Gérer les cookies » en bas de chaque page. \[Politique de confidentialité\]
>
> Boutons : Tout refuser · Personnaliser · Tout accepter

### Brouillon : ajout à la politique de confidentialité

> **BROUILLON – à valider par la direction.** *Mesure d'audience.* Avec votre accord, nous utilisons Google Analytics (Google Ireland Limited et Google LLC), chargé par Google Tag Manager, pour savoir quelles pages sont consultées et quels boutons sont utilisés, notamment le lien vers notre formulaire de pré-inscription Kidola. Ces mesures ne contiennent aucune information sur votre enfant ni sur vos revenus. Les cookies concernés ont une durée de vie de 13 mois au plus ; les statistiques sont conservées 14 mois. Des données peuvent être transférées aux États-Unis dans le cadre \[base légale à vérifier\]. Vous pouvez retirer votre accord à tout moment depuis « Gérer les cookies ». Le formulaire de pré-inscription est hébergé par Kidola, qui dispose de sa propre politique de confidentialité.

## 5. Calendrier, validations et points à vérifier

Le premier résultat exploitable arrive début décembre (test social) ; le test site ne démarre que fin novembre, après 3 semaines de mesure de référence, et se décide au plus tard le 20 janvier 2027. Les dates supposent une validation d'Alexandre cette semaine.

| Période | Étape |
| --- | --- |
| 21 – 27 oct. 2026 | Validation du plan |
| 28 oct. – 3 nov. | Installation et contrôle |
| 4 – 24 nov. | Mesure de référence |
| 10 nov. – 4 déc. | Test 2 : 8 Reels |
| 25 nov. | Calibrage du test site |
| 7 déc. | Décision test social |
| 25 nov. – 20 janv. 2027 | Test 1 : site (2 à 8 semaines) |
| 20 janv. 2027 | Décision test site |

Validation (21 – 27 oct.) : plan validé, brouillons juridiques relus, questions envoyées à Kidola, lien de la bio relevé. Installation (28 oct. – 3 nov.) : GTM, extrait dataLayer et CookieYes en prévisualisation, puis publication après accord. Calibrage (25 nov.) : selon le trafic mesuré, test A/B ou comparaison avant / après. Les semaines de fêtes (fermeture de la crèche : à vérifier) sont exclues de l'analyse du test site.

### Ce qu'Alexandre doit valider

- [ ] Le plan de taggage (1.2 à 1.5) et la publication de l'extrait dataLayer après contrôle
- [ ] Le pixel Meta laissé de côté tant qu'il n'y a pas de publicité payante
- [ ] La nomenclature UTM (1.7) et le lien en bio à boutons
- [ ] Le test 1 et son montant de reste à charge (après calcul de l'agent comptable)
- [ ] Le test 2, les 8 Reels et le remplacement éventuel du post du 14 novembre
- [ ] Les deux brouillons juridiques de la section 4 (par la direction)

### Points à vérifier

1. Site Kit : GA4 est-il connecté, et avec quel identifiant `G-…` ? (Site Kit > Réglages, ou rechercher `gtag/js?id=G-` dans le code source de l'accueil)
2. CookieYes : le mode de consentement Google v2 est-il activé, et le bouton « Tout refuser » est-il au premier niveau ?
3. Lien actuellement placé dans la bio Instagram, et outil possible pour la réponse automatique par mot-clé
4. Kidola : source des dossiers, question « Comment nous avez-vous connus ? », URL de retour après envoi, export des dossiers datés
5. Metricool : pourquoi la rétention des Reels (`IGRE27`) et les clics sur le lien du profil reviennent vides
6. Nouvelle mesure PageSpeed mobile de l'accueil (référence : 65/100, LCP 7,2 s le 6 octobre)
7. Sélecteurs du simulateur et des sections, à confirmer en prévisualisation GTM
8. Textes de loi cités en section 4 (Code de la consommation, loi de 2024 sur l'image des enfants, transferts UE – États-Unis)

### Sources

- Extraits WordPress lus le 20 octobre 2026 : « ASA BABY - Places disponibles » (4653), « Encart pré-inscription Kidola » (4847), « ancre Places disponibles » (4664) ; structure Elementor de l'accueil (page 95) ; état système WordPress
- Metricool, marque ASA BABY : statistiques Instagram du 20 septembre au 20 octobre 2026 et posts programmés jusqu'au 31 décembre
- Rank Math Analytics (Search Console), 30 derniers jours au 20 octobre 2026
- Audit du site du 6 octobre 2026 (dossier partagé `audits/`)
- [CNIL, « Cookies : solutions pour les outils de mesure d'audience »](https://www.cnil.fr/fr/cookies-solutions-pour-les-outils-de-mesure-daudience), consultée le 20 octobre 2026
- [CNIL, recommandation « cookies et autres traceurs » consolidée](https://www.cnil.fr/sites/default/files/2026-01/recommandation_cookies_consolidee.pdf), repérée le 20 octobre 2026, à relire en détail
