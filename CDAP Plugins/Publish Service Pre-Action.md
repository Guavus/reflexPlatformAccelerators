# Publish Service Pre-Action

## Description

Publish Service Pre-Action plugin fetches the hdfs path of the published IB. It provides support only for batch 
pipeline. It can be used to fetch either the latest version of the IB or any specific version that user specifies.
The fetched IB path is then stored as a plugin context global argument so that subsequent plugins in the cdap pipeline
can read from this variable.


## Pre-requisite

The plugin expects the IB service to be up and running.


## Use Case

This plugin can be used before any classifier plugin that needs the IB path of the of the trained model. 

## Properties

| Configuration | Label | Required? | Default | Description |
| :------------ | :---- | :-------- | :------ | :---------- |
| `ibServer` | IB Server | Mandatory | `N/A` | Guavus IB Management Service URL | 
| `ibName` | IB Name | Mandatory | `N/A` | Name of the published IB whose hdfs path needs to be fetched.| 
| `ibMajorVersion` | IB Major Version | Optional | `1` |The Major Version of the IB to be fetched| 
| `outputVariableName` | Global Variable Key | Optional | `ibpath` |  Key to be set as global variable.If nothing is specified, ibpath will be set as key | 


## Example

This example fetches the IB path and set the result in context global variable

    {
        "name": "Publish Service PreAction",
        "plugin": {
            "name": "PublishServicePreAction",
            "type": "Action",
            "properties": {
                "ibName": "xyz",
                "ibServer": "http://192.168.110.10:9170",
                "ibMajorVersion": "1523340062",
                "outputVariableName": "ibpath"
                
            }
        }
    }
