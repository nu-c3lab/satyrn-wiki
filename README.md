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


## Setting up the application

### Setting up the API

The API repo is [here](https://github.com/nu-c3lab/satyrn-api) and should be cloned into the whichever machine will be running the API. There is more in-depth documentation in that repo that details more about the inner workings of satyrn as well as how to set up the API. In this wiki we give a brief overview of the setup

#### Environment variables

These are the needed environment variables to be set up for the system. More information on these can be seen in the `env-example.txt` file in the repo.

```
FLASK_APP=core
FLASK_ENV="development" # the environment setting TODO: andrew check if there are extra things we wanna say about this; also in the server it is as SATYRN_ENV
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

#### Installing the needed libraries

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

#### Running the API

To run the API, we simply run the command
```
flask run
```
#### Testing the setup

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

### Setting up the UX



### Setting up the Plan Manager



## Creating a satyrn ring




### Sample ring

To see a sample ring, 