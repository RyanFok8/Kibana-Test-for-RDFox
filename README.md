# Using Kibana to visualise RDFox performance regression test
**remember to add license

## Contents
1. [Requirements](#requirements)
2. [Setting up](#setting-up)
   * [Windows](#windows)
   * [macOS](#macos)
3. [Quit/Cleanup](#quitcleanup)
4. [Getting started](#getting-started)
   * [To create a visualisation from scratch](#to-create-a-visualisation-from-scratch)
     * [Tag Clouds](#tag-clouds)
     * [Grouped bar charts](#grouped-bar-charts)
   * [If you have a saved dashboard `.ndjson` and a different `.jsonl` file](#if-you-have-a-saved-dashboard-ndjson-and-a-different-jsonl-file)
5. [Update API/Update By Query API](#update-api-update-by-query-api)
6. [Miscellaneous](#miscellaneous)
   * [Filtering data](#filtering-data)
   * [Formatting numbers](#formatting-numbers)
   * [Importing and exporting Dashboards `.ndjson`](#importing-and-exporting-dashboards-ndjson)
   * [Combining data views/index patterns](#combining-data-viewsindex-patterns)
   * [Double checking the basic license](#double-checking-the-basic-license)
   


## Requirements

* [Docker Engine](https://docs.docker.com/get-docker/) version **18.06.0** or newer
* [Docker Compose](https://docs.docker.com/compose/install/) version **1.28.0** or newer (including [Compose V2][compose-v2])
* 1.5 GB of RAM

## Setting up
### Windows

If you are using the legacy Hyper-V mode of _Docker Desktop for Windows_, ensure [File Sharing](https://docs.docker.com/desktop/settings/windows/#file-sharing) is enabled for the `C:` drive.

### macOS

The default configuration of _Docker Desktop for Mac_ allows mounting files from `/Users/`, `/Volume/`, `/private/`,
`/tmp` and `/var/folders` exclusively. Make sure the repository is cloned in one of those locations or follow the
instructions from the [documentation](https://docs.docker.com/desktop/settings/mac/#file-sharing) to add more locations.

Make sure your Docker Desktop is opened and clone this repository onto the Docker host that will run the stack:

```sh
git clone https://github.com/deviantony/docker-elk.git
```

Open `elasticsearch.yml` and switch the value of `xpack.license.self_generated.type` setting from `trial` to `basic` to disable paid features.

Then, open the folder in the terminal and initialize the Elasticsearch users and groups required by docker-elk by executing the command:

```sh
docker-compose up setup
```

If the setup completed without an error, start the other stack components:

```sh
docker-compose up
```

Give Kibana about a minute to initialize, then access the Kibana web UI by opening <http://localhost:5601> in a web
browser and use the following (default) credentials to log in:

* user: *elastic*
* password: *changeme*

You can change your password by clicking the icon located on the top left corner on the home page.

## Quit/Cleanup

To stop the application (stop and remove the containers but keep the data and volumes), use:
```sh
docker-compose stop
```

To shutdown the stack and remove all persisted data:
```sh
docker-compose down
```

## Getting started
**Note index patterns has been renamed to data views

### To create a visualisation from scratch
1. Run the regression test, and generate a `.jsonl` file by including 'jsonl' in the '-a'
2. Upload the `.jsonl` file in the **Get started by adding integrations** section on the home page and press import
3. Create an Index name to identify the source file and press import
4. When import successful, return back to home page

After ingesting the data, if we want to create a new dashboard from scratch:
1. Open the sidebar, choose **Dashboard** under **Analytics**
2. Click on **Create dashboard**
   * Under **Select type**, **Aggregation based --> Tag clouds** can be used to display fields directly, e.g., runId, system, architecture, step, stepType...
   * While **Lens** can be used to analyse numbers through bar chart, pie graph..., with different aggregations (count, median, average...)

#### Tag clouds
Tag clouds are used to display fields directly, if you want to display all the stepType(import, query, materialization):
- **Select type --> Aggregation based --> Tag clouds** and choose the source file
- Add a **bucket** on the right hand side, with **Terms** as **Aggregation**
- Select an appropriate **field**, **stepType** in this case
- The **size** should be bigger than the number of different stepTypes in order to display them all
- Notice on the top left, you can freely switch between different data source
- Press **Save and Return** on the upper right and the visualisation will be displayed on the dashboard

#### Grouped bar charts
Bar charts can be used to visualise and compare performance between different RDFox versions




### If you have a saved dashboard `.ndjson` and a different `.jsonl` file
1. Open the sidebar, scroll to the bottom, and click **Management**
2. In the **Kibana** section, choose **Saved objects**
3. Import the dashboard you want

## [Update API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)/ [Update By Query API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#docs-update-by-query-api-desc)

The APIs update a document using the specified script.

### Adding "facts_per_second" and "rules_per_second" fields 
1. Open the side bar
2. Scroll to bottom, click **Dev Tools** under **Management**
3. You will see a console

We first convert string values to integer/float:

```sh
POST /20230714-linux/_update_by_query
{
  "query": {
    "exists": {
      "field": "numFactsProcessed"
    }
  },
  "script": {
    "source": "if (ctx._source.numFactsProcessed != null) { ctx._source.numFactsProcessed = Long.parseLong(ctx._source.numFactsProcessed) }", 
    "lang": "painless"
  }
}

```

```sh
POST /20230714-linux/_update_by_query
{
  "query": {
    "exists": {
      "field": "numRulesProcessed"
    }
  },
  "script": {
    "source": "if (ctx._source.numRulesProcessed != null) { ctx._source.numRulesProcessed = Long.parseLong(ctx._source.numRulesProcessed) }", 
    "lang": "painless"
  }
}

```

```sh
POST /20230714-linux/_update_by_query
{
  "query": {
    "exists": {
      "field": "numFactsProcessed"
    }
  },
  "script": {
    "source": "if (ctx._source.containsKey('numFactsProcessed') && ctx._source.containsKey('time')) { ctx._source.time = Double.parseDouble(ctx._source.time) }",
    "lang": "painless"
  }
}
```
```sh
POST /20230714-linux/_update_by_query
{
  "query": {
    "exists": {
      "field": "numRulesProcessed"
    }
  },
  "script": {
    "source": "if (ctx._source.containsKey('numRulesProcessed') && ctx._source.containsKey('time')) { ctx._source.time = Double.parseDouble(ctx._source.time) }",
    "lang": "painless"
  }
}
```

```sh
# Create the index 'my_index'
PUT /my_index

# Index a document with ID '1' into 'my_index'
POST /my_index/_doc/1
{
  "name": "John Doe",
  "age": 30,
  "city": "New York"
}

# Update the 'city' field of the document with ID '1'
POST /my_index/_update/1
{
  "doc": {
    "city": "San Francisco"
  }
}
```

The API provides ... 

Open console bla bla

Assume you have a json file like this:

```sh
{
  "name": "John Doe",
  "age": 30,
  "salary": 50000,
  "month": 12
}
```
and if you want to add a field monthly salary:

```sh
curl -XPOST 'http://localhost:9200/my_index/_update/1' -d '{
  "script": {
    "source": "ctx._source.monthly_salary = ctx._source.salary / ctx._source.month",
    "lang": "painless"
  }
}'
```
so the document will look like this:

```sh
{
  "name": "John Doe",
  "age": 30,
  "salary": 50000,
  "month": 12,
  "monthly_salary": 4166.666666666667
}
```
To add a field "rate" to the documents in the "my_index" index when both "time" and "distance" exist, with the value calculated as "distance / time", you can use the Elasticsearch Update By Query API with a Painless script. The script will check if both "time" and "distance" fields exist in the document and then perform the calculation to add the "rate" field.
```sh
POST /20230714-linux/_update_by_query
{
  "query": {
    "bool": {
      "filter": [
        { "exists": { "field": "time" } },
        { "exists": { "field": "numRulesProcessed" } }
      ]
    }
  },
  "script": {
    "source": "ctx._source.rate = ctx._source.numRulesProcessed / ctx._source.time",
    "lang": "painless"
  }
}
```
In this example, we used the Update By Query API with a Painless script to add the "rate" field to the documents where both "time" and "distance" fields exist. The "query" section of the JSON payload uses a "bool" query with two "exists" queries to filter documents that have both "time" and "distance" fields.

OR 

This is string to integer
```sh
POST /20230714-linux/_update_by_query
{
  "query": {
    "exists": {
      "field": "numFactsProcessed"
    }
  },
  "script": {
    "source": "if (ctx._source.numFactsProcessed != null) { ctx._source.numFactsProcessed = Long.parseLong(ctx._source.numFactsProcessed) }", 
    "lang": "painless"
  }
}

```
```sh
POST /20230714-linux/_update_by_query
{
  "query": {
    "exists": {
      "field": "numFactsProcessed"
    }
  },
  "script": {
    "source": "if (ctx._source.containsKey('numFactsProcessed') && ctx._source.containsKey('time')) { ctx._source.time = Double.parseDouble(ctx._source.time) }",
    "lang": "painless"
  }
}
```
## Miscellaneous

### Filtering data
click tag clouds or KQL

### Formatting numbers
If you want to change the presentation of numbers in visualisation, e.g., to the nearest integer, to 3 decimal places...
1. Open the sidebar, scroll to the bottom, and click **Management**
2. In the **Kibana** section, click **Data Views**
3. Choose the specifc data view, and a specific key by clicking the pencil icon on the right
4. Toggle the **Set format**, and choose the appropriate **Format** and **Numeral.js format pattern** (Default is 3dp)
5. Save the settings

### Importing and exporting Dashboards `.ndjson`
1. Go to **Management**, and click **Saved objects** under **Kibana**
2. For import:
   * lll
   * lll
3. For export:
   * lll
   * lll
### Combining data views/index patterns
1. Go to **Management**, click **Data Views** in **Kibana**
2. Click **Create data view**
3. To combine two or more data views/index patterns, seperate them using comma, e.g., `index_1, index_2, index_3` in the index pattern row

### Double checking the basic license
1. Go to **Management** and scroll to the bottom
2. Click on **License Management** and it should show **Your Basic license is active**






some drafts:
- breakdown for grouped bar charts
- tag clouds in aggregation to show ids, system...
- apply filters



## vega

```sh
{
  $schema: https://vega.github.io/schema/vega-lite/v5.json
  title: Country fertility representation
  mark: {"type": "point", "tooltip": true}
  data: {
    url: {
      %context%: true
      index: scatter
      body: {
        size: 10000
        _source: ["country", "lifeExpectancy", "fertility"]
      }
    }
    format: {property: "hits.hits"}
  }
  encoding: {
    x: {field: "_source.lifeExpectancy", type: "quantitative", title: "Life Expectancy"}
    y: {field: "_source.fertility", type: "quantitative", title: "Fertility"}
    tooltip: [
      {"field": "_source.country", "type": "nominal", "title": "Country"}
      {"field": "_source.lifeExpectancy", "type": "nominal", "title": "Life Expectancy"}
      {"field": "_source.fertility", "type": "nominal", "title": "Fertility"}
    ]
  }
}
```
https://www.youtube.com/watch?v=5giacrHVYe4
https://www.elastic.co/blog/custom-vega-visualizations-in-kibana
https://afivan.com/2021/11/09/elastic-search-data-visualization-with-kibana-how-to-create-a-scatter-plot-with-vega-lite/
