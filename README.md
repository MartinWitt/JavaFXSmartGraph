# (JavaFX) SmartGraph

This project provides a generic (Java FX) **graph visualization library** that can automatically arrange the vertices' locations
through a [force-directed algorithm](https://en.wikipedia.org/wiki/Force-directed_graph_drawing) in real-time.

![smartgraph realtime](examples/smartgraph_realtime.gif)

## Features

- The visualization library can be used together with any abstract data type (ADT) that adheres to the `Graph<V,E>` or `Digraph<V,E>` interfaces. Default implementations are included, but you can devise your own;

- Instead of the force-directed automatic layout, vertices can be statically placed according to other algorithms and at *runtime*;

- *Actions* can be performed when clicking the vertices and/or the edges;

- Vertices can be freely moved, if the configuration allows it;

- The appearance of vertices and edges can are determined by a *properties* and a *css stylesheet* files;

- The *styles* can be changed at *runtime*.

## Documentation

### What's **new**?

- (1.0.0) Package now available through [Maven Central](https://central.sonatype.com/namespace/com.brunomnsilva). The library seems stable, after dozens of college projects of my students have used it. Hence, the version was bumped to 1.0.0.

- (0.9.4) You can now annotate a method with `@SmartLabelSource` within a model class to provide the displayed label for a vertex/edge; see the example at `com.brunomnsilva.smartgraph.example`. If no annotation is present, then the `toString()` method is used to obtain the label's text.

- (0.9.4) You can manually alter a vertex position on the panel at anytime, through `SmartGraphPanel.setVertexPosition(Vertex<V> v)`; see the example at `com.brunomnsilva.smartgraph.example`.

- (0.9.4) You can override specific default properties by using a *String* parameter to the `SmartGraphProperties` constructor; see the example at `com.brunomnsilva.smartgraph.example`. This is useful if you want to display visually different graphs within the same application.

- (0.9.4) You can now style labels and arrows individually.

### Using the library
 
The library is available through Maven Central. The coordinates are:

```xml
<dependency>
    <groupId>com.brunomnsilva</groupId>
    <artifactId>smartgraph</artifactId>
    <version>1.0.0</version>
</dependency>
```

:warning: Please note that the files `smartgraph.css` and `smartgraph.properties` **must be added manually** to your project.

:bulb: You should be able to use the artifact with Maven and Gradle projects with little effort. 

You can also find in the [releases section](https://github.com/brunomnsilva/JavaFXSmartGraph/releases) the binaries, source code and documentation.

### Basic usage

```java
// Create the graph
Graph<String, String> g = new GraphEdgeList<>();
// ... see Examples below

SmartPlacementStrategy strategy = new SmartCircularSortedPlacementStrategy();
SmartGraphPanel<String, String> graphView = new SmartGraphPanel<>(g, strategy);
Scene scene = new Scene(graphView, 1024, 768);

Stage stage = new Stage(StageStyle.DECORATED);
stage.setTitle("JavaFXGraph Visualization");
stage.setScene(scene);
stage.show();

//IMPORTANT! - Called after scene is displayed, so we can initialize the graph visualization
graphView.init();
```

This will display the graph using the instantiated placement `strategy`. You can **create your own static placement strategies** by implementing the `SmartPlacementStrategy` *interface* (see included examples).

```java
// Even with a static initial placement of vertices, we can toggle automatic layout
// at any time, e.g.:
graphView.setAutomaticLayout(true);
```

### API Reference

[![javadoc](https://javadoc.io/badge2/com.brunomnsilva/smartgraph/javadoc.svg)](https://javadoc.io/doc/com.brunomnsilva/smartgraph)


## Examples

Below are provided some graphs and the corresponding visualization, either using a static placement strategy or by the automatic force-directed layout algorithm.

### Sample Graph

The following code creates a sample graph:

```java
Graph<String, String> g = new GraphEdgeList<>();

g.insertVertex("A");
g.insertVertex("B");
g.insertVertex("C");
g.insertVertex("D");
g.insertVertex("E");
g.insertVertex("F");
g.insertVertex("G");

g.insertEdge("A", "B", "1");
g.insertEdge("A", "C", "2");
g.insertEdge("A", "D", "3");
g.insertEdge("A", "E", "4");
g.insertEdge("A", "F", "5");
g.insertEdge("A", "G", "6");

g.insertVertex("H");
g.insertVertex("I");
g.insertVertex("J");
g.insertVertex("K");
g.insertVertex("L");
g.insertVertex("M");
g.insertVertex("N");

g.insertEdge("H", "I", "7");
g.insertEdge("H", "J", "8");
g.insertEdge("H", "K", "9");
g.insertEdge("H", "L", "10");
g.insertEdge("H", "M", "11");
g.insertEdge("H", "N", "12");

g.insertEdge("A", "H", "0");
```

#### Sample Graph circular sorted placement (static)

![graph circular placement](examples/graph_circle_placement.png)

#### Sample Graph automatic layout

![graph automatic layout](examples/graph_automatic_layout.png)

### Sample Digraph (directed graph)

The following code creates a sample digraph:

```java
Digraph<String, String> g = new DigraphEdgeList<>();

g.insertVertex("A");
g.insertVertex("B");
g.insertVertex("C");
g.insertVertex("D");
g.insertVertex("E");
g.insertVertex("F");

g.insertEdge("A", "B", "AB");
g.insertEdge("B", "A", "AB2");
g.insertEdge("A", "C", "AC");
g.insertEdge("A", "D", "AD");
g.insertEdge("B", "C", "BC");
g.insertEdge("C", "D", "CD");
g.insertEdge("B", "E", "BE");
g.insertEdge("F", "D", "DF");
g.insertEdge("F", "D", "DF2");

//yep, its a loop!
g.insertEdge("A", "A", "Loop");
```

Please note that we use the property values `edge.arrow = true` and `vertex.label = true`.

Given it's a small graph, we increased the `layout.repulsive-force = 25000`. You should use higher values for smaller graphs; inversely, use smaller values for larger graphs.

> See Configuration and Styling section.
> 
#### Sample Digraph circular sorted placement (static)

![digraph circular placement](examples/digraph_circle_placement.png)

#### Sample Digraph automatic layout

![digraph automatic layout](examples/digraph_automatic_layout.png)

### Updating the view

When you make changes to the graph, you must update the visualization by calling `SmartGraphPanel#update()` or `SmartGraphPanel#updateAndWait()`. The latter version "blocks" the caller thread until the visualization has updated.

```java
// Any number of changes to the graph
g.removeVertex("A");
//...
        
// Update the visualization
graphView.update();
```

this will add/remove the corresponding vertices and edges from the visualization. If a new vertex is connected to an existing one, it will be initially placed in the vicinity of the latter. Otherwise, if it is an *isolated* vertex it will be placed in the center of the plot.

### Responding to click events on vertices and/or edges

You can attach actions with:

```java
graphView.setVertexDoubleClickAction(graphVertex -> {
    System.out.println("Vertex contains element: " + graphVertex.getUnderlyingVertex().element());
});

graphView.setEdgeDoubleClickAction(graphEdge -> {
    System.out.println("Edge contains element: " + graphEdge.getUnderlyingEdge().element());
    //dynamically change the style, can also be done for a vertex
    graphEdge.setStyle("-fx-stroke: black; -fx-stroke-width: 2;");
});
```

These actions will be performed whenever you *double-click* a vertex and/or an edge.

## Configuration and Styling

### SmartGraph Properties

:warning: The `smartgraph.properties` file must exist in you project folder.

You can set the graph visualization properties in the `smartgraph.properties` file:

```properties
# Vertex related configurations
#
vertex.allow-user-move = true
vertex.radius = 15 
vertex.tooltip = true
vertex.label = false

# Edge related configurations
#
edge.tooltip = true
edge.label = false
# only makes sense if displaying an oriented graph 
edge.arrow = false

# (automatic) Force-directed layout related configurations
#   -- You should experiment with different values for your 
#   -- particular problem, knowing that not all will achieve 
#   -- a stable state
layout.repulsive-force = 25000
layout.attraction-force = 30
layout.attraction-scale = 10
```

### SmartGraph CSS styling

:warning: Either using the library or including the source code, the `smartgraph.css` file must exist in you project folder. 
For example, if all your vertices turn out black, this is because this file cannot be found.

You can set the default CSS styles in the `smartgraph.css` file; see below.

The style classes `graph`, `vertex`, `vertex-label`, `edge`, `edge-label` and `arrow` contain the default styling for the nodes and must exist in the file. You can, however, provide different classes to apply to specific elements, e.g., vertices, edges and labels.

The following example contains an additional definition for the CSS class `.myVertex`. You can apply the style individually to an existing element, e.g., vertex, as follows:

```java
graphView.getStylableVertex("A").setStyleClass("myVertex");
```

```css
.graph {
    -fx-background-color: #F4FFFB;
}

.vertex {
    -fx-stroke-width: 3;
    -fx-stroke: #61B5F1;
    -fx-stroke-type: inside; /* you should keep this for vertex.radius to hold */
    -fx-fill: #B1DFF7;
}

.vertex:hover { /* pseudo-classes also work */
    /*-fx-cursor: default; */ /* You can use this style to override the hand/move cursors while hovering. */
    -fx-stroke-width: 4;
}

.vertex-label {
    -fx-font: bold 8pt "sans-serif";
}

.edge {
    -fx-stroke-width: 2;
    -fx-stroke: #FF6D66;  
    -fx-stroke-dash-array: 2 5 2 5; /* remove for clean edges */  
    -fx-fill: transparent; /*important for curved edges. do not remove */
    -fx-stroke-line-cap: round;
    -fx-opacity: 0.8;
}

.edge-label {
    -fx-font: normal 5pt "sans-serif";
}

.arrow {
    -fx-stroke-width: 2;
    -fx-stroke: #FF6D66;  
    -fx-opacity: 0.8;
}

/* Custom vertex. You should revert any unwanted styling in the default element, 
   style, since custom styles will be appended to the default style */
.myVertex {
    -fx-stroke-width: 5;
    -fx-stroke: green;
    -fx-stroke-type: inside; /* you should keep this if using arrows */
    -fx-fill: yellowgreen;
}
```



## Contributing

You can submit a pull request. Pull requests should adhere to the existing naming and *Javadoc* conventions.

:star: Also, if you use the library in your project, you can send me a message and I can showcase it in this page.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE.txt) file for details. **All derivative work should include this license**.

## Authors

Original author: **Bruno Silva** - [(GitHub page)](https://github.com/brunomnsilva) | [(Personal page)](https://www.brunomnsilva.com/)

---

I hope you find SmartGraph useful and look forward to seeing the projects you create with it!
