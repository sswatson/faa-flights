
# Common Patterns

## A Line Plot

Show a line plot of monthly flight counts over the course of each year in the dataset:

```rel
value type MonthYear = String, Int

module flight_chart
    with flight_graph use Flight, flight_departure_time

    def data:count[id] =
        count[datetime, month, year, x:
            Flight(x) and
            datetime = flight_departure_time[x] and
            month = datetime_monthname[datetime, "+00:00"] and
            year = datetime_year_UTC[datetime] and
            id = ^MonthYear[month, year]
        ]

    def data:month(id, month) =
        data:count(id, _) and
        ^MonthYear(month, _, id)

    def data:year(id, year) =
        data:count(id, _) and
        ^MonthYear(_, year, id)

    def color = {:year; :title, "Year"}
    def x = {:title, "Month"; :type, "nominal"}
    def y = {:title, "Total Flights"; (:scale, :zero, boolean_false)}
end

def graph = vegalite:line[:month, :count, flight_chart]
def output = vegalite:plot[graph]
```

Count flights in 2002 and 2003 and compute the percentage change from 2002 to 2003, facted by carrier:

```rel
with flight_graph use Flight, flight_departure_time, flight_carrier_code

def flights_per_year[year, carrier] =
    count[datetime, x:
        Flight(x) and
        datetime = flight_departure_time[x] and
        year = datetime_year_UTC[datetime] and
        carrier = flight_carrier_code[x]
    ]

module compare_years
    def yr2002[carrier] =
        flight_carrier_code(_, carrier),
        flights_per_year[2002, carrier] <++ 0

    def yr2003[carrier] =
        flight_carrier_code(_, carrier),
        flights_per_year[2003, carrier] <++ 0

    def percent_change[carrier] =
        flight_carrier_code(_, carrier),
        100 * (yr2003[carrier] - yr2002[carrier]) /
        (if yr2003[carrier] = 0 then 1 else yr2003[carrier] end)
end

def output = table[compare_years]
```

## Aggregate Statistics by Carrier

Compute several aggregate statistics, faceted by carrier:

```rel
with flight_graph use Flight, craft, model, flight_carrier_code,
    flight_departure_time, num_seats

module info_by_carrier
    def total_flights(c, f) =
        Flight(f) and
        c = flight_carrier_code[f] and
        // Filter by Jan 2003
        datetime_year_UTC[flight_departure_time[f]] = 2003 and
        datetime_month_UTC[flight_departure_time[f]] = 1

    def total_aircraft(c, a) =
        total_flights(c, f) and
        a = craft[f]
        from f

    def total_models(c, m) =
        total_aircraft(c, a) and
        m = model[a]
        from a
end

module stats_by_carrier
    def total_flights[c] =
        count[info_by_carrier:total_flights[c]]

    def total_aircraft[c] =
        count[info_by_carrier:total_aircraft[c]]

    def total_models[c] =
        count[info_by_carrier:total_models[c]]

    def total_seats_on_flights[c] =
        sum[f, seats:
            info_by_carrier:total_flights(c, f) and
            aircraft = craft[f] and
            m = model[aircraft] and
            seats = num_seats[m]
            from aircraft, m
        ]

    def total_seats_on_aircrafts[c] =
        sum[a, seats:
            info_by_carrier:total_aircraft(c, a) and
            m = model[a] and
            seats = num_seats[m]
            from m
        ]

    def avg_seats_per_model[c] =
        average[m, seats:
            info_by_carrier:total_models(c, m) and
            seats = num_seats[m]
        ]
end

def output = table[stats_by_carrier]
```

## Percent of Total

Create a table showing the percentage of the total number of flights that each carrier is responsible for:

```rel
with flight_graph use Flight, carrier_nickname, operated_by

def total_flight_count = count[Flight]

def count_by_carrier[c] =
    count[f:
        Flight(f) and
        c = carrier_nickname[operated_by[f]]
    ]

module flight_count_stats
    def flight_count = count_by_carrier
    def percent_of_total[c] =
        count_by_carrier[c] / total_flight_count * 100
end

def output = table[flight_count_stats]
```


## Carriers by destination

```rel
with flight_graph use operated_by, 
    flight_destination_code, carrier_name

def count_by_carrier_and_dest[carrier, dest] =
    count[f:
        dest = flight_destination_code[f] and
        carrier = carrier_name[operated_by[f]]
    ]

def output = count_by_carrier_and_dest
```

# Finding all unary relations

The cardinality of all unary relations in the graph:

```rel
def output[n] = count[v: flight_graph(n, v)]
```

```rel
def output(col) = flight_graph(col, _)
```

## Top Five Carriers

```rel
with flight_graph use operated_by, 
    flight_destination_code, carrier_name

def count_by_carrier =
    count[f:
        carrier = carrier_name[operated_by[f]]
    ], carrier from carrier

def top_5_carriers(ord, carrier, count) =
    bottom[5, count_by_carrier](ord, count, carrier)

def output = top_5_carriers
```

```rel
with flight_graph use operated_by, 
    flight_destination_code, carrier_name
def count_by_destination =
    count[f:
        dest = flight_destination_code[f]
    ], dest from dest

def top_5_destinations(ord, dest, count) =
    bottom[5, count_by_destination](ord, count, dest)

def output = top_5_destinations
```
