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