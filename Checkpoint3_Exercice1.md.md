Connexion à la VM SRVWIN01 :
- Utilisateur : Administrator
- %Adm!n@2k=T0p
### Partie 1 : Gestion des utilisateurs

L'utilisateur **Kelly Rhameur** a quitté l'entreprise.  
Elle est remplacée par **Lionel Lemarchand**

**Q.1.1.1** Créer l'utilisateur **Lionel Lemarchand** avec les même attribut de société que **Kelly Rhameur**.

Sur une console PowerShell, faites la commande suivante pour trouver l'utilisateur **Kelly Rhameur** :
```powershell
PS C:\Users\Administrator> get-aduser Kelly.Rhameur

DistinguishedName : CN=Kelly.Rhameur,OU=DirectionDesRessourcesHumaines,OU=LabUsers,DC=TSSR,DC=LAN
Enabled           : True
GivenName         : Kelly
Name              : Kelly.Rhameur
ObjectClass       : user
ObjectGUID        : 3401b8b0-de22-492a-a20b-b0bdbce7c0cb
SamAccountName    : Kelly.Rhameur
SID               : S-1-5-21-1634400263-1414186445-1484066983-6763
Surname           : Rhameur
UserPrincipalName : Kelly.Rhameur@TSSR.LAN

PS C:\Users\Administrator> 
```

Puis une fois sur l'utilisateur, faites un clique droit puis sélectionnez `Copy...`. Remplissez les champs puis faites `Next >`. Mettez un mot de passe, puis faites `Next`. Sur la dernière fenêtre (voir image ci-dessous), vérifier ce que vous avez mis, puis faites `Finish`.
**IMAGE 1-COPY**

Maintenant, sur la fenêtre `Active Directory Users and Computers`, dans l'OU `DirectionDesRessourcesHumaines`, l'utilisateur **Lionel Lemarchand** a bien été créée.
**IMAGE 2-AFTERCOPY**

Vérifions, si les attributs de société ont bien été copié. Double-cliquez sur l'utilisateur **Lionel Lemarchand** et allez dans l'onglet `Organization`. Pensez à changer le champ `Job Title` par Directeur des Ressources Humaines. Vous pouvez voir que le reste correspond aux même attributs de société que **Kelly Rhameur**.
**IMAGE 3-AFTERCOPY**

**Q.1.1.2** Créer une OU **DeactivatedUsers** et déplace le compte désactivé de **Kelly Rhameur** dedans.

Faites un clique droit sur `LabUsers`, puis allez dans `New > Organizational Unit`. Dans la fenêtre qui apparait, remplissez comme suit :
**IMAGE 4-CREATEDOU**

Pour déplacer le compte de **Kelly Rhameur**, faites un clique droit sur son compte et sélectionnez `Move ...`, puis dans la nouvelle fenêtre choisissez l'OU `DeactivatedUsers` qui se trouve dans l'OU `Lab Users`.
**IMAGE 5-MOVEUSER**

Maintenant, il faut désactiver le compte de **Kelly Rhameur**, sélectionnez son compte puis faites un clic droit sur `Disable Account`. Une fois fait, vous verrez un changement dans l'icône du compte de **Kelly Rhameur**.
**IMAGE 6-DISABLEUSER**

**Q.1.1.3** Modifier le groupe de l'OU dans laquelle était **Kelly Rhameur** en conséquence.

Double cliquez sur l'utilisateur **Kelly Rhameur**, et sélectionnez l'onglet `Member Of`. Dedans, vous trouverez les différents groupes. **Kelly Rhameur** appartient toujours à `GrpUsersDirectionDesRessourcesHumaines`, nous allons donc supprimer le lien en cliquant sur `Remove` et puis faites `Apply` pour appliquer les changements.

Avant de rajouter **Kelly Rhameur** dans le groupe `GrpUsersDeactivatedUsers`, il faut le créer. Faites un clique droit sur `TSSR.LAN` et sélectionnez `New > Group`, remplissez les champs comme suit :
**IMAGE7-NEWGROUP**

Maintenant, faites un clic droit sur **Kelly Rhameur** et sélectionnez `Add to a group...`, mettez le nom du nouveau groupe et cliquez sur `Check Names`, le nom que vous avez mis devrait passez en souligné, puis faites `OK`. Retournez dans `Member Of` et vous aurez ceci :
**IMAGE8-NewGroup**

**Q.1.1.4** Créer le dossier Individuel du nouvel utilisateur et archive celui de **Kelly Rhameur** en le suffixant par **-ARCHIVE**.

Dans l'explorateur Windows, allez dans le disque `DossiersIndividuels (F:)`, dedans vous trouverez le dossier de **Kelly Rhameur**. Renommez le `kelly.rhameur-ARCHIVE`. Maintenant, créer un nouveau dossier pour **Lionel Lemarchand**. Voici ce que ça donne :
**IMAGE 9-Folder**
### Partie 2 : Restriction utilisateurs

**Q.1.2.1** Faire en sorte que l'utilisateur **Gabriel Ghul** ne puisse se connecter que du lundi au vendredi, de 7h à 17h.

L'utilisateur **Gabriel Ghul** n'existe pas dans la base de donnée.
```powerhsell
PS C:\Users\Administrator> get-aduser Gabriel.Ghul
get-aduser : Cannot find an object with identity: 'Gabriel.Ghul' under: 'DC=TSSR,DC=LAN'.
At line:1 char:1
+ get-aduser Gabriel.Ghul
+ ~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (Gabriel.Ghul:ADUser) [Get-ADUser], ADIdentityNotFoun 
   dException
    + FullyQualifiedErrorId : ActiveDirectoryCmdlet:Microsoft.ActiveDirectory.Management.ADIdentity 
   NotFoundException,Microsoft.ActiveDirectory.Management.Commands.GetADUser
 

PS C:\Users\Administrator>
```
Toutefois, voici la marche à suivre pour qu'un utilisateur ne puisse se connecter que du lundi au vendredi & de 7h à 17h.

Sélectionnez un utilisateur et double cliquez dessus. Allez dans l'onglet `Account` où vous pourrez voir `Logon Hours`, cliquez dessus. Et remplissez comme suit puis faites `OK` et enfin sur `Apply`.
**IMAGE10-LoginHours**

**Q.1.2.2** De même, bloquer sa connexion au seul ordinateur **CLIENT01**.

Voici la marche à suivre pour bloquer la connexion d'un utilisateur sur l'ordinateur **CLIENT01**.

Toujours dans les propriétés de l'utilisateur et dans l'onglet `Account`, allez dans `Log On To...`. Sélectionnez `The following computers` et rajouter l'ordinateur **CLIENT1**. Puis cliquez sur `OK` et enfin sur `Apply`.
**IMAGE11-Computer**

**Q.1.2.3** Mettre en place une stratégie de mot de passe pour durcir les comptes des utilisateurs de l'OU **LabUsers**.

Allez dans l'outils `Group Policy Management`, cliquez droit sur l'OU `LabUsers` et sélectionnez `Create a GPO in this domain, and link it here...`. Donnez lui le nom **PasswordPolicy** puis cliquez sur `Ok`.
**IMAGE 12-GPO**

Faites un clique droit sur la nouvelle GPO, et sélectionnez `Edit...`. Puis allez dans `Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy`, vous aurez différentes stratégie à modifier.
**IMAGE 13-stratégie0**

Ici vous pourrez configurer différents options, pour l'exemple, je vais modifier :
- **Maximum password age** à 180 jours (environ 6 mois)
- **Minimum password age** à 30 jours (environ 1 mois)
- **Minimum password length** à 14 caractères
- Activer **Password must meet complexity requirements**.

**IMAGE 14-PasswordStrategy**

### Partie 3 : Lecteurs réseaux

**Q.1.3.1** Créer une GPO **Drive-Mount** qui monte les lecteurs **E:** et **F:** sur les clients.