# The Odyssey Graph

One story, three tellings, one graph: Homer's *Odyssey*, *EPIC: The Musical*, and Christopher Nolan's *The Odyssey* (2026) - every character, god, monster, place, song, saga, and actor, modelled as a Neo4j knowledge graph.

Built as a learning project and written up in full on the GraphAcademy blog: [The Odyssey in a Graph Database](https://graphacademy.neo4j.com/blog/odyssey-graph-database-neo4j).

## What's in the box

Two CSV files. The whole dataset.

### `nodes.csv` - the nouns

| column | what it is |
| --- | --- |
| `name` | Unique name (see "quirks" below) |
| `type` | One of: Hero, Mortal, Deity, Monster, Animal, Place, Saga, Song, Actor |
| `description` | What/who this is |
| `notes` | Extras: EPIC musical motifs, "Homer only" markers, casting notes |
| `source` | Where this row's facts came from |

130 nodes: 3 Heroes, 17 Mortals, 13 Deities, 5 Monsters, 1 very good dog, 14 Places, 9 Sagas, 40 Songs, 28 Actors.

### `relationships.csv` - the verbs

| column | what it is |
| --- | --- |
| `from`, `to` | Node names (must match `nodes.csv`) |
| `type` | Relationship type, e.g. `BLINDS`, `BETRAYS`, `PLAYS`, `TRAVELS_TO` |
| `detail` | The story behind the arrow |
| `source` | Where this claim comes from |

235 relationships across 28 types - family (`CHILD_OF`, `MARRIED_TO`), drama (`BLINDS`, `CURSES`, `BETRAYS`, `KILLS`), geography (`TRAVELS_TO`, `LIVES_IN`), the musical (`PART_OF`, `FEATURES`), and casting (`PLAYS`, with a `detail` marking film vs musical).

## Quickstart (free Neo4j Aura instance)

1. Create a free instance at [console.neo4j.io](https://console.neo4j.io)
2. In the Query tab, set the uniqueness rule:

```cypher
CREATE CONSTRAINT entity_name IF NOT EXISTS
FOR (n:Entity) REQUIRE n.name IS UNIQUE
```

3. Load the nodes:

```cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/KarayatAj/The_Odyssey_Graph/refs/heads/main/nodes.csv' AS row
MERGE (n:Entity {name: row.name})
SET n:$(row.type),
    n.description = row.description,
    n.notes = row.notes,
    n.source = row.source
```

4. Load the relationships:

```cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/KarayatAj/The_Odyssey_Graph/refs/heads/main/relationships.csv' AS row
MATCH (a:Entity {name: row.from})
MATCH (b:Entity {name: row.to})
MERGE (a)-[r:$(row.type)]->(b)
SET r.detail = row.detail,
    r.source = row.source
```

5. Drop the scaffolding (so Bloom colour-codes by real type):

```cypher
DROP CONSTRAINT entity_name
```

```cypher
MATCH (n) REMOVE n:Entity
```

6. Verify: you should have exactly 130 nodes and 235 relationships:

```cypher
MATCH (n) OPTIONAL MATCH (n)-[r]->()
RETURN count(DISTINCT n) AS nodes, count(r) AS relationships
```

> Re-running the import after step 5 will create duplicates (the `MERGE` matches on the
> `Entity` label you just removed). If you need to rebuild, use
> `MATCH (n) DETACH DELETE n` and repeat from step 2.

## Skip the queries

Download `odyssey_data_import_neo4j.zip`, then in Aura go to **Import → ⋯ → Open model (with data)**, choose the zip, and hit **Run import** - the full graph builds itself. 

And if you want it to look like the blog post too: in **Bloom → Perspectives → Import**, choose `odyssey-bloom-perspective.json` for the full colour-coded styling.

## Questions the graph can answer

```cypher
// Who has played Odysseus? (a Hollywood A-lister and a man who cast a musical over TikTok)
MATCH p = (:Actor)-[:PLAYS]->({name: "Odysseus"}) RETURN p

// What did EPIC: The Musical cut from Homer?
MATCH (n) WHERE n.notes CONTAINS "Homer only" RETURN n.name, n.description

// The full ten-year commute, Troy to Ithaca
MATCH p = ({name: "Troy"})-[:TRAVELS_TO*]->({name: "Ithaca"}) RETURN p

// Whose story is it, really?
MATCH (n)-[r]-()
WHERE NOT n:Song AND NOT n:Saga AND NOT n:Actor
RETURN n.name AS character, count(r) AS connections
ORDER BY connections DESC LIMIT 10
```

## Quirks and honest footnotes

- **Five songs are suffixed "(song)"** - Odysseus (song), Scylla (song), Charybdis (song),
  The Underworld (song), Polyphemus (song) - because EPIC names songs after characters and
  places, and node names must be unique. The blog post tells the full identity-collision story.
- **The Trojan War island:** Agamemnon, Menelaus, Helen, Clytemnestra, and their actors form
  a cluster with no connection to Odysseus's side of the graph. However, in
  Book 4, Telemachus visits Menelaus - which you can add yourself with:
  `MATCH (t {name:"Telemachus"}) MATCH (m {name:"Menelaus"}) MERGE (t)-[:VISITS {detail:"Homer Book 4"}]->(m)`
- **Casting dispute:** IMDb lists Charlize Theron as Circe; Wikipedia and Rotten Tomatoes
  say Calypso (with Samantha Morton as Circe). This dataset follows the two-source majority.
- **Film data is cast-only** - the film's plot details weren't public when this was compiled
  (checked 6 Jul 2026).
- Song-to-character `FEATURES` links were verified by ear against the official albums.
  Corrections welcome - open an issue or PR.

## Sources

Homer's *Odyssey* (Emily Wilson translation) - Theoi.com for lineages - EPIC: The Musical official albums and track lists - Wikipedia, Rotten Tomatoes, IMDb, and BroadwayWorld for cast details. Every row carries its own `source` column.

## Learn how this was built

- [The Odyssey in a Graph Database](https://graphacademy.neo4j.com/blog/odyssey-graph-database-neo4j) - the full write-up
  
### Free GraphAcademy courses:

- [Neo4j Fundamentals](https://graphacademy.neo4j.com/courses/neo4j-fundamentals)
- [Cypher Fundamentals](https://graphacademy.neo4j.com/courses/cypher-fundamentals)
- [Importing Data Fundamentals](https://graphacademy.neo4j.com/courses/importing-fundamentals)
- [Graph Data Modeling Fundamentals](https://graphacademy.neo4j.com/courses/modeling-fundamentals)
