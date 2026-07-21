# PSM SSE3 — mise à jour : inscriptions, approbation, export de prélèvement

## 1. À faire une seule fois dans Firebase avant de tester

**Activer l'authentification anonyme** (indispensable — sans ça, le formulaire de
demande d'accès ne fonctionnera plus) :
Console Firebase (`deploiement-psm-v3`) → **Authentication** → **Sign-in method**
→ activez **Anonyme**.

**Republier les règles** : le contenu de `firebase-rules.json` a changé (nouveau
nœud `/demandes`) → Realtime Database → Règles → coller → Publier.

## 2. Nouveau workflow d'inscription

- Un utilisateur remplit le formulaire "Demander un accès" (nom, e-mail,
  établissement, motif). Ça crée une entrée dans `/demandes/{id}` avec
  `statut: "attente"` — plus besoin que vous alliez créer le compte à la main
  dans la Console Firebase.
- **L'admin local** de l'établissement concerné (ou vous, super-admin) voit
  cette demande apparaître dans son panneau, sous "Demandes d'inscription en
  attente", à sa prochaine connexion.
- **Approuver** : cocher "Administrateur" si besoin, cliquer "Approuver". Le
  compte Firebase Auth est créé, rattaché à l'établissement, et un e-mail
  Firebase (« définissez votre mot de passe ») part automatiquement au
  demandeur. La demande passe en "validée" et disparaît de la liste.
- **Refuser** : la demande passe en "refusée", et **votre propre client mail
  s'ouvre avec un message pré-rempli** à destination du demandeur. C'est un
  clic manuel de votre part pour l'envoyer — sans backend serveur, je ne peux
  pas envoyer un e-mail automatique à quelqu'un qui n'a pas encore de compte
  dans le système. C'est la limite honnête de cette architecture 100% statique.

## 3. Export "Fin de prélèvement"

- Chaque prélèvement validé pendant que la page reste ouverte est journalisé
  en mémoire (pas en base — uniquement pour cette session/cet onglet).
- Bouton **"Fin de prélèvement"**, à côté de "Remise à zéro" : demande
  confirmation, puis télécharge automatiquement un fichier **Excel** et un
  **PDF** récapitulant les lignes prélevées (date, malle, produit, quantité).
- Un champ "Envoyer par e-mail" apparaît ensuite. **Attention** : l'envoi de
  pièces jointes par e-mail via Web3Forms est une **fonctionnalité payante
  (Pro)** chez eux. Si votre compte n'est pas sur ce plan, l'envoi échouera
  proprement avec un message d'erreur — mais les deux fichiers sont de toute
  façon déjà téléchargés localement à ce moment-là, donc rien n'est perdu :
  il suffit de les joindre manuellement à un e-mail si besoin.
- Le journal de session est vidé après un export réussi (nouvelle session de
  prélèvement à zéro).

## Limites à connaître

- Le journal de prélèvement n'est **pas partagé/persistant** : si l'utilisateur
  ferme l'onglet avant de cliquer "Fin de prélèvement", l'historique de cette
  session est perdu (les quantités en base, elles, restent bien à jour — seul
  le récapitulatif exportable disparaît).
- Le refus de compte n'envoie pas d'e-mail automatique — un brouillon prêt à
  envoyer s'ouvre dans votre client mail.
- L'envoi de récapitulatif par e-mail avec pièces jointes dépend du plan
  Web3Forms (Pro requis pour les pièces jointes).
