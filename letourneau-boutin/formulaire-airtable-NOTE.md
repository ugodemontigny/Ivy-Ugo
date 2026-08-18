# Formulaire Airtable — état et marche à suivre

Mis à jour le 18 août 2026.

## Deux questionnaires existent dans Airtable — lequel utiliser

1. **NOUVELLE base « Questionnaire découverte — Létourneau Boutin »** (`appQ72xtDfFJk1QP0`, table « Réponses » `tblMCaaN02Kh0qDpV`) — créée le 18 août. Les 12 questions FINALES approuvées par Jennifer, **anonyme par code répondant A–E**, connexions testées (écriture/lecture/suppression), table à zéro. ✅ **C'est celle-là qu'on utilise.**
2. **ANCIEN formulaire (17 juillet)** dans la base CRM (`appdr4ifunSEpMjEG`, table « Découverte — Audit IA ») — lien public déjà publié : `https://airtable.com/appdr4ifunSEpMjEG/shrmixjxGXLiEPDg0`. ⚠️ **Ne pas l'utiliser** : sa première question est « Nom » (version d'avant l'anonymisation), il vit dans la base de prospection, et sa mise en page ne peut être ni vérifiée ni corrigée par le connecteur. À l'occasion : ouvrir ce formulaire dans Airtable et le **désactiver** (Form → Share form → désactiver le lien) pour éviter toute confusion.

## Les liens (pour Ugo, connecté à son compte)

- Base : https://airtable.com/appQ72xtDfFJk1QP0
- Table Réponses : https://airtable.com/appQ72xtDfFJk1QP0/tblMCaaN02Kh0qDpV

## Étapes pour rendre le formulaire « live » (≈ 2 minutes, interface Airtable)

1. Ouvrir la base → table « Réponses ».
2. Barre de gauche (liste des vues) → **Create… → Form** — une vue formulaire se crée avec les champs de la table.
3. Dans le formulaire : **masquer le champ « Réf. »** (glisser hors du formulaire ou interrupteur), garder les questions dans l'ordre 1 → 11, marquer **« 1. Votre code répondant » obligatoire**.
4. Coller le **mot d'introduction** dans la description du haut et le **bloc confidentialité** dans la question 11 ou le message de fin (textes prêts dans `questionnaire-final.md`).
5. Personnaliser le message de remerciement (suggestion : « Merci! Vos réponses sont enregistrées sous votre code. — Ugo »).
6. **Share form → Create a shareable link** → copier le lien (format `https://airtable.com/appQ72xtDfFJk1QP0/pag…/form` ou `…/shr…`).
7. **Coller le lien dans la conversation avec Claude** — je prends le relais : vérification que le lien répond, insertion dans les 4 brouillons Gmail à la place de `[LIEN-FORMULAIRE]`, puis test complet (réponse bidon code E → vérification dans la table → suppression → compteur à zéro).

## Rappels confidentialité (Loi 25)

- Le lien est le même pour tout le monde; l'anonymat vient du **code répondant**, envoyé individuellement (idéalement à l'adresse personnelle).
- La liste code ↔ personne reste chez Ugo, jamais dans le dépôt ni dans un courriel de travail.
- Détruire les réponses ET la liste de correspondance au plus tard 6 mois après la collecte.
