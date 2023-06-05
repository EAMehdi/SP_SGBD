---
theme : gaia
marp: true
paginate: true
---

# Modélisation de la Base Carbone pour la Restauration

pour le  Cours de SQL, NoSQL et NewSQL

par : 

#### Ali BOUKHELIFA
#### & Mehdi EL AYADI

Université Paris Dauphine - PSL

---

## Plan
1. Introduction aux différents modèles de données avec des exemples de commandes de création
2. Présentation des requêtes pour chaque modèle avec des exemples de résultats
3. Analyse des avantages et des limites de chaque modèle
4. Comparaison des différents modèles de données
5. Conclusion : Discussion des difficultés rencontrées

---

## Modèle clé-valeur sous Redis
Le modèle clé-valeur stocke les données en paires clé-valeur. Redis est une base de données en mémoire qui est bien adaptée pour stocker des données de petite à moyenne taille. Les clés et les valeurs peuvent être des chaînes, des entiers, des listes, des sets, des hashes, etc. 

```redis
SET ingredient:légumes_de_saison name "légumes de saison" emission 53.4
SET ingredient:huile_olive name "huile d'olive" emission 18.2
SET plat:entrée:légumes_à_la_grecque name "Entrée : légumes à la grecque" ingredients "légumes_de_saison,huile_olive"
```


---

### Requêtes pour le modèle clé-valeur Redis

```redis
GET ingredient:légumes_de_saison emission

GET plat:entrée:légumes_à_la_grecque ingredients
```

---
### Modèle sous PostgreSQL avec des attributs en couples clé-valeur

Ce modèle stocke les données en utilisant le modèle relationnel, avec une extension pour prendre en charge les couples clé-valeur. C'est utile pour des données qui ont une structure variable ou qui nécessitent des attributs supplémentaires qui ne sont pas connus à l'avance.

```sql
CREATE TABLE Ingredients (
   id SERIAL PRIMARY KEY,
   name TEXT NOT NULL,
   emission FLOAT NOT NULL
);
```

---

### Requêtes pour le modèle PostgreSQL (couples clé-valeur)

```sql
SELECT emission FROM Ingredients WHERE name = 'légumes de saison';

SELECT name, ingredients->'légumes de saison' as legumes_de_saison_emission 
FROM Plats WHERE name = 'Entrée : légumes à la grecque';
```

---
### Modèle document sous MongoDB
Le modèle document stocke les données sous forme de documents, généralement en format BSON, un format binaire qui ressemble à JSON. Cela permet une grande flexibilité, car les documents n'ont pas besoin d'avoir une structure fixe.

```cypher
db.ingredients.insert({ name: "légumes de saison", emission: 53.4 });
db.plats.insert(
{ name: "Entrée : légumes à la grecque", category: "entrée",
 ingredients: [
     { name: "légumes de saison", emission: 53.4 }, 
     { name: "huile d'olive", emission: 18.2 }] 
});
```
---



### Requêtes pour le modèle MongoDB

```javascript
db.ingredients.find({ name: "légumes de saison" }, { emission: 1 });

db.plats.aggregate([
  { $match: { name: "Entrée : légumes à la grecque" } },
  { $unwind: "$ingredients" },
  { $project: { _id: 0, ingredients: 1 } }
]);
```

---

### Modèle sous PostgreSQL avec des attributs en données JSON
Ce modèle utilise la puissance de PostgreSQL et sa capacité à gérer des données JSON pour stocker les informations.

```sql
CREATE TABLE Ingredients (
   id SERIAL PRIMARY KEY,
   data JSONB
);

CREATE TABLE Plats (
   id SERIAL PRIMARY KEY,
   data JSONB
);
```

---

### Requêtes pour le modèle PostgreSQL (données JSON)

```sql
SELECT data->>'emission' FROM Ingredients WHERE data->>'name' = 'légumes de saison';

SELECT data->'ingredients' FROM Plats WHERE data->>'name' = 'Entrée : légumes à la grecque';
```

---

### Modèle graphe sous Neo4j
Le modèle graphe est excellent pour modéliser des données interconnectées, comme des réseaux sociaux, des arbres généalogiques, etc. Neo4j est une base de données orientée graphe qui peut gérer des requêtes complexes sur des données interconnectées.

```SQL
CREATE (i:Ingredient { name: "légumes de saison", emission: 53.4 })
CREATE (p:Plat { name: "Entrée : légumes à la grecque", category: "entrée" })
CREATE (p)-[:CONTAINS]->(i)
```


---



### Requêtes pour le modèle Neo4j

```cypher
MATCH (i:Ingredient) WHERE i.name = "légumes de saison" RETURN i.emission;

MATCH (p:Plat)-[:CONTAINS]->(i:Ingredient)
 WHERE p.name = "Entrée : légumes à la grecque" RETURN i.name, i.emission;
```

---

## Analyse des Modèles (1)

#### Redis:
- Avantages: Performance rapide, facile à utiliser, flexible.
- Limites: Pas idéal pour les relations complexes entre les données.

#### PostgreSQL (couples clé-valeur):
- Avantages: Supporte les relations complexes, flexible avec l'extension hstore.
- Limites: Moins performant que les bases de données en mémoire comme Redis.

---
## Analyse des Modèles (2)

**MongoDB**:
- Avantages: Grande flexibilité, facile à utiliser pour les données orientées document.
- Limites: Moins performant pour les requêtes relationnelles complexes.

---
## Analyse des Modèles (3)
**PostgreSQL (données JSON)**:
- Avantages: Supporte les relations complexes, flexible avec le support JSON.
- Limites: Moins performant que les bases de données en mémoire comme Redis.

---
## Analyse des Modèles (4)
#### Neo4j:
- Avantages: Excellent pour les données interconnectées, supporte des requêtes complexes.
- Limites: Peut être plus difficile à configurer et à utiliser que d'autres modèles.

---

## Comparaison des Modèles 

- Le modèle Redis est le plus rapide pour lire et écrire des données simples, mais n'est pas adapté aux relations complexes.
- Les modèles PostgreSQL sont plus lents mais plus adaptés aux données relationnelles complexes.
- MongoDB offre une grande flexibilité pour les données orientées document.
- Neo4j est le meilleur choix pour les données interconnectées.

---

- **Redis** : Dans notre cas, chaque plat et chaque ingrédient peut avoir plusieurs relations (par exemple, un plat est composé de plusieurs ingrédients, un ingrédient peut faire partie de plusieurs plats), donc Redis **pourrait ne pas être le meilleur choix**.

--- 
- **PostgreSQL** : 
	* **Facile** gérer les relations entre les plats, les ingrédients et les émissions de GES => **jointure** 
	* Fonctionnalités telles que les transactions et la cohérence des données, qui peuvent être utiles pour garantir l'intégrité des données.
	* Facile à mettre en place et à utiliser
	* Mais lecture et d'écriture plus lent par rapport à Redis.

---

- **MongoDB** : 
	* **flexibilité** pour le stockage et la requête de données. 
	* **utile** nous avons des plats ou des ingrédients avec :
**attributs variables** (ex.informations nutritionnelles supplémentaires, notes de plat, commentaires). 
	* **Mais** gestion des relations complexes entre les documents difficile 
et moins performant par rapport PostgreSQL
	* Difficulté à mettre en place et tester (contrairement aux autres)

---

- **Neo4j** : 
	* un bon choix si nous voulons analyser les relations entre les plats, les ingrédients et les émissions de GES de manière plus complexe
	* un peu "overkill" pour notre cas d'utilisation et les requêtes 


--- 
## Conclusion

- Répartition du travail équitable
- Utilisation de chatGPT, nécessite beaucoup de temps pour revérifier et corriger ses erreurs	
