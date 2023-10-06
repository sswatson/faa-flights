# Generic Shortest Path

```rel

entity type Path:nil
entity type Path:cons = Char, Entity

def Graph = my_graph

@inline def nil_graph[E] =
    (E, ^Path:nil);
    (:Path, ^Path:nil);
    (:nil, ^Path:nil)

@inline def cons_graph[elem, tail in Graph:Path, E] =
    (E, ^Path:cons[elem, tail]);
    (:Path, ^Path:cons[elem, tail]);
    (:element, ^Path:cons[elem, tail], elem);
    (:tail, ^Path:cons[elem, tail], tail)

def length[x in Graph:Path] = 0, ^Path:nil(x)
def length[x in Graph:Path] = 1 + length[get_tail[x]]

def get_tail(x, p) = ^Path:cons(e, p, x) from e in Graph:node


def shortest_path =
    nil_graph[(:shortest_path, x, x)] from x in Graph:node

def shortest_path =
    cons_graph[x, p2, (:shortest_path, x, y)],
    argmin[length[p] for p in candidate[x, y]](p2)
    from x, y, p2

def candidate[x, y] = Graph:edge[x] . Graph:shortest_path[y], x != y

module my_graph
    with my_graph use shortest_path

    def node = 'a'; 'b'; 'c'; 'd'
    def edge = ('a', 'b'); ('b', 'c'); ('a', 'c'); ('c', 'd'); ('d', 'a')
    def shortest_distance[x, y] = length[shortest_path[x, y]]
end

 // inject shortest path into the graph
def my_graph = shortest_path

def output = my_graph
```
