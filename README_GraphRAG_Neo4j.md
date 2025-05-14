# Graph RAG with Neo4j and the New Data Format

This README explains how to adapt your Neo4j code to work with the new data format (`entities_relations.json`) for Graph RAG implementation.

## Overview of Changes

The main changes required to adapt the Neo4j code for the new data format are:

1. **Entity Extraction**: Extract entities from the `entities` array in the new format
2. **Relationship Mapping**: Process relationships from the `relationships` array
3. **Graph Structure Adaptation**: Transform the data into triplets for Graph RAG
4. **Property Utilization**: Leverage the additional metadata (entity_type, description, etc.)

## Implementation Details

The implementation is provided in the `GraphRAG_Neo4j.ipynb` notebook, which includes:

1. **Neo4j Integration**: Code to load and process the new data format into Neo4j
2. **Triplet Extraction**: Conversion of the new format into triplets for Graph RAG
3. **Vector Storage**: Using Milvus for storing embeddings of entities, relations, and passages
4. **Graph Structure**: Building adjacency matrices to represent the graph structure
5. **Query Processing**: Functions to process queries using the graph structure

## Key Components

### 1. Data Loading and Neo4j Integration

```python
# Load JSON data
with open("/Users/shiva/Documents/Regal/data/enities_realtions.json", "r") as f:
    kg_data = json.load(f)

# Insert entities and relationships into Neo4j
with driver.session() as session:
    # Insert entities
    for entity in kg_data["entities"]:
        session.write_transaction(
            insert_entity,
            entity["name"],
            entity["entity_type"],
            entity["singular"],
            entity["description"]
        )

    # Insert relationships
    for rel in kg_data["relationships"]:
        session.write_transaction(
            insert_relationship,
            rel["entity_1"],
            rel["entity_2"],
            rel["relationship_type"],
            rel["description"]
        )
```

### 2. Triplet Extraction for Graph RAG

```python
def extract_triplets_from_new_format(kg_data):
    triplets = []
    triplets_with_passages = []
    
    # Create a mapping of entity names to their descriptions
    entity_descriptions = {}
    for entity in kg_data["entities"]:
        entity_descriptions[entity["name"]] = entity["description"]
    
    # Create triplets from relationships
    for rel in kg_data["relationships"]:
        entity_1 = rel["entity_1"]
        entity_2 = rel["entity_2"]
        relationship_type = rel["relationship_type"]
        
        # Create a triplet in the format expected by Graph RAG
        triplet = [entity_1, relationship_type, entity_2]
        triplets.append(triplet)
        
        # Create a passage from the relationship and entity descriptions
        passage = f"{entity_1} {relationship_type} {entity_2}. "
        if entity_1 in entity_descriptions:
            passage += f"Description of {entity_1}: {entity_descriptions[entity_1]}. "
        if entity_2 in entity_descriptions:
            passage += f"Description of {entity_2}: {entity_descriptions[entity_2]}. "
        if "description" in rel and rel["description"]:
            passage += f"Relationship details: {rel['description']}"
        
        # Store the passage with the triplet
        triplet_with_passage = {
            "passage": passage,
            "triplets": [triplet]
        }
        triplets_with_passages.append(triplet_with_passage)
    
    return triplets, triplets_with_passages
```

### 3. Graph Structure Building

```python
# Process triplets for Graph RAG
entityid_2_relationids = defaultdict(list)
relationid_2_passageids = defaultdict(list)

entities = []
relations = []
passages = []

# Process the triplets with passages
for passage_id, dataset_info in enumerate(triplets_with_passages):
    passage, triplet_list = dataset_info["passage"], dataset_info["triplets"]
    passages.append(passage)
    
    for triplet in triplet_list:
        if triplet[0] not in entities:
            entities.append(triplet[0])
        if triplet[2] not in entities:
            entities.append(triplet[2])
        
        relation = " ".join(triplet)
        if relation not in relations:
            relations.append(relation)
            entityid_2_relationids[entities.index(triplet[0])].append(len(relations) - 1)
            entityid_2_relationids[entities.index(triplet[2])].append(len(relations) - 1)
        
        relationid_2_passageids[relations.index(relation)].append(passage_id)
```

### 4. Adjacency Matrix Construction

```python
# Build adjacency matrices
entity_relation_adj = np.zeros((len(entities), len(relations)))
for entity_id, entity in enumerate(entities):
    entity_relation_adj[entity_id, entityid_2_relationids[entity_id]] = 1

entity_relation_adj = csr_matrix(entity_relation_adj)

entity_adj_1_degree = entity_relation_adj @ entity_relation_adj.T
relation_adj_1_degree = entity_relation_adj.T @ entity_relation_adj

target_degree = 1

entity_adj_target_degree = entity_adj_1_degree
for _ in range(target_degree - 1):
    entity_adj_target_degree = entity_adj_target_degree * entity_adj_1_degree
relation_adj_target_degree = relation_adj_1_degree
for _ in range(target_degree - 1):
    relation_adj_target_degree = relation_adj_target_degree * relation_adj_1_degree

entity_relation_adj_target_degree = entity_adj_target_degree @ entity_relation_adj
```

## How to Use

1. **Setup**: Install the required dependencies
   ```
   pip install neo4j pymilvus numpy scipy langchain langchain-core langchain-openai tqdm
   ```

2. **Neo4j Configuration**: Ensure Neo4j is running and update the connection details
   ```python
   uri = "bolt://localhost:7687"
   driver = GraphDatabase.driver(uri, auth=("neo4j", "123456789"))
   ```

3. **OpenAI API Key**: Set your OpenAI API key
   ```python
   os.environ["OPENAI_API_KEY"] = "your-api-key-here"
   ```

4. **Run the Notebook**: Execute the cells in the notebook to:
   - Load the data into Neo4j
   - Extract triplets for Graph RAG
   - Build the graph structure
   - Create and populate Milvus collections
   - Test queries against the Graph RAG system

5. **Query the System**: Use the `query_graph_rag` function to query the system
   ```python
   test_query = "What is the relationship between Reserve Bank and Foreign Exchange Management Act?"
   result = query_graph_rag(test_query, ["Reserve Bank", "Foreign Exchange Management Act"])
   ```

## Advantages of the New Approach

1. **Richer Metadata**: The new format includes entity types, descriptions, and relationship details
2. **Better Context**: Passages include descriptions of entities and relationships
3. **Improved Query Understanding**: Entity types help in disambiguating entities
4. **More Accurate Retrieval**: The graph structure allows for more accurate retrieval of relevant passages

## Conclusion

By adapting the Neo4j code to work with the new data format and integrating it with Graph RAG, you can create a more powerful and accurate retrieval system that leverages both the graph structure and the rich metadata provided in the new format.
