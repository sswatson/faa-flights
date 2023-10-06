
# Small Queries

Find all of the states containing an airport in this dataset:

```rel
with flight_graph use Airport, located_in, abbr
def output = Airport.located_in.located_in.located_in.abbr
```

Find the state abbreviation of each airport:

```rel
with flight_graph use Airport, located_in, State, abbr
def output = (Airport <: located_in :> State).abbr
```