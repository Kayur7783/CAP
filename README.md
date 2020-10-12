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


```namespace my.bookshop;
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
  entity Authors @readonly as projection on my.Authors;
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



