# Textplot

# Python文本网络分析工具包textplot

<sup><a href="http://textplot.s3-website-us-west-1.amazonaws.com/#mental-maps/war-and-peace" target="_new">**_War and Peace_**</a> (click to zoom)</sup>

<a href="http://textplot.s3-website-us-west-1.amazonaws.com/#mental-maps/war-and-peace" target="_new">![War and Peace](notes/mental-maps/networks/war-and-peace.jpg)</a>

**Textplot** is a little program that converts a document into a network of terms, with the goal of teasing out information about the high-level topic structure of the text. For each term:

1. Get the set of offsets in the document where the term appears.

1. Using [kernel density estimation](http://en.wikipedia.org/wiki/Kernel_density_estimation), compute a probability density function (PDF) that represents the word's distribution across the document. Eg, from _War and Peace_:

  ![War and Peace](notes/mental-maps/figures/war.png)

1. Compute a [Bray-Curtis](http://en.wikipedia.org/wiki/Bray%E2%80%93Curtis_dissimilarity) dissimilarity between the term's PDF and the PDFs of all other terms in the document. This measures the extent to which two words appear in the same locations.

1. Sort this list in descending order to get a custom "topic" for the term. Skim off the top N words (usually 10-20) to get the strongest links. Here's "napoleon":

  ```bash
  [('napoleon', 1.0),
  ('war', 0.65319871313854128),
  ('military', 0.64782349297012154),
  ('men', 0.63958189887106576),
  ('order', 0.63636730075877446),
  ('general', 0.62621616907584432),
  ('russia', 0.62233286026418089),
  ('king', 0.61854160459241103),
  ('single', 0.61630514751638699),
  ('killed', 0.61262010905310182),
  ('peace', 0.60775702746632576),
  ('contrary', 0.60750138486684579),
  ('number', 0.59936009740377516),
  ('accompanied', 0.59748552019874168),
  ('clear', 0.59661288775164523),
  ('force', 0.59657370362505935),
  ('army', 0.59584331507492383),
  ('authority', 0.59523854206807647),
  ('troops', 0.59293965397478188),
  ('russian', 0.59077308177196441)]
  ```

1. Shovel all of these links into a network and export a GML file.

## Generating graphs

There are two ways to create graphs - you can use the `textplot` executable from the command line, or, if you want to tinker around with the underlying NetworkX graph instance, you can fire up a Python shell and use the `build_graph()` helper directly.

Either way, first install Textplot. With PyPI:

`pip install textplot`

Or, clone the repo and install the package manually:

```bash
pyvenv env
. env/bin/activate
pip install -r requirements.txt
python setup.py install
```

### From the command line

Then, from the command line, generate graphs with:

`texplot generate [IN_PATH] [OUT_PATH] [OPTIONS]`

Where the input is a regular `.txt` file, and the output is a [`.gml`](http://en.wikipedia.org/wiki/Graph_Modelling_Language) file. So, if you're working with _War and Peace_:

`texplot generate war-and-peace.txt war-and-peace.gml`

The `generate` command takes these options:

- **`--term_depth=1000` (int)** - The number of terms to include in the network. For now, Textplot takes the top N most frequent terms, after stopwords are removed.

- **`--skim_depth=10` (int)** - The number of connections (edges) to skim off the top of the "topics" computed for each word.

- **`--d_weights` (flag)** - By default, terms that appear in similar locations in the document will be connected by edges with "heavy" weights, the semantic expected by force-directed layout algorithms like Force Atlas 2 in Gephi. If this flag is passed, the weights will be inverted - use this if you want to do any kind of pathfinding analysis on the graph, where it's generally assumed that edge weights represent _distance_ or _cost_.

- **`--bandwidth=2000` (int)** - The [bandwidth](http://en.wikipedia.org/wiki/Kernel_density_estimation#Bandwidth_selection) for the kernel density estimation. This controls how "smoothness" of the curve. 2000 is a sensible default for long novels, but bump it down if you're working with shorter texts.

- **`--samples=1000` (int)** - The number of equally-spaced points on the X-axis where the kernel density is sampled. 1000 is almost always enough, unless you're working with a huge document.

- **`--kernel=gaussian` (str)** - The kernel function. The [scikit-learn implementation](http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.KernelDensity.html) also supports `tophat`, `epanechnikov`, `exponential`, `linear`, and `cosine`.

### From a Python shell

Or, fire up a Python shell and import `build_graph()` directly:

```bash
In [1]: from textplot.helpers import build_graph

In [2]: g = build_graph('war-and-peace.txt')

Tokenizing text...
Extracted 573064 tokens

Indexing terms:
[################################] 124750/124750 - 00:00:06

Generating graph:
[################################] 500/500 - 00:00:03
```

`build_graph()` returns an instance of `textplot.graphs.Skimmer`, which gives access to an instance of `networkx.Graph`. Eg, to get degree centralities:

```bash
In [3]: import networkx as nx
In [4]: nx.degree_centrality(g.graph)
```

---

Texplot uses **[numpy](http://www.numpy.org)**, **[scipy](http://www.scipy.org)**, **[scikit-learn](http://scikit-learn.org)**, **[matplotlib](http://matplotlib.org)**, **[networkx](http://networkx.github.io)**, and **[clint](https://github.com/kennethreitz/clint)**.
