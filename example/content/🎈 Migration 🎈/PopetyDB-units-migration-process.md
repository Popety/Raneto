/*
Title: PopetyDB Unit Migration Process
Sort: 2
*/

> The aim of this document is to explain the units migration process from
PopetyDB to popetyProd.

List of unit data that may be updated on Popety Prod during the migration:
* unit number
* level
* unit
* block
* area sqft
* bedrooms
* bathrooms
* profile progress

> Something wrong or missing ? Please let us know

## Migration Cycle

<img src="%image_url%/Units-migration.png" style="text-align:left; display:inline"/>

### 1 - Enter Data

 | | |
-|-|-|
**Who** | **Fountain member**
**Input** | Beginning of the cycle
**Output** | Email indicating the completion of data and summing up the properties updated

1. Before entering data you need to start a new cycle.

 <a href="http://popetydb.popety.com/#/migration" target="_blank">Go to popetydb app -> migration and click on the Start new cycle button;</a>

 <img src="%image_url%/start_new_cycle.png" width="500" style="text-align:left; display:inline"/>

1. As soon you finish to complete or update a property click on completed checkbox

 <img src="%image_url%/complete_property.png" width="500" style="text-align:left; display:inline"/>

1. Then you can check that the property is automatically include in the current migration cycle on <a href="http://popetydb.popety.com/#/migration" target="_blank">popetydb app -> migration</a> -> migration number

 <img src="%image_url%/afftected_properties.png" width="500" style="text-align:left; display:inline"/>

### 2 - Validate Data

| | |
-|-|-|
**Who** | **Popety member**
**Input** | Receiving Enter Data completion email
**Output** | Email validating of data

1. Go to <a href="http://popetydb.popety.com/#/migration" target="_blank">popetydb app -> migration</a> -> migration number to see properties affected by the migration and perform a checking

 <img src="%image_url%/afftected_properties.png" width="500" style="text-align:left; display:inline"/>

### 3 - Migrate Data

| | |
-|-|-|
**Who** | **Popety member**
**Input** | Go from management
**Output** | Email summing up the migration

> These steps using command console will be replace soon by a software Factory job

#### Requirement
* Git Install
* NodeJS Install
* Github account


#### Steps

1. Start Popety VPN

1. Go to a directory of your choice and get the migration code source
 ```
 git clone https://github.com/Popety/popety-v2
 ```

1. Enter your Github credentials

1. Run the following commands (Set the expected MIGRATION_NUMBER)
 ```
 cd popety-v2
 git checkout fix/migration-update
 ```
1. Then Run
 ```
 node migration/unitMigrationWithCollectionStatus.js collection_status=MIGRATION_NUMBER
 ```

1. When the migration is finish you must see a message like this
 ```
 Migration finished. Total units migrated: 12751 in 269567 ms
 ```

1. Please go to the /migration folder and get the log file to attach it to the Migration confirmation Email

1. Check the migration units not found errors on <a href="http://popetydb.popety.com/#/migration" target="_blank">popetydb app -> migration</a> -> migration number

 <img src="%image_url%/unit-not-found.png" width="500" style="text-align:left; display:inline"/>
