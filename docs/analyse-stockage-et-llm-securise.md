# Analyse — Stockage des « cerveaux » clients et LLM maison

Note interne IA avec Ugo. Objectif : évaluer (1) un stockage multi-fournisseur
(DigitalOcean, GitHub, bientôt AWS) où chaque client choisit où réside son
« cerveau », et (2) la construction d'un LLM maison pour la sécurité.

Demande d'origine : « For the storage of data we use DigitalOcean and GitHub
and soon moving to AWS so the storage of clients' brains can be where they
want. Also building our own LLM too for more security. »

---

## 1. Résumé exécutif

| Sujet | Verdict | Pourquoi |
|---|---|---|
| Stockage au choix du client (DO / AWS) | ✅ Bonne idée, faisable | Les deux offrent un API compatible S3 et une région canadienne. Une couche d'abstraction unique suffit. |
| GitHub comme lieu de stockage des cerveaux clients | ❌ À éviter | GitHub est un outil de code, pas un stockage de données personnelles. Pas de choix de région pour un compte standard, historique Git impossible à purger proprement (droit à l'effacement, Loi 25). |
| « Construire notre propre LLM » | ⚠️ Reformuler | Entraîner un modèle de zéro est hors de portée (coût, équipe, qualité). Ce qui est réaliste et sécuritaire : **héberger soi-même un modèle à poids ouverts** ou utiliser une **inférence en région canadienne** avec zéro rétention. |

---

## 2. Stockage : où vivent les cerveaux clients

### 2.1 Ce qu'un « cerveau » contient

Documents internes du client, courriels, procédures, données de clients finaux
→ souvent des **renseignements personnels** au sens de la Loi 25. Donc :

- Registre des traitements et lieu d'hébergement documenté par client.
- **ÉFVP** (évaluation des facteurs relatifs à la vie privée) obligatoire avant
  tout transfert de renseignements personnels hors Québec (art. 17 Loi 25).
  Une région canadienne hors Québec (Toronto, Calgary) reste un transfert
  « hors Québec » à documenter, même si le risque est faible.
- Contrat de traitement avec chaque fournisseur infonuagique (déjà couvert par
  leurs DPA standard, à conserver au dossier).

### 2.2 Régions disponibles

| Fournisseur | Région canadienne | Stockage objet | Chiffrement clés client |
|---|---|---|---|
| DigitalOcean | Toronto (`tor1`) | Spaces (API S3) | Limité (SSE côté serveur, pas de BYOK complet) |
| AWS | Montréal (`ca-central-1`), Calgary (`ca-west-1`) | S3 | KMS, clés gérées par le client (BYOK), journalisation CloudTrail |
| GitHub | Aucun choix de région pour un compte standard | n/a | n/a |

**Montréal est la seule région qui permet de dire « vos données restent au
Québec ».** C'est un argument commercial fort pour une PME québécoise.
À vérifier avant de promettre : les régions et fonctionnalités évoluent;
confirmer sur les pages officielles au moment de signer.

### 2.3 Architecture recommandée

```
Client A ──┐                       ┌── DO Spaces tor1  (bucket a-…)
Client B ──┼─► API IA avec Ugo ───►│── AWS S3 ca-central-1 (bucket b-…)
Client C ──┘   (couche stockage)   └── AWS S3 ca-central-1 (bucket c-…)
```

Principes :

1. **Une seule interface de stockage** dans le code (interface S3). Le
   fournisseur et la région sont une **configuration par client**, pas du
   code différent. Ajouter AWS = ajouter un adaptateur, pas réécrire.
2. **Un bucket (ou préfixe) par client**, jamais de mélange. Politique IAM
   qui interdit l'accès croisé.
3. **Chiffrement** : TLS en transit, chiffrement au repos partout. Sur AWS,
   une clé KMS par client; offrir le BYOK aux clients exigeants.
4. **Effacement** : procédure documentée et testée pour supprimer un cerveau
   complet (données, index vectoriel, sauvegardes, journaux) sur demande.
5. **Sauvegardes** dans la même région que la donnée primaire, sinon on
   annule la promesse de résidence.
6. **Journal d'accès** conservé (qui a lu quoi, quand) — requis pour répondre
   à un incident (obligation de notification, Loi 25).
7. **Index vectoriel / base de données** : il doit suivre la même règle de
   résidence que les fichiers. Un cerveau à Montréal avec un index vectoriel
   aux États-Unis n'est pas « au Canada ».

### 2.4 Rôle de GitHub

- Code source, infrastructure-as-code, documentation publique : oui.
- Cerveaux clients, exports, fichiers de formation : **non**.
- Activer : secret scanning, push protection, 2FA obligatoire, revue des
  accès des collaborateurs. Vérifier qu'aucun cerveau client n'est déjà
  commité dans un dépôt (historique inclus).

### 2.5 Plan de migration vers AWS

1. Créer l'organisation AWS (Organizations, comptes séparés prod/dev),
   région par défaut `ca-central-1`.
2. Implémenter l'adaptateur S3 + KMS derrière l'interface existante.
3. Migrer un client pilote (copie, vérification d'intégrité, bascule, purge
   de l'ancien emplacement, preuve de purge au dossier).
4. Ajouter la question « où voulez-vous héberger vos données? » au processus
   d'intégration client, avec les options : Toronto (DO), Montréal (AWS).
5. Mettre à jour la politique de confidentialité et les contrats clients.

---

## 3. « Notre propre LLM » : ce que ça veut dire vraiment

### 3.1 Trois options, très différentes

| Option | Coût / effort | Sécurité | Qualité | Recommandation |
|---|---|---|---|---|
| A. Entraîner un modèle de fondation de zéro | Millions $, équipe ML, mois | Totale | Très inférieure aux modèles de pointe | ❌ Non |
| B. Héberger soi-même un modèle à poids ouverts (Llama, Mistral, Qwen…), éventuellement affiné (fine-tuning) sur nos données | GPU loués (~2 000–10 000 $/mois selon la taille), DevOps | Très forte : aucune donnée ne sort de notre infra | Bonne pour RAG et tâches ciblées, sous les meilleurs modèles commerciaux | ✅ Pour les clients qui l'exigent |
| C. Inférence commerciale en région canadienne, zéro rétention (ex. Claude via Amazon Bedrock dans `ca-central-1`) | Paiement à l'usage, aucune infra | Forte : données traitées au Canada, non utilisées pour l'entraînement, non conservées | Meilleure qualité disponible | ✅ Par défaut |

### 3.2 Ce que la sécurité exige réellement

Le risque principal n'est pas « le modèle », c'est **le chemin que prennent
les données** : où elles sont envoyées, si elles sont conservées, qui peut
les voir, si elles servent à entraîner un autre modèle. Les options B et C
répondent toutes deux à ces questions. L'option A n'apporte rien de plus en
sécurité et coûte énormément.

À exiger quel que soit le choix :

- Contrat qui interdit l'entraînement sur nos données et fixe la rétention à
  zéro (ou quelques jours pour la détection d'abus, documenté).
- Résidence de l'inférence au Canada, ou sur nos propres serveurs.
- Journalisation des requêtes chez nous (pas chez le fournisseur).
- Tests de fuite entre clients : un cerveau ne doit jamais apparaître dans
  la réponse d'un autre client (isolation par client au niveau du RAG).
- Défenses contre l'injection de prompt dans les documents clients.

### 3.3 Plan recommandé

1. **Maintenant** : option C. Inférence en région canadienne, contrat sans
   entraînement, zéro rétention. Argument marketing valide : « Vos données
   sont traitées au Canada et ne servent jamais à entraîner un modèle. »
2. **Dans 6 à 12 mois, si des clients l'exigent** (secteur public, santé,
   juridique) : option B en offre premium. Un modèle à poids ouverts hébergé
   sur nos serveurs (DO GPU Droplets à Toronto ou AWS à Montréal), affiné au
   besoin. Facturer ce surcoût.
3. **Ne pas** communiquer « nous construisons notre propre LLM ». Dire plutôt
   « modèle hébergé de façon privée » ou « inférence souveraine ». C'est
   précis, vérifiable, et ça n'engage pas à un chantier impossible.

---

## 4. Questions ouvertes pour Ugo

1. Combien de clients actifs et quel volume par cerveau (Go)? Ça détermine si
   le multi-fournisseur vaut la complexité tout de suite.
2. Des clients ont-ils déjà demandé une résidence précise (Québec vs Canada)?
3. Y a-t-il des cerveaux clients dans un dépôt GitHub aujourd'hui?
4. L'index vectoriel actuel est hébergé où?
5. Budget mensuel acceptable pour une offre « modèle privé »?

## 5. Prochaines étapes concrètes

- [ ] Audit : lister chaque emplacement actuel de données clients (DO, GitHub,
      index vectoriel, sauvegardes, outils tiers).
- [ ] Créer le compte AWS, région `ca-central-1`, un bucket pilote + KMS.
- [ ] Implémenter l'interface de stockage unique + adaptateur AWS.
- [ ] Rédiger le modèle d'ÉFVP par client et le gabarit de contrat.
- [ ] Choisir et contractualiser l'inférence en région canadienne.
- [ ] Rédiger la procédure d'effacement complet et la tester sur un client
      test.

---

Sources à citer dans toute communication externe tirée de ce document :
Commission d'accès à l'information du Québec (cai.gouv.qc.ca) pour la Loi 25;
pages officielles de régions AWS et DigitalOcean pour la résidence des
données. Aucun chiffre de ce document ne doit être publié sans vérification.
