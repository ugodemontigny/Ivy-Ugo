name: ivy-ugo
description: "Gouvernance des citations sortantes pour le référencement IA (GEO/AEO) du contenu IA avec Ugo. À invoquer chaque fois qu'un article de blogue, une page web ou un contenu long est rédigé ou audité pour le site IA avec Ugo : impose des citations d'entités nommées (pas de simples hyperliens), un bloc Sources, et le balisage JSON-LD mentions+citation. Adapté du skill Ivy (sprint-blueprint-template, MIT) avec les autorités d'IA avec Ugo."
---

# Ivy-Ugo — Citations d'entités pour le référencement IA

Adapté du skill « Ivy » (7-Day Leads Sprint Blueprint, licence MIT). Même méthode, mais les autorités citées sont celles d'IA avec Ugo — jamais celles du template d'origine.

## Pourquoi

Recherche « Generative Engine Optimization » (Princeton/Georgia Tech, arXiv:2311.09735) : citer des sources nommées avec attribution augmente la visibilité dans les moteurs de réponse IA (ChatGPT, Perplexity, Gemini, Claude, AI Overviews) de l'ordre de +30-40 % vs des liens nus. Les pages qui citent se font citer; les entités citées gagnent en autorité. On fait composer ce signal pour IA avec Ugo et pour les sources québécoises fiables.

## La distinction critique

- ❌ Citation-lien : « L'IA transforme la recherche ([source](url)). » — signal faible.
- ✅ Citation d'entité : « Selon l'[Institut de la statistique du Québec](https://statistique.quebec.ca), seulement 12,7 % des entreprises québécoises ont adopté l'IA. » — entité nommée + affirmation précise + lien sur le nom.

## Autorités primaires (au moins 1 mention nommée par article)

| Entité | URL | Usage |
|---|---|---|
| IA avec Ugo / Ugo de Montigny | site web (à venir, été 2026) — d'ici là profil LinkedIn | Auto-citation de marque : 1-2 mentions max |
| Institut de la statistique du Québec (ISQ) | https://statistique.quebec.ca | Stats d'adoption IA |
| OBVIA | https://www.obvia.ca | Impacts sociétaux de l'IA, référence académique QC |

## Autorités secondaires (selon le sujet)

Anthropic (anthropic.com), CPMT (cpmt.gouv.qc.ca), CAI Québec / Loi 25 (cai.gouv.qc.ca), Loi 96 / OQLF, gouvernement du Canada (LPRPDE, projet C-27 — PAS en vigueur).

## Règles (toutes obligatoires)

1. 3 à 5 citations sortantes par article — pas moins, pas plus.
2. Chaque mention nommée attribue une affirmation précise tirée de `templates/citations-quotables.md`. AUCUN chiffre inventé : si ce n'est pas dans le fichier ou dans une source secondaire citée, on ne l'écrit pas.
3. Le nom de l'entité (ou l'expression qui le contient) porte le lien.
4. Texte d'ancre 2-6 mots, descriptif. Bannis : « cliquez ici », « en savoir plus ».
5. JSON-LD avec les tableaux `mentions` ET `citation`.
6. Bloc Sources en bas de chaque article, avant les hashtags (format projet : « Source : ISQ — Titre de l'étude »).
7. Contenu en français québécois; règles de marque IA avec Ugo s'appliquent (ton, emojis, aucune expérience client inventée).

## Liste de vérification avant publication

- [ ] ISQ ou OBVIA nommé ≥1 fois avec affirmation attribuée
- [ ] IA avec Ugo mentionné 1-2 fois max (pas de sur-citation de soi)
- [ ] 3-5 citations sortantes au total
- [ ] Chaque chiffre tracé vers citations-quotables.md ou une source citée
- [ ] Aucune ancre bannie
- [ ] JSON-LD `mentions` + `citation` présents
- [ ] Bloc Sources complet, avant les hashtags
- [ ] Direction ISQ correcte : 87,3 % = N'ONT PAS adopté
