# Satyrn Wiki
Repo describing how to set up an instance of a Satyrn application

## Overview of the needed repos and files

A `satyrn` application instance contains code from three main repos:
- `satyrn-api` is the analytical backbone and the backend of `satyrn`. It is a Flask application that is listening for api requests and hands back the needed information about a given request, such as what are the available entities for analysis, returning the results for a search, and returning the results for an analysis request.
- `satyrn-ux` contains both the user-facing code (i.e. the front-end that displays notebooks, visually shows results, and shows analyses), and the code that handles the user-creation system (i.e. creating, storing, and maintaining the databases for users in the platform).
- `satyrn-PlanManager` is a separate JavaScript library imported by the satyrn-ux build and contains the logic for plan creations (i.e. based on a given ring that specifies the domain, creating the analytic plans that can be answered by satyrn about the domain) and results rendering.

In addition to installing and running the above repos, we also require 
- the creation of a `ring`: a json file that describes the semantics and the database of the domain at hand
- read access to the domain database via its location string (which is entered into the `ring` json file)
- the creation of a `site.json` file that contains a pointer to the `ring` as well as additional information on how to run the satyrn application

This repo will briefly walkthrough the three repos, with mostly an emphasis on how to set up each of the repos for running the entire platform. We also provide a `ring` and `site.json` example along with the database that it is describing.

For a walkthrough of each of the components, visit the wiki in the github repo