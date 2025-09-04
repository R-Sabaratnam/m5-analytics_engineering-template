# JSON Operators in PostgreSQL
_______________

In our case, the result from fetching the weather data using an API is a large JSON from which we will extract 2 tables for:  

- daily weather
- hourly weather

To extract, flatten, or normalize JSON data from a column in PostgreSQL, we can employ JSON operators and choose individual fields as columns for our new table. We will only use operators `->` and `->>`. 

#### An Example

here is a snippet of a JSON from one cell from the daily data, but only two days.

```json
{
  "meta": {
    "generated": "2024-07-17 14:38:45"
  },
  "data": [
    {
      "date": "2023-01-01",
      "tavg": 9.8,
      "tmin": 5.6,
      "tmax": 13.3,
      "prcp": 0,
      "snow": 0,
      "wdir": 263,
      "wspd": 16.2,
      "wpgt": null,
      "pres": 1012.9,
      "tsun": null
    },
    {
      "date": "2023-01-02",
      "tavg": 7.8,
      "tmin": 3.9,
      "tmax": 12.8,
      "prcp": 0.5,
      "snow": 0,
      "wdir": 225,
      "wspd": 10.4,
      "wpgt": null,
      "pres": 1019.9,
      "tsun": null
    },
  ]
}
```

To extract the value of the `"data"` key (selecting a JSON object field by key), we can use the following query:

```sql
SELECT extracted_data -> 'data' FROM weather_daily_raw;
```

As you can see, each cell has a whole sequence, an array, of daily values. To handle this, we use the `JSON_ARRAY_ELEMENTS` function, which expands the array into a set of rows (per day), allowing us to break down the daily data for further analysis.

```sql
SELECT JSON_ARRAY_ELEMENTS(extracted_data -> 'data') FROM weather_daily_raw;
```

Now we can access the key of every available metric. For example we could extract the field `"temp"` 

```sql
SELECT JSON_ARRAY_ELEMENTS(extracted_data -> 'data') -> 'temp' FROM weather_daily_raw;
```

The `->` operator returns values in JSON format. When casting individual JSON values to `VARCHAR` or `TEXT` the double quotes from the JSON format will persist.

```sql
SELECT (JSON_ARRAY_ELEMENTS(extracted_data -> 'data') -> 'temp')::VARCHAR(255) FROM weather_daily_raw;
```

It is impossible to cast JSON values to some types, it's essential to first cast them as `TEXT` or `VARCHAR`. 

```sql
SELECT (JSON_ARRAY_ELEMENTS(extracted_data -> 'data') -> 'date')::TEXT::DATE FROM weather_daily_raw;
```

Alternatively, we can use the `->>` operator. The output is already in text format, so it can easily be converted to a different data type.

```sql
SELECT (JSON_ARRAY_ELEMENTS(extracted_data -> 'data') ->> 'tavg')::NUMERIC FROM weather_daily_raw;

SELECT (JSON_ARRAY_ELEMENTS(extracted_data -> 'data') -> 'date')::DATE FROM weather_daily_raw;
```



**For more information:**

- [PostgreSQL Documentation: JSON Functions and Operators](https://www.postgresql.org/docs/9.5/functions-json.html)
- [Tutorial: PostgreSQL JSON Operators/](https://www.postgresqltutorial.com/postgresql-json-functions/postgresql-jsonb-operators/)
- [Tutorial: PostgreSQL JSON](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-json/)
