# SSE - Passive SSH Key Compromise via Lattices
## Authors
Ferrara Justin

## Notes
- get les clés de signature RSA si une faute dans la signature
- de base pas possible pour SSH car D-H
- Implem avec théorème des restes chinois non protégé
- Besoins d'une signature fausse, public key, calc un GCD
- RSA PKCS v1.5
- TLS 1.2 handshake avec signature RSA en clair -> écoute passive possible
- TLS 1.3 handshake chiffré après le D-H -> écoute active nécessaire
- Possible même si partie du message inconnu

### SSH
- SSH handshake pour décider de l'algo et ensuite D-H, auth faite avec signature du session identifier
- Signature avec RSA, DSA, ECDSA, EdDSA, ...
- Si trouve la clé privée, peut faire un man in the middle mais pas déchiffrer le traffic ???
- Signe un message qui est le hash du session identifier qui contient le D-H, id client et id serveur
- On connait pas le D-H mais uniquement la signature
- Peut ensuite get le password en faisant une attaque active
- Password authentication / Public key authentication (MitM pas possible ???)
- SSH agent forwarding ???
- Ne voit pas le message car utilise D-H pour le chiffrer

### IPsec
- Internet Key Exchange (IKE) cipher suite, key exchange, authentification, ...
- IKEv1 et IKEv2
- ... ???

### PKCS v1.5 padding pour signature RSA
Le padding PKCS v1.5 est donné par :

$$ 00 || 01 || FF ... FF || ASN.1 || Hash(m) $$

Où :
- `ASN.1` est un identifiant pour définir la fonction de hashage utilisée
- `Hash(m)` est le hash du message `m`

### RSA Signatures avec padding PKCS v1.5

La signature `s` RSA d'un message `m` sans padding est donnée par :

$$ N = pq $$
$$ \phi(N) = (p-1)(q-1) $$
$$ d = e^{-1} \mod \phi(N) $$
$$ s = f(m)^d \mod N $$

La clé privée est donnée par :

$$ (N, d) $$

La clé publique est donnée par :

$$ (N, e) $$

La vérification de cette signature est donnée par :

$$ f(m') = s^e \mod N $$
$$ f(m) = f(m') $$

### Théorème des restes chinois
Le théorème des restes chinois et ici nécessaire car nous savons qu'il y a une erreur de calcul dans la signature.

Nous pouvons donc poser les équations suivantes pour définir l'erreur de calcul :

$$ x \mod p = x' mod p $$
$$ x \mod q \neq x' mod q $$

Avec `x` la signature correcte et `x'` la signature incorrecte par exemple.

On peut donc déduire que :

$$ (x - x') \mod p = 0 $$
$$ (x - x') \mod q \neq 0 $$
$$ (x - x') \mod n = k \cdot p \mod n \quad \text{avec} \quad k \in \mathbb{Z} $$


Avec ces équations, nous pouvons donc déduire `p` de la manière suivante :

$$ p = \gcd(N, s'-s) $$
$$ p = \gcd(N, m'-m) $$
$$ p = \gcd(N, s'^e - m) $$
$$ q = N / p $$

Avec s' la signature incorrecte, s la signature correcte, m' le message incorrect et m le message correct. Ce cas fonctionne si nous possèdons donc une signature correcte et une signature incorrecte du même message ce qui n'est pas le cas ici.


### PACD (Partial Approximate Common Divisors)
PACD est une généralisation du PGDC :

$$ N_i = p \cdot q_i $$

Avec `p` le facteur commun et `q_i` un diviseur commun approximatif. ???

Pour pouvoir résoudre le problème PACD, il faut que les diviseurs communs approximatifs soient suffisamment proches pour que l'algorithme LLL puisse les trouver. Nous pouvons donc essayer de poser les équations suivantes :

$$ N_0 = p \cdot q_0 $$
$$ N_1 = p \cdot q_1 + r_1 $$

Avec `r_1` une erreur de calcul :

$$ |r_1| < 2^{log_2(r)} $$

Avec `r` l'espace de la fonction de hashage. ???

Comme nous n'avons pas `N1`, nous pouvons essayer de le trouver en utilisant le théorème des restes chinois :

$$ (m' - m) \mod N = k \cdot p$$
$$ (h(m') - h(m)) \mod N = k \cdot p$$

Nous pouvons donc déduire les équations suivantes :

$$ N_0 = p \cdot q_0 $$
$$ N_1 = (s'^e - h(m)) \mod N = k \cdot p + h $$

Avec `h` une erreur de calcul. ???

### Lattice
Nous pouvons poser cette fonction qui à une petite racine mod p en `x = r_1`

$$ f(x) = N_1 - x $$

On peut ensuite en dériver des équations en utilisant la formule suivante :

$$ Q_j(x) = N^{\max(k - j, 0)} f(x)^{\min(j, k)} x^{\max(j - k, 0)} $$
$$ 0 \leq j \leq k $$

Avec `(t, k) = (2, 1)` pour notre cas. ??? pourquoi dériver des équations ???

Nous pouvons ensuite remplir la matrice suivante :

$$
B =
\begin{bmatrix}
-2^{2 \log r} & 2^{\log r} N_1 & 0 \\
0 & -2^{\log r} N_1 & 0 \\
0 & 0 & N_0
\end{bmatrix}
$$

Avec `r` l'espace de la fonction de hashage.

En utilisant l'algorithme LLL, nous pouvons ensuite poser : ???

$$ g(y) = 0 $$
$$ y = r_1 $$

Avec `r_1` une petite racine mod p.

Comme nous connaissons `r_1`, nous pouvons donc déduire `p`:

$$ p = \gcd(N_0, N_1 - r_1) $$


!!!!! fonctionne uniquement si log r < log N / 4

## Bibliography

**Keegan Ryan, Kaiwen He, George Arnold Sullivan, Nadia Heninger**. *Passive SSH Key Compromise via Lattices*. Cryptology ePrint Archive, Paper 2023/1711, 2023. [DOI: 10.1145/3576915.3616629](https://doi.org/10.1145/3576915.3616629), [URL: https://eprint.iacr.org/2023/1711](https://eprint.iacr.org/2023/1711).


------------------------------------------------------------------------------------------------------------------

## 1.1. Explain why is the vector (b1, b2, . . . , bm, Bα/p, B) is a linear combination of the rows of the matrix M.

On peut voir que la matrice M ressemble à ceci:

$$
M = \begin{pmatrix}
    p & 0 & \cdots & & & \\
    0 & p & \cdots & & & \\
    & & \ddots & & & \\
    & & & p & & \\
    t_1 & t_2 & \cdots & t_m & \frac{B}{p} & 0 \\
    a_1 & a_2 & \cdots & a_m & 0 & B \\
\end{pmatrix}
$$

En sachant que:

$$ t_i * \alpha - a_i \bmod p = b_i $$

On remarque que si on multiplie la dernière ligne de la matrice par \*-1 on obtient:

$$ { -a_1, -a_2, \dots, -a_m, 0, -B } $$

Et que si on multiplie l'avant dernière ligne par \*alpha on obtient:

$$ { t_1*\alpha, t_2*\alpha, \dots, t_m*\alpha, \frac{B*\alpha}{p}, 0 } $$

En faisant une combinaison linéaire des deux lignes, on obtient la ligne suivante:

$$ (b_1, b_2,  \dots, b_m, \frac{- B\alpha}{p}, B) $$


## 1.2. Explain why the vector (b1, b2, . . . , bm, Bα/p, B) is small compared to p.

Car il est défini dans la consinge que:

$$ B < p $$
$$ b_i < B $$

On peut également noter que B doit être beacoup plus petit que p, sinon l'algorithme LLL serait trop lent.

On peut donc en déduire que ceci sera toujours plus petit que p:

$$ t_i * \alpha - a_i \bmod p = b_i $$

avec b_i < B qui sera donc toujours plus petit que p.

On peut également noter que alpha est de toute façon plus petit que p car c'est la clé privée qui est modulo p.

On peut donc en déduire que le vecteur
$$ (b_1, b_2,  \dots, b_m, \frac{-B\alpha}{p}, B) $$
 est petit par rapport à p.

## 1.3. Explain what the LLL algorithm does.

L'algorithme LLL prend comme entrée une base de vecteurs. Il va ensuite essayer de trouver une base plus courte et plus proche de l'orthogonalité que la base d'entrée:
- appliquer la réduction de Gram-Schmidt pour décomposer chaque vecteur
- modifier un peu les vecteurs en les ajustant, en remplacant certains vecteurs par des combinaisons linéaires des autres vecteurs
- vérifie la condition de Lovász pour chaque vecteur

À la fin de l'algorithme, on obtient une base plus courte et plus proche de l'orthogonalité que la base d'entrée. Si la réduction a fonctionné, on peut trouver un vecteur qui a comme coefficients les valeurs suivantes dont la clé privée a:

$$ (b_1, b_2,  \dots, b_m, \frac{-B\alpha}{p}, B) $$

À noter que l'algorithme LLL va retourner un vecteur de vecteurs avec pour chaque vecteur une possibilité de réduction de la matrice de base. On peut ensuite parcourir toutes ces réductions possible de la matrice pour trouver la clé privée a dans un des vecteurs.