# vuln-server
# vuln-server

# Modifications du Jenkinsfile pour l'Analyse Dynamique des Vulnérabilités

Dans le cadre de ce projet, le Jenkinsfile a été modifié pour répondre aux exigences suivantes :  
1. Suppression de l'analyse statique avec **Semgrep**.
2. Ajout de l'étape de démarrage du serveur **Node.js**.
3. Exécution de l'outil de test dynamique **Nuclei**.
4. Analyse des résultats de Nuclei et échec du build si des vulnérabilités de niveau "high", "medium" ou "low" sont détectées.

## Fonctionnement du Jenkinsfile Modifié

Voici les principales étapes du pipeline :

1. **Clonage du dépôt Git** : Le pipeline commence par cloner le dépôt contenant l'application vulnérable (ici, un serveur Node.js).

2. **Démarrage du serveur Node.js** : Une fois le dépôt cloné, le serveur Node.js est lancé en arrière-plan avec la commande `nohup node server.js &`. Cela permet au serveur de fonctionner pendant l'exécution du scan dynamique.

3. **Exécution du Scan Dynamique avec Nuclei** : Après que le serveur soit lancé, le pipeline exécute un scan dynamique avec **Nuclei**, un outil d'analyse de vulnérabilités. Ce scan est effectué sur l'application en local (`http://localhost:3000`). Les résultats du scan sont sauvegardés dans un fichier de rapport (`report-nuclei.txt`).

4. **Analyse des Résultats du Scan** : Le fichier de rapport généré par Nuclei est ensuite analysé. Ce rapport contient les vulnérabilités détectées, classées par niveau de gravité : **high**, **medium**, et **low**.

5. **Échec du Build en Cas de Vulnérabilités** : Si des vulnérabilités de niveaux **high**, **medium** ou **low** sont détectées dans le rapport, le build échoue (le statut de `currentBuild.result` est défini sur `FAILURE`), et un message est affiché pour indiquer qu'une vulnérabilité a été détectée.

6. **Nettoyage** : Enfin, après l'analyse, le serveur Node.js est arrêté et l'environnement de travail est nettoyé.

## Explication des Vulnérabilités de Niveaux "High", "Medium", "Low"

Dans le rapport généré par **Nuclei**, les vulnérabilités sont classées en trois niveaux de gravité qui sont essentiels pour prioriser les actions de sécurité :

- **High** : Les vulnérabilités de niveau "high" sont les plus critiques et doivent être traitées en priorité. Elles représentent un risque majeur, comme l'injection SQL ou l'exécution de code à distance, et peuvent avoir des conséquences dramatiques sur la sécurité de l'application. Un attaquant peut potentiellement exploiter ces vulnérabilités pour prendre le contrôle de l'application ou du système sous-jacent.  
  **Exemple** : SQL Injection, Command Injection.

- **Medium** : Les vulnérabilités de niveau "medium" sont sérieuses, mais elles sont généralement plus difficiles à exploiter que celles de niveau "high". Elles nécessitent souvent des conditions spécifiques ou un vecteur d'attaque moins évident. Cependant, elles ne doivent pas être ignorées, car elles pourraient permettre à un attaquant de causer des dégâts importants si elles sont exploitées.  
  **Exemple** : Cross-Site Scripting (XSS), Authentification faible.

- **Low** : Les vulnérabilités de niveau "low" sont moins critiques. Elles peuvent être liées à des mauvaises pratiques de sécurité, à des erreurs de configuration ou à des informations non sensibles mais potentiellement nuisibles. Bien qu'elles n'aient pas un impact immédiat sur la sécurité, elles devraient être corrigées pour maintenir de bonnes pratiques de sécurité et éviter des problèmes à long terme.  
  **Exemple** : Exposition de version de l'application, cookies non sécurisés.

## Processus de Gestion des Rapports

Lorsque **Nuclei** effectue un scan, il génère un rapport dans lequel chaque vulnérabilité est listée avec sa description, son URL et son niveau de gravité.

# Rapport de Vulnérabilités

## Exemple de Rapport

Voici un exemple simplifié de ce à quoi pourrait ressembler un rapport généré par Nuclei dans le fichier `report-nuclei.txt` :

  
```bash 
[HIGH] SQL Injection: http://localhost:3000/user?id=1

[MEDIUM] Open Redirect: http://localhost:3000/redirect?url=http://malicious.com

[LOW] Sensitive Information Disclosure: http://localhost:3000/config
```
Dans cet exemple :

- La vulnérabilité "SQL Injection" est une vulnérabilité de niveau **HIGH**. Cela signifie qu'un attaquant pourrait injecter des commandes SQL malveillantes dans l'application, compromettant potentiellement la base de données.
- La vulnérabilité "Open Redirect" est de niveau **MEDIUM**. Elle pourrait permettre à un attaquant de rediriger un utilisateur vers un site malveillant.
- La vulnérabilité "Sensitive Information Disclosure" est de niveau **LOW**. Elle implique l'exposition de données sensibles dans les réponses du serveur, mais sans impact immédiat.

## Analyse du Jenkinsfile

Une fois le rapport généré, le Jenkinsfile analyse son contenu pour détecter la présence de vulnérabilités. Si des vulnérabilités de niveau **high**, **medium**, ou **low** sont trouvées, le build échoue. Le pipeline lit le rapport et utilise une simple recherche pour détecter la présence de ces termes :

```yml
if (report.contains("high") || report.contains("medium") || report.contains("low")) {     echo 'Vulnérabilités détectées dans le scan Nuclei, échec du build.'     currentBuild.result = 'FAILURE' } else {     echo 'Aucune vulnérabilité détectée, build réussi.' }
```

## Résultat de l'Analyse

Si des vulnérabilités de niveau **high**, **medium** ou **low** sont détectées, le pipeline échoue, et un message d'erreur est généré pour informer l'équipe qu'une ou plusieurs vulnérabilités ont été trouvées. Le message pourrait être quelque chose comme :

`Vulnérabilités détectées dans le scan Nuclei, échec du build.`

Si aucune vulnérabilité n'est détectée, le pipeline passe avec succès et un message de succès est affiché :

`Aucune vulnérabilité détectée, build réussi.`




