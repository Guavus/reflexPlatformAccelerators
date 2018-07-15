# JanusGraph Sink Connector

JanusGraph Sink Connector is a Guavus provided accelerator that converts the incoming data into graph data. For conversion to graph data it uses user provided vertex and edge schema, and writes the result in JanusGraph database backed by HBase as storage, and Elasticsearch as indexing backends. This Plug-in provides following features:

-   Extracts vertex and edge data using user-defined configuration (labels, properties, and indexes) from the incoming RDD of StructuredRecords.
-   Writes vertex and edge data in JanusGraph.
-   Uses embedded mode to call JanusGraph API(s) for writing in backend storages.
-   Performs auto registration of vertex and edge schema (labels, properties and indexes) in case of a non-existant graph.

## Use Case
Janusgraph Sink Plugin can be used to write incoming RDD of records in graph form in Hbase. 

## Pre-requisite
Data should be transformed in a format as required by the specified schema.

## Configuration

* **Graph Schema**: It is the input JSON configuration file provided by the user.
    
  Graph Schema majorly has following sections :
  
  * vertexCol : All vertices(of any type) are expected to exist in this column.
  * vartexLabelCol : This column contains `type` for the enteries in `vertexCol`.
  * propertyKeys : In this section, all graph properties are globally defined. They may belong to either vertices or edges or both.
  * vertices : This section is used to define all possible labels expected for the `vartexLabelCol`. Also, all
    properties which we want to associate with this label types should be specified here (note that definition of
    these properties are not defined here, they are gloabbly defined in `propertyKeys` section.
  * edges : This section is used to define all possible edge label along with other configuration properties to be
    associated with this edge type. Every edge label is expected to be present as an individual column in input data and should contain values for the other vertex which this edge represents (note that first vertex for this edge is contained by `vertexCol` value for this row data.
  * indexes : This section lists all type of indexes to be created along with other attributes.


  All inidividual properties for specifying Graph schema are further defined below:
    
| Configuration | Required ?|Default|Description|
|:--|:--|:--|:--|
| `vertexCol` | Yes | N/A |This is the common vertex column which uniquely identifies the vertex in the output graph. For example, `column : "name" , value : "Jupiter"` |
| `vertexLabelCol` | Yes | N/A | This column identifies the column type. For example, `column : "type" , value : "God"`|
| `vertices` | Yes | N/A | It is an array of `vertexLabelCol` values, along with their individual properties. | 
| `vertices.label` | Yes | N/A | The name of the specific `vertexLabelCol`. For example, `column : "label" , value : "titan"` |
| `vertices.properties` | No | N/A | The properties for this label value. They can be more than one. For example, `column : "properties" , value : ["age", "height"` |
| `edges` | Yes | N/A | This is the relationship between two vertices. |
| `edges.label` | Yes | N/A | The name of the specific `vertexLabelCol`. For example, `column : "father" , value : "titan"` |
| `edges.multiplicity` | No | N/A | This represents whether the relationship between two vertices is unidirectional or bidirectional. For unidirectional, it will be "MANY2ONE", and for bidirectional it will be "MULTI". For example, `column : "father" , value : "MANY2ONE"` . Note: For "MULTI" the "updateProperty" should be "false" in propertyKeys section|
| `propertyKeys` | Yes | N/A | These are the list of all possible properties of vertices and edges.|
| `propertyKeys.name` | Yes | N/A | The name of the property. For example, `column : "name" , value : "time"` |
| `propertyKeys.dataType` | Yes | N/A | The data type of that property. For example, `column : "property" , value : "String"` |  
| `propertyKeys.cardinality` | Optional | N/A | This provides the option for making the property as List or Set. |
| `propertyKeys.updateProperty` | Optional | false | This provides the option for updating the property. |
    
* **Graph Name**: The name of output HBase table. This field is mandatory.
* **Storage Hostname** : The hostname(s) on which HBase master is running. This field is mandatory.
* **Index Name**: The Elasticsearch index name value to identify the index.
* **Index Storage Hostname**: The connection IP and port for Elasticsearch.
    
  Properties for specifying Index schema are listed below:
    
| Configuration | Required? | Default | Description |
| :-- | :-- | :-- | :-- |
| `indexes.indexClass` | No | N/A | The class extending element on which you create index. For example, `org.apache.tinkerpop.gremlin.structure.Vertex` or `org.apache.tinkerpop.gremlin.structure.Edge` |
| `indexes.name` | Yes | N/A | The name to uniquely identify the index. |
| `indexes.keys` | No | N/A | These are the list of index keys. |
| `indexes.unique` | No | N/A | This binary value to identify if the index name is unique. |
| `indexes.indexType` | Yes | N/A | Specifies if the index is of type Composite or Mixed. |
| `consistencyModifier` | No | N/A | Specifies if you can lock the index, only if the index type is composite. |
| `mixedIndexName` | Yes | N/A | If the index type is mixed, specify the name of mixed index. |
    


* **Additional Properties**: Additional properties that you want to specify. It is an optional field.

| Configuration | Required? | Default | Description |
| :-- | :-- | :-- | :-- | 
| dynamicVertexCreation | No | True | Specifies if the vertex should be created dynamically. |
| enableExceptionLogging | No | True | Specifies if exception logging should be enabled. |
| updateLabel | No | False | Specifies if the vertex label should be updated or not |
 
## Example 

### Sample Input Data to ingest using this plugin

This is the example of an input CSV file for JanusGraph (data corresponds to  [official Graph of Gods example](https://docs.janusgraph.org/latest/getting-started.html)):

```
name,type,age,father,mother,battled,time,place,lives,reason,pet,brother

saturn,titan,10000,null,null,null,null,null,null,null,null,null

jupiter,god,5000,saturn,null,null,null,null,sky,loves fresh breeze,null,neptune

sky,location,null,null,null,null,null,null,null,null,null,null

sea,location,null,null,null,null,null,null,null,null,null,null

neptune,god,4500,null,null,null,null,null,sea,loves waves,null,jupiter

hercules,demigod,30,jupiter,alcmene,nemean,1,38.1,27.3,null,null,null,null

hercules,demigod,30,null,null,hydra,2,37.7,23.9,null,null,null,null

hercules,demigod,30,null,null,cerburus,12,39,22,null,null,null,null

alcmene,human,45,null,null,null,null,null,null,null,null,null

pluto,god,4000,null,null,null,null,null,tarturus,no,fear of death,cerberus,neptune

pluto,god,4000,null,null,null,null,null,null,null,null,jupiter

nemean,monster,null,null,null,null,null,null,null,null,null,null

hydra,monster,null,null,null,null,null,null,null,null,null,null

cerberus,monster,null,null,null,null,null,null,tartarus,null,null,null 

```


### Sample JSON schema to ingest above data into JansgraphDB

This is the example of an input configuration for JanusGraph in JSON format:

```json
{
    "vertexCol": "name",
    "vartexLabelCol": "type",
    "vertices": [
        {
            "label": "titan",
            "properties": [
                "age"
            ]
        },
        {
            "label": "location",
            "properties": []
        },
        {
            "label": "god",
            "properties": [
                "age"
            ]
        },
        {
            "label": "demigod",
            "properties": [
                "age"
            ]
        },
        {
            "label": "human",
            "properties": [
                "age"
            ]
        },
        {
            "label": "monster",
            "properties": []
        }
    ],
    "propertyKeys": [
        {
            "name": "time",
            "dataType": "Integer",
	    "updateProperty": "false"
        },
        {
            "name": "name",
            "dataType": "String",
            "updateProperty": "false"
        },
        {
            "name": "age",
            "dataType": "Integer",
            "updateProperty": "false"
        },
        {
            "name": "reason",
            "dataType": "String",
            "updateProperty": "false"
        }
    ],
    "edges": [
        {
            "label": "father",
            "multiplicity": "MANY2ONE"
        },
        {
            "label": "mother",
            "multiplicity": "MANY2ONE"
        },
        {
            "label": "battled",
            "signature": [
                "time"
            ],
            "properties": [
                "time"
            ]
        },
        {
            "label": "lives",
            "signature": [
                "reason"
            ],
            "properties": [
                "reason"
            ]
        },
        
            "label": "pet"
        },
        {
            "label": "brother"
        }
    ],
    "indexes": [
        {
            "name": "name",
            "indexClass": "org.apache.tinkerpop.gremlin.structure.Vertex",
            "keys": [
                "name"
            ],
            "unique": true,
            "indexType": "Composite",
            "consistencyModifier": "LOCK"
        },
        {
            "name": "age",
            "indexClass": "org.apache.tinkerpop.gremlin.structure.Vertex",
            "keys": [
                "age"
            ],
            "indexType": "Mixed",
            "mixedIndexName": "search"
        },
        {
            "name": "edges",
            "indexClass": "org.apache.tinkerpop.gremlin.structure.Edge",
            "keys": [
                "reason"
            ],
            "indexType": "Mixed",
            "mixedIndexName": "search"
        }
    ]
}
```

### Validating ingested graph data using Gremlin Console

This is an example of a gremlin command. This command results the locations where Pluto's brothers live (refer [God of Graphs](https://docs.janusgraph.org/latest/getting-started.html) graph example) :
```
gremlin> tv.V(pluto).out('brother').out('lives').values('name')  
==>sky  
==>sea
```

This is an example of a gremlin command. This command results the Vertices having label as "demigod" (refer [God of Graphs](https://docs.janusgraph.org/latest/getting-started.html) graph example) :
```
gremlin> tv.V().has('vertexLabel','demigod').values('name')
==>hercules
```
