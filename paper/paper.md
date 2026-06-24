---
title: 'RGraphSpace: A lightweight interface between igraph and ggplot2 graphics'
tags:
- R
- graph visualization
- ggplot2
- igraph
- network analysis
- spatial visualization
date: "`r Sys.Date()`"
output:
  html_document:
    df_print: paged
authors:
- name: Flávio Gabriel Carazza Kessler
  orcid: "0000-0002-5309-8043"
  affiliation: '1'
- name: Jonathan André Back
  orcid: "0009-0008-7338-1197"
  affiliation: '1'
- name: Lana Bazan Peters Querne
  orcid: "0000-0001-9967-028X"
  affiliation: '1'
- name: Victor Henrique Apolonio dos Santos
  orcid: "0000-0002-6394-5840"
  affiliation: '1'
- name: Vinícius de Saraiva Chagas
  orcid: "0000-0001-5160-0450"
  affiliation: '1'
- name: Mauro Antônio Alves Castro
  orcid: "0000-0003-4942-8131"
  corresponding: true
  affiliation: '1'
bibliography: paper.bib
affiliations:
- name: "Bioinformatics and Systems Biology Laboratory, Federal University of Paraná,
    Curitiba, Paraná, 81520-260, Brazil"
  index: 1
---

# Summary

Network visualization is a fundamental component of computational biology and systems science research. While R offers powerful tools for network analysis through `igraph` [@Nepusz:2006] and sophisticated plotting capabilities through `ggplot2` [@Wickham:2016], integrating these frameworks for spatial network visualization has remained challenging. `RGraphSpace` addresses this gap by providing a lightweight interface that seamlessly integrates `igraph` objects with `ggplot2` graphics within a normalized coordinate system. The package implements new geometric objects using `ggplot2` prototypes, specifically customized for side-by-side visualization of multiple graphs. By scaling shapes and graph elements to fit within a standardized unit space, `RGraphSpace` enables layered visualizations with consistent spatial alignment, making it particularly valuable for comparative network analysis and spatial mapping applications.

# Statement of need

Researchers working with network data frequently need to visualize graphs in spatial contexts, such as overlaying networks on geographical maps, comparing multiple networks side-by-side, or creating layered visualizations where network topology is constrained to specific spatial regions. While `igraph` provides comprehensive network analysis capabilities and various layout algorithms, its native plotting functions lack the flexibility and aesthetic control offered by modern `ggplot2` graphics. Conversely, while `ggplot2` excels at creating publication-quality visualizations with extensive customization options, it does not natively support network graph objects.

Existing solutions typically require researchers to manually convert `igraph` objects into data frames, calculate edge coordinates, and handle scaling issues—a process that is both time-consuming and error-prone. Furthermore, when working with multiple graphs or attempting to overlay networks on background images, maintaining consistent scaling and spatial alignment becomes increasingly complex.

`RGraphSpace` was designed to solve these challenges by providing:

1.  **Seamless integration**: Direct conversion of `igraph` objects into `ggplot2`-compatible graphics without manual data transformation
2.  **Normalized coordinate system**: Automatic scaling of graph elements to fit within a standardized unit space (0-1), enabling consistent visualization across multiple graphs
3.  **Spatial awareness**: Support for background images with proper coordinate mapping, allowing networks to be overlaid on spatial maps or other contextual imagery
4.  **Flexible customization**: Full access to `ggplot2`'s extensive theming and aesthetic options while maintaining network-specific attributes
5.  **Edge and node scaling**: Intelligent handling of edge arrows, node shapes, and other graph elements that scale appropriately with the visualization space
6.  **Interoperability with other packages**: Designed to be an extension for existing network analysis workflows, not a replacement for it.

The package is particularly valuable for researchers in systems biology, neuroscience, social network analysis, and any field requiring visualization of networks within spatial contexts. By reducing the technical barriers to creating sophisticated network visualizations, `RGraphSpace` enables researchers to focus on their scientific questions rather than visualization implementation details.

# State of the field

Several R packages address network visualization, each with different strengths and limitations:

-   **`igraph`** [@Nepusz:2006]: Provides comprehensive network analysis and basic plotting capabilities but lacks the aesthetic flexibility and customization options of modern visualization frameworks. Its plotting system is based on base R graphics, which can be limiting for publication-quality figures.

-   **`ggraph`** [@Pedersen:2025]: Extends `ggplot2` to support network visualizations and offers excellent integration with the tidyverse ecosystem. However, it is primarily designed for standalone network plots and does not emphasize spatial mapping or the normalization of coordinates for multi-graph comparisons. `ggraph` excels at traditional network layouts (force-directed, hierarchical, etc.) but is less suited for situations where network coordinates are predetermined by spatial constraints.

-   **`RedeR`** [@Castro:2012]: An R/Bioconductor package that provides interactive network visualization with support for nested networks and hierarchical structures. While `RedeR` offers rich interactivity through a Java-based interface, it operates in a separate visualization environment rather than integrating with `ggplot2`. `RGraphSpace` complements `RedeR` by providing static, publication-ready visualizations using familiar `ggplot2` syntax.

-   **`visNetwork`** [@Almende:2025]: Offers interactive network visualizations using the vis.js JavaScript library, but focuses on web-based interactivity rather than static publication graphics and spatial integration.

`RGraphSpace` occupies a unique niche by specifically targeting use cases where networks must be visualized within spatial contexts (e.g., overlaid on maps or images) while maintaining the power and flexibility of `ggplot2`. Its normalization approach makes it particularly well-suited for comparative visualizations where multiple graphs need to be displayed with consistent scaling.

# Software design


As shown in \autoref{fig:network-layout}, RGraphSpace maps graph elements to the image coordinate system.

![**Figure 1. Graph layout generated using RGraphSpace.** []{label="fig:network-layout"}](figures/Architecture_transparent_highres.png)



`RGraphSpace` is built around an S4 class system that wraps `igraph` objects and manages their transformation into `ggplot2`-compatible data structures. The core workflow involves:

1.  **Graph preprocessing**: The `GraphSpace()` constructor accepts an `igraph` object with `x`, `y`, and `name` vertex attributes, along with optional layout matrices or background images. The constructor normalizes all coordinates to a unit space and validates graph attributes.

2.  **Attribute management**: The package recognizes standard `igraph` vertex attributes (e.g., `nodeSize`, `nodeShape`, `nodeColor`, `nodeLineColor`, `nodeLineWidth`) and edge attributes (e.g., `edgeLineWidth`, `edgeLineColor`, `edgeLineType`, `arrowType`, `arrowLength`) and automatically maps them to appropriate `ggplot2` aesthetics.

3.  **Geometric objects**: `RGraphSpace` implements custom `ggproto` objects that extend `ggplot2`'s geom system. Three specialized `geoms` translate graph data into geometric layers. These `geoms` use a dual-anchor normalization approach to align layers, required for analysis where network elements must be accurately referenced to a spatial map.

-   `geom_graphspace()`: A high-level layer that processes both nodes and edges in a single call.
-   `geom_nodespace()`: Dedicated to rendering nodes. Inherits `GeomPoint` aesthetic mappings, modified to inform the edge layer on node states. It can be used with the `inject_nodespace()` function to handle the scaling of node shapes to ensure they maintain consistent size relative to the plot space.
-   `geom_edgespace()`: Handles the relational data between nodes. Inherits GeomSegment aesthetic mappings; unlike standard segments, it is “node-aware” and dynamically calibrates start and end points.

4.  **Edge rendering**: The package includes sophisticated edge rendering that handles both directed and undirected graphs, calculates appropriate arrow coordinates, and supports self-loops and multiple edges between the same vertices.

5.  **Theming system**: Multiple pre-configured themes (`th0`, `th1`, `th2`, `th3`) provide different aesthetic styles while remaining fully customizable through standard `ggplot2` theme functions.

The package also provides accessor methods compatible with common `igraph` functions, allowing users to query and modify graph attributes without breaking the `GraphSpace` object structure. This design enables an intuitive workflow where users can leverage their existing `igraph` knowledge while gaining access to `ggplot2`'s visualization capabilities.

# Example Usage

## Example 1

::: Usar exemplo com dado espacial (mapas):::

## Example 2

::: Usar exemplo com dado de alta dimensionalidade (Seurat) :::

encerrar com painel das possibilidades de uso do RGraphSpace

# Research impact statement

`RGraphSpace` has been designed to support a wide range of research applications in computational biology and network science. The package enables researchers to create sophisticated visualizations that combine network topology with spatial information, facilitating the exploration of relationships between network structure and physical or conceptual space.

Potential applications include:

-   **Systems biology**: Overlaying molecular interaction networks on cellular compartment maps or tissue sections
-   **Neuroscience**: Visualizing brain connectivity networks in anatomical space
-   **Spatial transcriptomics**: Integrating gene regulatory networks with spatial expression data
-   **Social networks**: Mapping social connections within geographical contexts
-   **Ecological networks**: Displaying species interaction networks over habitat maps

The package's ability to handle multiple graphs with consistent scaling also makes it valuable for temporal analyses, where network evolution can be visualized across time points or experimental conditions.

# Availability and documentation

`RGraphSpace` is available on CRAN and can be installed using standard R package installation procedures. The development version is hosted on GitHub at <https://github.com/sysbiolab/RGraphSpace>. Comprehensive documentation is provided through:

-   *A detailed vignette demonstrating the complete workflow from `igraph` object creation to customized `ggplot2` visualizations*
-   Function-level documentation accessible through R's help system
-   *Example datasets (`gtoy1`, `gtoy2`) that facilitate learning and testing*

The package requires R ≥4.5 and depends on `igraph`, `ggplot2`, `methods`, and several utility packages (`grDevices`, `scales`, `grid`). Continuous integration testing ensures compatibility across platforms. Moreover, all current and futures vignettes of the package can be found in [Get Started](https://sysbiolab.github.io/RGraphSpace/articles/get-started.html) section from `RGraphSpace's` GitHub.

# Citations

If the reader want to cite **RGraphSpace**, is possible to cite this manuscript and also the publication on CRAN:

```         
@Manual{SysbiolabTeam:2026, 
title = {RGraphSpace: A lightweight interface between ‘igraph’ and ‘ggplot2’ graphics}, 
author = {Sysbiolab Team},
year = {2026},
note = {R package version 1.2.0}, 
organization = {CRAN}, 
doi = {10.32614/CRAN.package.RGraphSpace}, 
url = {https://cran.r-project.org/web/packages/RGraphSpace/index.html}
}
```
# AI usage disclosure

During this work's preparation, ChatGPT (OpenAI) was used by the authors to improve the comprehensibility of the R package’s documentation, using RStudio Desktop (<https://posit.co/>). The authors carefully reviewed and polished the content as needed after using this tool/service and assume full responsibility for the published content.

# Acknowledgements

We acknowledge the broader R community, particularly the developers of `igraph` and `ggplot2`, whose excellent software made this integration possible. This work was funded by CNPq (316622/2021-4; 440412/2022-6), CAPES (88882.632783/2021-01), and Fundação Araucária (NAPI Bioinformática) and supported by the Bioinformatics and Systems Biology Laboratory at the Federal University of Paraná, Brazil.

# References
