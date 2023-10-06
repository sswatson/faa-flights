
# Rel By Example

## `SELECT`

```rel
with flight_graph use airport_code, airport_name,
    located_in, airport_region, facility_type,
    airport_elevation, State
def select_airports =
    (:code, id, airport_code[id]);
    (:name, id, airport_name[id]);
    (:state, id, (located_in[id] :> State));
    (:region, id, airport_region[id]);
    (:facility_type, id, facility_type[id]);
    (:elevation, id, airport_elevation[id])
    from id
    
def output = table[select_airports]
```

## `ORDER BY`

```rel
with flight_graph use airport_code, airport_name,
    located_in, airport_region, facility_type,
    airport_elevation, State

def select_airports =
    (:code, id, airport_code[id]);
    (:name, id, airport_name[id]);
    (:state, id, (located_in[id] :> State));
    (:region, id, airport_region[id]);
    (:facility_type, id, facility_type[id]);
    (:elevation, id, airport_elevation[id])
    from id

def airports_sorted_by_code[col, i, k, v] =
    sort[airport_code[_]](i, code) and
    airport_code(k, code) and
    select_airports(col, k, v)
    from code

def output = table[airports_sorted_by_code]
```

## `GROUP BY`
```rel
with flight_graph use State, lookup_state,
    facility_type, located_in

def CA_airports_groupby[fac_type] =
    count[id:
        (located_in[id] :> State) = lookup_state["CA"] and
        fac_type = facility_type[id]
    ]

def output = CA_airports_groupby
```


# Reduction with `ORDER BY`

```rel
with flight_graph use State, lookup_state,
    facility_type, located_in

def CA_airports_groupby[fac_type] =
    count[id:
        (located_in[id] :> State) = lookup_state["CA"] and
        fac_type = facility_type[id]
    ]

def CA_airports_ordered[i, k, v] =
    reverse_sort[CA_airports_groupby[_]](i, v) and
    CA_airports_groupby(k, v)

def output = CA_airports_ordered
```

```rel
with flight_graph use airport_elevation, Airport,
    abbr, located_in, State, name, County, facility_type

def elevation_in_meters[id] =
    feet_to_meters[airport_elevation[id]]

def state_and_county[id in Airport] =
    concat[
        concat[abbr[located_in[id] :> State], "-"],
        name[located_in[id] :> County]
    ]

def avg_elevation_in_meters[state, fac_type] =
    average[elevation_in_meters[id] for id where
        state = (located_in[id] :> State) and
        fac_type = facility_type[id]
    ]

def airport_count[state, fac_type] =
    count[id:
        state = (located_in[id] :> State) and
        fac_type = facility_type[id]
    ]

def output = airport_count
```

```rel
with flight_graph use airport_elevation, Airport,
    abbr, located_in, State, name, County, facility_type

def elevation_in_meters[id] =
    feet_to_meters[airport_elevation[id]]

def state_and_county[id in Airport] =
    concat[
        concat[abbr[located_in[id] :> State], "-"],
        name[located_in[id] :> County]
    ]

def avg_elevation_in_meters[state, fac_type] =
    average[elevation_in_meters[id] for id where
        state = (located_in[id] :> State) and
        fac_type = facility_type[id]
    ]

def airport_count[state, fac_type] =
    count[id:
        state = (located_in[id] :> State) and
        fac_type = facility_type[id]
    ]
    
def heliport:state = abbr

def heliport:airport_count[state] =
    airport_count[state, "HELIPORT"]

def heliport:avg_elevation_in_meters[state] =
    avg_elevation_in_meters[state, "HELIPORT"]

def output = table[heliport]
```