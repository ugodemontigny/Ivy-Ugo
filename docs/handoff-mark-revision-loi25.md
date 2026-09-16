# Handoff pour Mark — révision de l'analyse stockage / LLM / Loi 25

**De :** Ugo (via Claude Code)
**Pour :** Mark
**Date :** 2026-09-16
**Rôle demandé :** réviseur. Lire, contester, corriger. Ne pas réécrire.

## 1. Contexte en trois phrases

IA avec Ugo stocke les « cerveaux » de ses clients (documents, courriels,
procédures, souvent des renseignements personnels) chez DigitalOcean et dans
GitHub, et prévoit une migration vers AWS pour laisser chaque client choisir
où résident ses données. Ugo envisage aussi de « construire son propre LLM »
pour la sécurité. Le document à réviser évalue ces options sous l'angle de
la Loi 25 (secteur privé, Québec) et propose une décision.

## 2. Où se trouve le travail

| Élément | Emplacement |
|---|---|
| Document à réviser | `docs/analyse-stockage-et-llm-securise.md` |
| Pull request | https://github.com/ugodemontigny/Ivy-Ugo/pull/1 |
| Branche | `claude/storage-llm-security-11wicm` |
| Base | `main` |

Le document compte sept sections. Les sections 6 et 7 sont le cœur :
comparaison Loi 25 et mini-analyse de risque pour l'option Toronto.

## 3. Les conclusions à contester

1. **AWS Montréal comme défaut** pour les nouveaux clients.
2. **DigitalOcean Toronto acceptable** pour les clients existants, avec une
   ÉFVP art. 17 documentée et huit mesures d'atténuation (section 7.2).
3. **GitHub exclu** pour toute donnée client, y compris l'historique Git.
4. **Palier « souverain »** (hébergeur québécois + modèle à poids ouverts
   hébergé chez nous) réservé aux données de santé, financières détaillées,
   de mineurs ou d'organismes publics (section 7.3).
5. **« Construire notre propre LLM » reformulé** en « modèle hébergé de façon
   privée ». Entraîner de zéro est écarté; inférence commerciale en région
   canadienne, zéro rétention, sans entraînement, comme défaut.
6. **Le chiffrement côté client** (clés chez IA avec Ugo) est présenté comme
   la mesure qui pèse le plus, parce qu'elle neutralise l'exposition au
   CLOUD Act autant à Toronto qu'à Montréal.

## 4. Ce que je te demande de vérifier

### Juridique (priorité haute)

- [ ] Les numéros d'articles de la LPRPSP cités (3.1, 3.2, 3.3, 3.5, 8, 10,
      12.1, 17, 18.3, 27, 28.1, 90.1) correspondent bien au texte en vigueur
      sur legisquebec.gouv.qc.ca.
- [ ] L'affirmation « la Loi 25 n'interdit pas d'héberger hors Québec, elle
      exige une ÉFVP concluant à une protection adéquate » est exacte et
      complète.
- [ ] La lecture de l'art. 17 pour AWS Montréal : données au Québec mais
      fournisseur soumis au CLOUD Act. Est-ce que la CAI a publié une
      position sur ce point? Si oui, citer.
- [ ] Le seuil de la section 7.3 (santé, financier détaillé, mineurs,
      organismes publics) est-il le bon découpage, ou trop large / trop
      étroit?
- [ ] Les montants de sanctions (10 M$ / 2 %, 25 M$ / 4 %) sont à jour.

### Technique (priorité moyenne)

- [ ] Régions : DigitalOcean `tor1`, AWS `ca-central-1` (Montréal) et
      `ca-west-1` (Calgary). Confirmer que rien n'a changé.
- [ ] La limite de BYOK chez DigitalOcean Spaces est-elle correctement
      décrite?
- [ ] Le chiffrement côté client est-il compatible avec l'index vectoriel et
      le RAG tels qu'ils sont implémentés chez IA avec Ugo? Si les données
      doivent être déchiffrées pour l'indexation, où se fait-elle et cela
      annule-t-il l'argument T1?

### Scores de risque (priorité basse)

- [ ] Les scores P × I de la section 7.2 et la comparaison 7.4 sont des
      estimations. Contester ceux qui te semblent faux, surtout T1 (accès
      autorité étrangère) et T4 (effacement GitHub).

## 5. Ce que le document ne couvre pas volontairement

- Coûts chiffrés par fournisseur.
- Choix d'un hébergeur québécois précis.
- Implémentation de l'adaptateur de stockage.
- Les obligations fédérales (LPRPDE) au-delà d'une mention.

Si tu penses qu'un de ces points doit entrer dans ce document plutôt que
dans un suivant, dis-le.

## 6. Format de retour attendu

Commentaires directement sur la PR, ligne par ligne quand c'est possible.
Pour chaque point : « erreur » (à corriger avant fusion), « nuance » (à
ajouter) ou « ok ». Un verdict final : fusionner tel quel, fusionner après
corrections, ou ne pas fusionner.

## 7. Questions ouvertes pour Ugo (à ne pas trancher dans la révision)

Reprises de la section 4 du document : nombre de clients et volume par
cerveau, demandes de résidence déjà reçues, présence de cerveaux dans
GitHub aujourd'hui, emplacement de l'index vectoriel, budget pour l'offre
« modèle privé ».
