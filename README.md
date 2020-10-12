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

## Step 3
Create the file db/data-model.cds and add the following data definition.


``namespace my.bookshop;
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
}``

## Step 4





