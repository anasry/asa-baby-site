# CLAUDE.md : point d'entrée

Ce fichier ne contient **aucune règle propre**. La seule source de vérité du projet ASA BABY est [`AI-RULES.md`](AI-RULES.md), complétée par les agents spécialisés du dossier [`.ai-agents/`](.ai-agents/).

## Ordre de lecture obligatoire

1. [`AI-RULES.md`](AI-RULES.md) : identité de la micro-crèche (Poissy, premium), ton, règles d'intégrité, écosystème d'outils et auto-vérification en boucle fermée, règles permanentes de travail sur le site, socle technique.
2. Le fichier de l'agent adapté à la tâche :
   - design, CSS, intégration Elementor : [`.ai-agents/agent-web-designer.md`](.ai-agents/agent-web-designer.md) (design system en ligne : jetons `--asa-*`, typographies, rayons, ombres, mouvement) ;
   - parcours, friction mobile, conversion Kidola : [`.ai-agents/agent-audit-ux.md`](.ai-agents/agent-audit-ux.md) ;
   - juridique, PMI, RGPD, santé : [`.ai-agents/agent-juriste.md`](.ai-agents/agent-juriste.md) ;
   - tarifs, CMG, rentabilité : [`.ai-agents/agent-comptable.md`](.ai-agents/agent-comptable.md) ;
   - réseaux sociaux, Canva, Metricool : [`.ai-agents/agent-social-media.md`](.ai-agents/agent-social-media.md).

## Priorité en cas de doute

La demande explicite d'Alexandre dans la conversation en cours, puis `AI-RULES.md`, puis le fichier de l'agent. Si une règle semble manquer ou se contredire, on corrige `AI-RULES.md` ou l'agent concerné, jamais ce fichier : aucune règle ne doit être ajoutée ici.

L'ancien `CLAUDE.md` de standards génériques (branche `claude/design-standards`) est obsolète et remplacé par ce fichier depuis le 2026-10-20.
