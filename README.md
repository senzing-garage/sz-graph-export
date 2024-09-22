# sz-graph-export

## Overview

This is a python utility used to export senzing entities in networkx json format for loading graph databases.

### Prerequisites
- Python 3.6 or higher
- Senzing API version 3.0 or higher

### Installation

Place the following file in a directory of your choice:
- [sz_graph_export.py](sz_graph_export.py)

*Note: This utility must be run within a Senzing environment set to your project!*

## Usage

```console
usage: sz_graph_export.py [-h] [-o OUTPUT_PATH] [-S] [-D]

optional arguments:
  -h, --help            show this help message and exit
  -o OUTPUT_PATH, --output_path OUTPUT_PATH
                        Path to desired output files including base file name if desired as multiple files are created
  -S, --include_source_nodes
                        Only resolved entity nodes and relationships are exported by default
```

## Typical use

```console
sz_graph_export.py -o /project/export/foo_
```

This will export a networkx compliant json file named **/project/export/foo_senzing_graph.json**

*Ideally, the graph database already contains the source nodes and their relationships and this export is used to add the resolved entity nodes and their relationships to source nodes and each other to it.*

However, the -S argument can be used if you want to export the source nodes from Senzing.  If you add this an additional file named **/project/source/foo_source_graph.json** will be created.


## Structure of Senzing nodes and edges

Notice the Senzing graph is not directed.  This is because edges between two senzing entities should be considered bi-directional. The fact that they are sharing an address has no real direction.
```console
{
    "directed": false,
    "multigraph": false,
    "graph": {},
```
A Senzing node only contains enough attributes to help display and query the graph.  For instance, this node is for an "entity" that is an "organization" with 3 source nodes: 2 from "SOURCE1" and 1 from "SOURCE3".

- Use node_class to distinguish Senzing nodes from other nodes.
- Use node_type to determine which icon to use.
- Use record_count to help size your node.
- Use the data_sources to help determine the color of your node.  For instance, if SOURCE2 is for a watch list, this node might be red.
- A graph query should be able to quickly find ...
	- entities that contain multiple records
	- entities that contain duplicate SOURCE1 records
	- entities that have both SOURCE1 and SOURCE2 records

```console
    "nodes": [
        {
            "id": "SENZING:225",
            "node_class": "ENTITY",
            "node_type": "ORGANIZATION",
            "node_name": "ABC Company",
            "entity_id": 225,
            "record_count": 3,
            "data_sources": [
                {
                    "SOURCE1": 2
                },
                {
                    "SOURCE2": 1
                }
            ],
```
A Senzing EDGE contains enough attributes to display, filter and query relationships detected by Senzing.
- Use edge_class / edge_type to color, filter or query nodes on your graph.  For instance,
	- Resolved edges are higher confidence then possible matches or relationships
	- Easily remove all possibly related nodes from a graph display
- Use edge_type for a succint edge label
- Use edge_details to display the Senzing match_key on a hover over the edge.
```console
    "links": [
        {
            "edge_class": "Resolved",
            "edge_type": "Resolved to",
            "edge_details": "Use Senzing's whyRecordInEntity call to get current details",
            "source": "SOURCE1-1001",
            "target": "SENZING:225"
        },
    ...
        {
            "edge_class": "Derived",
            "edge_type": "Possibly related",
            "edge_details": "+ADDRESS",
            "source": "SENZING:9",
            "target": "SENZING:225"
        },
