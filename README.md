E-Mail Autoconfigure
====================

Certains clients de messagerie recueillent des informations de configuration avant de configurer des comptes de messagerie. Ce projet permet de fournir à des clients comme Outlook, Thunderbird et Mail la bonne configuration de serveur de messagerie, afin que les utilisateurs puissent simplement entrer leur courriel et mot de passe pour configurer un nouveau compte de messagerie.

### Microsoft Outlook

Le logiciel Outlook de Microsoft permet l'autoconfiguration en utilisant des résolution DNS et des requêtes HTTP. Il vérifie d'abord la zone DNS pour trouver un enregistrement SRV pour le service `autodiscover` utilisant le protocole TCP (`_autodiscover._tcp.<DOMAINE>`). Cet enregistrement à comme valeure le nom d'hôte du serveur où se trouve les fichiers d'autoconfiguration. Une requête HTTP vers le `https://<SRV_RECORD_VALUE>/autodiscover/autodiscover.xml` sera faite pour récupérer le fichier de configuration XML.


### Mozilla Thunderbird

Le logiciel Thunderbird utilise un principe similaire à Outlook pour la découverte des fichiers d'autoconfiguration. Il 

### IOS


Installation
------------

### Serveur Apache

Pour configurer l'Autodiscover sur un domaine unique, vous pouvez configurer votre hôte virtuel comme suit:

```
<VirtualHost *:443>
  ServerName autodiscover.{{$DOMAIN}}
  ServerAlias autodiscover.{{$DOMAIN}} autoconfig.{{$DOMAIN}}

  <Location />
    Options -Indexes
    AllowOverride All
  </Location>

  ...
</VirtualHost>
```

Copiez maintenant `settings.json.json.sample` à la racine de votre répertoire Virtual Host et appliquez vos variables de configuration.


### Autoconfig pour plusieurs domaines sur le même serveur

Lorsqu'un utilisateur met son adresse e-mail `user@example.org` dans son client de messagerie, il fera probablement une requête GET sur https://autodiscover.example.org/autodiscover/autodiscover.xml

Si vous avez plusieurs domaines hébergés sur votre serveur de messagerie, vous pouvez rediriger ces requêtes vers votre serveur main-autoconfig. Ajoutez cette configuration à votre configuration d'hôte virtuel existante :

```
<VirtualHost *:443>
  ServerName example.org
  SSLEngine On
  ...

  RewriteEngine On
  RewriteCond %{HTTP_HOST} ^autodiscover\. [NC]
  RewriteRule ^/(.*)      https://autodiscover.{{$DOMAIN}}/$1 [L,R=301,NE]

  RewriteCond %{HTTP_HOST} ^autoconfig\. [NC]
  RewriteRule ^/(.*)      https://autoconfig.{{$DOMAIN}}/$1 [L,R=301,NE]
  ...
</VirtualHost>

<VirtualHost *:80>
  ServerName example.org
  
  RewriteEngine On
  RewriteCond %{HTTP_HOST} ^autodiscover\. [NC]
  RewriteRule ^/(.*)      https://autodiscover.{{$DOMAIN}}/$1 [L,R=301,NE]

  RewriteCond %{HTTP_HOST} ^autoconfig\. [NC]
  RewriteRule ^/(.*)      https://autoconfig.{{$DOMAIN}}/$1 [L,R=301,NE]
  ...
</VirtualHost>

```


### DNS Setup

```
autoconfig              IN      CNAME   www
autodiscover            IN      CNAME   www

@                       IN      MX 10   {{$MX_DOMAIN}}.
@                       IN      TXT     "mailconf=https://autoconfig.{{$DOMAIN}}/mail/config-v1.1.xml"
_imaps._tcp             SRV 0 1 993     {{$MX_DOMAIN}}.
_submission._tcp        SRV 0 1 465     {{$MX_DOMAIN}}.
_autodiscover._tcp      SRV 0 0 443     autodiscover.{{$DOMAIN}}.
```

Au lieu d'un CNAME, vous pouvez bien sûr aussi choisir un A-record

```
autoconfig              IN      A      {{$AUTODISCOVER_IP}}
autodiscover            IN      A      {{$AUTODISCOVER_IP}}
```

Remplacer les variables ci-dessus par des données selon le tableau suivant

Variable         | Description
-----------------|-------------------------------------------------------------
MX_DOMAIN        | Le nom d'hôte de votre serveur MX
DOMAIN           | Le domain
AUTODISCOVER_IP  | L'adresse IP du server AutoDiscover

ToDo
----
 * Faire une version plus moderne utilisant des technologies comme Flask & NodeJS
 * Configuration de vhosts pour NGINX
 * Création de l'interface graphique
