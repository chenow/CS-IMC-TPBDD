# Travaux Pratiques Bases de Données Relationnelles et Graphes
Dans ce TP, nous allons travailler sur des jeux de données publics issus des [datasets IMDB](https://datasets.imdbws.com/)). Ces données sont disponibles dès le démarrage du TP dans une base de données relationelle [Azure SQL Database](https://docs.microsoft.com/fr-fr/azure/azure-sql/database/sql-database-paas-overview).

Les objectifs du TP sont les suivants:

1. Se familiariser avec les différences de paradigme entre le requêtage relationnel et graphe
2. Transformer les données pour les exporter depuis la base de données relationelle SQL, vers une base de données graphe Neo4j
3. Implémenter un ensemble de requêtes:
    - en SQL sur la base de donnée relationelle [Azure SQL Database](https://docs.microsoft.com/fr-fr/azure/azure-sql/database/sql-database-paas-overview)
    - en [Cypher](https://neo4j.com/developer/cypher/) sur la base de données graphe [Neo4j](https://neo4j.com/)
    - en [Apache Gremlin](https://tinkerpop.apache.org/docs/3.3.2/reference/#graph-traversal-steps) sur une base de données graphe répartie [Cosmos Db](https://docs.microsoft.com/en-us/azure/cosmos-db/graph/graph-introduction)
4. (optionnel) Encapsuler ces requêtes dans des API serverless (Azure Functions) et mesurer les performances

## Prérequis
### GitHub Codespaces et récupération du code
L'installation des prérequis à ce TP étant fastidieuse, nous vous avons préconfiguré un [GitHub Codespace](https://github.com/features/codespaces) pour lequel un compte GitHub est nécessaire. Notons que les étudiants bénéficient de certains avantages, dont des heures Codespace supplémenaires, via le programme [GitHub Student Developer Pack](https://education.github.com/pack).

1. Effectuez un [fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo) privé de ce repository, fork sur lequel vous travaillerez pour toute la durée du TP.
   
   ![image](https://github.com/user-attachments/assets/cb2263fe-19f9-4005-98fe-fcad811a0d5e)

3. Assurez-vous d'être dans votre fork en vérifiant l'URL du navigateur.

4. Créez votre Codespace en cliquant sur *Create a codespace on main*. Choisissez la plus petite machine possible: le TP ne demande pas beaucoup de ressource car les tâches de calcul sont surtout effectuées par les bases de données auquelles vous vous connecterez. Si vous utilisez Firefox, désactivez l'*Enhanced Tracking Protection* sur l'URL de votre Codespace sans quoi vous ne pourrez vous y connecter.

![image](https://github.com/user-attachments/assets/337199dd-4223-4326-9e7d-4180c0792082)

### Création d'une base graphe Neo4j
La partie 2 du TP requiert la création d'une base de données Neo4j en mode "bac à sable" (Sandbox). Pour cela:
1. Créez une base de données  [Neo4j Sandbox](https://neo4j.com/sandbox/). Utilisez les informations que vous souhaitez pour la création de compte.
2. Parmi les templates, créez une base vierge (*Blank sandbox*)
3. Dans le portail Neo4j (un lien vous a été envoyé par email), dépliez la ligne correspondant à votre sandbox puis allez dans l'onglet **Connection details** pour noter ces informations de connexion
![image](https://user-images.githubusercontent.com/22498922/147907013-ae0f0d32-7982-464b-969a-576646407c9c.png)
4. ⚠️ Modifiez votre fichier `.env` les variables d'environnement nécessaires à la connexion à votre base Neo4j. Notez que les bases sandbox sont automatiquement détruites au bout de 2 jours mais vous pouvez la prolonger ou la reconstruire très facilement une fois la Partie 3 du TP réalisée.

### Mise à jour du fichier de configuration (.env)
1. Téléchargez le fichier fourni par l'enseignant `.env`:
    ```
    wget <URL> -O .env
    ```
2. Mettez à jour ce fichier avec le contenu suivant:
    ```sh
    TPBDD_SERVER=tpbdd-sqlserver.database.windows.net
    TPBDD_DB=tpbdd-sql
    TPBDD_USERNAME=sqlread-user
    TPBDD_PASSWORD=<pré-rempli>
    ODBC_DRIVER={ODBC Driver 18 for SQL Server}

    TPBDD_NEO4J_SERVER=<Bolt URL au format bolt://...>
    TPBDD_NEO4J_USER=neo4j
    TPBDD_NEO4J_PASSWORD=<Neo4j password>
    ```
3. Démarrez un *nouveau* terminal dans VS Code et exécutez la commande suivante:
    ```sh
    python3 pyodbc-py2neo-test.py
    ```
4. Si la commande fonctionne alors votre environnement est pleinement fonctionnel.

⚠️ Le nombre d'heure gratuites octroyées par GitHub est limité. Lorsque votre session de travail est terminée, veillez impérativement à:
- Commiter et pusher (sync) votre code dans votre repository

    ![image](https://github.com/user-attachments/assets/a8f39b87-94da-4eb3-a89c-3c539c17ac88)![image](https://github.com/user-attachments/assets/ddab4349-fdab-4dc8-ac66-d725ea66303a)

- Eteindre ou détruire votre Codespace depuis le [tableau de bord Codespaces](https://github.com/codespaces). Il est important de pusher votre code car même en éteignant le Codespace, votre code non commité peut être supprimé lors d'un auto-delete par GitHub.

    ![image](https://github.com/user-attachments/assets/f4ded3ba-31c4-4d1b-a5da-e422576a8e15)

    Pour reprendre votre travail, retournez sur [tableau de bord Codespaces](https://github.com/codespaces) puis cliquez sur le nom de votre Codespace si vous l'aviez éteint, ou créez-en un nouveau depuis votre fork.

### Optionnel: Outillage
1. Il est conseillé de passer l'interface du portail Azure en **anglais** afin de suivre plus facilement les instructions du TP.
2. Pour la Partie 1 consacrée à SQL et si vous le pouvez, installez [Azure Data Studio](https://docs.microsoft.com/fr-fr/sql/azure-data-studio/download-azure-data-studio?view=sql-server-ver15) pour faciliter la création de vos requêtes SQL. Si ce n'est pas possible, vous pourrez également effectuer les requêtes depuis votre navigateur via [portail Azure](https://portal.azure.com), en ouvrant la page correspondant à la base de données SQL `tpbdd-sqlserver/tpbdd-sql`.

# Partie 1 - Base de données relationnelle - Azure SQL Database
Ecrire les requêtes ci-dessous, et les expliquer en deux ou trois phrases maximum en français ou en anglais.

## Requêtes SQL

**Exercice 0**: Décrivez les tables et les attributs.

tArtist
idArtist (primary key)
primaryName (nvarchar)
birthYear (smallint)

tFilm
idFilm (primary key)
primaryTitle (nvarchar)
startYear (smallint)
averageRating (decimal)
runtimeMinutes (smallint)

tFilmGenre
idFilm (primary key)
idGenre (foreign key)

tGenre
idGenre (primary key)
genre (nvarchar)

tJob
idArtist (primary key)
category (nvarchar)
idFilm (foreign key)


**Exercice 1** (¼ pt): Visualisez l'année de naissance de l'artiste `Inci Pars`.

```sql
SELECT birthYear FROM tArtist where primaryName =  'Inci Pars';
>>> 1980
```

**Exercice 2** (¼ pt): Comptez le nombre d'artistes présents dans la base de donnee.

```sql
SELECT COUNT(*) FROM tArtist;
>>> 84166
```

**Exercice 3** (¼ pt): Trouvez les noms des artistes nés en `1960`, affichez ensuite leur nombre.

```sql
SELECT primaryName FROM tArtist where birthYear = 1960;
SELECT COUNT(*) FROM tArtist where birthYear = 1960;
>>> 211
```

**Exercice 4** (1 pt): Trouvez l'année de naissance la plus représentée parmi les acteurs (sauf 0!), et combien d'acteurs sont nés cette année là.
```sql
SELECT TOP 1 birthYear, COUNT(*) as nb 
FROM tArtist 
WHERE birthYear != 0 
GROUP BY birthYear 
ORDER BY nb DESC;

>>> 1982, 459
```

**Exercice 5** (½ pt): Trouvez les artistes ayant joué dans plus d'un film

```sql
SELECT j.idArtist, a.primaryName, COUNT(DISTINCT j.idFilm) AS nb_films
FROM tJob j JOIN tArtist a ON j.idArtist = a.idArtist
WHERE j.category = 'acted in'
GROUP BY j.idArtist, a.primaryName
HAVING COUNT(DISTINCT j.idFilm) > 1;
>>> 8538 lignes
```

explication:
La requête joint les tables `tArtist` et `tJob` sur la clé `idArtist` et filtre les lignes où la catégorie est `acted in`. Ensuite, elle groupe les résultats par `idArtist` (et ensuite primaryName pour gérer les cas où plusieurs artistes ont les mêmes noms) et compte le nombre de films pour chaque artiste. Enfin, elle filtre les artistes ayant joué dans plus d'un film.

**Exercice 6** (½ pt): Trouvez les artistes ayant eu plusieurs responsabilités au cours de leur carrière (acteur, directeur, producteur...).

```sql
SELECT a.primaryName, COUNT(DISTINCT j.category) AS nb_responsibilities
FROM tJob j
JOIN tArtist a ON j.idArtist = a.idArtist
GROUP BY j.idArtist, a.primaryName
HAVING COUNT(DISTINCT j.category) > 1
>>> 6028 lignes
```

explication:
La requête joint les tables `tJob` et `tArtist` sur la clé `idArtist` et groupe les résultats par `idArtist` puis `primaryName`. Ensuite, elle compte le nombre de catégories distinctes pour chaque artiste et filtre les artistes ayant eu plusieurs responsabilités au cours de leur carrière.

**Exercice 7** (¾ pt): Trouver le nom du ou des film(s) ayant le plus d'acteurs (i.e. uniquement *acted in*).

```sql
WITH FilmActorCount AS (
    SELECT j.idFilm, f.primaryTitle, COUNT(DISTINCT j.idArtist) AS nb_actors
    FROM tJob j
    JOIN tFilm f ON j.idFilm = f.idFilm WHERE j.category = 'acted in'
GROUP BY j.idFilm, f.primaryTitle
)

SELECT idFilm, primaryTitle, nb_actors
FROM FilmActorCount
WHERE nb_actors = (SELECT MAX(nb_actors) FROM FilmActorCount);
>>> 5781 lignes
```

Cette requête utilise une CTE (Common Table Expression) pour compter le nombre d'acteurs distincts par film dans FilmActorCount. Ensuite, elle sélectionne le ou les films ayant le plus grand nombre d'acteurs en comparant chaque valeur à la valeur maximale des acteurs obtenue via une sous-requête.

**Exercice 8** (1 pt): Montrez les artistes ayant eu plusieurs responsabilités dans un même film (ex: à la fois acteur et directeur, ou toute autre combinaison) et les titres de ces films.

```sql
SELECT 
    a.primaryName,
    f.primaryTitle,
    COUNT(DISTINCT j.category) AS nb_responsibilities
FROM tJob j
JOIN tArtist a ON j.idArtist = a.idArtist
JOIN tFilm f ON j.idFilm = f.idFilm
GROUP BY a.idArtist, f.idFilm, a.primaryName, f.primaryTitle
HAVING COUNT(DISTINCT j.category) > 1
>>> 6195 lignes
```

Explication:

La requête joint les tables `tJob`, `tArtist` et `tFilm` sur les clés `idArtist` et `idFilm`. Ensuite, elle groupe les résultats par `idArtist`, `idFilm`, `primaryName` et `primaryTitle`. Enfin, elle compte le nombre de catégories distinctes pour chaque artiste et film et filtre les artistes ayant eu plusieurs responsabilités dans un même film.


# Partie 2 - Base de données graphe - Neo4j
Neo4j est une base de données graphe. Les données sont représentées par des nœuds, des relations entre les nœuds et des propriétés:
1. Les nœuds et les relations contiennent des propriétés
2. Les *relations* connectent les nœuds
3. Les *propriétés* sont des paires clé-valeur pouvant être associées à des nœuds ou des relations
4. Les relations ont des *directions*: unidirectionnelles et bidirectionnelles

## Cypher 
Le langage de requêtage utilisé par Neo4j est [Cypher](https://neo4j.com/developer/cypher/). Vous trouverez sa [documentation officielle](https://neo4j.com/docs/cypher-refcard/current/) sur le site de Neo4j.

Cypher est basé sur des **patterns**. Un pattern décrit les données et permet d'interroger le graphe par le biais d'une syntaxe visuelle très similaire à la façon dont sont généralement illustrées sur papier les données dans un graphe.

On utilise des:
* Cercles `()` pour représenter les **nœuds**
* Flèches `-->` pour représenter les **relations**

**Patterns** :
1. nœud a : `(a)`
2. une relation entre deux nœuds : `(a)–>(b)` 
3. un chemin : `(a)–>(b)<–(c)`
4. un label (ou étiquette) : `(a:label)`

Les patterns sont utilisés pour le requêtage et visent à surtout réprésenter la structure des liens entre les nœuds, comme on peut le voir dans les exemples suivants:
1. `(a)–(b)`
2. `(a)-[r]- >(b) `
3. `(a)-[r:REL_TYPE]->(b) `
4. `(a)-[:REL_TYPE]->(b) `
5. `(a)-[*]->(b)`

### Création de noeuds
Le statement `CREATE` nous permet de créer des noeuds selon la structure suivante :  
```
CREATE (nodePseudoVariable:nodeLabel { nodePropertyName: nodePropertyValue, .. })
```

Supposons que nous voulions créer deux noeuds: `Alice` et `Bob` de type `Person` avec la propriéte `name`, les requêtes seraient les suivantes: 
```
query = ("CREATE (Alice:Person { name: 'Alice' }) "
query = ("CREATE (Bob:Person { name: 'Bob' }) "
```

Nous pouvons aussi lier les noeuds par des relations, comme par exemple en indiquant que `Bob` et `Alice` se connaissent:

```
MATCH
  (a:Person),
  (b:Person)
WHERE a.name = 'Alice' AND b.name = 'Bob'
CREATE (a)-[r:KNOWS]->(b)
RETURN type(r)
```

### Création de relations
Le statement `MATCH` permet également de créer des relations entre des noeuds préalablement identifiés (*matchés*) en fonction de conditions:
```
MATCH (a:NodeLabel),(b:NodeLabel)
WHERE .....
CREATE (a)-[r:RELTYPE]->(b)
RETURN type(r)
```

Pour sélectionner tous les noeuds, utilisez la requête suivante:
```
MATCH ( n ) 
RETURN n
```

Autre exemple, pour sélectionner tous les noeuds disposant de la propriété `label`

```
MATCH (n:label) 
RETURN n
```

Pour affiner encore d'avantage, voici comment sélectionner tous les noeuds ayant une propriété `label` dont la valeur vaut `value` :


```
MATCH ( n )
WHERE n.label = 'value'
RETURN n
```

Pour supprimer des noeuds, utilisez le statement `DELETE`.

# Partie 3 - Export des données vers un modèle graphe
1. Dans le dossier `tp`, complétez le programme [export-neo4j.py](export-neo4j.py) aux endroits notés `A COMPLETER`. N'hésitez pas à déboguer en ajoutant des `print`, créer des programmes de test etc. Utilisez les fonctions [`create_nodes` et `create_relationships`](https://neo4j-contrib.github.io/py2neo/bulk/index.html) de **py2neo**.
2. Effectuez l'export vers votre base Neo4j Sandbox
3. (optionnel, 2 points bonus) Décrivez de quelle manière l'environnement de développement a été préconfiguré pour vous:
    - Environnement Python
    - Informations de connexion aux bases de données

# Partie 4 - Requêtes graphe (Cypher)
Ecrire les requêtes ci-dessous, et les expliquer en deux ou trois phrases maximum en français ou en anglais.

**Exercice 1** (¼ pt): Ajoutez une personne ayant votre prénom et votre nom dans le graphe. Verifiez qui le noeud a bien éte crée. 

**Exercice 2** (¼ pt): Ajoutez un film nommé `L'histoire de mon 20 au cours Infrastructure de donnees`

**Exercice 3** (½ pt): Ajoutez la relation `ACTED_IN` qui modélise votre participation à ce film en tant qu'acteur/actrice

**Exercice 4** (½ pt): Ajoutez deux de vos professeurs/enseignants comme réalisateurs/réalisatrices de ce film.

**Exercice 5** (½ pt): Affichez le noeud représentant l'actrice nommée `Nicole Kidman`, et visualisez son année de naissance.

**Exercice 6** (½ pt): Visualisez l'ensemble des films.

**Exercice 7** (½ pt): Trouvez les noms des artistes nés en `1963`, affichez ensuite leur nombre.

**Exercice 8** (1 pt): Trouver l'ensemble des acteurs (sans entrées doublons) qui ont joué dans plus d'un film.

**Exercice 9** (1 pt): Trouvez les artistes ayant eu plusieurs responsabilités au cours de leur carrière (acteur, directeur, producteur...).

**Exercice 10** (1 pt): Montrez les artistes ayant eu plusieurs responsabilités dans un même film (ex: à la fois acteur et directeur, ou toute autre combinaison) et les titres de ces films.

**Exercice 11** (2 pt): Trouver le nom du ou des film(s) ayant le plus d'acteurs.

## Requêtes graphe (Gremlin)
Une autre base de données graphe a été créée pour ce TP.  Elle utilise la technologie Cosmos DB, une base de donnée multi-paradigme sur Azure. Pour le paradigme graphe et contrairement à Cypher, Cosmos DB utilise un langage de requêtage open source: [Apache Gremlin](https://tinkerpop.apache.org/docs/3.3.2/reference/#graph-traversal-steps).

Vous pourrez trouver cette base, déjà préremplie, dans le portail Azure sous le nom `tpbdd-movies-cdb`. Vous pourrez y effectuer des requêtes en utilisant l'onglet **Data Explorer**.

**Exercices**: Effectuez toutes les requêtes de la section précédentes en utilisant ce langage de requêtage.

Barême: [Ex1:0]  [Ex2:0] [Ex3:0] [Ex4:1] [Ex5:1] [Ex6:1] [Ex7:1] [Ex8:1] [Ex9:1.5] [Ex10:2] [Ex11:5(bonus)]

## (optionnel, 3 points bonus) Encapsulation dans des APIs serverless
Implémentez quelques unes des requêtes du TP sous forme d'API Serverless avec Azure Function ou AWS Lambda. Mesurez les performances.

