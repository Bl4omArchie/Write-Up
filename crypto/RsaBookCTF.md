# Deuxième édition bookCTF 2020 - RSA#1 et RSA#2

## Introduction
Voici les write-up des deux challanges RSA du bookCTF. Sera expliqué comme résoudre le challenge avec les outils nécessaires et des exemples de codes en python.

## RSA#1
Nous disposons de deux fichiers, l'un au format pem contenant une clé publique et privée et un autre fichier binaire qui contient le ciphertext.
Pour déchiffrer un message avec RSA, il faut trouver la clé privée mais comme elle nous l'est déjà donné, une simple commande openSSL suffit.

Ce challenge n'est donc pas bien compliqué à condition de connaître la commande openSSL nécessaire !

Exécuter ceci et tout se fera automatiquement:
```
openssl rsautl -decrypt -inkey <fichiers avec les clés> -in <ciphertext> -out <fichier où sera stocké le message>

openssl rsautl -decrypt -inkey private.pem -in flag.txt.enc -out flag.txt

>> BC{s1mpl3_Rs4_1nTr0dUct10n}

```

Et voilà la premier chall rsa est flag ! 



## RSA#2:
Dans ce challenge, nous possédons un fichier pem contenant la clé publique et un fichier binaire avec le ciphertext.
Contrairement au challenge précédent, nous n'avons pas la clé privée et l'objectif et de la trouver ! 
Si vous ne connaissez pas la base du fonctionnement de RSA, je vous conseille cette vidéo qui vous permettra de le comprendre: [ici](https://youtu.be/MuNyEoU5tSo?t=209).

La première chose que j'ai faite est d'extraire la clé publique avec openSSL:
![alt text](https://github.com/Bl4omArchie/Write-Up/blob/main/crypto/images/rsa%232.PNG)

Je possède donc N et e, je convertis alors N en décimal grâce à ce script python:

```py
N = int("""41:0e:eb:e4:a6:71:9e:a1:e7:30:40:e5:25:61:93:
    f4:4e:e3:f4:df:cd:e0:7c:b8:f1:dc:8c:db:95:86:
    dd:b6:63
""".replace("\n", "").replace(":", "").replace(" ",""), 16) #On enlève les retours à la ligne et les doubles points en rapprochant les charactères
```

J'ai donc ensuite essayer de factoriser N grâce à ce [site](http://factordb.com/)

Ce qui s'est avéré payant: 
```
N = 7533234967451425113839122328357652882257791636101415254442914553640192738965091
p = 1984353979129643887715281539913891079263
q = 3796316104224293711606659339243752901757 
e = 48653
```

J'ai donc réussi à réupérer p et q ce qui me permit de continuer mes calculs:
```
phiN = 7533234967451425113839122328357652882252010966018061316843592612761035094984072
d = 4008239031558486463773114909958164145332411318465655834360267238742624617266205
```

Pour calculer phi(N), j'ai fais: (p-1)*(q-1) et pour trouver d j'ai fais l'invmod de e et phi(N) grâce à ce [site](https://www.dcode.fr/inverse-modulaire)

J'ai ensuite extrait et convertis en décimal le ciphertext grâce à script python:
``` py
with open("flag.txt.enc", "rb") as fp:  #J'ouvre mon fichier en mode "rb" qui permet d'ouvrir les fichiers binaires et j'y écris récupère le contenu
    r = fp.read()
    r = int(r.hex(), 16)     

>> 5461468309655872738700035603457241238681569801180183582502634691203218427997766
```

Pour déchiffrer le message, j'ai utilisé la fonction pow() qui revient à faire c**d % N ainsi que la fonction long_to_bytes() de pycryptodome qui permet de convertir directement de décimal à ascii string (c'est à dire le texte en clair). Si vous n'arrivez pas à installer le module pycryptodome, vous pouvez le faire manuellement [ici](
https://coding.tools/) (décimal -> hexa puis hexa -> ascii string)
```py
from Crypto.Util.number import long_to_bytes

long_to_bytes(pow(c, d, N))

>> b'BC{sm4ll_M0dulU5_15_B4d}'
```

Et voilà, le chall est flag !


