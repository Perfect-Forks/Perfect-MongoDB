Perfect - MongoDB Connector
===========================

[![Perfect logo](http://www.perfect.org/images/perfect-git-banner.png)](http://perfect.org/get-involved.html)

[![Swift 3.0](https://img.shields.io/badge/Swift-3.0-orange.svg?style=flat)](https://developer.apple.com/swift/)
[![Platforms OS X | Linux](https://img.shields.io/badge/Platforms-OS%20X%20%7C%20Linux%20-lightgray.svg?style=flat)](https://developer.apple.com/swift/)
[![License Apache](https://img.shields.io/badge/License-Apache-lightgrey.svg?style=flat)](http://perfect.org/licensing.html)
[![Docs](https://img.shields.io/badge/docs-100%25-yellow.svg?style=flat)](http://www.perfect.org/docs/)
[![GitHub version](https://badge.fury.io/gh/PerfectlySoft%2FPerfect-MongoDB.svg)](https://badge.fury.io/gh/PerfectlySoft%2FPerfect-MongoDB)
[![Donate](https://img.shields.io/badge/Donate-PayPal-blue.svg?style=flat)](https://paypal.me/perfectlysoft)
[![Twitter](https://img.shields.io/badge/Twitter-@PerfectlySoft-brightgreen.svg?style=flat)](http://twitter.com/PerfectlySoft)
[![Join the chat at https://gitter.im/PerfectlySoft/Perfect](https://img.shields.io/badge/Gitter-Join%20Chat-brightgreen.svg)](https://gitter.im/PerfectlySoft/Perfect?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)



This project provides a Swift wrapper around the mongo-c client library,
enabling access to MongoDB servers.

This package builds with Swift Package Manager and is part of the
[Perfect](https://github.com/PerfectlySoft/Perfect) project. It was written to
be stand-alone and so does not require PerfectLib or any other components.

Ensure you have installed and activated the latest Swift 3.0 tool chain.




## Issues

We are transitioning to using JIRA for all bugs and support related issues, therefore the GitHub issues has been disabled.

If you find a mistake, bug, or any other helpful suggestion you'd like to make on the docs please head over to [http://jira.perfect.org:8080/servicedesk/customer/portal/1](http://jira.perfect.org:8080/servicedesk/customer/portal/1) and raise it.

A comprehensive list of open issues can be found at [http://jira.perfect.org:8080/projects/ISS/issues](http://jira.perfect.org:8080/projects/ISS/issues)

OS X Build Notes
----------------

This package requires the [Home Brew](http://brew.sh) build of mongo-c.

To install Home Brew:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To install mongo-c:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
brew install mongo-c
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Linux Build Notes
-----------------

Ensure that you have installed libmongoc.

Building
--------

Add this project as a dependency in your Package.swift file.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.Package(url:"https://github.com/PerfectlySoft/Perfect-MongoDB.git", versions: Version(0,0,0)..<Version(10,0,0))
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Quick Start
-----------

The following will clone an empty starter project:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add to the Package.swift file the dependency:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
let package = Package(
 name: "PerfectTemplate",
 targets: [],
 dependencies: [
     .Package(url:"https://github.com/PerfectlySoft/Perfect.git", versions: Version(0,0,0)..<Version(10,0,0)),
     .Package(url:"https://github.com/PerfectlySoft/Perfect-MongoDB.git", versions: Version(0,0,0)..<Version(10,0,0))
    ]
)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create the Xcode project:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
swift package generate-xcodeproj
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open the generated `PerfectTemplate.xcodeproj` file in Xcode.

The project will now build in Xcode and start a server on localhost port 8181.

>   **Important:** When a dependancy has been added to the project, the Swift
>   Package Manager must be invoked to generate a new Xcode project file. Be
>   aware that any customizations that have been made to this file will be lost.

Creating a MongoDB connection and query a collection
----------------------------------------------------

In Xcode, open Sources/PerfectTemplate/main.swift, and update the code to the
following:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
import PerfectLib

// Initialize base-level services
PerfectServer.initializeServices()

// Add routes
addURLRoutes()

do {
    // Launch the HTTP server on port 8181
    try HTTPServer(documentRoot: "./webroot").start(port: 8181)
} catch PerfectError.networkError(let err, let msg) {
    print("Network error thrown: \(err) \(msg)")
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create a new file at the same level as `main.swift`, called
`routingHandlers.swift`

Next

-   add the import directives for PerfectLib and MongoDB connector;

-   add some testing routes;

-   register the routes with Perfect.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
import PerfectLib
import MongoDB

func addURLRoutes() {
    Routing.Routes["/test" ] = testHandler
    Routing.Routes["/mongo" ] = mongoHandler
}

// Register any handlers or perform any one-time tasks.
public func PerfectServerModuleInit() {
    addURLRoutes()
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note that the two routes are handed off to functions.

The function handling the “/test” url simply return some “Hello, World!” JSON.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
func testHandler(request: WebRequest, _ response: WebResponse) {
    let returning = "{Hello, World!}"
    response.appendBody(string: returning)
    response.requestCompleted()
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

More information is available in the “URL Routing” example
(<https://github.com/PerfectlySoft/PerfectExample-URLRouting>)

A sample MongoDB handler function would be as follows:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
func mongoHandler(request: WebRequest, _ response: WebResponse) {

    // open a connection
    let client = try! MongoClient(uri: "mongodb://localhost")

    // set database, assuming "test" exists
    let db = client.getDatabase(name: "test")

    // define collection
    guard let collection = db.getCollection(name: "testcollection") else {
        return
    }

    // Here we clean up our connection, 
    // by backing out in reverse order created
    defer {
        collection.close()
        db.close()
        client.close()
    }

    // Perform a "find" on the perviously defined collection
    let fnd = collection.find(query: BSON())

    // Initialize empty array to receive formatted results
    var arr = [String]()

    // The "fnd" cursor is typed as MongoCursor, which is iterable
    for x in fnd! {
        arr.append(x.asString)
    }

    // return a formatted JSON array.
    let returning = "{\"data\":[\(arr.joined(separator: ","))]}"

    // Return the JSON string
    response.appendBody(string: returning)
    response.requestCompleted()
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For detailed information about the MongoDB connector check out the
Perfect-MongoDB project from GitHub and view the latest generated documentation
at `/Perfect-MongoDB/docs/index.html`

## Repository Layout

We have finished refactoring Perfect to support Swift Package Manager. The Perfect project has been split up into the following repositories:

* [Perfect](https://github.com/PerfectlySoft/Perfect) - This repository contains the core PerfectLib and will continue to be the main landing point for the project.
* [PerfectTemplate](https://github.com/PerfectlySoft/PerfectTemplate) - A simple starter project which compiles with SPM into a stand-alone executable HTTP server. This repository is ideal for starting on your own Perfect based project.
* [PerfectDocs](https://github.com/PerfectlySoft/PerfectDocs) - Contains all API reference related material.
* [PerfectExamples](https://github.com/PerfectlySoft/PerfectExamples) - All the Perfect example projects and documentation.
* [PerfectEverything](https://github.com/PerfectlySoft/PerfectEverything) - This umbrella repository allows one to pull in all the related Perfect modules in one go, including the servers, examples, database connectors and documentation. This is a great place to start for people wishing to get up to speed with Perfect.
* [PerfectServer](https://github.com/PerfectlySoft/PerfectServer) - Contains the PerfectServer variants, including the stand-alone HTTP and FastCGI servers. Those wishing to do a manual deployment should clone and build from this repository.
* [Perfect-Redis](https://github.com/PerfectlySoft/Perfect-Redis) - Redis database connector.
* [Perfect-SQLite](https://github.com/PerfectlySoft/Perfect-SQLite) - SQLite3 database connector.
* [Perfect-PostgreSQL](https://github.com/PerfectlySoft/Perfect-PostgreSQL) - PostgreSQL database connector.
* [Perfect-MySQL](https://github.com/PerfectlySoft/Perfect-MySQL) - MySQL database connector.
* [Perfect-MongoDB](https://github.com/PerfectlySoft/Perfect-MongoDB) - MongoDB database connector.
* [Perfect-FastCGI-Apache2.4](https://github.com/PerfectlySoft/Perfect-FastCGI-Apache2.4) - Apache 2.4 FastCGI module; required for the Perfect FastCGI server variant.

The database connectors are all stand-alone and can be used outside of the Perfect framework and server.

## Further Information
For more information on the Perfect project, please visit [perfect.org](http://perfect.org).
