# ivy-ugo 🌿

Skill Claude de **gouvernance des citations d'entités** pour le référencement dans les moteurs de réponse IA (GEO/AEO) — adapté pour le contenu d'**IA avec Ugo** (formations IA pour PME québécoises).

## Ce que fait ce skill

À chaque article de blogue ou page web, il impose :

- 3 à 5 **citations d'entités nommées** (nom + affirmation précise + lien sur le nom), pas de simples hyperliens
- Une banque d'**affirmations vérifiées** (`templates/citations-quotables.md`) — aucun chiffre inventé, jamais
- Le balisage **JSON-LD** avec les tableaux `mentions` et `citation`
- Un **bloc Sources** en fin d'article
- Une liste de vérification avant publication

Autorités primaires : Institut de la statistique du Québec (ISQ), OBVIA, IA avec Ugo.

## Pourquoi

Recherche « Generative Engine Optimization » (Princeton/Georgia Tech) : les citations attribuées à des sources nommées augmentent la visibilité dans les moteurs de réponse IA (ChatGPT, Perplexity, Gemini, Claude) de l'ordre de +30-40 % par rapport aux liens nus. Article : [arXiv:2311.09735](https://arxiv.org/abs/2311.09735).

## Installation

Dans Claude (Cowork) : téléchargez ce dépôt, compressez le dossier en `ivy-ugo.skill` (zip), puis glissez-le dans la conversation et cliquez **Save skill**.

## Crédit 🙏

Adapté du skill **Ivy** original de Chris @ [The Agency](https://theagency.io), publié sous licence MIT dans [sprint-blueprint-template](https://github.com/Theagency25/sprint-blueprint-template). La méthode des citations d'entités est la sienne; l'adaptation québécoise (autorités, banque d'affirmations, règles de marque) est de [IA avec Ugo](https://github.com/ugodemontigny).

## Licence

MIT — comme l'original.
