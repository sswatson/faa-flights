
# Introduction

The files in this directory will guide you through the project. Complete the setup steps in `README.md` before beginning.

If you open your RelationalAI view using the RelationalAI icon in the left sidebar, you should see your select database in the Databases section. You can open that tree item to see a list of base relations, derived relations, and models. We'll look at the base relations first and then the models.

## Base Relations

Let's begin understanding this dataset by looking at the base relations.

There are five relations obtained by importing five CSV files that make up the `faa-flights` dataset. In addition, there are two supplementary relations that contain general information about US states. Let's take a look at each relation.

### Core Dataset

First up in alphabetical order is `aircraft_data`:

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

The relation `top_table` is defined in `util.rel`; it takes a number `n` and a relation `R` and returns the top `n` rows of `R` as a wide table.

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

### Supplementary Data

The `states_data` table is also really simple; just ID, name, abbreviation:

```rel
table[states_data]
```

Finally, the `geojson` relation just contains some geographic coordinates for purposes of drawing maps:

```rel
us_states_geojson
```

## Models

Next, let's take a look at the models. We can start with `util.rel`, since those relations are general-purpose and self-contained.

### `util.rel`

The `ratio` relation computes the proportion of tuples in `S` that are also in `R`:

```rel
@inline
def ratio[R, S] {
    count[intersect[R, S]] / count[S]
}
```

This relation requires an annotation (`@inline`, in this case) because it is higher-order. In other words, because it takes entire relations as arguments, its contents cannot be explicitly enumerated by the engine.

The other utility relation is also an `@inline` relation:

```rel
doc"""
If `R` represents a table in stacked form, select the triples from the top `n` rows of that table and display
the resulting relation as a wide table.
"""
@inline
def top_table[n, R] {
    table[{
        a, b, c :
        R(a, b, c)
        and top[n, {mid : R(_, mid, _)}](_, b)
    }]
}
```

Here `{mid : R(_, mid, _)}` is the set of all elements in the middle column of `R`. In other words, it's the set of all row identifiers in the table of which `R` is the stacked form. The relation `top` then picks out the first `n` such identifiers and prepends a column that enumerates those items.

So ultimately the body of this relation says "show us, in table form, the set of all triples of the form `(a, b, c)` such that `(a, b, c)` is in `R` and `b` is one of the first `n` row identifier values.

### `schema-mapping.rel`

Next let's take a look at the file `schema-mapping.rel`.

The purpose of this file will be to transform the data from the raw CSV form to a form where all elements are equipped with appropriate types.

To see how this works, we're going to go from the bottom of the file to the top. This will allow us to see the overall architecture of the schema-mapping logic. Then we will go back from the top to the bottom to understand concretely how each relation works.

The rule that defines the relation `airport` is this one:

```rel
def airport =
    flight_file_mapping[airport_data, flight_dataset_info:airport]
```

This rule says that the relation `airport` is defined as the result of applying the relation `flight_file_mapping` to the relation `airport_data` and the relation `flight_dataset_info:airport`. You can see that the other relations (`carrier`, `aircraft_model`, etc.) are defined similarly.

To understand what these rules will evaluate to, we'll need to look at `flight_file_mapping`:

```rel
@inline
def flight_file_mapping[CSV, Schema](col_renamed, k, v...) =
    CSV(key_col, pos, key_str) and
    CSV(col, pos, val_str) and
    Schema:primary_key(key_col) and
    Schema:entity_name(entity_name) and
    Schema:rename(col, col_renamed) and
    Schema:convert(col, val_str, v...) and
    create_entity(entity_name, key_str, k)
    from key_col, key_str, val_str, col, pos, entity_name
```

This says that `flight_file_mapping` takes two arguments, `CSV` and `Schema`, and evaluates to a relation with the columns `col_renamed`, `k`, and zero or more additional columns called `v...`.

The `v...` syntax is called a *varargs* expression, and the idea is that it's the same as if you'd inserted `v`, or `v1, v2`, or `v1, v2, v3`; whichever is appropriate for the relational application in the body in which `v...` appears. It's useful because it means you don't have to repeat the logic in the rule for each possible number of arguments.

The first argument, called `CSV`, is the one that contains the raw data. The second argument, called `Schema`, contains rules about how things should be mapped in this relation.

Let's start with the variables being introduced in the rule:

* `key_col` is the name of the column in the CSV file that contains the primary key for the table.
* `key_str` is the value of the primary key for a given row.
* `col` is the name of the column in the CSV file that contains the value for a given attribute.
* `pos` is the position of the column `col` in the CSV file.
* `val_str` is the value of the column `col` for a given row.
* `entity_name` is the name of the entity that we're creating.

The clauses in the body are providing constraints that establish the appropriate relationships between all of these variables.

* The first clause `CSV(key_col, pos, key_str)` says that `key_str` is the string value from the CSV file for the column `key_col` at position `pos`.
* The second one `CSV(col, pos, val_str)` says that `val_str` is the string value from the CSV file for the column `col` at position `pos`.
* The four lines that begin with `Schema` take advantage of table-specific rules to say that `key_col` is the primary key, `entity_name` is the name of the entity in question, and so on.
* `create_entity(entity_name, key_str, k)` binds the variable `k` to the entity that we're creating.

The next relation in this file that we want to understand is `flight_dataset_info`. Here it is with some of the more verbose parts truncated:

```rel
@inline
module flight_dataset_info
    with dataset_info use
        int_column, float_column, three_valued_bool_column,
        datetime_column, date_column, table_info, rename_column,
        primary_key, bool_column, year_column, month_year_column,
        int_feet_column, float_degree_column

    def carrier = table_info[
        primary_key[:Carrier, :code]
    ]

    def aircraft = table_info[
        primary_key[:Aircraft, :tail_num];
        int_column:aircraft_type_id;
        //...
    ]

    def aircraft_model = table_info[
        primary_key[:AircraftModel, :aircraft_model_code];
        int_column:aircraft_type_id;
        int_column:aircraft_engine_type_id;
        //...
    ]

    def airport = table_info[
        primary_key[:Airport, :id];
        int_column:id;
        //...
    ]

    def flight = table_info[
        primary_key[:Flight, :id2];
        int_column:flight_time
        //...
    ]

end
```

It begins with a `with`-`use` declaration, which is comparable to an import statement in traditional programming languages. Using the declaration `with dataset_info use int_column` inside the `module` body means that we can say `int_column` instead of the full name `dataset_info:int_column`. This reduces boilerplate.

The five declarations define five subrelations of `flight_dataset_info`. Each of these subrelations is obtained by applying the relation `table_info` to a relation containing information about the primary key and column types for the corresponding CSV table.

Finally we can look at the first relation defined in the file:

```rel
@inline
module dataset_info
    module int_column[c]
        def column_name = c
        def convert = c, parse_int
    end

    module float_column[c]
        def column_name = c
        def convert = c, parse_float
    end

    module bool_column[c]
        def column_name = c
        def convert = c, "Y"
        // If not "Y", will be false by default
   end

    module three_valued_bool_column[c]
        def column_name = c
        def convert = c, "Y", boolean_true
        def convert = c, "N", boolean_false
    end

    module datetime_column[c]
        def column_name = c
        def convert = c, (v: parse_datetime[v, "YYYY-mm-dd HH:MM:SS Z"])
    end

    module date_column[c]
        def column_name = c
        def convert = c, (v: parse_date[v, "Y-m-d"])
    end

    module year_column[c]
        def column_name = c
        def convert = c, (v: Year[date_year[parse_date[v, "Y"]]])
    end

    module month_year_column[c]
        def column_name = c
        def convert = c, parse_month_year
    end

    module int_feet_column[c]
        def column_name = c
        def convert = c, parse_int_feet
    end

    module float_degree_column[c]
        def column_name = c
        def convert = c, parse_float_degree
    end

    module table_info[R]
        def primary_key = R:primary_key
        def column_name = R:column_name
        def entity_name = R:entity_name
        def convert[c, v] = v, not(R:column_name(c))
        def convert[c, v] = R:convert[c, v]
        def rename[c] = c, not(exists R:rename[c])
        def rename(c1, c2) = R:rename(c1, c2)
    end

    module rename_column[c1, c2]
        def rename = (c1, c2)
    end

    module primary_key[name, c]
        def entity_name = name
        def primary_key = c
    end
end
```

Let's break this down one rule at a time:

```rel
@inline
module dataset_info
    module int_column[c]
        def column_name = c
        def convert = c, parse_int
    end
    //...
end
```

The declaration that starts with `module int_column[c]` is a **parameterized module** declaration. It means that we will have to supply an argument to `int_column` in order to get the subrelations being defined in the body.

For example, `dataset_info:int_column[:foo]` evaluates to the following relation (conceptually; `parse_int` has to be applied to a string to realize a concrete value):

```rel
(:column_name, :foo           );
(:convert,     :foo, parse_int)
```

Note: the expression `dataset_info:int_column` by itself can't be evaluated because there are infinitely many values `c` that could be supplied. If it could be printed, it would look like this:

```rel
:foo,  :column_name, :foo            ;
:foo,  :convert,     :foo,  parse_int;
:bar,  :column_name, :bar            ;
:bar,  :convert,     :bar,  parse_int;
:baz,  :column_name, :baz            ;
:baz,  :convert,     :baz,  parse_int;
"foo", :column_name, "foo"           ;
"foo", :convert,     "foo", parse_int;
//...
```

Most of the other conversion subrelations work similarly. Some of them have something interesting going on:

```rel
module date_column[c]
    def column_name = c
    def convert = c, (v: parse_date[v, "Y-m-d"])
end
```

With this declaration, we can do `date_column[:date_of_birth]`, for example, and get the following relation:

```rel
(:column_name, :date_of_birth);
(:convert,     :date_of_birth, (v: parse_date[v, "Y-m-d"]))
```

```rel
(v: parse_date[v, "Y-m-d"])["2023-08-11"]
```

This syntax, which looks like anonymous function syntax in other languages, is what we call a *relational abstraction*. It generalizes the following construction:

```rel
def R = {1; 2; 3}

def output = { x : x*x < 8 and R(x) }
```

| output |
| ------ |
| 1      |
| 2      |

The expression `{ x : x*x < 8 and R(x) }` is a *set comprehension*. It means "the set of all `x` such that `x*x < 8` and `x` is in `R`". This set comprehension evaluates to `{1; 2}` since those two numbers square to give a vlaue less than 8, while `3` does not.

The expression after the colon in a relational abstraction is allowed to evaluate to something other than `true` or `false`; in that case the value is appended to the variable value as part of the result:

```rel
def R = {1; 2; 3}
def output = { x : x + 5, x*x < 8, R(x) }
```

|     | output |
| --- | ------ |
| 1   | 6      |
| 2   | 7      |

Note that we have to switch to using commas to join the expressions; this is because `and` only works for boolean expressions, while the comma performs the same filtering operation when used with boolean expressions (while handling non-boolean expressions like `x + 5`).

In any case, the upshot is that a relational abstraction like `x: x + 5` is effectively an anonymous function (no restrictions needed if we're going to evaluate it right away):

```rel
(x: x + 5)[10]
```

Another noteworthy rule in this module is `three_valued_bool_column`:

```rel
module three_valued_bool_column[c]
    def column_name = c
    def convert = c, "Y", boolean_true
    def convert = c, "N", boolean_false
end
```

This rule is saying `three_valued_bool_column[:foo][:convert, :foo]` should map `"Y"` to a value called `boolean_true`.

The values `boolean_true` and `boolean_false` exist in the language because the relations `true` and `false` -- which are aliases for `{()}` and `{}` -- are not suitable for all situations. For example, this expression evaluates to `{:name, "Alice"}`:

```rel
:name, "Alice";
:is_active_customer, false
```

These are only equivalent if you know that the relation would have an :is_active_customer subrelation if Alice were an active customer. This is called the **closed world** assumption -- the absence of a datum is a basis for drawing a conclusion, not an expression of uncertainty about that datum. The values `boolean_true` and `boolean_false` can be used to make things explicit in situations where the closed-world assumption is not appropriate.

*Note: the expression above is equivalent to this JSON:*

```json
{
    "name": "Alice",
    "is_active_customer": false
}
```

Lastly, let's take a look at `table_info`:

```rel
module table_info[R]
    def primary_key = R:primary_key
    def column_name = R:column_name
    def entity_name = R:entity_name
    def convert[c, v] = v, not(R:column_name(c))
    def convert[c, v] = R:convert[c, v]
    def rename[c] = c, not(exists R:rename[c])
    def rename(c1, c2) = R:rename(c1, c2)
end
```

This relation takes a relation `R` and preserves its subrelations `primary_key`, `column_name`, `entity_name`, `convert`, and `rename`. It also adds rules for the `convert` and `rename` subrelations that say that the conversion and renaming relations should map any value to itself if the value is not accounted for in `R:convert` or `R:rename`, respectively.

That concludes our inspection of the Rel code in this file. Now let's follow one thread of evaluation so we can understand how it fits together: In the module `flight_dataset_info`, we have this rule:

```rel
def airport = table_info[
    primary_key[:Airport, :id];
    int_column:id;
    int_column:cbd_dist;
    bool_column:cntl_twr;
    bool_column:major;
    float_degree_column:longitude;
    float_degree_column:latitude;
    int_feet_column:elevation;
    month_year_column:act_date;
    three_valued_bool_column:cust_intl;
    three_valued_bool_column:c_ldg_rts;
    three_valued_bool_column:joint_use;
    three_valued_bool_column:mil_rts
]
```

The relation `primary_key[:Airport, :id]` evaluates to:

```rel
:entity_name, :Airport;
:primary_key, :id
```
The relation `int_column:id` evaluates to:

```rel
:column_name, :id;
:convert, :id, parse_int
```

(Again, conceptually; if you try to run this code it won't display anything because the relation `parse_int` has to be evaluated to reach a concrete element that can be shown in the output table.)

Continuing in this way, we get an `:airport` subrelation that looks like this:

```rel
:entity_name, :Airport;
:primary_key, :id;
:column_name, :id;
:convert, :id, parse_int;
:column_name, :cbd_dist;
:convert, :cbd_dist, parse_int;
:column_name, :cntl_twr;
:convert, :cntl_twr, parse_bool;
:column_name, :major;
:convert, :major, parse_bool;
:column_name, :longitude;
:convert, :longitude, parse_float_degree;
:column_name, :latitude;
:convert, :latitude, parse_float_degree;
:column_name, :elevation;
:convert, :elevation, parse_int_feet;
:column_name, :act_date;
:convert, :act_date, parse_month_year;
:column_name, :cust_intl;
:convert, :cust_intl, parse_three_valued_bool;
:column_name, :c_ldg_rts;
:convert, :c_ldg_rts, parse_three_valued_bool;
:column_name, :joint_use;
:convert, :joint_use, parse_three_valued_bool;
:column_name, :mil_rts;
:convert, :mil_rts, parse_three_valued_bool;
:rename, :id, :id;
:rename, :cbd_dist, :cbd_dist;
:rename, :cntl_twr, :cntl_twr;
:rename, :major, :major;
:rename, :longitude, :longitude;
:rename, :latitude, :latitude;
:rename, :elevation, :elevation;
:rename, :act_date, :act_date;
:rename, :cust_intl, :cust_intl;
:rename, :c_ldg_rts, :c_ldg_rts;
:rename, :joint_use, :joint_use;
:rename, :mil_rts, :mil_rts
```

Note that all those `:rename` tuples are supplied by the logic that says that missing `:rename`s should be filled in with the identity function; we didn't have to list them explicitly.

This relation appears once in the file, in this expression that defines the relation `airport`: 

```rel
flight_file_mapping[airport_data, flight_dataset_info:airport]
```

We can see in the logic defining `flight_file_mapping` that the second-argument relation is supposed to have first-column values of `:primary_key`, `:entity_name`, `:rename`, and `:convert`, and that lines up with the expression for `flight_dataset_info:airport` detailed above.

One remaining question we have to answer is why it was necessary to use varargs in the definition of `flight_file_mapping`.

If you look carefully at the definition of `dataset_info`, you'll see that one of the `convert` relations, namely the one under `bool_column`, has an arity of 2 rather than 3. This is because `def convert = c, "Y"` is equivalent to `def convert = c, "Y", true` -- note that `true` has an arity of zero.

### `state.rel`

To use entity types and value types in Rel, it's important to define various relations to help convert back and forth between those types and other values.

The model `state.rel` contains some helpful examples of how that can be done:

```rel
module flight_graph
    doc"""
        State
    """
    def State(x in Entity) =
        ^State(s, x) and
        airport:state(_, s)
        from s

    doc"""
        Two-letter state abbreviation (ANSI, USPS)
    """
    def abbr(x in State, s in String) =
        ^State(s, x) and
        airport:state(_, s)

    def lookup_state(s in String, x in State) =
        abbr(x, s)

    def lookup_state(s in String, x in State) =
        flight_graph:postal_abbrevation(x, s)

end
```

Starting at the top, wrapping this code in a `module` block will mean that we'll write `flight_graph:State` rather than just `State` when we want to access this relation elsewhere.

It's helpful to put everything related to your knowledge graph in one module because then you can ask questions like "what are all my top-level relation names?" as a Rel query rather than having to infer that from static code analysis.

Let's take a look at the first rule in the module:

```rel
def State(x in Entity) =
    ^State(s, x) and
    airport:state(_, s)
    from s
```

The point of this relation is to determine whether a given entity is a `State`. This will be a finite relation that draws from the base data to determine the valid states.

```rel
airport:state
```