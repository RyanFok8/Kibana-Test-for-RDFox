# Using Kibana to visualise RDFox performance regression test
## Contents
1. [Requirements](#requirements)
2. [Setting up](#setting-up)
   * [Windows](#windows)
   * [macOS](#macos)
3. [Quit/Cleanup](#quitcleanup)
4. [Getting started](#getting-started)
   * [Update API/Update By Query API](#update-api-update-by-query-api)
   * [To create a visualisation from scratch](#to-create-a-visualisation-from-scratch)
     * [Tag Clouds](#tag-clouds)
     * [Grouped bar charts](#grouped-bar-charts)
     * [Filtering data](#filtering-data)
   * [Vega-Lite for scatter plot](#vega-lite-for-scatter-plot)
   * [If you have a saved dashboard `.ndjson`](#if-you-have-a-saved-dashboard-ndjson)
6. [Others](#others)
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
(Note: This task only needs to be performed once, during the *initial* startup ofthe stack)

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
docker-compose down -v
```

## Getting started
**Note index patterns has been renamed to data views

1. Run the regression test, and generate a `.jsonl` file by including 'jsonl' in the '-a'
2. Upload the `.jsonl` file in the **Get started by adding integrations** section on the home page and press import
3. Create an **index name** to identify the source file, do not create **data view**, as we will have to edit the file
4. When import successful, return back to home page

### [Update API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)/ [Update By Query API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#docs-update-by-query-api-desc)

We would like to add **facts_per_second** and **rules_per_second** for comparison  using the two APIs:
1. Open the side bar
2. Scroll to bottom, click **Dev Tools** under **Management**
3. Copy the following code in the console and run it sequentially 

We first convert string values to integer/float: 
(change the index name **20230714-linux** accordingly)

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

POST /20230725-linux/_update_by_query
{
  "query": {
    "match_all": {}
  },
  "script": {
    "source": "if (ctx._source.containsKey('time') && ctx._source.time instanceof String) { ctx._source.time = Double.parseDouble(ctx._source.time) }",
    "lang": "painless"
  }
}
```


And perform the required calculation:
```sh
POST /20230714-linux/_update_by_query
{
  "query": {
    "bool": {
      "filter": [
        { "exists": { "field": "time" } },
        { "exists": { "field": "numFactsProcessed" } }
      ]
    }
  },
  "script": {
    "source": "ctx._source.facts_per_second = ctx._source.numFactsProcessed / ctx._source.time",
    "lang": "painless"
  }
}

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
    "source": "ctx._source.rules_per_second = ctx._source.numRulesProcessed / ctx._source.time",
    "lang": "painless"
  }
}
```

### To create a visualisation from scratch
After ingesting and editing the data, we want to add the data in Kibana and create a new dashboard:
1. Open the sidebar, click **Management**
2. Under **Kibana**, choose **Data Views**
3. Create a new data view by choosing the index pattern you have defined earlier, the name and index pattern could be the same
4. After adding it to Kibana, click **Dashboard** under **Analytics** in the side bar
5. Click on **Create dashboard**
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
Bar charts can be used to visualise and compare performance between different RDFox versions, e.g., if we want to have a bar chart comparing facts_per_second for every test(LUBM-10, SUP-140...):
- **Select type --> Lens** and drag the required fields into the display
- Choose the appropriate **Visualisation type** and function (median, average...)
- To turn from a bar chart to grouped bar chart, we add a field in the **Breakdown**, on the right hand side
- Choose **Filters**, and filter out the 3 RDFox versions
- We can compare different tests by clicking the **horizontal axis** and filter out different test names

**Right next to visualisation type, there are settings to modify the bar chart, e.g., adding labels, scaling of x-axis and y-axis...

#### Filtering data
In order to filter data to a specific system, or a specific test name, you can either:
1. Use Kibana Query Language (KQL) on the upper bar
2. Click on a specific tag clouds

### Vega-Lite for scatter plot
There is no build-in scatter plot graph type so we will have to create from scratch using a high level visualisation grammar called [Vega-Lite](https://vega.github.io/vega-lite/). To open the Vega editor, when creating a new visualisation, choose **Custom visualisation** this time.
Before writing any code in the editor, we will have to edit our jsonl file first.

Upload a new copy of jsonl file, and filter out those data who is not query in the console: (change your index name accordingly)

```sh
POST /your_index/_delete_by_query
{
  "query": {
    "bool": {
      "must_not": [
        {
          "term": {
            "stepType": "query"
          }
        }
      ]
    }
  }
}
```
We then change the 3 rdfox_versions to "oldest", "previous", "latest" respectively as we will have to do [pivot transform](https://vega.github.io/vega-lite/docs/pivot.html) later on.
Format:
```sh
POST /20230725-linux-query/_update_by_query
{
  "query": {
    "match": {
      "rdfox_version": "3.1.1"
    }
  },
  "script": {
    "source": "ctx._source.rdfox_version = \"oldest\""
  }
}
```
And edit the repetition id to (#01, #02, #03) as we have to group queries results by repetition id later on:

```sh
POST /your_index/_update_by_query
{
  "query": {
    "match_all": {}
  },
  "script": {
    "source": "ctx._source.repetition_id = ctx._source.repetition_id.substring(ctx._source.repetition_id.length() - 3)"
  }
}
```

The data is now in the format that we want. Reopen the Vega editor and paste the following code:

``` sh
{
  $schema: https://vega.github.io/schema/vega-lite/v5.json
  title: RDFox 
  "mark": {"type": "point", "tooltip": {"content": "data"}}, 
  data: {
    url: {
      %context%: true
      index: 20230725-linux-query
      body: {
        size: 10000
        _source: ["test_name", "rdfox_version", "step", "stepType", "time", "repetition_id"]
      }
    }
    format: {property: "hits.hits"}
  }
  "transform": [
    {"pivot": "_source.rdfox_version", "value": "_source.time", "groupby": ["_source.step", "_source.test_name", "_source.repetition_id"]}
  ], 
  "encoding": {
    "x": {"field": "previous", "type": "quantitative", "title": "Time in Version 1"},
    "y": {"field": "latest", "type": "quantitative", "title": "Time in Version 2"}
  }
}
```




### If you have a saved dashboard `.ndjson`
1. Open the sidebar, scroll to the bottom, and click **Management**
2. In the **Kibana** section, choose **Saved objects**
3. Import the dashboard you want




## Others


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
```sh
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "data": {
    "values": [
      {"test": "ESWC2018", "Version": "1", "stepType": "query", "step": "query10", "time": "10"},
      {"test": "ESWC2018", "Version": "1", "stepType": "import", "step": "import-ttl", "time": "10"},
      {"test": "ESWC2018", "Version": "1", "stepType": "query", "step": "query11", "time": "9.5"},
      {"test": "ESWC2018", "Version": "2", "stepType": "query", "step": "query10", "time": "11"},
      {"test": "ESWC2018", "Version": "2", "stepType": "query", "step": "query11", "time": "10.3"},
      {"test": "ESWC2018", "Version": "3", "stepType": "query", "step": "query10", "time": "4"},
      {"test": "ESWC2018", "Version": "3", "stepType": "query", "step": "query11", "time": "6.5"},
      {"test": "LUBM-10", "Version": "1", "stepType": "query", "step": "query10", "time": "5.3"},
      {"test": "LUBM-10", "Version": "1", "stepType": "query", "step": "query11", "time": "4.6"},
      {"test": "LUBM-10", "Version": "2", "stepType": "query", "step": "query10", "time": "5.4"},
      {"test": "LUBM-10", "Version": "2", "stepType": "query", "step": "query11", "time": "4.5"},
      {"test": "LUBM-10", "Version": "3", "stepType": "query", "step": "query10", "time": "5"},
      {"test": "LUBM-10", "Version": "3", "stepType": "query", "step": "query11", "time": "6"}
    ]
  },
  "transform": [
    {"filter": "datum.stepType === 'query'"},
    {"pivot": "Version", "value": "time", "groupby": ["step", "test"]}
  ],
  "mark": {"type": "point", "tooltip": {"content": "data"}},
  "encoding": {
    "x": {"field": "1", "type": "quantitative", "title": "Time in Version 2"},
    "y": {"field": "2", "type": "quantitative", "title": "Time in Version 1"}
  }
}
```
```sh
{
  $schema: https://vega.github.io/schema/vega-lite/v5.json
  title: RDFox Compare
  "mark": {"type": "point", "tooltip": {"content": "data"}},
  data: {
    url: {
      %context%: true
      index: scatter
      body: {
        size: 10000
        _source: ["rdfox_version", "time", "repetition_id", "runId", "step", "stepType", "system", "architecture", "test_name"]
      }
    }
    format: {property: "hits.hits"}
  }
  "transform":[
    {"pivot": "_source.rdfox_version", "value": "_source.time", "groupby": ["_source.step", "_source.test_name"]}
  ]
  
  encoding: {
    x: {field: "3.1.1", type: "quantitative", title: "Version 3.1.1"}
    y: {field: "6.2", type: "quantitative", title: "Version 6.2"}
  }
}
```
```sh
{
  $schema: https://vega.github.io/schema/vega-lite/v5.json
  title: RDFox Compare
  "mark": {"type": "point", "tooltip": {"content": "data"}},
  data: {
    url: {
      %context%: true
      index: scatter1
      body: {
        size: 10000
        _source: ["test", "Version", "step", "stepType", "time"]
      }
    }
    format: {property: "hits.hits"}
  }
  "transform":[
    {"pivot": "_source.Version", "value": "_source.time", "groupby": ["_source.step", "_source.test"]}
  ],
  encoding: {
    x: {field: "1", type: "quantitative", title: "Version 3.1.1"}
    y: {field: "2", type: "quantitative", title: "Version 6.2"}
  }
}
```
```sh
POST /your_index/_delete_by_query
{
  "query": {
    "bool": {
      "must_not": [
        {
          "term": {
            "stepType": "query"
          }
        }
      ]
    }
  }
}
```
```sh
POST /20230725-linux-query/_update_by_query
{
  "query": {
    "match": {
      "rdfox_version": "3.1.1"
    }
  },
  "script": {
    "source": "ctx._source.rdfox_version = \"oldest\""
  }
}
```

Worked code:
```sh
{
  $schema: https://vega.github.io/schema/vega-lite/v5.json
  title: RDFox
  "mark": {"type": "point", "tooltip": {"content": "data"}},
  data: {
    url: {
      %context%: true
      index: 20230725-linux-query
      body: {
        size: 10000
        _source: ["test_name", "rdfox_version", "step", "stepType", "time", "repetition_id"]
      }
    }
    format: {property: "hits.hits"}
  }
  "transform": [
    {"pivot": "_source.rdfox_version", "value": "_source.time", "groupby": ["_source.step", "_source.test_name", "_source.repetition_id"]}
  ],
  "encoding": {
    "x": {"field": "previous", "type": "quantitative", "title": "Time in Version 1"},
    "y": {"field": "latest", "type": "quantitative", "title": "Time in Version 2"}
  }
}
```
