# satyrn-wiki
Repo describing how to set up an instance of a satyrn application

## Overview of the needed repos and files

A `satyrn` application instance contains code from three main repos:
- `satyrn-api` is the analytical backbone and the backend of `satyrn`. It is a Flask application that is listening for api requests and hands back the needed information about a given request, such as what are the available entities for analysis, returning the results for a search, and returning the results for an analysis request.
- `satyrn-ux` contains both the user-facing code (i.e. the front-end that displays notebooks, visually shows results, and shows analyses), and the code taht handles the user-creation system (i.e. creating, storing, and maintaining the databases for users in the platform).
- `satyrn-PlanManager` contains the logic for plan creations (i.e. based on a given ring that specifies the domain, creating the analytic plans that can be answered by satyrn about the domain).

In addition to installing and running the above repos, we also require 
- read access to the domain database via its location string
- the creation of a `ring`: a json file that describes the semantics and the database of the domain at hand
- the creation of a `site.json` file that contains a pointer to the `ring` as well as additional information on how to run the satyrn application

This repo will briefly walkthrough the three repos, with mostly an emphasis on how to set up each of the repos for running the entire platform. We also provide a `ring` and `site.json` example along with the database that it is describing.

## Setting up the API

The API repo is [here](https://github.com/nu-c3lab/satyrn-api) and should be cloned into the whichever machine will be running the API. There is more in-depth documentation in that repo that details more about the inner workings of satyrn as well as how to set up the API. In this wiki we give a brief overview of the setup

### Environment variables

These are the needed environment variables to be set up for the system. More information on these can be seen in the `env-example.txt` file in the repo.

TODO: andrew check if there are extra things we wanna say about FLASK_ENV; also in the server where stuff is deployed it is as SATYRN_ENV

```
FLASK_APP=core
FLASK_ENV="development" # the environment setting 
SATYRN_CONFIG_VERSION=2 # the configuration version of satyrn. As we continue development, this would get upgraded

API_KEY="thisisatestkey" # an API key for more secure API calls; can be any string but we recommend something stronger than the sample
UX_SERVICE_KEY="anothertestkey" # another API key for the UX service; if none is given for this, it defaults to the API_KEY
SATYRN_SECRET_KEY= 'sample secret key value' # Generate a good key with secrets.token_urlsafe()
SATYRN_SECURITY_SALT='sample security salt' # Generate a good salt with secrets.SystemRandom().getrandbits(128)

SATYRN_ROOT_DIR=<relevant pointer to the root directory of the satyrn-api repo>
SATYRN_SITE_CONFIG=<relevant pointer to a site.json configured like the one in the example below which should include references to ring config JSON files to bootstrap at platform initialization>

USER_DB=<relevant pointer to the database file for storing user information>
UX_SERVICE_API=<relevant pointer to the url where the UX service is being hosted. e.g., for local development it can be set to "http://localhost/api/">
```

### Installing the needed libraries

We recommend creating a python virtual environemnt for this application, which can be down with the following steps:
```
cd satyrn-platform
python -m venv venv   # <-- yup, the .gitignore assumes you have a venv dir named venv
source venv/bin/activate
```
Then, install the needed requirements
```
pip install -r requirements.txt
```

### Running the API

To run the API, we simply run the command
```
flask run
```
### Testing the setup

To test the setup, we need to send requests to the running API via an interface such as [Postman](https://www.postman.com/downloads/).
For instance, a properly formed Postman request for getting the sum of contribution amounts in the database would contain the following:
- In the headers, we have a key `x-api-key` with the value `<API_KEY>`
- The url would be `<api_host>:<api_port>/api/analysis/<ring_id>/1/<searchable_entity>`
	- for example, if we are running this in localhost on port 5000, on a ring with an id of "thisisaringid" on the searchable entity "Contribution", we would have the following url
	```
	http://127.0.0.1:5000/api/analysis/thisisaringid/1/Contribution/
	```
- In the json body of the request, we would have
	```
	{
	    "target": {
	        "entity": "Contribution",
	        "field": "amount"
	    },
	    "op": "sum",
	    "relationships": []

	}
	```

## Setting up the UX

The UX consists of two main parts: the client and the server. The client contains the front-facing code, meaning most of the things that the end user see, such as the notebooks themselves or the general platform UI. The server supports the client by providing it all the information that it needs, such as maintaining the user database system.

The server and the client should be run on different terminals simultaneously.

### Server

The server will be a Node.js application that will utilize a Postgres database to store all of the user information.

#### Database

For local development make sure to have a Postgres instance up and running.
https://www.codecademy.com/article/installing-and-using-postgresql-locally

#### Environment variables

To set up the environment variables, create a file called `.env` and save it in `./server/` (i.e. the file path is `./server/.env`).
These are the needed environment variables to be set up for the system. More information on these can be seen in the `env.example` file in the repo.

TODO: Fill in these
```
UX_SERVER_PORT=
DB_USERNAME=
DB_PASSWORD=
DB_NAME=
DB_HOST=
DB_DIALECT=
DB_PORT=
JWT_SECRET=
JWT_EXP=
CORE_API_KEY=
CORE_API_ENDPOINT=
SENDGRID_API_KEY=
SENDGRID_FROM_SENDER=
UX_CLIENT_URL=
RATE_LIMITER_MAX_REQUESTS=
RATE_LIMITER_TIME_WINDOW=
```

#### Running

To run the server, the following commands should suffice
```
cd server 
npm install
npm run start
```

### Client

The client will be a Node.js application that will be making calls to the server to fetch the required information.

####  Environment variables

To set up the environment variables, create a file called `.env` and save it in `./client/` (i.e. the file path is `./client/.env`).
These are the needed environment variables to be set up for the system. More information on these can be seen in the `env.example` file in the repo.

TODO: fill in these
```
## CLIENT
PORT=80
INLINE_RUNTIME_CHUNK=false
REACT_APP_VERSION=$npm_package_version
REACT_APP_NAME=$npm_package_name
```

#### Running

To run the server, the following commands should suffice

```
cd client 
npm install
npm run start
```


#### Testing

To test if the system is working, you can utilize the following test user and admin accounts:

User: user@testing.test:Pass-word-25

Admin: admin@testing.test:Pass-word-25


TODO: doublecheck that these get automatically created or are valid

### Deployment

TODO: check with andrew
TODO: check the remaining environment variables in `.env.example`, i.e.
```
## DEPLOYMENT
STAGE=

## SEEDS
SEED_ADMIN_PASSWORD=
```


## Setting up the Plan Manager

TODO: go over this with andrew


## Creating a `satyrn` ring

As stated in the introduction, a `satyrn ring` defines the database and the semantics of a domain so that the `satyrn` platform can provide search and analysis capabilities. Based on the configuration of the `ring`:
- the `planManager` creates possible analytic plans for the users to ask questions
- the API:
	- creates an ORM to connect to the database specified in the `ring`
	- creates the searching options for the `ux` to display to users
	- computes the needed answers for searching and analysis

To see a sample ring and a sample site file, check the `sample_ring_folder/` in this repo. We will go through the main parts of the ring and what each of these corresponds to utilizing the sample ring file

### High level keys

```
{
  "id": 1,
  "userId": 1,
  "rid": "20e114c2-ef05-490c-bdd8-f6f271a6733f",
  "name": "Basic Ring",
  "description": "A simple ring for testing and development based on the political contributions data.",
  "version": 1,
  "schemaVersion": 2.1,
  "dataSource": {...},
  "ontology": {...},
  "visibility": "public",
  "createdAt": "2021-11-18T01:07:54.696Z",
  "updatedAt": "2021-11-18T01:07:54.696Z"
}
```

The high-level keys are mostly metadata that help contextualize the `ring`:
- `id`:
- `userId`:
- `name`: the human-readable name for the `ring`/domain
- `description`: a brief description of the `ring`
- `version`: the version of the `ring`. If the `ring` is updated, the version number of the `ring` would be upgraded, but we would still be able to associate the different versions with the same domain
- `schemaVersion`: the schema version that is utilized. As `satyrn` is upgraded, rings that utilize a higher schema version would indicate so here
- `dataSource`: a dictionary that contains the information for the database utilized in the ring
- `ontology`: a dictionary containing the semantics for plan generation, search, and analysis
- `visibility`: whether we want "public" or "private" visibility of this `ring`
- `createdAt`: 
- `updatedAt`: 

TODO: are the createdAt and updatedAt created automatically?
TODO: blurbs for id and userId

The bulk of the information utilized by the `api` and `planManager` are in the `dataSource` and `ontology` keys.

### `dataSource`

```
  "dataSource": {
    "type": "sqlite",
    "connectionString": "<insert_rest_of_path>/satyrn-wiki/sample_ring_folder/basic.db",
    "tables": [...],
    "joins": [...]
  },
```

The `dataSource` key contains the information for the database utilized in the ring. The main four keys are:
- `type`: the underlying database type that is used in this data source. The available types are sqlite, postgres, and csv
- `connectionString`: the read access connection string to the data source. This can be either a local path for the file (sqlite), a local path for a folder of files (csv), or a path for a remote database (postgres)
- `tables`: a list of dictionaries that specify the tables to be used in this `ring`. Note that not all tables in the data source have to be listed here; however, only the ones in this list will be available for the system to use (if they are utilized within the `ontology`). Additionally, not all listed tables need to be utilized in the rest of the system.
- `joins`: a list of dictionaries that specify the joins within tables to be used in this `ring`. Similar to the caveat of the `tables`, not all joins need to be listed and only the ones listed can be used. We assume that any table listed in a join will be listed under the `tables` list.

#### `tables`

```
  {
    "name": "contribution",
    "primaryKey": "id",
    "pkType": "integer"
  }
```

Each table is specified by three main attributes:
- `name`: the name of the table in the database (sqlite/postgres) or the name of a file containing a tabular table (csv)
- `primaryKey`: the name of the column utilized as a primary key for that table
- `pkType`: the data type for the primary key

#### `joins`

```
  {
    "name": "contributor",
    "from": "contribution",
    "to": "contributor",
    "path": [
      [
        "contribution.contributor_id",
        "contributor.id",
        "integer"
      ]
    ],
    "bidirectional": true
  }
```

Each join requires five main keys:
- `name`: the name of the join (utilized internally)
- `from`: the left table for the join
- `to`: the right table for the join
- `path`: the join path from the `from` table to the `to` table. A join path is a list of tuples with 3 attributes: (leftTable.joinKey, rightTable.joinkey, joinType)
	- Note that a join path can contain multiple "hops". Simply by adding more tuples, we can extend the join path across multiple tables. So long as all the tables utilized are listed in the `tables` list and the beginning and end tables for the join line up with the `from` and `to` tables, the join should be valid
	- Currently the only joinType available is "integer" (meaning that we expect integer equality between the two join keys)
- `bidirectional`: whether the join works for both directions (meaning we can swap the left and right tables for an equally valid join)

### `ontology`

```
"ontology": {
    "defaultTargetEntity": "Contribution",
    "relationships": [...],
    "entities": [...]
  },
```

An ontology is defined by three main keys:
- `defaultTargetEntity`: the name of the entity in `entities` that will be utilized as the default entity for searching
- `relationships`: a list of dictionaries that specify how each of the entities are related to each other (if at all)
- `entities`: a list of dictionaries that specify each of the entities (e.g., their attributes, their naming convention, how to display them, etc.)

#### `relationships`

```
  {
    "name": "ContribToContributor",
    "from": "Contribution",
    "to": "Contributor",
    "join": "contributor",
    "relation": "m2o",
    "bidirectional": true
  }
```

A relationship has the following keys:
- `name`: the name of the relationship (utilized internally and in the analytics plans)
- `from`: the name of the first entity
- `to`: the name of the second entity
- `join`: any sql joins needed to be able to conduct analyses utilizing both entities (join names utilized from the `joins` key defined in `dataSource`)
	- Note that joins are optional in the case where there might be multiple entities within the same table. For instance, even in this sample ring we could have created a third entity called "Contribution Recipient" rather than having it as an attribute in the "Contributions" entity. This hypothetical entity would have had a one to many relation to "Contributions" (one recipient can have many contributions), yet it would not have had any SQL joins since it resides on the same table as "Contributions"
- `relation`: whether the relationship is many to one (m2o), one to many (o2m), many to many (m2m) or one to one (o2o). The concept of `relation` follow analogously to that of types of joins in SQL databases
- `bidirectional`: whether the relationship is bidirectional between the entities

#### `entities`

```
{
    "name": "Contributor",
    "table": "contributor",
    "id": "id",
    "idType": "integer",
    "renderable": false,
    "reference": "${name} from ${parentOrg}",
    "attributes": {...}
}
```

An entity contains the following keys:
- `name`: name of the entity
- `table`: the main table that defines an entity (from the `tables` list in `dataSource`)
- `id`: the name of the id that uniquely identifies an entity (should be a column in the table defined in `table` in this entity)
- `idType`: the data type of the id
- `renderable`: whether the entity has associated with it an HTML rendering
- `reference`: the human-readable way of referencing this entity. Anything within the format `${attributeName}` will be referencing an attribute of that entity (e.g.,in that case it will expect an attribute called "attributeName"). Anything outside `${}` will be rendered as is
- `attributes`: a dictionary of attributes for that entity

TODO: donna add any modifications needed for the renderable bit

##### `attributes`

Attributes are among the most important building blocks of doing analyses in `satyrn`, since often we care not only about the concept of the entity itself, but also about certain attributes that each entity has. The main keys that each attribute has are:
- `nicename`: a list of length two detailing the human readable name of the attribute. The first entry is in singular form, the second is in plural
- `units`: (optional) a list of lenght two detailing the units utilized (singular, plural). If not given, it will utilize the `nicename` as units
- `isa`: the data type of the attribute
	- Note: eventually this will be expanded upon to allow for more semantic typing
- `source`: a dictionary detailing the data source of the attribute
	- `table`: the table from which we can obtain this attribute (often it is the same as the `table` in the entity)
	- `columns`: the a list of columns to obtain the attribute. Often, these are of length 1, but can be longer if we need multiple columns to assemble the attribute (e.g. a name could be composed of columns first_name, middle_name, last_name)
- `metadata`: additional metadata that details how `satyrn` is able to utilize this attribute in its searching and analytics capabilities
	- `searchable`: whether we want to allow searching over this attribute
	- `allowMultiple`: when searching, whether we allow multiple filters over this attribute (for instance, in a domain where one judicial case has many judges, we might want to serach for cases with judge 123 and judge 234)
	- `searchStyle`: style of search widget in the UX. Available styles are "range", 
	- `analyzable`: whether this attribute is available for analysis (often this is set to true)
	- `threshold`: (optional) for numeric attributes, allows bucketing the attribute into different categories to be able to conduct different analyses
		- e.g., in the sample `ring` for Contribution, we can count Contributions goruped by their amount, where the different categories are less than or equal to 250, between 250 and 500, between 500 and 1000, and over 1000
	- `autocomplete`: (optional, default false) whether we want to allow the UX to do autocompleting when searching over that attribute

TODO: donna add things about multi-table entity for attributes on different tables
TODO: double check search styles. rightn ow i think a bunch of them are just range?


We list a couple of examples of attributes that follow the format described above
```
"amount": {
    "nicename": [
      "Contribution Amount",
      "Contribution Amounts"
    ],
    "units": [
      "dollar",
      "dollars"
    ],
    "isa": "float",
    "source": {
      "table": "contribution",
      "columns": [
        "amount"
      ]
    },
    "metadata": {
      "searchable": true,
      "allowMultiple": false,
      "searchStyle": "range",
      "analyzable": true,
      "threshold": ["x <= 250", "250 < x <= 500", "500 < x <= 1000", "1000 < x"]
    }
}
```

```
"contributionRecipient": {
    "nicename": [
      "Recipient",
      "Recipients"
    ],
    "isa": "string",
    "source": {
      "table": "contribution",
      "columns": [
        "contribution_recipient"
      ]
    },
    "metadata": {
      "searchable": true,
      "allowMultiple": false,
      "searchStyle": "range",
      "analyzable": true,
      "autocomplete": true
    }
}
```


```
"parentOrg": {
    "nicename": [
      "Contributor Parent Organization",
      "Contributor Parent Organizations"
    ],
    "isa": "string",
    "source": {
      "table": "contributor",
      "columns": [
        "parent_org"
      ]
    },
    "metadata": {
      "searchable": true,
      "allowMultiple": false,
      "searchStyle": "range",
      "analyzable": true,
      "description": "The organization this contributor is associated with."
    }
}
```

### `site.json`


```
{
    "name": "Sample Ring",
    "icon": "",
    "description": "A simple political contributions app leveraging the 2.1 config schema.",
    "rings": ["<insert_rest_of_path>/satyrn-wiki/sample_ring_folder/ring.json"]
}
```

The `site.json` file describes some metadata about the `satyrn` site application. The four main keys are:
- `name`: the name of the site
- `icon`: ,
- `description`: a brief description of the site
- `rings`: a list of file paths to the rings that are to be utilized in the site. This allows for a single `satyrn` site to contain multiple rings available for analysis

TODO: what do we say about icon

## Putting it all together

TODO: final thing about how it all comes together