# Deuxième édition bookCTF 2020 - RSA#1 et RSA#2

## Introduction
Voici les write-up des deux challanges RSA du bookCTF. Sera expliqué comme résoudre le challenge avec les outils nécessaires et des exemples de codes en python.

## RSA#1
Nous disposons de deux fichiers, l'un au format pem contenant une clé publique et privée et un autre fichier binaire qui contient le ciphertext.
Lorsque nous voulons chiffrer un message avec le système RSA, il nous faut générer une clé publique et une clé privée afin de pouvoir déchiffrer ce message. Comme la clé privée est déjà donné pas besoin de la chercher, il nous suffit simplement de déchiffrer le message avec. 
Le but du challenge est donc d'extraire le ciphertext pour pouvoir le déchiffrer avec la clé privé. 

Comme vous l'aurez pu constater, le ciphertext peut être un peu perturbant, stocké dans un fichier binaire, votre éditeur tente de décoder les charactères et vous affiche un peu n'importe quoi. Pour remédier à cela, voici un script python qui stockera dans un fichier nommé ciphertext.txt la forme hexadécimal du ciphertext.
```py
with open("flag.txt.enc", "rb") as fp:  #J'ouvre mon fichier en mode "rb" qui permet d'ouvrir les fichiers binaires et j'y écris récupère le contenu
    r = fp.read()
    r = r.hex()     #.hex() donne plus proprement la forme hexadécimal du ciphertext 

with open("ciphertext.txt", "w") as fp2:    #Ici, je stock le ciphertext dans un autre fichier que j'utiliserais pour déchiffrer
    fp2.write(r)

```
Ensuite, nous devons déchiffrer ce message.
D'habitude, nous n'avons pas la paire de clé privée et nous devons extraire la clé publique pour trouver la clé privée. 
Mais comme elle est donné, il nous suffit simplement, grâce à openSSL, d'exécuter la commande suivante:
```
openssl rsautl -decrypt -inkey private.pem -in flag.txt.enc -out flag.txt
```

Et voilà la premier chall rsa est flag ! 



## RSA#2:
Dans ce challenge, nous possédons la clé publique et le ciphertext dans un fichier binaire. Nous allons donc devoir trouver la clé privée par nous même.
Pour extraire le ciphertext, vous devez faire la même opération que sur le premier challenge. Une fois que c'est fait nous devons faire de même avec la clé publique grâce à openSSL et la commande suivante:
```
openssl rsa -
```
Nous avons donc le modulus (n), sous forme hexadécimal, et l'exposant publique (e). Voici le code en python pour récupérer la version décimal de n:
```py
N = int("""41:0e:eb:e4:a6:71:9e:a1:e7:30:40:e5:25:61:93:
    f4:4e:e3:f4:df:cd:e0:7c:b8:f1:dc:8c:db:95:86:
    dd:b6:63
""".replace("\n", "").replace(":", "").replace(" ",""), 16) #On enlève les retours à la ligne et les doubles points en rapprochant les charactères
```

Là attention, si vous ne vous y connaissez pas suffisament avec rsa, je vous conseille de regarder d'abord cette vidéo afin de bien comprendre ce qui va suivre: [ici](https://youtu.be/MuNyEoU5tSo?t=209).

Pour trouver la clé privée à partir de la clé publique, il nous faut l'exposant privé d

Voici le processus de calcul:
```
factor(N) = p, q        #Je factorise N se qui me donne deux nombres premiers p et q
phi(N) = (p-1) * (q-1)  #phi(N) correspond à l'indicatrice d'Euler
d = invmod(e, phiN)     #la fonction invmod correspond à l'inverse de e modulo phiN
```

Nous avons donc:
```
N = 7533234967451425113839122328357652882257791636101415254442914553640192738965091
p = 1984353979129643887715281539913891079263
q = 3796316104224293711606659339243752901757 
e = 48653
phiN = 7533234967451425113839122328357652882252010966018061316843592612761035094984072
d = 4008239031558486463773114909958164145332411318465655834360267238742624617266205
c = 301738122221323616019412722717929172721914151471822201851502202231881120024724529230235123614411110415491111221911861901748454624025574252150233218135248231188138192154219179149602321264715862173154200122156156159882613824250221201189168225432487417020115318523220215712110665902371081212541692552162041605189132078127190943954118710411691571071701382341361101215314516141120921515418615251601219381157244417321946579951581395
```

Pour déchiffrer le message, je vais utiliser la fonction pow() qui revient à faire c**d % N ainsi que la fonction long_to_bytes() de pycryptodome qui permet de convertir directement de décimal à ascii string (c'est à dire le texte en clair). Si vous n'arrivez pas à installer le module pycryptodome, vous pouvez le faire manuellement ici (décimal -> hexa puis hexa -> ascii string)
```py
from Crypto.Util.number import long_to_bytes

long_to_bytes(pow(c, d, N))

>> b'BC{sm4ll_M0dulU5_15_B4d}'
```

Et voilà, le chall est flag !


https://www.dcode.fr/inverse-modulaire
https://coding.tools/