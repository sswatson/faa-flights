```rel
@inline def cnil[x] = last[hash[(:nil, x)]]
@inline def ccons[x, p] = last[hash[(:cons, x, p)]]

with my_graph use nil, element, tail, length, shortest_path, node, edge, show

def my_graph:node = 'a'; 'b'; 'c'; 'd'
def my_graph:edge = ('a', 'b'); ('c', 'c'); ('b', 'c'); ('a', 'c'); ('c', 'd'); ('d', 'a')

def my_graph:length[p] = 0, nil(_, p)
def my_graph:length[p] = 1 + length[tail[p]]

def my_graph:shortest_distance[x, y] = length[shortest_path[x, y]]

def my_graph:show[x in node] = string[x]
def my_graph:show[p] = show[x], nil(x, p) from x
def my_graph:show[p] = concat[concat[show[element[p]], " -> "], show[tail[p]]]

def my_graph = (:nil, x, cnil[x]) from x in node
def my_graph =
    (:shortest_path, x, y, ccons[x, p2]);
    (:element, ccons[x, p2], x);
    (:tail, ccons[x, p2], p2)
    from x, y, p2
    where p2 = argmin[length[p] for p where my_graph:candidate(x, y, p)]

def my_graph:candidate[x, y] = nil[y], edge(x, y)
def my_graph:candidate[x, y] = edge[x] . shortest_path[y]

def output[x, y] = length[shortest_path[x, y]], show[shortest_path[x, y]]
```