# Designing Data Intensive Applications - Data Models and Query Languages
> Created: 2022-11-02 18:29
> #book 

The key question when designing data models is: how is each layer represented in the layer below it?

As an example, the layers can be: real world models → objects, data structures and APIs → general purpose data model (JSON/XML/tables) → bytes in memory/disk → electrical impulses.

## Relational Model vs Document Model

**Relational Model:** Data is represented as _relations_ (tables) containing unordered _tuples_ (rows). Created by Edgar Codd, 1970.

The usecases for relational models is generally in business data processing, either in _transaction processing_ or _batch processing_ ([[OLTP]] and [[OLAP]] are related concepts). Relational models generalized and scaled far better than expected, and went way beyond their original usecases.

### NoSQL

Term used to reference recent, non-relational databases. However, some partially relational databases have also been added into this definition. Why did this kick off?

+ Need for greater scalability, either very large datasets or high write throughput.
+ Widespread preference for FOSS over proprietary DB tech
+ Specialized query operations that are hard to do in SQL models
+ Desire for more dynamic / expressive data models

**Polyglot Persistence:** The idea of using relational models alongside non-relational / NoSQL, as per individual use case within the project.

### Object-Relational Mismatch

If data is stored in relational structures in the database, but the program itself requires in-memory objects to represent the data, then we end up needing a layer in the middle to translate to and from the database to the program. This layer is called Object Relational Mapping (ORM). We call the problem itself an _impedance mismatch_.

ORM frameworks include Hibernate, ActiveRecord, Room etc. These reduce the amount of boilerplate required for the translation.

#### Case Study: Representing a Resume in a DB

While you can put things like `firstName` and `lastName` directly in the `users` table, you cannot put something like `positionsHeld` since one user will have multiple positions, each with its own data.

Options for representing one-to-many relationships:
+ Traditional SQL: Put the right hand side in separate tables, with a foreign key reference to the left hand table.
+ Later versions of the SQL standard allow structured data (and sometimes JSON documents) which can be queried for their internals, allowing putting multi-value data in a single row.
+ Convert the entire right hand side collection into JSON/XML, then save it as text linked to the left hand side (cannot query within this data though).

Alternatively, you can use a JSON document that represents the whole resume and put it in a NoSQL DB.

### Data normalization

There are reasons why you would want to use identified objects over free text when storing data, for example a `town` field in an address could link to a `towns` table which contains a list of allowed towns, rather than simply keeping it as a text field.

Advantages:
+ Consistent style and spelling across records
+ Avoiding ambiguity (same name on different towns?)
+ Ease of updating if it ever needs to be changed (Bangalore → Bengaluru)
+ Localization support
+ Better search

The question really is one of duplication -- if duplication is _okay_ in the context of the data you are storing, then it's fine to store it as text. Text has meaning to humans but not computers -- the database itself will not know when some piece of text has changed and something else needs to be updated to keep it in line. But IDs on the other hand are meaningless to humans but can be used by computers to link together different bits of data and remove duplication. This is the key idea behind _normalization_ in databases.

Document databases (NoSQL) generally have weak support for joins, therefore duplication is more common and normalization is difficult. If the DB does not support joins, you (the app) have to perform multiple queries to the DB to fetch all the data. Often you'll have issues with atomicity and performance when this happens. The work of making an atomic join and fetch of the data is transferred to the application.

Also, often the structure of data changes over time -- for example, in a resume we might have stored `schools` as just the names (text), but might need to change it in the future to objects which link to details of specific schools. This might mean that an initially join-free model can require joins later.

### Data Model Comparison

**Simplicity of Application Code.** If the data has a document-like structure, where the whole tree is generally loaded at once, then it's better to use the document model. Relational models _shred_ data (splitting into multiple tables) leading to unnecessarily complex app code. If you frequently need to reference nested items directly, prefer a relational model (where the individual tables can be directly queried). 

If the app uses a lot of many-many joins, then generally avoid document structures. You will have to write the joins (and handle atomicity and partial query failures) within the application itself.

**Schema Flexibility.** Document databases are sometimes called *schemaless*, but that’s misleading, as the code that reads the data usually assumes some kind of structure—i.e., there is an implicit schema, but it is not enforced by the database. A more accurate term is *schema-on-read* (the structure of the data is implicit, and only interpreted when the data is read), in contrast with *schema-on-write* (the traditional approach of relational databases, where the schema is explicit and the database ensures all written data conforms to it).

Schema changes on relational DBs (`ALTER TABLE`) is generally fast. MySQL is a notable exception -- it creates a copy of the table before altering it, and can therefore take hours depending on the size of the table. See https://arctype.com/blog/mysql-alter-table-problems/.

With schema-on-read DBs, the application code will generally handle old schemas using an `if` block, deciding how to process the data depending on the shape of the data at runtime. Think of it as a dynamically typed language, as opposed to static typing for schema-on-write DBs.

**Data locality.** A document store generally keeps the whole document in contiguous memory blocks, making faster retrieval for when you want the whole document at once. But if you only want parts of the document, or make edits to a tiny part (especially if they increase the size of the document as a whole), then it gets worse for perf since the whole thing needs to be picked. Some relational DBs (e.g. Google Spanner) also give data interleaving options for locality on disk.

## Query Languages

Generally, **declarative query languages** are preferred over imperative languages, as they simply describe the *shape* of the data, and don't give any specifics about how to actually retrieve it. This makes performance improvements (like parallelization across cores) doable in the background without requiring to change how the query is written. SQL is a declarative language.

### MapReduce Querying

MongoDB allows MapReduce querying. How does this work?

1. Map: For each document, call a `map` function that returns a `(key, value)` pair.
2. Reduce: For each set of `(key, listOfValues)`, call a `reduce` function that returns a new `summaryValue` for that key (can be the original `listOfValues` as well.

Both of the above should be _pure_ functions. MongoDB allows adding an additional filtering query on top of this.

## Graph Data Models

If you have tons of many-to-many relations in your data, then it might make sense to model as a graph (vertices + edges). Practical examples are social graphs, the web graph, rail/road networks etc.

You can also use graph DBs to store _heterogenous data_, where vertices can have different types and edges can mean different kinds of connections within the same graph.

### Property Graphs

Basically an unordered set of vertices, and another unordered set of edges, containing full details pertaining to the specific vertex or edge, including whatever properties (key-values) are denoted by that vertex, or whatever relationship type is denoted by that edge. Also has what incoming/outgoing edges are connected to each vertex, and what are the tail and head vertices of each edge.

```sql
CREATE TABLE vertices (
    vertex_id integer PRIMARY KEY,
    properties json
);
CREATE TABLE edges (
    edge_id integer PRIMARY KEY,
    tail_vertex integer REFERENCES vertices (vertex_id),
    head_vertex integer REFERENCES vertices (vertex_id),
    label text,
    properties json
);
CREATE INDEX edges_tails ON edges (tail_vertex);
CREATE INDEX edges_heads ON edges (head_vertex);
```

Important properties:
1. Schema does not limit which vertex can be connected to which other vertex/edge.
2. We can quickly traverse the graph because we can find which edges are connected to each vertex directly.
3. Has different labels for different types of relationships.

### Cypher Query Language

[Declarative query language for property graphs][cypher], created for Neo4j. Example insert query:

```cypher
CREATE
    (NAmerica:Location {name:'North America', type:'continent'}),
    (USA:Location {name:'United States', type:'country' }),
    (Idaho:Location {name:'Idaho', type:'state' }),
    (Lucy:Person {name:'Lucy' }),
    (Idaho) -[:WITHIN]-> (USA) -[:WITHIN]-> (NAmerica),
    (Lucy) -[:BORN_IN]-> (Idaho)
```

Querying for data:

```cypher
MATCH
    (person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
    (person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (eu:Location {name:'Europe'})
RETURN person.name
```

Explanation:
1. `person` has outgoing `BORN_IN` edge to some vertex, which eventually follows (via a chain of 0 or more `WITHIN` edges) to `name=United States`.
2. `person` also has an outgoing `LIVES_IN` edge to some vertex, which eventually follows (via a chain of 0 or more `WITHIN` edges) to `name=Europe`.

The query optimizer can automatically figure out the best way to run this query -- you don't specify it within the query itself (declarative, not imperative).

Note that expressing the same thing in SQL would be significantly more difficult -- you need to specify the joins beforehand and it can get extremely clumsy (or impossible) to represent something like "`WITHIN` edge repeated 0 or more times".

### Triple-Stores and SPARQL

All entries are stored as triples of `(subject, predicate, object)`.

Subject is basically the vertex of the graph. Object is either a primitive (string, number etc. which forms a key-value pair as `(vertex, key, value)`), or another vertex, in which case the predicate is the edge-type.

Previous edge graph represented in _Turtle_ triples:

```turtle
@prefix : <urn:example:>.
_:lucy      a            :Person.
_:lucy      :name        "Lucy".
_:lucy      :bornIn      _:idaho.
_:idaho     a            :Location.
_:idaho     :name        "Idaho".
_:idaho     :type        "state".
_:idaho     :within      _:usa.
_:usa       a            :Location.
_:usa       :name        "United States".
_:usa       :type        "country".
_:usa       :within      _:namerica.
_:namerica  a            :Location.
_:namerica  :name        "North America".
_:namerica  :type        "continent".
```

It can also be compressed to:

```turtle
@prefix : <urn:example:>.
_:lucy      a :Person;    :name "Lucy";          :bornIn _:idaho.
_:idaho     a :Location;  :name "Idaho";         :type "state";     :within _:usa.
_:usa       a :Location;  :name "United States"; :type "country";   :within _:namerica.
_:namerica  a :Location;  :name "North America"; :type "continent".
```

#### SPARQL

Query language for triple-stores using the [RDF][rdf] data model. It pre-dates Cypher. Same Cypher query from above in SPARQL:

```sparql
PREFIX : <urn:example:>
SELECT ?personName WHERE {
	?person :name ?personName.
	?person :bornIn / :within* / :name "United States".
	?person :livesIn / :within* / :name "Europe".
}
```

### Datalog

Much older language than the above two, studied in the 1980s. Datalog facts:

```prolog
name(namerica, 'North America').
type(namerica, continent).
name(usa, 'United States').
type(usa, country).
within(usa, namerica).
name(idaho, 'Idaho').
type(idaho, state).
within(idaho, usa).
name(lucy, 'Lucy').
born_in(lucy, idaho).
```

Same query from above:

```prolog
within_recursive(Location, Name) :- name(Location, Name). /* Rule 1 */
within_recursive(Location, Name) :- /* Rule 2 */
	within(Location, Via),
	within_recursive(Via, Name).
migrated(Name, BornIn, LivingIn) :- /* Rule 3 */
	name(Person, Name),
	born_in(Person, BornLoc),
	within_recursive(BornLoc, BornIn),
	lives_in(Person, LivingLoc),
	within_recursive(LivingLoc, LivingIn).
?- migrated(Who, 'United States', 'Europe').
/* Who = 'Lucy'. */
```

Here, we define rules that give us new "intermediate" predicates, which can be built up whenever all the predicates on the RHS of `:-` are satisfied. This is a more complex approach, but is very powerful as it allows combining rules and reusing them in different queries.

----

## References
1. [Designing Data Intensive Applications](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/) Chapter 2

[cypher]: https://neo4j.com/developer/cypher/
[rdf]: https://en.wikipedia.org/wiki/Resource_Description_Framework
