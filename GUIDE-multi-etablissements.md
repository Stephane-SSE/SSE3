# PSM Inventory Manager — passage au multi-établissements

## Ce qui a changé

- Chaque établissement (site PSM) a maintenant son propre stock, isolé des autres :
  `/etablissements/{etabId}/produits/...`
- Chaque établissement a ses propres administrateurs locaux :
  `/etablissements/{etabId}/admins/{uid}: true`
- Chaque utilisateur est rattaché à **un seul** établissement :
  `/users/{uid}/etablissementId: "etabId"`
- Un ou plusieurs **super-administrateurs** gèrent tous les établissements :
  `/superadmins/{uid}: true`

Un utilisateur connecté mais non rattaché voit un écran bloquant "Compte non rattaché" et ne peut rien consulter.

## Étapes de déploiement

1. Remplacez `index.html` dans `github.com/Stephane-SSE/SSE2` par le fichier fourni (`psm.json` est inchangé, il ne sert qu'à amorcer l'établissement historique).
2. Mettez à jour les règles de la Realtime Database avec le contenu de `firebase-rules.json` (Console Firebase > Realtime Database > Règles > Publier).
3. **Bootstrap du premier super-administrateur** (étape manuelle obligatoire, une seule fois) :
   - Console Firebase > Realtime Database > données.
   - Ajoutez manuellement le nœud `superadmins/{votre UID}` = `true` (votre UID est visible dans Authentication).
   - C'est la seule étape qui doit être faite à la main : aucune interface ne peut créer le tout premier super-admin (il faudrait déjà en être un).
4. Connectez-vous sur l'application avec ce compte. Comme il n'a pas encore d'`etablissementId`, vous verrez le panneau **Super-administration** avec un message vous invitant à créer/rattacher votre établissement.

## Migrer les données existantes (site actuel `deploiement-psm2`)

Vos données actuelles sont à la racine (`/produits`, `/admins`), hors de toute structure d'établissement.

1. Connectez-vous en tant que super-administrateur (étape ci-dessus).
2. Dans le panneau **Super-administration**, un bloc rouge **"Migration des anciennes données"** apparaît automatiquement s'il détecte des données à la racine.
3. Cliquez sur **"Migrer vers « etab-principal »"**. Cela :
   - crée l'établissement `etab-principal` ("Établissement historique"),
   - copie `/produits` vers `/etablissements/etab-principal/produits`,
   - copie `/admins` vers `/etablissements/etab-principal/admins` (ces comptes redeviennent admins, mais localement à cet établissement),
   - rattache automatiquement chaque ancien admin, ainsi que vous-même, à `etab-principal`.
4. Rechargez la page.
5. Une fois vérifié que tout fonctionne, supprimez manuellement `/produits` et `/admins` à la racine dans la Console Firebase (l'app ne le fait pas automatiquement, par sécurité).

## Ajouter un nouvel établissement (nouvel hôpital)

1. Panneau **Super-administration** > "Créer un établissement" > saisir le nom > Créer.
2. Créer le compte Firebase Auth du premier utilisateur de ce site (Console Firebase > Authentication > Ajouter un utilisateur), ou attendre sa demande d'accès via le formulaire (qui indique maintenant l'établissement souhaité).
3. Copier l'UID de ce compte (colonne UID dans Authentication).
4. Dans le panneau Super-administration, section "Rattacher un utilisateur" : coller l'UID, choisir l'établissement dans la liste, cocher "Admin local" si c'est la personne qui doit pouvoir importer le fichier Excel, puis "Rattacher".
5. Ce nouvel établissement démarre **vide** : son admin local doit importer son propre `Liste_PSM.xlsx` via la zone d'administration habituelle pour peupler son stock.

## Ce qui n'a pas changé

- Recherche, prélèvement de quantité, synchronisation temps réel : identiques, mais désormais scopés à l'établissement de l'utilisateur connecté.
- Import xlsx : toujours réservé aux administrateurs (super-admin ou admin local de l'établissement), remplace uniquement le stock de cet établissement.
- Demande d'accès (Web3Forms) : le formulaire inclut maintenant un champ "Établissement / site PSM" pour indiquer au super-admin où rattacher le compte une fois créé.

## Limite connue

Firebase Auth ne permet pas, côté client, de retrouver un UID à partir d'un e-mail. Le rattachement d'un utilisateur nécessite donc de copier son UID depuis la Console Firebase (Authentication) — il n'y a pas moyen de l'éviter sans backend serveur, ce que l'hébergement GitHub Pages ne permet pas.
