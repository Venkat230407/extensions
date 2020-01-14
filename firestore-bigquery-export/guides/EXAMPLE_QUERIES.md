These example queries are for use with the official Firebase Extension
[_Export Collections to BigQuery_](https://github.com/firebase/extensions/tree/master/firestore-bigquery-export)
and its associated [`fs-bq-schema-views` script](TODO) (referred to as the "schema-views script").

The queries use the following parameter values from your installation of the extension:

+   `${param:PROJECT_ID}`: the project ID for the Firebase project in
    which you installed the extension
+   `${param:DATASET_ID}`: the ID that you specified for your dataset during
    extension installation
+   `${param:TABLE_ID}`: the common prefix of BigQuery views to generate

**NOTE:** You can, at any time, run the schema-views script against additional schema files
to create different schema views over your raw changelog. When you settle on a fixed schema,
you can create a [scheduled query](https://cloud.google.com/bigquery/docs/scheduling-queries)
to transfer the columns reported by the schema view to a persistent backup table.

Assume that you have a schema view matching the following configuration from a
schema file:

```
{
  "fields": [
    {
      "name": "name",
      "type": "string"
    },
    {
      "name":"favorite_numbers",
      "type": "array"
    },
    {
      "name": "last_login",
      "type": "timestamp"
    },
    {
      "name": "last_location",
      "type": "geopoint"
    },
    {
      "fields": [
        {
          "name": "name",
          "type": "string"
        }
      ],
      "name": "friends",
      "type": "map"
    }
  ]
}
```

### Example query for a timestamp

You can generate a listing of users that have logged in to the app as follows:

```
SELECT name, last_login
FROM ${param:PROJECT_ID}.${param:DATASET_ID}.${param:TABLE_ID}_schema_${SCHEMA_FILE_NAME}_latest
ORDER BY last_login DESC
```

In this query, note the following:

+   `${SCHEMA_FILE_NAME}` is the name of the schema file that you
    provided as an argument to run the schema-views script.

+   The `last_login` column contains data that is stored in the `data`
    column of the raw changelog. The type conversion and view generation is
    performed for you by the
    [_Export Collections to BigQuery_](https://github.com/firebase/extensions/tree/master/firestore-bigquery-export)
    extension.

### Example queries for an array

The example schema configuration (see above) stores each user's favorite number
in a Cloud Firestore array called `favorite_numbers`. Here are some example
queries for that data:

+   If you wanted to determine how many favorite numbers each user
    currently has, then you can run the following query:

    ```
    SELECT document_name, MAX(favorite_numbers_index) 
    FROM ${param:PROJECT_ID}.users.users_schema_user_full_schema_latest 
    GROUP BY document_name
    ```

+   If you wanted to determine the what the current favorite numbers are
    of the app's users (assuming that number is stored in the first position of
    the `favorite_numbers` array), you can run the following query:

    ```
    SELECT document_name, favorite_numbers_member 
    FROM ${param:PROJECT_ID}.users.users_schema_user_full_schema_latest 
    WHERE favorite_numbers_index = 0
    ```

### Example query if you have multiple arrays

If you had multiple arrays in the schema configuration, you might have to select
all `DISTINCT` documents to eliminate the redundant rows introduced by the
cartesian product of `CROSS JOIN`.

```
SELECT DISTINCT document_name, favorite_numbers_member 
FROM ${param:PROJECT_ID}.users.users_schema_user_full_schema_latest 
WHERE favorite_numbers_index = 0
```