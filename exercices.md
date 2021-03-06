# Liste des exercices proposés

### reader-writer : Gestion de readers/writers

*RESERVATION* : <1 groupe de max. 2 personnes>

Vous développez un système de cache générique qui permet d'associer à un identifiant entier unique un pointeur vers un objet quelconque en mémoire. Vous souhaitez que votre cache soit thread-safe. Cependant, vu la structure de votre programme, la lecture ou l'écriture d'un élément dans le cache peut se faire via différentes fonctions différentes. Vous souhaitez alors coder 2 fonctions, `readerAllow` et `writerAllow` qui permettront aux fonctions de lecture et d'écriture de s'exécuter de manière thread-safe. Vous souhaitez que cette gestion thread-safe aie les propriétés suivantes :

- Il ne peut y avoir qu'un seul writer qui accède à la base de données à tout moment
- Un writer doit toujours avoir la priorité sur un reader s'ils tentent simultanément d'accéder au cache

Déclarez les variables globales qui seront utilisées par votre programme. Vous avez droit aux fonctions des headers pthread.h et semaphore.h.

Écrivez une fonction d'initialisation `void initRW()` qui initialisera les structures globales que vous avez déclarées précédemment. Cette fonction sera lancée en même temps que l'initialisation du cache.

Écrivez une fonction `void *readerAllow(int key, void *(*reader)(int key))` qui exécutera de manière thread-safe la fonction `reader` avec la clé passée en argument à `readerAllow` et renverra le résultat fourni par `reader`.

Écrivez une fonction `void writerAllow(int key, int value, void *(*writer)(int key, int value))` qui exécutera de manière thread-safe la fonction `writer` avec la clé et la valeur passées en argument à `writerAllow`.

*NOTES IMPORTANTES* : Ici, la solution de l'exercice en elle-même est déjà toute faite, il s'agit du code du writer/reader fournie par le cours (http://sites.uclouvain.be/SystInfo/notes/Theorie/html/Threads/coordination.html#probleme-des-readers-writers) avec priorité aux writers. Vous devrez juste remplacer les sections critiques par un appel aux fonctions passées comme pointeurs en argument.

La difficulté de l'exercice réside dans l'écriture des tests. Vous devez réfléchir à des tests qui permettent de s'assurer qu'un writer est toujours exécuté seul, et que les writers ont priorité sur les readers. Le fait que les fonctions `readerAllow` et `writerAllow` utilisent des pointeurs de fonction n'est pas anodin : vous pouvez par exemple faire un writer bloquant à l'aide d'un mutex, puis de lancer un reader et vous assurer que le reader n'est pas exécuté tant que vous n'avez pas déverouillé le mutex du writer. Soyez créatifs dans vos tests et faites-en sorte que ceux-ci vérifient bien tous les cas limites du problème du reader/writer.

### my-sem : Écriture de sémaphores

*RESERVATION* : <1 groupe de max. 2 personnes>

On souhaite écrire notre propre type de sémaphore à l'aide de mutex. On déclare pour ce faire les 2 structures suivantes :

```
typedef struct semProcess {
    pthread_mutex_t mutex;
    struct semProcess next;
} sem_process_t;

typedef struct mySem {
    int value;
    int capacity;
    sem_process_t *head;
} mysem_t;

```

Chaque sémaphore contient une valeur et une liste de processus bloqués. 

Écrivez une fonction `int mysem_init(mysem_t *sem, unsigned int value);` qui assigne à `value` et `capacity` `sem` la valeur de l'argument `value` et initialise à `NULL` la liste des processus bloqués.

Écrivez une fonction `int mysem_wait(mysem_t *sem);` qui bloque le fil d'exécution si `value` de `sem` vaut 0 et ajoute le processus à la **fin** de la liste des processus bloqués. Si `value` est plus grand que 0, il est décrémenté.

Écrivez une fonction `int mysem_post(mysem_t *sem);` qui incrémente `value` de `sem` si aucun autre processus n'est bloqué, et sinon débloque le premier processus de la liste des processus bloqués. `value` ne peut jamais excéder `capacity`.

Écrivez une fonction `mysem_close(mysem_t *sem);` qui libère toutes les ressourcées associées à un sémaphore. Si ce sémaphore a des processus bloqués, il les libère.

*Tests imposés :* Vous devez tester les 4 fonctions. Il est important de tester le flux d'exécution, s'assurer que `sem_wait` est bien bloquant quand `value` vaut 0 et qu'il ajoute le processus bloqué à la fin de la liste, que `sem_post` va bien chercher le premier élément bloqué de la liste, que `value` n'excède jamais `capacity`, que `mysem_init` initialise proprement la sémaphore avec les bonnes valeurs et que `mysem_close` libère toutes les ressources correctement. 
Pour la vérification de la libération des ressources, utilisez le mécanisme de LD_PRELOAD pour s'assurer que le nombre d'appels à `free` correspond bien au nombre de processus bloqués dans la liste (et que le mutex de chacun de ces processus bloqués est bien déverrouillé !), de même, testez aussi le cas où vous faites échouer `malloc`.

### pgcd : Plus Grand Commun Diviseur

*RESERVATION* : kevin STOFFLER 66271200

On cherche à calculer le plus grand commun diviseur de deux très grands nombres. Pour ce faire, on calcule tous les diviseurs de chacun des 2 nombres, et on regarde quel est leur PGCD. Pour ce faire, on déclare la liste chaînée suivante, permettant d'enregistrer en mémoire les diviseurs d'un nombre :

```
struct Node {
    unsigned int divisor;
    struct Node *next;
};
```

Écrivez une fonction `struct Node *factorize(unsigned int n)` qui retourne la liste chaînée contenant tous les diviseurs du nombre `n` dans l'ordre décroissant.

Écrivez une fonction `unsigned int gcd(unsigned int a, unsigned int b)` qui va lancer l'exécution de `factorize` pour `a` et `b` dans 2 threads différents et va extraire des deux listes renvoyées le PGCD. Le nombre 1 est considéré comme un diviseur. Cette fonction renvoie 0 si une erreur s'est produite.

*Tests imposés :* Il y a quelques points auxquels il faut faire attention en plus de tester les valeurs renvoyées par `gcd`.
- Écrire des tests pour `factorize`
- Utiliser le mécanisme de LD_PRELOAD pour compter le nombre d'appels à `pthread_create` ou `pthread_join` et s'assurer que 2 threads ont bien été créés.
- Utiliser le mécanisme de LD_PRELOAD pour s'assurer que le nombre d'appels à malloc est égal au nombre d'appels à free et est égal à la somme des diviseurs des 2 nombres (soit la somme des tailles des listes chainées).
- Egalement à l'aide du mécanisme de LD_PRELOAD, faire échouer malloc ou `pthread_create` et s'assurer que `gcd` retourne alors 0.

### insertion-sort : Tri par insertion

*RESERVATION* : Devillez Henri, Siciliano Damiano-Joseph

On désire implémenter un algorithme de tri par insertion sur un tableau de N entiers, le tableau et sa taille étant passés en argument.

L'algorithme de tri est le suivant :
Pour chaque élément d'indice `i` (`i` variant de 1 à N-1)
* cet élément devient la clé
* on la compare avec l'élément d'indice `i-1`
* si la clé est plus petite, on les échange et on recommence la comparaison avec l'élément précédent (d'indice `i-2`) et ainsi de suite, tant que la clé est plus petite que l'élément qui lui précède (ou qu'on est revenu au début du tableau)
* quand la clé est à sa place (c'est-à-dire qu'elle est plus grande que ou égale à l'élément qui lui précède), la boucle intérieure est finie et on passe à l'élément d'indice `i+1`.

Écrivez le corps de la fonction `void tri(int T[], int taille);`.

_Tests imposés :_ Aucun.

### matrix-mult : Multiplication de matrices

*RESERVATION* : Adrien BALLET 20971400, Sébastien MOTTET 18521400

Écrivez une fonction `int access(int *A, int taille, int ordonnee, int abscisse);` qui renvoie l'élément d'abscisse et d'ordonnée indiquées dans une matrice carrée `taille x taille` d'entiers (il s'agit donc de l'élément A\[ordonnee][abscisse]).

Écrivez une fonction qui effectue la multiplication de deux matrices carrées `taille x taille` d'entiers. Le prototype de la fonction est la suivante : `void mult(int *A, int *B, int *C, int taille)` et elle calcule `C = A x B`. Les trois matrices sont stockées sous forme de tableau ligne par ligne et la place de la matrice C est déjà allouée. 

_Tests imposés :_ Aucun.

### poly : Polynomes

*RESERVATION* : Maxime Dimidschstein 62641400, AIGRET Julien 83431300

On souhaite gérer des polynômes réels de degré inférieur ou égal à 10 en utilisant la structure suivante :

```typedef struct {
    double coeff[10];
} poly;```

qui servira à représenter le polynôme où coeff[0] est le coefficient du terme indépendant, coeff[1] le coefficient du terme en x, etc.

Écrivez une fonction `double eval(poly *P, double x)` qui calcule la valeur du polynôme P au point x. On pourra se servir de la formule de Horner : `P(x) = ((...(a_9*x + a_8)*x + a_7)*x + ...)x + a_1)*x + a_0` où `a_i` est `coeff[i]`.

Écrivez une fonction `poly *derivee(poly *P)` qui renvoie le polynôme exprimant la dérivée du polynôme P.

Écrivez une fonction `double racine(poly *P, double x0)` qui calcule via la méthode de Newton la racine du polynôme P.
La méthode est la suivante : on part d'un point initial, et on construit une suite de points qui se rapprochent de la racine en calculant à chaque étape un nouveau point à partir de la valeur du polynôme et de sa dérivée : `x_n+1 = x_n  - P(x_n)/P'(x_n)`. La fonction s'arrête lorsque `|P(x_n)| < 0.0001`. On suppose que le calcul converge toujours vers une racine.

_Tests imposés_ : Les tests à réaliser ici sont intuitifs, il faut toujours s'assurer que les valeurs renvoyées correspondent bien à la solution mathématique demandée. Vu les problèmes de précision des calculs en virgule flottante, vous devez tolérer les résultats avec un certain intervalle d'erreur. Fournissez au moins 3 tests par fonction, et au moins un des 9 tests doit aller jusqu'au degré 9.

### bookstore : Gestion d'une librairie

*RESERVATION* : Thomas Wirtgen 1331-1400 & Florent Mabille 4456-1400, 

On souhaite gérer le catalogue d'une librairie dans lequel chaque livre est identifié par son auteur et son titre. La structure de données choisie est la suivante : il y a une liste chaînée d'auteurs dont chaque élément pointe vers une liste chaînée d'ouvrages.

```
typedef struct cellAuteur {
    char *auteur;
    struct cellLivre *Livres;
    struct cellAuteur *next;
} cellAuteur;

typedef struct cellLivre {
    char *titre;
    struct cellLivre *suiv;
} cellLivre;
```

Écrivez une fonction `cellAuteur *existe(cellAuteur *librairie, char *strAuteur)` qui teste si un auteur existe dans la liste `librairie` et dans ce cas renvoie un pointeur sur sa cellule de la liste (et `NULL` sinon).

Écrivez une fonction `int compteOuvrage(cellAuteur *librairie, char *strAuteur)` qui compte le nombre de livres d'un auteur dans la liste `librairie`.

Écrivez une fonction `void add(cellAuteur *librairie, char *strAuteur, char *strTitre)` qui ajoute dans le catalogue un livre de l'auteur indiqué. L'auteur existe dans le catalogue.

Écrivez une fonction `void supprimer(cellAuteur *librairie, char *strAuteur)` qui supprime du catalogue un auteur et tous ses livres. L'auteur existe dans le catalogue.

_Tests imposés_ : Outre les tests classiques, assurez-vous que la fonction `add` ne segfault pas lorsque vous lui indiquez un auteur dont la liste de livres est vide. Utilisez les fonctions `signal`, `sigsetjmp`et `siglongjmp` pour attraper un éventuel segfault dans `add` et pouvoir indiquer à l'utilisateur l'origine précise de ce segfault (il ne prend pas en compte le fait que la liste d'articles d'un auteur puisse être vide). Réécrivez également la fonction `free` pour vous assurer que l'étudiant utilise `supprimer` correctement : comptez le nombre d'appels à `free` effectués et assurez-vous que ce nombre correspond au nombre de structures qui ont dû être désallouées (1 pour l'auteur + le nombre de livres qu'il avait).

### btree-access : Parcours d'un arbre binaire de recherche

*RESERVATION* : Bertrand Van Ouytsel Charles-Henry 15221400, William Salame 50691400

On souhaite parcourir un arbre binaire de recherche. Un arbre de recherche binaire est une structure de données où chaque nœud possède une clé et une valeur. En outre, chaque nœud peut avoir 2 nœuds fils : un à gauche dont la clé est toujours inférieure à la sienne, et un à droite dont la clé est toujours supérieure à la sienne. Autrement dit si vous êtes à un nœud dont la clé vaut 10 et que vous cherchez un nœud dont la clé vaut 5, vous savez que vous devez descendre à gauche pour espérer trouver un éventuel nœud dont la clé vaut 5.

```
typedef struct BSTreeNode {
    int key;
    int value;

    struct BSTreeNode *left;
    struct BSTreeNode *right;
} Node;
```

![Arbre binaire](https://upload.wikimedia.org/wikipedia/commons/thumb/f/f7/Binary_tree.svg/192px-Binary_tree.svg.png)

Écrivez une fonction `int *has_key(Node root, int key)` qui vérifie si l'arbre binaire dont le sommet est le nœud root possède un nœud dont la clé vaut `key` et renvoie alors un pointeur vers `value` de ce nœud, renvoie `NULL` sinon.

Écrivez une fonction `int compare(Node root_a, Node root_b)` qui vérifie si les 2 arbres binaires passés en argument sont identiques (ils ont la même structure et tous les nœuds possèdent la même paire clé/valeur). Renvoie 1 si les arbres sont identiques, 0 sinon. Indice : pensez récursivement.

_Tests imposés_ : Les tests pour `has_key' devraient être assez intuitifs. Pour `compare`, n'hésitez pas à créer 2 arbres identiques mais différents du point de vue de la mémoire. Créer un seul arbre `b1`et s'attendre à ce que `compare(b1, b1)` renvoie 1 n'est pas suffisant ! L'étudiant ne doit pas comparer les pointeurs de 2 noeuds pour vérifier s'ils sont identiques ou non.

### hexadecimal : Conversion d'un unsigned int vers hexadécimal et vice-versa

*RESERVATION* : Sébastien Strebelle 8143-1300, Simon HICK 8853-1200

On souhaite convertir un entier non signé vers sa représentation hexadécimale. Par exemple l'entier 42 vaut "2A" en hexadécimal. De même, on souhaite faire la conversion dans l'autre sens.

Écrivez une fonction `unsigned int hex_to_int(char *hex)` qui prend en argument une chaîne de caractères représentant un nombre hexadécimal (cette chaîne ne peut comporter que les chiffres de 0 à 9 et les lettres A à F).

Écrivez une fonction `char *int_to_hex(unsigned int value, char *dest)` qui prend en argument un entier non-signé et enregistre sa représentation hexadécimale dans la chaîne de caractères indiquée par `dest`. On suppose que `dest` est un tableau de 9 char au minimum. La fonction devra toujours renvoyer le pointeur `dest`.

### bitwise-ops : Opérations sur les bits

*RESERVATION* : Robin HORMAUX 5666-1300, Maxime WATTIAUX 6558-1400

On souhaite effectuer des opérations spécifiques sur certains bits d'un entier non-signé.

Écrivez une fonction `char get_bit(unsigned int char, int pos)` qui renvoie le bit à la position `pos` de `x`.

Écrivez une fonction `unsigned char set_bit(unsigned int char, int pos, char value)` qui met le bit à la position `pos` de `x` à la valeur `value` (qui ne pourra jamais être que 0 ou 1).

Écrivez une fonction `unsigned char get_3_leftmost_bit(unsigned char x)` qui renvoie les 3 bits les plus à gauches de `x`. Par exemple, si on a la séquence 11011001, la fonction doit renvoyer la valeur correspondant à 00000110.

Écrivez une fonction `unsigned char get_4_rightmost_bit(unsigned char x)` qui renvoie les 4 bits les plus à droite de `x`. 

Écrivez une fonction `unsigned char unset_first_bit(unsigned char x)` qui met à 0 le premier bit à 1 qu'il trouve en partant de la gauche, et ne fait rien s'il n'y a pas de bit mis à 1.

Écrivez une fonction `unsigned char cycle_bits(unsigned char x, int n)` qui déplace tous les bits de `n` places vers la gauche selon la formule `x[(i+n)%8] = x[i]` où `x[i]` représente le ième bit de x.

_Tests imposés_ : Aucun.

### index-text : Indexation d'un texte

*RESERVATION* : Thomas Beznik & Vianney Coppé

On souhaite indexer un texte afin de savoir quels mots reviennent le plus fréquemment dans un corpus. Le processus d'indexation se fait en 2 phases : on compte d'abord le nombre d'occurrences de chaque mot, et on supprime ensuite de la table d'indexation tous les mots qui n'ont pas été indexés au moins `N` fois. Le corpus est une chaîne de caractères composées uniquement de minuscules et où le seul délimiteur est un espace (pas de ponctuation).

On définit la structure suivante représentant une entrée de l'index :
```
typedef struct indexEntry {
    char word[26];
    int count; //nombre de fois qu'un mot est apparu dans le corpus
    struct indexEntry *next;
} Entry;
```

Écrivez une fonction `Entry *build_index(char *corpus)` qui renvoie l'index associé au corpus passé en paramètre. Vous pouvez modifier la chaine passée en argument.

Écrivez une fonction `void filter_index(Entry *index_head, int treshold)` qui supprime de l'index tous les mots qui n'ont pas été recensés au moins `treshold` fois.

_Tests imposés_ : Réécrivez la fonction `malloc` pour vérifier que l'étudiant gère le cas où `malloc` renvoie NULL, pensez à intercepter un éventuel segfault lorsque vous faites "bugger" `malloc`. Réécrivez la fonction `free` afin de compter le nombre d'appels à `free` effectués pour s'assurer que l'étudiant libère bien de la mémoire les entrées de l'index qui sont éliminées par le filtre. Assurez-vous que l'étudiant ne fait pas de buffer-overflow, c'est-à-dire qu'il ne dépasse pas par la droite la chaîne "corpus" : écrivez dans un test unitaire le corpus à la fin d'une page mémoire et mettez la page mémoire suivante en PROT_NONE, et pensez à intercepter un éventuel segfault.

### rpn-calc : Calculatrice en notation polonaise inversée


*RESERVATION* : Mathias NOVAK 41501400, Margerie Huet

La notation polonaise inversée permet d'écrire de façon non-ambigüe sans parenthèses des formules arithmétiques. Par exemple, le calcul `((1 + 2) × 4) + 3` peut être noté `1 2 + 4 * 3 +` en notation polonaise inverse, ou encore `3 4 1 2 + * +`. L'avantage de cette notation est qu'elle est très facilement compréhensible par un ordinateur : on imagine une pile où on peut soit ajouter un élément sur la pile, soit retirer le dernier élément ajouté. En parcourant la formule arithmétique, si on rencontre un nombre, on l'ajoute à la pile, si on rencontre une opérande (par ex. le symbole '+'), on retire les 2 derniers éléments de la pile, on en fait la somme et on ajoute le résultat à la pile.

Pour ce problème, vous ne pourrez utiliser que la variable globale `double stack[STACK_SIZE]`, représentant la pile, et `int stack_height`, représentant la hauteur actuelle de la pile, qui seront déjà initialisées à 0 et accessibles par vos fonctions.
Vous pouvez supposer que les exemples utilisés par les tests feront en sorte que le nombre d'éléments actuels dans la pile ne dépassera jamais `STACK_SIZE`.

Écrivez une fonction `void push(double value)` qui permet d'ajouter l'élément `value` à la pile.

Écrivez une fonction `double pop(void)` qui enlève et retourne l'élément au sommet de la pile.

Écrivez une fonction `double rpn(char *expr)` qui calcule l'expression en notation polonaise inverse contenue dans `expr` et retourne le résultat. Vous pouvez supposer que `expr` contiendra toujours une expression correcte où il ne restera jamais qu'un seul élément sur la pile à la fin de l'exécution. Indice : utilisez la fonction `strtok(3)` pour séparer les différents éléments de la chaîne et la fonction `atof(3)` pour convertir l'éventuel nombre rencontré en double. Exemple : "4 2 5 * + 1 3 2 * + /" est censé renvoyer 2. Les opérandes possibles sont + (addition), - (soustraction), * (multiplication) et / (division).

_Tests imposés_ : Faites bien attention à ne tester que des expressions valides. Testez bien les différentes opérandes et réalisez également des tests uniquement pour les fonctions `pop` et `push`.

