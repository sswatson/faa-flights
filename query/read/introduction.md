# FAA Flight Data

## Base Relations

Let's begin understanding this dataset by looking at the base relations. First up in alphabetical order is `aircraft_data`:

```rel
table[aircraft_data]
```

Each row of this table corresponds to an aircraft, and there are quite a few attributes:

```rel
count[first[aircraft_data]]
```

Next, let's look at `aircraft_model_data`:

```rel
top_table[10, aircraft_model_data]
```

The relations in this table can be joined with `aircraft_data` to get more information about an aircraft based on its model identifier.

Next up is `airport_data`, in which rows correspond to airports and columns correspond to identifiers and geographic information:

```rel
top_table[10, airport_data]
```

Now `carrier_data`:

```rel
top_table[10, carrier_data]
```

Rows are airlines in this table, and there are just three columns.

The table `flight_data` has more columns:

```rel
flight_data
```

The `states_data` table is also really simple; just ID, name, abbreviation:

```rel
table[states_data]
```

Finally, the `geojson` relation just contains some geographic coordinates for purposes of drawing maps:

```rel
us_states_geojson
```