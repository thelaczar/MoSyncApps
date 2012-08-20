<!-- # Using Web SQL in MoSync Apps  -->
<!-- C:\md>perl Markdown.pl C:\MoSyncProjects\MoSyncApps\WebSQLDemo\WebSQLTutorial.md > output.txt -->

<!--
<style type="text/css">
p>img {
  width: 550px;
}
</style>
-->

In this tutorial we will take a look at the Web SQL Databse API, a standard way of accessing SQL databases from JavaScript. We introduce the API and show how to use it in a MoSync application. In addition, we discuss other available options for accessing databases from a JavaScript-based MoSync app.

The source code for this tutorial is available on [GitHub](https://github.com/divineprog/MoSyncApps/tree/master/WebSQLDemo).

## Background

[Web SQL Database](http://www.w3.org/TR/webdatabase/) is a database API proposed by the W3C Working Group. However, the standard is not actively maintained at present, because of lack of independent implementations (multiple implementations are required for W3C to proceed with the standards process). 

Currently, Web SQL is implemented in WebKit, which means that it is available on iOS and Android (and in Chrome on the destktop).

There exists competing standards for data storage in browsers, as outlined in the article [HTML5 Storage Wars](http://csimms.botonomy.com/2011/05/html5-storage-wars-localstorage-vs-indexeddb-vs-web-sql.html). In the favour of Web SQL is that it is considered reliable and mature, and it is based on SQLite, a proven database engine that has become popular on mobile devices.

## MoSync support for Web SQL

## A tour of the Web SQL API

Use the global function openDatabase to open a [Database](http://www.mosync.com/files/imports/doxygen/latest/html5/database.md.html#Database):

    var db = openDatabase("MyDB", "1", "MyDB", 65536);

Parameters are:

* database name
* database version
* database display name
* database size

Use the transaction function in the database object to excute queries, and execute actual queries using the executeSql function in the [SQLTransaction](http://www.mosync.com/files/imports/doxygen/latest/html5/sqltransaction.md.html#SQLTransaction) object:

    db.transaction(
        function(tx)
        {
            tx.executeSql(
                "INSERT INTO pet VALUES (?, ?, ?)", 
                ["Charmy", 7, 0.6],
                function(tx, result) {
                    console.log("Query Success") },
                function(tx, error) {
                    console.log("Query Error: " + error.message) }
            );
        },
        function(error) {
            // Note that this error function does not get 
            // called if the above query should fail, since
            // we supply an error function to executeSql.
            console.log("Transaction Error: " + error.message) },
        function() {
            console.log("Transaction Success") }
    );

The executeSql function takes up to four parameters:

* query string (mandatory)
* query parameter array (optional)
* query success function (optional)
* query error function (optional)

One nice thing with the Web SQL API is that it encourages the use of query parameters. It is very easy to supply parameters in the query parameter array.

The transaction function takes up to three parameters:

* transaction function, execute queries in this function (mandatory)
* transaction error function (optional)
* transaction success function (optional)

Note the order of the success and error functions, which is the reverse for the transcation function, compared to the executeSql function. Errors are reported using the [SQLError](http://www.mosync.com/files/imports/doxygen/latest/html5/sqlerror.md.html#SQLError) object.

The transaction success and transaction error functions are useful for handling of the entire transaction.

Note that the transaction error function will be called only if you do not supply a query error function to executeSql. If you provide a query error function that will be called, and the transcation success function will be called, rather than the transaction error function. Thus, in the above example, the transaction error function will not be called if the query fails.

Also note that queries are executed asynchronously. When the transaction success function is called, you should be guaranteeded that all queries have been executed.

When doing a search query, the result is obtained as a set of rows, which involves using the [SQLResultSet](http://www.mosync.com/files/imports/doxygen/latest/html5/sqlresultset.md.html#SQLResultSet) and the [SQLResultSetList](http://www.mosync.com/files/imports/doxygen/latest/html5/sqlresultsetlist.md.html#SQLResultSetList) objects:

    tx.executeSql("SELECT * FROM pet", [],
        function(tx, result)
        {
            console.log("All rows:");
            for (var i = 0; i < result.rows.length; ++i)
            {
                var row = result.rows.item(i);
                console.log("  " + row.name + " " + row.age + " " + row.curiosity);
            }
        });

## Basic examples

File [example1.js](https://github.com/divineprog/MoSyncApps/blob/master/WebSQLDemo/LocalFiles/example1.js) provides a basic example of how to use the Web SQL API. This example illustrates the techniques shown above.

File [example2.js](https://github.com/divineprog/MoSyncApps/blob/master/WebSQLDemo/LocalFiles/example2.js) shows how to use a lightweight wrapper library, found in file [database.js](https://github.com/divineprog/MoSyncApps/blob/master/WebSQLDemo/LocalFiles/database.js), which simplifies the Web SQL API a bit. In this library, each query is run in its own transaction, so the handling of the transaction object is abstracted away. The drawback is that you then cannot use the transaction success function to determine when a sequence of queries are complete.

Note that in example1.js, the transaction success function runs the code in example2.js. This makes the two examples run sequentially.

You can run both examples, e.g. in Chrome, by opening the file [example.html](https://github.com/divineprog/MoSyncApps/blob/master/WebSQLDemo/LocalFiles/example.html). Inspect the console log in Chrome to see the output.

One this that is of interest is how retrieving the result of SELECT COUNT(*) works in Web SQL. The restult is retured as a row, and the name of the fioeld that contains the row count is called "COUNT(*)". See the example code for how this is used.

## Demo application

The app WebSQLDemo is a simple turn-based game of luck (much like playing by tossing a dice). The "dice" in the app is a "wheel" with numbers 1 tho 50. Whoever gets the highest number wins the current round, and the total score gets updated. 

You challenge the app and can affect the outcome by taking a low risk or a high risk. With high risk, you can double your score, but the app gets to roll the wheel twice.

This is a screenshoot of the app:

![Web SQL Demo Screenshot](https://raw.github.com/divineprog/MoSyncApps/master/WebSQLDemo/Tutorial/WebSQLDemo.png)

The source code is available on [GitHub](https://github.com/divineprog/MoSyncApps/tree/master/WebSQLDemo). The main entry point of the program is file [index.html](https://github.com/divineprog/MoSyncApps/blob/master/WebSQLDemo/LocalFiles/index.html). The app uses the database.js wrapper library discussed above.

The database is used to store the total score of the two players (the user and the app). This makes the score persistent. While not a big database with lots of objects, it shows how to perform basic databse operations, like querying and updating rows.

## Alternative techniques


