```rel
def labelled_edge =
    "craft", "Flight", "Aircraft"; "destination", "Flight", "Airport"; "located_in", "Airport", "City"; "located_in", "Airport", "County";
    "located_in", "Airport", "State"; "located_in", "City", "County"; "located_in", "City", "State"; "located_in", "County", "State";
    "model", "Aircraft", "AircraftModel"; "operated_by", "Flight", "Carrier"; "operates", "Carrier", "Flight"; "origin", "Flight", "Airport"

@inline def create_node(xs..., h) = hash[(xs...)](xs..., h)

def sg = (:node, h); (:label, h, x) from x, h where (labelled_edge(_, x, _) or labelled_edge(_, _, x)) and create_node(x, h)
def sg = (:node, h); (:label, h, x); (:edge, yh, h); (:edge, h, zh)
    from x, y, z, h, yh, zh where labelled_edge(x, y, z) and create_node(x, y, z, h) and create_node(y, yh) and create_node(z, zh)

@inline def cnil[x] = last[hash[(:nil, x)]]
@inline def ccons[x, p] = last[hash[(:cons, x, p)]]

with my_graph use nil, element, tail, length, shortest_path, node, edge, show

def my_graph:node = sg:node
def my_graph:edge = sg:edge
def my_graph:length[p] = 0, nil(_, p)
def my_graph:length[p] = 1 + length[tail[p]]
def my_graph:shortest_distance[x, y] = length[shortest_path[x, y]]
def my_graph:show[x in node] = sg:label[x]
def my_graph:show[p] = show[x], nil(x, p) from x
def my_graph:show[p] = concat[concat[show[element[p]], " -> "], show[tail[p]]]

def my_graph = (:nil, x, cnil[x]) from x in node
def my_graph = (:shortest_path, x, y, ccons[x, p2]);
    (:element, ccons[x, p2], x);
    (:tail, ccons[x, p2], p2)
    from x, y, p2
    where p2 = argmin[length[p] for p where my_graph:candidate(x, y, p)]

def my_graph:candidate[x, y] = nil[y], edge(x, y)
def my_graph:candidate[x, y] = edge[x] . shortest_path[y]

def output[x, y] = length[shortest_path[x, y]], show[shortest_path[x, y]], sg:label(x, "Flight"), sg:label(y, "State")
```