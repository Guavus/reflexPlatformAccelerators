# Guavus CDAP Plugins


## Overview

Plugins are categorized as - source, transform, analytics, sink, and action or conditional. Plugins are available as pluggable modules that enable you to define a job pipeline on a UI canvas by stitching various stages of the pipeline using these pluggable modules. This job pipeline is then deployed and run as an application using CDAP deploy mechanism (through UI). Guavus provides custom plugins of above categories that are used as CDAP plugins to create a job, however you can also create your own custom plugins to add the capability as per your requirement.

## Prerequisites

In general, you should setup your plugin project as detailed [here](#creating-your-own-plugin-repo-from-scratch).
For this project, clone this repo :
```
git clone https://github.com/Guavus/cdap-plugins.git
```
To use plugins, you must have CDAP version 4.3.3 or later.


## Build Plugins


* Build plugin jar and json

RUN from this directory:
```
    make all
```
**NOTE** : Running above script will first check and locally install any required internal artifact before building the plugins.

The build will create a .jar and .json file under the `target` directory.
These files can be used to deploy your plugins.

* Create plugin rpm

RUN from this directory:
```
    make publish-all
```
**NOTE** : Running above script will create rpm for each plugin in `dist/cdap-plugins/` directory. On installing these generated rpm on any machine,the jar and json would be present at path `/opt/guavus/cdap-plugins`. If you want these generated rpms to be pushed to artifactory , set PUSH_TO_ARTIFACTORY=1 in `artifactory_mgmt.sh` file before running `make all` and also make sure there is keyless access between artifactory server and build server.

## Deploy Plugins

**Method 1 : CLI [Preferred option to simulate production deployments]**

You can deploy your plugins using the CDAP CLI:
```
    load artifact <target/<component>-plugins-<version>.jar config-file <target<component>-plugins-<version>.json>
```
For example, if your artifact is named 'ibpublisher-plugins-<version>':
```
    load artifact target/ibpublisher-plugins-5.3.0-1.0.0.jar config-file target/ibpublisher-plugin-5.3.0-1.0.0.json 
```

**Method 2 : HTTP REST**

You can also deploy your plugins using the CDAP HTTP Rest API as documented in CDAP's online documentation here : 
`https://docs.cask.co/cdap/4.3.3/en/reference-manual/http-restful-api/artifact.html`


**METHOD 3 : CDAP UI**

CDAP UI is the most convenient interface to upload above jar and json to the running CDAP instance and the UI interface is
self-intutive to do the upload by navigating to `localhost:11011` or `<cdap-master-ip>:11011`.


## Creating your own Plugin Repo from Scratch

If you want to create your own repository for building custom plugins, all you need is a basic pom.xml to start with and
the directory structure for creating plugins.
To understand the basics, one may also refer to standard [CDAP
link](https://docs.cask.co/cdap/4.3.3/en/developer-manual/pipelines/developing-plugins/index.html) for this.

In summary, you may either use pom.xml from this project and follow the CDAP defined directory structure or 
follow below steps :

1. Use the Maven artifcat `cdap-data-pipeline-plugins-archetype` to create your project :
```
mvn archetype:generate -DarchetypeGroupId=co.cask.cdap -DarchetypeArtifactId=cdap-data-pipeline-plugins-archetype
-DarchetypeVersion=4.3.3 -DgroupId=org.example.plugin
```
For example, for this project, we used following :
```
mvn archetype:generate -DarchetypeGroupId=co.cask.cdap -DarchetypeArtifactId=cdap-data-pipeline-plugins-archetype
-DarchetypeVersion=4.3.3 -DgroupId=com.guavus.cdap.plugins.transform
```

It will prompt you for your artifactId and version. In this project, we specified it as :
```
artifactId = guavus-plugins
version = 5.3.0
```

2. This will create your project by name `artifactId` given above and create the directory structure as per provided
   groupId.

3. It will also generate default example files in `src/main`, `src/test/`, `docs` and `widgets` directories which you
   may clean.

4. Write your plugin code in `src/main`, `src/test/`, `docs` and `widgets` as also provided in this project. You may
   further define more directories as per your requirement like `icons` for providing plugin logos and `libs` to carry
   the external dependency jars (if not available thorugh maven) to be maintained with your project.

5. You might need to update `pom.xml` to specify more dependencies, plugins, goals, etc as per your project need.

6. Finally, build your plugins:
```
    mvn clean package -DskipTests
```
The build will create a .jar and .json file under the ``target`` directory.
These files can be used to deploy your plugins.


## UI Integration

The CDAP UI displays each plugin property as a simple textbox. To customize how the plugin properties
are displayed in the UI, you can place a configuration file in the ``widgets`` directory.
The file must be named following a convention of ``[plugin-name]-[plugin-type].json``.

See [Plugin Widget Configuration](http://docs.cask.co/cdap/current/en/hydrator-manual/developing-plugins/packaging-plugins.html#plugin-widget-json)
for details on the configuration file.

The UI will also display a reference doc for your plugin if you place a file in the ``docs`` directory
that follows the convention of ``[plugin-name]-[plugin-type].md``.

When the build runs, it will scan the ``widgets`` and ``docs`` directories in order to build an appropriately
formatted .json file under the ``target`` directory. This file is deployed along with your .jar file to add your
plugins to CDAP.
