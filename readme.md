<h5>
Un servlet est une application web java qui respecte l’API Java Servlet.

Un servlet peut être écrit en pur code Java ou être généré à partir d’une page JSP.

Un servlet est exécuté par un conteneur de servlets.

Tomcat est le conteneur de servlets de la fondation Apache.

Jenkins est une application Java fournie sous forme de servlet.
</h5>

<H1>Introduction à Tomcat et Jankins</H1>
Votre serveur LAMP va vous permettre de bénéficier des nombreuses applications PHP existantes. Cependant, toutes les applications ne sont pas développées en PHP et pour pouvoir installer certains logiciels, vous aurez peut-être besoin d’installer d’autres environnements.

Par exemple, le logiciel Jenkins est de plus en plus populaire dans le monde du développement logiciel. Il permet d’automatiser les tests, la compilation et le déploiement des logiciels. Ce programme est écrit dans un autre langage très populaire : Java.

Je vous propose donc, dans la suite de ce chapitre, d’installer Jenkins et de découvrir ainsi l’univers des applications Java.

Pour cela, vous aurez besoin d’avoir quelques notions de vocabulaire propres à cet environnement.

Découvrez l’univers des applications web Java : Servlets, JSP, etc.
Dans l’univers Java, les applications destinées à générer dynamiquement du code HTML se présentent généralement sous la forme de servlets. Un servlet est donc simplement une application web java qui respecte l’API Java Servlet. Jenkins est fourni sous forme de servlet. Concrètement, il se présente sous la forme d’un fichier binaire d’extension .war.

Pour créer un servlet, un développeur peut :

soit écrire du code “pur java” et le compiler ;

soit écrire une JSP (JavaServer Page).

Une JSP est en fait une page HTML à laquelle ont été ajoutés des appels à du code Java. La JSP sera alors compilée par un compilateur JSP pour la transformer en servlet.

Ensuite, pour pouvoir exécuter vos servlets sur votre serveur et leur transmettre les requêtes HTTP, vous aurez besoin d’un conteneur de servlets. Dans ce chapitre, vous utiliserez Tomcat qui est un autre projet de la fondation Apache.

Tomcat est constitué de plusieurs composants :

<H4>Catalina :</H4> qui est le conteneur de servlets en lui-même, et qui est chargé de leur exécution ;

<H4>Coyote :</H4>qui est un connecteur HTTP, donc un mini-serveur web qui va transmettre les requêtes HTTP à Catalina ;

<H4>Jasper :</H4> qui est le compilateur JSP de Tomcat.

Vous comprenez maintenant que pour pouvoir utiliser l’application Jenkins, la première étape va être d’installer un serveur Tomcat.

# Installez le serveur Apache Tomcat

Sur la page de téléchargement de Jenkins, téléchargez la version Long-Term Support sous forme d’un “Generic Java Package (.war)” :
```
$ cd /tmp
$ wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war

```
Sous Ubuntu, vous pouvez simplement installer Tomcat par la commande :
```
$ sudo apt install tomcat9
```
Installez ensuite toutes les polices utilisées par Java et recommandées pour l’installation de Tomcat, sans quoi certaines applications ne pourront pas fonctionner.
```
$ sudo apt install  fonts-dejavu-extra fonts-ipafont-gothic fonts-ipafont-mincho fonts-wqy-microhei fonts-indic
```
Par défaut, Tomcat écoute sur le port TCP 8080. Vous pouvez tester la bonne installation de Tomcat en vous connectant depuis le navigateur de votre client sur ce port :

<image  src="page_connexion_tomcat.png" alt="page connexion tomcat" width="500" height="300" center>


Tomcat dispose de deux interfaces web d’administration :

<h4>Manager App : </h4> qui vous aidera à gérer vos applications et vous fournira de précieuses statistiques sur l’utilisation de votre serveur.

<H4>Host Manager App </H4>qui vous permettra de gérer vos virtual hosts.

Sous Ubuntu, vous pouvez les installer par la commande :
```
$ sudo apt install tomcat9-admin
```

Pour les utiliser, vous devrez ensuite ajouter un utilisateur autorisé dans votre configuration Tomcat. Commencez par éteindre votre serveur :
```
$ sudo systemctl stop tomcat9
```
Allez dans le répertoire de configuration de Tomcat  /etc/tomcat9  . Les fichiers de configuration sont tous au format XML. Éditez le fichier  tomcat-users.xml  en ajoutant la ligne suivante entre les balises  <tomcat-users />  :
```html
<tomcat-users …>
    <user username=”admin” password=”password” roles=”manager-gui,admin-gui” />
</tomcat-users>
```
Cette ligne ajoute un utilisateur “admin” avec les rôles “manager-gui” et “admin-gui”.

Il serait également plus sûr de restreindre l’accès à l’IP de votre client. Pour cela, allez dans le répertoire de configuration des applications Catalina  /etc/tomcat9/Catalina  . Vous y trouverez un répertoire localhost correspondant à votre seul “virtual host” actuel. Dans ce répertoire, vous avez un fichier XML pour la configuration de chaque application. Dans le fichier  manager.xml   , vous trouvez la balise  <Context />  suivante :
```html

<Context path=”/manager”
    docBase=”/usr/share/tomcat9-admin/manager”
    antiResourceLocking=”false” privileged=”true” />
```
L’attribut  path  indique l’URL d’accès à l’application, et l’attribut  docBase  indique le répertoire où se trouvent les fichiers de l’application. Ajoutez une balise  <Valve />  à cette balise  <Context />  de cette façon :
```html

<Context path=”/manager”
   docBase=”/usr/share/tomcat9-admin/manager”
   antiResourceLocking=”false” privileged=”true” >
   <Valve className=”org.apache.catalina.valves.RemoteAddrValve”
       allow=”192.168.29.2” />
</Context>
```
Cette configuration permet de restreindre l’accès à l’application Manager à l’IP 192.168.29.2 de votre client. Vous pouvez éditer de la même façon le fichier  host-manager.xml  puis redémarrer le serveur Tomcat :
```
$ sudo systemctl start tomcat9
```
Depuis votre client, vous pouvez alors vous connecter à  http://www.example.com:8080/manager/html  et  http://www.example.com:8080/host-manager/html  en utilisant les login et mot de passe que vous avez définis dans le fichier  tomcat-users.xml  .

<image src="tomcat_application_manager.png" alt="page connexion tomcat" width="500" height="300" center>

# Installez Jenkins comme servlet Tomcat
Sur la page de téléchargement de Jenkins, téléchargez la version Long-Term Support sous forme d’un “Generic Java Package (.war)” :
```
$ cd /tmp
$ wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
```
Les fichiers de Tomcat sont dans  /var/lib/tomcat8  . Pour installer Jenkins, il vous suffit de déposer votre fichier  .war  dans le répertoire  /var/lib/tomcat8/webapps  qui contient les applications :

```
$ sudo mv /tmp/jenkins.war /var/lib/tomcat9/webapps
```
Votre fichier  .war  est automatiquement déployé, et un répertoire Jenkins est créé.

Avant de pouvoir utiliser Jenkins, il vous reste juste à configurer la variable  JENKINS_HOME  qui indique le chemin dans lequel Jenkins va stocker configurations, logs et builds. Par défaut, ces fichiers seront stockés dans  /root/.jenkins/   , mais vous allez plutôt les mettre dans  /var/lib/jenkins/  . Commencez par créer ce répertoire et donnez les droits à l’utilisateur  tomcat9  qui gère Tomcat :
```
$ sudo mkdir /var/lib/jenkins
$ sudo chown tomcat9:tomcat9
```
Puis éditez le fichier  /etc/tomcat9/context.xml  et ajoutez la ligne suivante entre les balises  <Context />  :
```html
<Environment name=”JENKINS_HOME” value=”/var/lib/jenkins” type=”java.lang.String” />
```
Redémarrez Tomcat et connectez-vous à l’adresse http://www.example.com:8080/jenkins depuis votre client :
```
$ sudo systemctl restart tomcat9
```
Capture d'écran du navigateur d'un client montrant l'interface d'installation de Jenkins
Interface d'installation de Jenkins : vérification de la clé secrète
Suivrez les instructions à l’écran et recopiez le mot de passe que vous trouverez dans  /var/lib/jenkins/secrets/InitialAdminPassword  .

Capture d'écran du navigateur d'un client connecté à l'interface de configuration de Jenkins (phase d'installation des plugins)
Interface d'installation de Jenkins : installation des plugins
Si vous rencontrez l’erreur “This Jenkins instance appears to be offline”, ça n’est probablement pas un problème de connexion, c’est un problème de certificat HTTPS. Vous pouvez contourner le problème en éditant l’adresse contenue dans  /var/lib/jenkins/hudson.model.UpdateCenter.xml  pour mettre l’adresse en http:// plutôt que https://. Redémarrez Tomcat, retapez le mot de passe et le problème devrait être résolu.

Suivez ensuite le protocole d’installation : installez les plugins par défaut, créez un compte administrateur et voilà, vous avez installé Jenkins !

En production, il faudra supprimer le répertoire  /var/lib/tomcat9/webapps/ROOT  qui contient l’application “par défaut” de Tomcat, et qui pour l’instant ne contient que la page d’accueil de Tomcat.
<p style ="color:red">Attention
 Actuellement, vous utilisez le serveur web Coyote, intégré à Tomcat, sur le port 8080 pour vous connecter à Jenkins. Ce serveur web n’est pas fait pour être utilisé pour servir directement les connexions des clients. Effectivement, il est moins performant pour servir des fichiers statiques, et ne supporte pas nativement le HTTPS. Il est recommandé d’utiliser un serveur web plus robuste tel qu’Apache ou Nginx comme proxy pour renvoyer les requêtes vers Tomcat. Le serveur web gérera la connexion HTTPS, les restrictions d’accès et les pages statiques. Seules les requêtes vers l’application Java seront renvoyées vers Tomcat.
 </p>

Il existe alors deux moyens de faire communiquer votre serveur web et votre serveur Tomcat :

par le protocole HTTP, le serveur web se contente alors de faire une redirection HTTP vers le serveur Tomcat ;

par le plugin mod_jk (uniquement pour Apache) qui utilise un protocole spécial pour faire communiquer Apache et Tomcat.
<p style="color:red">Attention
 
En résumé
Un servlet est une application web java qui respecte l’API Java Servlet.

Un servlet peut être écrit en pur code Java ou être généré à partir d’une page JSP.

Un servlet est exécuté par un conteneur de servlets.

Tomcat est le conteneur de servlets de la fondation Apache.

Jenkins est une application Java fournie sous forme de servlet.

Dans le chapitre suivant, vous verrez comment configurer Nginx en tant que proxy HTTP pour accéder à Tomcat.
</p>