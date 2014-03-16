VIVO DASHBOARD
==============

VIVO Dashboard is a Drupal-based application for analyzing publication metadata coming from VIVO, a semantic researcher profile system. VIVO Dashboard allows end users (primarily administrators) to easily generate reports in one of the three way: bar graphs, HTML lists, and spreadsheets. End users can facet publication records in a variety of ways including:

- year
- publication type
- journal ranking
- journal name
- author name
- author type
- organizational affiliation
- first-last author affiliation


HOW IT WORKS
============

Import process
--------------

All VIVO sites expose their data as RDF. VIVO Dashboard routinely retrieves batches of RDF data from VIVO and turns it into Drupal content using Drupal's "Feeds" module.

VIVO's index pages contain links to RDF files listing all individuals in a particular class. VIVO Dashboard uses these RDF lists as feeds, similar to RSS feeds, periodically checking the lists to see if publications have been added or removed. Individuals linked to publications, such as authors and journals, are imported on demand.

The import process is quite slow. Tens of thousands of publications will typically take between a few days and week to import, depending on the performance of the VIVO site.

Drupal interface
----------------

VIVO Dashboard saves imported data as standard Drupal content.  

- Publications -> nodes
- Authorships -> relations
- Authors -> nodes
- Journals -> taxonomy terms
- Departments -> taxonomy terms

By storing imported data as Drupal content it becomes possible to leverage the thousands of available Drupal modules. VIVO Dashboard uses a number of popular modules to power its faceted search interface, including:

- Views
- Search API
- Facet API

The main interface included with the application is highly customizable. These modules allow for great flexibility, and administrators can adjust the interface to suit their institution's needs.

INSTALLATION
============

VIVO Dashboard is made available as a Drupal install profile and does not actually include all the required packages. Before installing, the codebase must be "built" using Drush. 

Building with Drush
-------------------

Install Drush by following the instructions at: https://github.com/drush-ops/drush

Then run:

    drush make https://raw.github.com/milesw/vivodashboard/master/distro.make vivo-dashboard

If no errors were reported, you should should have a complete VIVO Dashboard codebase inside the "vivo-dashboard" directory.

Drupal hosting
--------------

If you have your own hosting, or want to install VIVO Dashboard in your own local development environment, review the documentation on installing Drupal here : https://drupal.org/documentation/install (Step 1 is already done).

If you have don't have a preference for hosting, Pantheon is an excellent option. Development sites are free. 

If you want to get up and running quickly on your local machine, Acquia Dev Desktop is a great self-contained LAMP environment for Windows and Mac, designed for Drupal.

Installing on Pantheon
----------------------

1. Create a new site on Pantheon and proceed through the Drupal installation. Info you enter does not matter at this point.
2. Put the Pantheon site in SFTP mode.
3. Inside the "vivo-dashboard" directory you built using Drush, locate the "profiles/vivodashboard" directory.
4. Connect to the Pantheon FTP server and upload your "profiles/vivodashboard" to the corresponding "profiles" directory on Pantheon. This may be inside a "code" directory.
5. In the Pantheon dashboard go to "Workflow" and then "Wipe". Confirm and wipe the environment.
6. Click on "Visit Development Site".
7. You should see a Drupal installation page again, with an option to install VIVO Dashboard. Choose that option and proceed through the installation. If there is no such option, you probably did not upload "profiles/vivodashboard" to the correct location. 
8. When the installation is finished you will be taken to VIVO Dashboard's main page.

Note: If you're comfortable using Git, steps 2-4 can be done with Pantheon's Git repository for the site. You'll need to add an SSH key in the Pantheon dashboard.

Installing with Acquia Dev Desktop
----------------------------------

1. Install the Acquia Dev Desktop application
2. Open the Acquia Dev Desktop Control Panel, go to Settings, then Sites, then Import.
3. For the "Site path" locate the "vivo-dashboard" directory you built using Drush.
4. For the database select "Create new database".
5. Enter anything you want for the domain. 
6. After clicking "Import" you should be brought to a Drupal installation page.
7. Choose the "VIVO Dashboard" option and proceed through the installation.
8. When the installation is finished you will be taken to VIVO Dashboard's main page.

SETUP
=====

Starting an Import
------------------

Once installed, the first thing to do is start the import.

1. Visit the /import page, then choose the "VIVO Publications" importer.
2. Enter your VIVO site's URL in the form.
3. Enter the URI for the class you'd like to import (e.g. http://purl.org/ontology/bibo/AcademicArticle). 
4. Click "Import". 

The process will take a few moments to initialize. Once you see a progress bar showing a percentage, the import is underway. Now you'll want to set up cron to keep it moving along. Once cron is configured you can close the progress window and cron will take over in the background. 

Note: VIVO importers other than "VIVO Publications" can be ignored. These get triggered automatically.

If you see a status message reporting "There are no new nodes" your class URI is likely incorrect. 

If you see an error message, your VIVO site URL might be incorrect or VIVO could be failing to produce the RDF list for the specified class. Some VIVO sites seem to have trouble when RDF lists containing a large number of individuals. You can test this by instead trying a class that contains a smaller number of individuals (hundreds instead of thousands). See the Troubleshooting section for more information.

Configuring cron
----------------

Most VIVO sites contain thousands of publications, and a complete publication import may take anywhere from a few days to over a week. Because PHP has limits on the execution time for a request, Drupal must import one small chunk of data at a time via cron runs.

Review the documentation on setting up cron for Drupal: https://drupal.org/cron

For VIVO Dashboard, the optimal cron frequency is 1 minute.

A typical crontab entry looks like this:

    */1 * * * * wget -O - -q -t 1 http://mysite.gotpantheon.com/cron.php?cron_key=rUncI0PLGq7_56SrZzNCrnerLoHZOHb7pCoFSFhBwdE

Note: The cron_key is unique for each site. You can find this URL on the Status Report (Reports -> Status report).

More cron
---------

Cron runs are used to keep Drupal working. The two main jobs that VIVO Dashboard performs during a cron run are:

1. Importing data from VIVO
2. Indexing imported content for search 

VIVO Dashboard comes with a module called Elysia Cron. This module acts as a manager responsible for delegating work. Elysia Cron knows how frequently jobs should be performed and will trigger them accordingly. It also collects statistics on job execution and can globally disable jobs.

Elysia Cron can be configured at: Configuration -> System -> Cron Settings (/admin/config/system/cron/settings)

CUSTOMIZATION
=============

Journal rankings
----------------

SCImago journal ranking data can be imported using the Journal Rankings importer (/import/journal_rankings), which expects a TSV file. Column 1 should contain the journal name, column 2 should contain the rank, and column 3 should contain the ISSN.

Once these are imported you'll need to do a bulk update to add ranking data to previously-imported journals (/admin/structure/taxonomy/journal_rankings/update), and run a search reindex (/admin/config/search/search_api/index/authorships and /admin/config/search/search_api/index/publications). Journals imported after ranking data is imported will automatically pick up ranking.

Institution-specific data
-------------------------

Chances are you will want to adjust the existing data mapping or perhaps add new data specific to your institution. All the mapping from VIVO to Drupal happens in the Feeds importer configurations under Structure -> Feeds importers, then under Processor -> Mapping.

- VIVO Publications: /admin/structure/feeds/vivo_publications/mapping
- VIVO Authors: /admin/structure/feeds/vivo_authors/mapping
- VIVO Journals: /admin/structure/feeds/vivo_journals/mapping

It should be fairly easy to add new data properties and map them to Drupal fields. Object properties (relationships), however, are a bit more complex and mapping for these has yet to be streamlined.

Hiding rdf:type items
---------------------

You will often get not-so-useful superclasses, such as "Thing" or "Agent", appearing in facets and lists. To hide those, edit the corresponding taxonomy term (/admin/structure/taxonomy/rdf_types) and click the "Hidden" option at the bottom. You will need to reindex before the change takes effect.

Including/excluding facet items
-------------------------------

When logged in you can edit configuration for facets by hovering over them and clicking the gear icon. Select "Configure facet filters", where you can enable either "Include" or "Exclude" filters..

TROUBLESHOOTING
===============

The first thing to try when troubleshooting Drupal is clearing caches. You can find "Flush all caches" under the Home icon in the toolbar. 

Database log
------------

Drupal maintains a comprehensive system log at: Reports -> Recent log messages (/admin/reports/dblog)

Feeds log
-------------

The Feeds module maintains its own log of events. Import-related troubleshooting should start here: Reports -> Feeds log (/admin/reports/feeds)

Per-importer logs can also be found on import pages. For example, the "VIVO Publications" import page (/import/vivo_publications) has a "Log" link at the top. 

Problems during a VIVO import, such as failed linked data requests, are usually logged with a severity of "warning".

Failed linked data requests
---------------------------

If you see warnings in the Feeds log indicating that certain VIVO URIs returned no data, the requests are likely timing out. This typically happens with VIVO individuals containing very large amounts of data. You can try increasing the timeout for the importer in the parser settings (e.g. /admin/structure/feeds/vivo_publications/settings/LdImportParser).

VIVO RDF list errors
--------------------

If you get error messages such as "500 Internal Server Error" after submitting the Import form, the VIVO site is likely failing to produce the RDF list for the class you specified. You can verify this by visiting the index page on the VIVO site, choosing the respective class, and trying the RDF link at the top.

One way to work around this is by manually creating a text file containing all the publication URIs to be imported, with one URI per line. This can be done using VIVO's internal SPARQL endpoint. You'll then need to change the Fetcher plugin for the importer (/admin/structure/feeds/vivo_publications/fetcher) to either "File upload" or "HTTP Fetcher". Then you can specify this text file on the import form (/import/vivo_publications). 

PERFORMANCE
===========

Drupal's great flexibility often comes at the expense of performance. As the amount of data increases, VIVO Dashboard will become slower. There are some things you can do to give Drupal a boost.

Entity Cache
------------

VIVO Dashboard comes with a module called Entity Cache that is intended to reduce the load on the database. This module is disabled by default only because it has not had enough testing. Enabling it is perfectly safe and should lead to noticeable performance gains. However, it should be suspect if pieces of data start disappearing.

Apache Solr
-----------

The primary module behind VIVO Dashboard's faceted search is Search API. Search API relies on a "backend" module and clever indexing to do its magic. Out of the box the database backend (search_api_db) is used for VIVO Dashboard in order to minimize hosting requirements. While this is the most convenient backend, it's not the most performant.

Using Apache Solr as the backend for Search API will yield substantially better performance. Although it hasn't been fully tested, it should simply be a matter of changing the Search API indexes to use the Solr backend already included with VIVO Dashboard (/admin/config/search/search_api). 

You also, of course, must set up the Apache Solr server itself. Once you do so, you'll need to copy configuration files from the search_api_solr Drupal module to Apache Solr's "conf" directory. Instructions and be found on the search_api_solr module page: https://drupal.org/project/search_api_solr
