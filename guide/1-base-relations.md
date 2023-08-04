
# Introduction

The files in this directory will guide you through the project. Complete the setup steps in `README.md` before beginning.

If you open your RelationalAI view using the RelationalAI icon in the left sidebar, you should see your select database in the Databases section. You can open that tree item to see a list of base relations, derived relations, and models. We'll look at the base relations first and then the models.

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