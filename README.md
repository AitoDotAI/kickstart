# kickstart
This guide provides practical and immediately actionable steps for getting started with Aito.

# 1. Introduction
Welcome to the kickstart guide with Aito.ai with all the necessary steps to get started! We start with looking at how Aito works when everything has already been set up, and then we see how to set everything up ourselves. To follow this guide, you may use your favorite REST client or paste the queries directly to your command line. Let’s go!

# 2. Aito in Action
We’ll first show you how Aito looks like in action. We have already set up a public Aito instance for everyone to play with. 

Let’s start off with a fun example where we’ll automatically detect sarcasm in Reddit discussions. Below is a snapshot of how the data looks like. The “label” column indicates whether or not the comment is sarcastic; it is the first thing we want to predict.

![Sample data image](https://github.com/AitoDotAI/kickstart/blob/master/Screenshot%202019-10-15%20at%2015.28.34.png)

To predict if the sentence “dude you are so smart” is sarcastic on Reddit, simply copy the following query to your command line and hit enter (pay attention to the bolded parts, they are our playground):

```
curl --request POST \
  --url https://aito-reddit-sarcasm.api.aito.ai/api/v1/_predict \
  --header 'content-type: application/json' \
  --header 'x-api-key: 9Ik1wJQ1tq86vMQG7taDB2cgfpSogUFu69lBGTnV' \
  --data '{
    "from": "comments",
    "where": {
      "comment": "dude you are so smart"
    },
    "predict": "label"
  }
'
```

The response looks like this:

```
{
  "offset" : 0,
  "total" : 2,
  "hits" : [ {
    "$p" : 0.8432771866750243,
    "field" : "label",
    "feature" : 1
  }, {
    "$p" : 0.15672281332497587,
    "field" : "label",
    "feature" : 0
  } ]
}
```

The response to this query contains the likelihood (“$p”) of each class (“feature”). This comment is indeed quite sarcastic. In fact, Aito thinks it is sarcastic with 84% probability.

Looking at the query you’ll notice that we only provided Aito with the comment and not the other data features. Let’s include “subreddit” in the query and see what happens:

```
curl --request POST \
  --url https://aito-reddit-sarcasm.api.aito.ai/api/v1/_predict \
  --header 'content-type: application/json' \
  --header 'x-api-key: 9Ik1wJQ1tq86vMQG7taDB2cgfpSogUFu69lBGTnV' \
  --data '{
    "from": "comments",
    "where": {
      "comment": "dude you are so smart",
	     "subreddit": "politics"
    },
    "predict": "label"
  }
'
```

And the response you get from Aito:
```
{
  "offset" : 0,
  "total" : 2,
  "hits" : [ {
    "$p" : 0.9031693595826128,
    "field" : "label",
    "feature" : 1
  }, {
    "$p" : 0.09683064041738716,
    "field" : "label",
    "feature" : 0
  } ]
}
```
Being posted in the “Politics” Subreddit increases the chances that this comment is sarcastic to 90%. Meanwhile, if you try the same query but replace “politics” with “science”, the probability of sarcasm reduces dramatically to just 62%.

Go ahead and change the sentence to whatever you want and select a new subreddit. Here’s a few random subreddits for you to try:

AskReddit
nba
science
videos
pcmasterrace

Well done! Now, after asking Aito if there’s sarcasm in the comment, let’s ask why it thinks so. Understanding the reasoning of a prediction is critical in any Machine Learning application! To receive more information than just the prediction, add a “select” line in the query which returns the predicted label (“feature”), its probability (“$p”) and most importantly, the reasoning (“$why”). Copy the following curl:
```
curl --request POST \
  --url https://aito-reddit-sarcasm.api.aito.ai/api/v1/_predict \
  --header 'content-type: application/json' \
  --header 'x-api-key: 9Ik1wJQ1tq86vMQG7taDB2cgfpSogUFu69lBGTnV' \
  --data '{
    "from": "comments",
    "where": {
      "comment": "dude you are so smart",
	     "subreddit": "politics"
    },
	  "select": ["feature","$p","$why"],
   "predict": "label"
  }
'
```

As requested, the query returns:
```
{
  "offset" : 0,
  "total" : 2,
  "hits" : [ {
    "feature" : 1,
    "$p" : 0.6155384924100668,
    "$why" : {
      "type" : "product",
      "factors" : [ {
        "type" : "baseP",
        "value" : 0.5
      }, {
        "type" : "normalizer",
        "name" : "exclusiveness",
        "value" : 0.9015948211626251
      }, {
        "type" : "relatedVariableLift",
        "variable" : "subreddit:science",
        "value" : 0.3999850073479482
      }, {
        "type" : "relatedVariableLift",
        "variable" : "comment:smart",
        "value" : 1.520886821001524
      }, {
        "type" : "relatedVariableLift",
        "variable" : "comment:dude",
        "value" : 1.257895343712437
      } ]
    }
  }, { ...
```
A lot of stuff is crammed in the response but bear with us. What’s important right now are the factors called “relatedVariableLift”. They tell us what Aito thinks during inference. Lifts represent multipliers which impact the final outcome, and according to Aito, the words “smart” and “dude” are especially likely to be sarcastic. For example, the word “smart” increases the sarcasm probability of the comment by 1.52x but being on the science subreddit decreases it by 0.39x.

Another very common type of inference is recommendation. With the below curl, Aito will predict the subreddit where a specific user is least likely to be sarcastic. Laid out, it says “recommend me a subreddit where the value of ‘label’ is likely 0 when the value of ‘author’ in the ‘comments’ table is ‘Creepeth’”.
```
curl --request POST \
  --url https://aito-reddit-sarcasm.api.aito.ai/api/v1/_recommend \
  --header 'content-type: application/json' \
  --header 'x-api-key: 9Ik1wJQ1tq86vMQG7taDB2cgfpSogUFu69lBGTnV' \
  --data '{
	"from": "comments",
	"where": {
		"author": "Creepeth"
	},
	"recommend": "subreddit",
	"goal": {"label": 0}
}'
```
# 3. Setting up your own Aito
Now we take a few steps back and start from uploading your own data into a schema. Or in case you don’t right now have suitable data available, you may download and use the same CSV files we use in this demo. The demo files are found in this repositry: "reddit_sample.csv" contains a sample of the full comments dataset and "users.csv" is a completely fabricated table to represent the Reddit users. To keep following the guide, you’ll need API keys which allow you to access your Aito instance. If you don’t have an API key yet, please contact us here.

Setting up your own schema has three distinct steps. First we look at the schema structure, then create an empty schema and tables, and finally upload the data into their correct tables. We highly recommend using a REST client for HTTP-API interaction.

## Schema planning
Very often your data is stored in a collection of connected tables. In our Reddit example, the data is divided into two tables: Comments and Users. Aito will be able to find information across linked tables, and in our case the tables are linked together with “author” column. We could also link in a third table describing for example the subreddits, daily weather or something else. At this point, you need to go through your files and determine which tables and columns are linked.

![Schema example image](https://github.com/AitoDotAI/kickstart/blob/master/Screenshot%202019-10-15%20at%2015.26.39.png)

Now that we know the structure, we can start creating the database schema.

## Creating your Aito schema
Now we define the tables, columns and their individual types and other specifications in JSON format. This important step is a little tedious but necessary for ensuring the correct format for the data. In the next chapter we introduce our Command Line Interface (currently alpha version) to automate some of this process.
```
The structure of the JSON is following:
{
“schema”: {
		“table1”: {
			“columns”: {
				“column1”: {“type”: “Data type”},
        “column2”: {“type”: “Data type”, “analyzer”: “english”},
        ...
        },
        “type”: “table”
      },
      “table2”: {
        “columns”: {
          “column1”: {“type”: “Data type”, “link”: “table1.column2”},
        “column2”: {“type”: “Data type”},
        …
      },
      “type”: “table”
  },
    ...
  }
}
```
Double check the JSON with care and add any necessary specifications such as [links](https://aito.ai/docs/api/#post-api-v1-search) (connects two tables on one column) and [text analyzers](https://aito.ai/docs/api/#schema-analyzer) (treats the value as separate words instead of a single class). Make sure the linked columns have the same type. With our sample data, here’s how the full schema creation request looks like (replace the environment name and api key with yours):

```
curl --request PUT \
  --url https://your-env-name.api.aito.ai/api/v1/schema \
  --header 'content-type: application/json' \
  --header 'x-api-key: your-rw-api-key' \
  --data '{
  "schema": {
    "users": {
      "type": "table",
      "columns": {
        "registered": {
          "type": "String",
          "nullable": false
        },
        "score": {
          "type": "Int",
          "nullable": false
        },
        "user": {
          "type": "String",
          "nullable": false
        }
      }
    },
    "comments": {
      "type": "table",
      "columns": {
        "subreddit": {
          "type": "String",
          "nullable": false
        },
        "author": {
          "type": "String",
          "nullable": false,
          "link": "users.user"
        },
        "score": {
          "type": "Int",
          "nullable": false
        },
        "label": {
          "type": "Int",
          "nullable": false
        },
        "downs": {
          "type": "Int",
          "nullable": false
        },
        "date": {
          "type": "String",
          "nullable": false
        },
        "comment": {
          "type": "Text",
          "nullable": false,
          "analyzer": "english"
        },
        "ups": {
          "type": "Int",
          "nullable": false
        },
        "parent_comment": {
          "type": "Text",
          "nullable": false,
          "analyzer": "english"
        },
        "created_utc": {
          "type": "String",
          "nullable": false
        }
      }
    }
  }
}'
```
If there were no errors, the request sets up and returns the full schema structure. You can view your empty schema again with:
```
curl --request GET \
  --url https://your-env-name.api.aito.ai/api/v1/schema \
  --header 'content-type: application/json' \
  --header 'x-api-key: your-rw-api-key'
```
## Uploading data to Aito
Aito expects data in a records-oriented JSON where each row is an individual item, such as:
```
[
{"user":"Trumpbart","registered":"2018-5-30","score":-111},
{"user":"Shbshb906","registered":"2018-7-10","score":124},
{"user":"Creepeth","registered":"2013-4-6","score":10},
…
]
```
To convert CSV into JSON format, you may use any script or converter you like, or try the Command Line Interface discussed in the next chapter. With your data in JSON, you may now upload it to each table. This batch upload request can support up to 50 000 rows at a time. If your file contains more rows, you may use a script to loop through the data. The following curl uploads up to 50 000 rows to the “users” table:

```
curl --request POST \
  --url https://your-env-name.api.aito.ai/api/v1/data/users/batch \
  --header 'content-type: application/json' \
  --header 'x-api-key: your-rw-api-key' \
  --data '
  [
		{"user":"Trumpbart","registered":"2018-5-30","score":-111},
		{"user":"Shbshb906","registered":"2018-7-10","score":124},
		{"user":"Creepeth","registered":"2013-4-6","score":10}
		...
  ]'
```
Repeat the process for each table you wish to upload data to. To view the content of the “users” table, you may use:
```
curl --request POST \
  --url https://your-env-name.api.aito.ai/api/v1/_query \
  --header 'content-type: application/json' \
  --header 'x-api-key: your-rw-api-key' \
  --data '{
	"from": "users",
	"limit": 10
}'
```
Your data is now ready for predictions!

## Using Command Line Interface
The Command Line Interface (CLI) is a tool to introduce automation into this process. It helps you by generating the table JSONs required for schema creation, converts CSV data to JSON, and uploads data to your schema. Please do note it’s still an early alpha version. You'll need to run the commands for each data table in the schema.

Here’s how to get started:
1. Install Aito CLI (requires Python 3.6+) on command line: ```pip install aitoai``` https://github.com/AitoDotAI/aito-python-tools
2. On your command line, cd to the folder with your data files
3. Run the following command for each of your CSV data files:
   ```aito convert -c tablefile.json -f json csv datafile.csv datafile.json```

The last command generates two JSON files from your CSV file. “tablefile.json” contains the structure of the table which you can copy to your schema creation request. “datafile.json” contains all the CSV data converted into JSON format for uploading. Make sure to change the file names for each CSV file you convert.

To upload data with CLI, use the following command for each of your data files.
```aito client -u https://your-env-name.api.aito.ai -r your-ro-key -w your-rw-key upload-batch your-table-name < your-datafile.json```

This might take a while but you’ll be able to follow the progress on the command line.

## Inference
Finally your schema is ready for the fun part. Try again the prediction queries of the first chapter but this time in your own environment. Enjoy!
