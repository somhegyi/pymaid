pymaid
==================

Collection of [Python](http://www.python.org) 3 tools to interface with [CATMAID](https://github.com/catmaid/CATMAID "CATMAID Repo") servers.

Pymaid has been tested with CATMAID latest release version 2017.04.20 - if you are working with older versions, you may experience incompatibilities.

`pymaid.pymaid` is the low-level library to connect to CATMAID servers and fetch data. Most data is returned as Pandas dataframe

`pymaid.igraph_catmaid` contains a wrapper to turn CATMAID skeletons in [iGraph](http://www.igraph.org) objects which can then be used to e.g. quickly calculate geodesic (along the arbor) distances and cluster synapses. 

`pymaid.morpho` contains wrappers to analyse or manipulate neuron morphology

`pymaid.plot` contains a wrapper to generate 2D or 3D morphology plots of neurons

`pymaid.cluster` contains wrappers to cluster neurons 

## Installation
I recommend using [Python Packaging Index (PIP)](https://pypi.python.org/pypi) to install pymaid.
First, get [PIP](https://pip.pypa.io/en/stable/installing/) and then run in terminal:  

`pip install git+git://github.com/schlegelp/pymaid@master`  

This command should also work to update the package.

*Attention*: on Windows, the dependencies (i.e. Numpy, Pandas and SciPy) will likely fail to install. Your best bet is to get a Python distribution that already includes them (e.g. [Anaconda](https://www.continuum.io/downloads)).

#### Dependencies:
`pymaid` requires [Pandas](http://pandas.pydata.org/)

`catmaid_igraph` requires [iGraph](http://www.igraph.org), [SciPy](http://www.scipy.org), [Numpy](http://www.scipy.org) and [Matplotlib](http://www.matplotlib.org)

`plot` requires [matplotlib](http://matplotlib.org/) and [Plotly](http://plot.ly)

`morpho` uses standard Python 3 libraries, [iGraph](http://www.igraph.org) is optional 

`cluster` requires [SciPy](http://www.scipy.org)

## Basic examples:

### Retrieve 3D skeleton data
```python
from pymaid.pymaid import CatmaidInstance, get_3D_skeleton

#Initialize Catmaid instance 
myInstance = CatmaidInstance( 'www.your.catmaid-server.org' , 'user' , 'password', 'token' )

#Retrieve skeleton data for two neurons
skdata = get_3D_skeleton ( ['12345','67890'] , myInstance )

#skdata is a Pandas dataframe. Each row holds a single neuron. Some examples:

#Access treenodes of the first neuron
nodes = skdata.ix[0].nodes

#Get coordinates of all connectors
coords = skdata.ix[0].connectors[ ['x','y','z'] ].as_matrix()

#Classify nodes into branches, leafs and slabs:
from pymaid.morpho import classify_nodes
df = classify_nodes( skdata.ix[0] )

#This new dataframe has a new column 'type'. Let's use this to get treenode ids for all branch points
branch_points = df.nodes[ df.nodes.type == 'branch' ].treenode_id

```
### Cluster synapses based on distance along the arbor using iGraph
```python
from pymaid.pymaid import CatmaidInstance, get_3D_skeleton
from pymaid.igraph_catmaid import igraph_from_skeleton, cluster_nodes_w_synapses

#Initiate Catmaid instance
remote_instance = CatmaidInstance( 'www.your.catmaid-server.org' , 'user' , 'password', 'token' )

#Example skid
skid = '12345'

#Retrieve 3D skeleton data for neuron of interest
skdata = get_3D_skeleton ( [ example_skid ], remote_instance, connector_flag = 1, tag_flag = 0 )

#(Optional) Consider downsampling for large neuronns (preverses branch points, end points, synapses, etc.)
from pymaid.morpho import downsample_neuron
ds_neuron = downsample_neuron( skdata.ix[0] , 4 )

#Generate iGraph object from node data
g = igraph_from_skeleton( ds_neuron )

#Cluster synapses - generates plot and returns clustering for nodes with synapses
syn_linkage = cluster_nodes_w_synapses( g, plot_graph = True )

#Find the last two clusters (= the two largest clusters):
clusters = cluster.hierarchy.fcluster( syn_linkage, 2, criterion='maxclust')

#Print summary
print('%i nodes total. Cluster 1: %i. Cluster 2: %i' % (len(clusters),len([n for n in clusters if n==1]),len([n for n in clusters if n==2])))
```

## Additional examples:
Check out [/examples/](https://github.com/schlegelp/PyMaid/tree/master/examples) for a growing list of Jupyter notebooks.

## Contents:
### pymaid.pymaid:
Currently **pymaid** features a range of wrappers to conveniently fetch data from CATMAID servers.
Use e.g. `help(get_edges)` to learn more about their function, parameters and usage.

- `add_annotations()`: use to add annotation(s) to neuron(s)
- `add_tags()`: add tags to treenodes and connectors
- `get_3D_skeleton()`: get neurons' skeleton(s) - i.e. what the 3D viewer in CATMAID shows
- `get_arbor()`: similar to get_3D_skeleton but more detailed information on connectors
- `get_annotations_from_list()`: get annotations of a set of neurons (annotation only)
- `get_connectors()`: get connectors (synapses, abutting and/or gap junctions) for set of neurons
- `get_connector_details()`: get details for connector (i.e. all neurons connected to it)
- `get_contributor_statistics()`: get contributors (nodes, synapses, etc) for a set of neurons
- `get_edges()`: get edges (connections) between sets of neurons
- `get_history()`: retrieve project history similar to the project statistics widget
- `get_logs()`: get what the log widged shows (merges, splits, etc.)
- `get_names()`: retrieve names of a set of skeleton IDs
- `get_neuron_annotation()`: get annotations of a **single** neuron (includes user and timestamp)
- `get_neurons_in_volume()`: get neurons in a defined box volume
- `get_node_lists()`: retrieve list of nodes within given volume
- `get_node_user_details()`: get details (creator, edition time, etc.) for individual nodes
- `get_partners()`: retrieve connected partners for a list of neurons
- `get_partners_in_volume()`: retrieve connected partners for a list of neurons within a given Catmaid volume
- `get_review()`: get review status for set of neurons
- `get_review_details()`: get review status (reviewer + timestamp) for each individual node
- `get_skids_by_annotation()`: get skeleton IDs that are annotated with a given annotation
- `get_skids_by_name()`: get skeleton IDs of neurons with given names
- `get_skeleton_list()`: retrieve neurons that fit certain criteria (e.g. user, size, dates)
- `get_user_list()`: get list of users in the project
- `get_volume()`: get volume (verts + faces) of CATMAID volumes
- `skid_exists()`: checks if a skeleton ID exists

### pymaid.igraph_catmaid:
- `calculate_distance_from_root()`: calculates geodesic (along-the-arbor) distances for nodes to root node
- `cluster_nodes_w_synapses()`: uses iGraph's `shortest_paths_dijkstra` to cluster nodes with synapses
- `igraph_from_adj_matrix()`: generates iGraph representation of network
- `igraph_from_skeleton()`: generates iGraph representation of neuron morphology

### pymaid.plot:
- `plot2d()`: generates 2D plots of neurons
- `plot3d()`: uses [Plotly](http://plot.ly) to generate 3D plots of neurons
- `plot_network()`: uses iGraph and [Plotly](http://plot.ly) to generate network plots

### pymaid.cluster:
- `create_adjacency_matrix()`: create a Pandas dataframe containing the adjacency matrix for two sets of neurons
- `create_connectivity_distance_matrix()`: returns distance matrix based on connectivity similarity (Jarrell et al., 2012)
- `synapse_distance_matrix()`: cluster synapses based on eucledian distance

### pymaid.morpho:
- `calc_cable()`: calculate cable length of given neuron
- `calc_strahler_index()`: calculate strahler index for each node
- `cut_neuron()`: virtually cuts a neuron at given treenode and returns the distal and the proximal part
- `classify_nodes()`: adds a new column to a neuron's dataframe categorizing each node as branch, slab, leaf or root
- `cut_neuron2()`: similar to above but uses iGraph (slightly faster)
- `downsample_neuron()`: takes skeleton data and reduces the number of nodes while preserving synapses, branch points, etc.
- `in_volume()`: test if points are within given CATMAID volume
- `synapse_root_distances()`: similar to `pymaid.igraph_catmaid.calculate_distance_from_root()` but does not use iGraph

### pymaid.user_stats:
- `get_time_invested()`: calculate the time users have spent working on a set of neurons


## License:
This code is under GNU GPL V3
