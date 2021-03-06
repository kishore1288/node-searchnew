﻿# NodeSearch Docs
A Node.js implementation of CloudService's SME Search

```js
    // Create our client socket
    var socket = io.connect('http://' + window.location.hostname + ':' + window.location.port);

    // Search the root folder for any files with an extension of PDF
    var searchFilter = {
        FolderId: 0,
        Token: 'a1b2c3d4e5f6',
        Extensions: [
            'pdf'
        ],
        Type: 'x'
    };

    // Perform our search passing the searchFilter object we just created
    socket.emit('search', searchFilter);

    // If we receive a result, log it to the console
    socket.on('search-result', function(resultData){
        console.log(JSON.stringify(resultData));
    });

    // If we didn't receive any results log the message to the console
    socket.on('search-noresult', function(resultData){
        console.log(JSON.stringify(resultData.statusMessage));
    });

    // If the search failed alert the user and log it to the console
    socket.on('search-failure', function(resultData){
        alert('Search Failed');
        console.log(JSON.stringify(resultData));
    });
```

## Description
NodeSearch searches through files within a SME(StorageMadeEasy) Repository and returns a JSON result set based on a set of filters set by the user.

#### Dependencies
NodeSearch uses the Socket.io framework for Node.js, all clients must implement a Socket.io cilent in order to interact with and perform searches in NodeSearch.

##### Socket.io CDN Link for Script Reference 
https://cdn.socket.io/socket.io-1.3.5.js

---

## Required Search Parameters
When performing a search, there are 3 required parameters. In addition to the explicitly required parameters, a filter must also be used. If a filter is not passed 
to the search a 'search-failure' event will be emitted back over the socket to the client.

* **FolderId** - This is the base folder that the search will be performed in. 
* **Token/Username** - In order to perform a search a valid SME Token must be used. NodeSearch can either generate one for a valid user passed through or it can accept a Token to use for 
the search.
* **Type of Search** (Default Type: e)
* **1 or more Filters** (Filename, File Description, File Extension(s), Tag(s), From/To Modification Dates, Metadata)

### Basic Search Examples

#### 
```js    
    // This example passes a Token
    // Search in the Root folder for any files that have an extension of 'pdf'
    var searchFilterWithToken = {
        FolderId: 0,
        Token: 'a1b2c3d4e5f6',
        Extensions: [
            'pdf'
        ],
        Type: 'x'
    };

    // This example passes a Username
    // Search in the Root folder for any files that have an extension of 'pdf'
    var searchFilterWithUsername = {
        FolderId: 0,
        Username: 'demo'
        Extensions: [
            'pdf'
        ],
        Type: 'x'
    };
```    

 ---
## Filters
When searching for files, NodeSearch will apply filters to pass to SME depending on which files you would like returned.
All Filters are optional, however there must be at least 1 filter used in the search request in order for the search to occur.

### Available Filters

| Search Filter | Param Name | Type | Comments |
| ------------- | ---------- | ---- | --------
| File Name | Filename| string | |
| File Description | Description | string | | 
| File Extension | Extensions | string array | |
| File Tags | Tags | string array | |
| From Modification Date | FromDate | date string | Can only be used during an Advanced Search (Type: 'v') |
| To Modification Date | ToDate | date string | Can only be used during an Advanced Search (Type: 'v') |
| Metadata | Metadata | Metadata Filter Object (see below) |  |

### Metadata Filter Object

Metadata Filter Objects have 4 required properties

#### 1. Id

The ID of the Metadata field to search within SME.

#### 2. TextClause

The TextClause is used to determine how the value of the Criteria property is searched for.

TextClause can be equal to 1 of 2 possible values:

1. Equals (0), The Text being searched against must equal the value within the Criteria property.
2. Contains (1), The Text being searched against must contain the value within the Criteria property.

#### 3. Criteria

The Value that we are actually searching for within SME.

#### 4. SearchClause

```js    
    var mdFilter1 = {
        Id: 1,
        TextClause: 1,
        Criteria: 'abc',
        SearchClause: 1
    };
    
    var mdFilter2 = {
        Id: 1,
        TextClause: 1,
        Criteria: 'def',
        SearchClause: 0
    };

    // Metadata Filter Objects can be construced and then added to the Metadata Filter Array
    var searchFilter = {
        FolderId: 0,
        Token: 'a1b2c3d4e5f6',
        Type: 'v',
        Metadata: [
            mdFilter1,
            mdFilter2
        ]
    };

    // Metadata Filter Objects can also be constructed inline within the Metadata Filter Array
    var searchFilter = {
        FolderId: 0,
        Token: 'a1b2c3d4e5f6',
        Type: 'v',
        Metadata: [
            {
                Id: 1,
                TextClause: 1,
                Criteria: 'abc',
                SearchClause: 1
            },
            {
                Id: 1,
                TextClause: 1,
                Criteria: 'def',
                SearchClause: 0
            }
        ]
    };
```

---
## Search Types

The Type of Search performed is specified by the 'Type' parameter 


| Search Type Description | 'Type' Param Value | Filter to Use |
| ----------------------- | ------------------ | ------------- |
| Advanced Search | 'v' | Filename |
| All Words | 'a' | Filename |
| Any Words | 'o' | Filename |
| Annotation Search | 'h' | Filename |
| File Name Starts With | 's' | Filename |
| File Name Ends With | 'd' | Filename |
| File Name Matches Exact Phrase | 'e' | Filename |
| File Extension | 'x' | Extensions |
| File Content | 'c' | Filename | 
| Filename & Content | 'n' | Filename |


### TextOption
The TextOption Parameter is used in conjunction with the Search Type parameter when performing either an Advanced Search (Type: 'v') or an Annotation Search (Type: 'h'). 
Using TextOption extends the capability of a search against the Filename or Content. 

#### TextOption Example 
```js
    // The TextOption Parameter is an array of strings
    TextOption: [
        'bw',
        'wx'
    ]

    // Example of TextOption being used in a search 
    var searchFilter = {
        FolderId: 0,
        Token: 'a1b2c3d4e5f6',
        Filename: 'NodeSearch Example'
        Type: 'v',
        TextOption: [
            'bw',
            'wx'
        ]
    }

```
| TextOption Description | 'TextOption' Param Value |
| ---------------------- | ------------------------ |
| Match Exact Word or Phrase | 'x' |
| Match Whole Word | 'wh' |
| Begins With Word | 'bw' | 
| Ends With Word | 'ew' |
| Use Wildcards for Words | 'wx' |


## Get Folders
The 'get-folders' is an entirely separate function supported outside of searching. The purpose of this function is to obtain the folders that are available to a particular user.
With this list the user then may select which folder they would like to search within. 'get-folders' always retrieves all subfolders including nested sub-folders and will build 
a JSON Object Tree. 

*Note: 'get-folders' hasn't been fully implemented and will only return the root folder until further development occurs.

```js
    // Establish our connection to NodeSearch
    var socket = io.connect('http://node.search:port'');
    
    // Build our get-folders Parameter JSON  
    var folderRequestData = {
        Token: 'a1b2c3d4e5f6',
        FolderId: 0
    };
    
    socket.emit('get-folders', folderRequestData);
    
    // If we get results back print them to the console in a readable format 
    socket.on('get-folders-result', function(data){
        console.log(JSON.stringify(data, null, 2));
    });
    
    // If we get an error log it to the console and display an alert of the error
    socket.on('get-folders-failure', function(data){
        console.log(JSON.stringify(data, null, 2));
        alert(data.error);        
    });
```

### Required Parameters
Making calls to 'get-folders' are similar to those of 'search'. There are only 2 parameters and both of those parameters are required. 
Failure to provide both parameters will result in a 'get-folders-failure' with a single property of 'error'.

* **Token/Username** - Either a valid SME API Token or LawStudio/SME Username
* **FolderId** - This is the ID of the Folder to get all Sub-Folders from.

#### Sample 'get-folders' JSON Request Object
```js
    // Example using Username parameter
    // Get the Folders within Root (FolderId = 0) for SomeUser 
    var folderRequestData = {
        Username: 'someUser',
        FolderId: 0
    };        
    
    // Example using Token parameter
    // Get the Folders within Root (FolderId = 0) and with Token 'a1b2c3d4e5f6'  
    var folderRequestData = {
        Token: 'a1b2c3d4e5f6'
        FolderId: 0
    };
``` 
### 'get-folders-result' Event
This event is emitted by NodeSearch after the final JSON Object Tree has been built.
The data returned by the 'get-folders-result' is an array of Folder Objects. 

#### Folder Object
```js
    // Folder Object
    {
        FolderId: Number,
        ParentFolderId: Number,
        FolderName: String,
        Children: [ Folder Object ]
    }
```