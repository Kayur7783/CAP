# Download the project from git

`git clone https://github.com/Kayur7783/bookshop.git`
`cd bookshop`
`cds watch`

or you can follow the following steps to create this sample project.

## Step 1

Execute `cds init bookshop` command in the terminal.

## Step 2

Start a cds live re-load server using `cds watch`. 
At this point we do not have any service definitions hence the console will be logged as follows.
   ``` 
   No models found at db/,srv/,app/,schema,services.
   Waiting for some to arrive...
```

## Step 3
Create the file db/data-model.cds and add the following data definition.


```
namespace my.bookshop;
using { Country, managed } from '@sap/cds/common';

entity Books {
  key ID : Integer;
  title  : localized String;
  author : Association to Authors;
  stock  : Integer;
}

entity Authors {
  key ID : Integer;
  name   : String;
  books  : Association to many Books on books.author = $self;
}

entity Orders : managed {
  key ID  : UUID;
  book    : Association to Books;
  country : Country;
  amount  : Integer;
}
```

## Step 4
Add the service definition file catalog-service.cds in srv folder.
Add the following code to the file. 
```
using my.bookshop as my from '../db/data-model';

service CatalogService {
  entity Books @readonly as projection on my.Books;
  entity Authors as projection on my.Authors;
  entity Orders @insertonly as projection on my.Orders;
}
```

As soon as we define the service definition cds file the console log changes to as follows. 
```
[cds] - model loaded from 3 file(s):

  db/data-model.cds
  srv/catalogService.cds
  ../../.node_modules_global/lib/node_modules/@sap/cds-dk/node_modules/@sap/cds/common.cds

[cds] - using bindings from: { registry: '~/.cds-services.json' }
[cds] - connect to db > sqlite { database: ':memory:' }
/> successfully deployed to sqlite in-memory db

[cds] - connect to messaging > local-messaging {}
[cds] - serving CatalogService { at: '/catalog', impl: 'srv/catalogService.js' }

[cds] - launched in: 1490.002ms
[cds] - server listening on { url: 'http://localhost:4004' }
[ terminate with ^C ]
```
Now, we have a boilerplate service running. 
We can open the same and test in the browser. 
We will get a pop-up saying 
```
A service is listening to port 4004. Click "Expose and Open" to access the service externally, and preview it in a new tab.
```
Click on Expose and open to test the service. 

## Step 5
Adding mock data for testing.
In the db folder, create a new `csv` folder. 
Create following files in the folder. 
* my.bookshop-Authors.csv
* my.bookshop-Books.csv

The naming convention for the files is `<namespace>-<entity name>.csv`.
Add the following data in my.bookshop-Authors.csv
```
ID;name
101;Emily Brontë
107;Charlote Brontë
150;Edgar Allen Poe
170;Richard Carpenter
```

Add the following data in my.bookshop-Books.csv
```
ID;title;author_ID;stock
201;Wuthering Heights;101;12
207;Jane Eyre;107;11
251;The Raven;150;333
252;Eleonora;150;555
271;Catweazle;170;22
```
With cds watch running, you will notice that the data gets loaded in the in memory sqlite database immediately. 
Now, when we try and get data for an entity from the browser, we get data that we have loaded just now. 

## Step 6
Add sqlite persistence database.
Execute the following command. 
```
cds deploy --to sqlite:bookshop.db
```

This would create a sqlite file bookshop.db at the root of the project. 
Lets open the file in the database explorer and check what is happening behind the scenes. 
* Navigate to the explorer.
* Select sqlite as database option. 
* Add following file path in the input field for database file. `/home/user/projects/bookshop/bookshop.db`
* Click Save Connection

Your sqlite database created by cds deploy command is now available for examination and learning. 

## Step 7
Define nodejs service handler. 
In the srv folder. Create the file catalog-service.js
Note - if the js handler file name is same as cds service definition file, the cds runtime will automatically detect it as handler file. 

Paste the following code in the catalog-service.js
```
  module.exports = (srv) => {

  const {Books} = cds.entities ('my.bookshop')

  // Reduce stock of ordered books
  srv.before ('CREATE', 'Orders', async (req) => {
    const order = req.data
    if (!order.amount || order.amount <= 0)  return req.error (400, 'Order at least 1 book')
    const tx = cds.transaction(req)
    const affectedRows = await tx.run (
      UPDATE (Books)
        .set   ({ stock: {'-=': order.amount}})
        .where ({ stock: {'>=': order.amount},/*and*/ ID: order.book_ID})
    )
    if (affectedRows === 0)  req.error (409, "Sold out, sorry")
  })

  // Add some discount for overstocked books
  srv.after ('READ', 'Books', each => {
    if (each.stock > 111)  each.title += ' -- 11% discount!'
  })

}
```
Lets check the output of the `cds watch` command and changes to our service output. 

## Step 8
Testing and debugging. 
Stop the cds watch command by using Ctrl+C key press. 
Open the find file option using command+P key press.
Search for file launch.json.
Provide the following launch configuration. 
```
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run bookshop",
      "request": "launch",
      "type": "node",
      "cwd": "/home/user/projects/bookshop",
      "runtimeExecutable": "npx",
      "runtimeArgs": [
        "-n"
      ],
      "args": [
        "--",
        "cds",
        "run",
        "--in-memory?"
      ],
      "console": "internalConsole",
      "internalConsoleOptions": "openOnSessionStart",
      "skipFiles": [
        "<node_internals>/**"
      ],
      "env": {
        "run.config": "{\"handlerId\":\"cap_run_config_handler_id\",\"runnableId\":\"/home/user/projects/bookshop\"}"
      }
    }
  ]
}
```
The business application studio would open a debugger console, host the application like we have in cds watch command. 
Navigate to the hosted application in the new tab. 
Go back to business application studio and place a breakpoint on the line 20, i.e., `if (each.stock > 111)  each.title += ' -- 11% discount!'`
Switch back to application tab and click on Books. This would initiate Read for books and the call would stop at our breakpoint. 
Likewise we can debug all our custom handlers. 
Also note we can create API test calls in business application studio. 

Create the file tests/bookshop.http and provide the following code. 
```
### Service Document
GET http://localhost:4004/

### Service $metadata document
GET http://localhost:4004/catalog/$metadata

### Browsing Books
GET http://localhost:4004/catalog/Books?&$select=title,author


### Browsing Authors
GET http://localhost:4004/catalog/Authors?&$select=name&$expand=books($select=title)&$filter=ID eq 180


### Create Author
POST http://localhost:4004/catalog/Authors
Content-Type: application/json;IEEE754Compatible=true

{
    "ID": 180,       
    "name": "JK Rowling"
}   


```
Thats how we can test and debug our applications. 

Thank you!
