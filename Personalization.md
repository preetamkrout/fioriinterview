# Personalization Service in SAP Fiori apps
Personalization is used for storing user-specific configuration and personalization data for particular app. e.g. column sequence in Table, filter queries for data selection or any kind of user-preference for the app.
The functionality is implemented as service in unified shell and has two platforms or ways to achieve it:
* Sandbox shell in local development environment (e.g. SAP WebIDE [Run As > Component])
* ABAP Shell in productive environment (e.g. ABAP Frontend server)

_NOTE:_ 
* sandbox shell use browses localStorage for storing the preference and ABAP shell uses tables (**/UI2/CONTAINER**, **/UI2/ITEM**)
* Expired Personalization data can be deleted or cleaned using TCODE: **/UI2/PERS_EXPIRED_DELETE**
* Personalization data can be cleaned using following IMG configuration
```
IMG > UI Technologies > SAP Fiori > Data Adapter > Create Cleanup
```

## Modes of Personalization implementation
* Direct [one entity, easy (one step), no p13n variants]
* Container [multiple entities, complex(seperate load & save), p13n variants available]

## Direct Mode
It is used for simple use-case and is easy to implement (e.g. Table P13N, custom user selection for one entity). In this mode p13n service creates a personalizer object and that objects hold the item/data in JSON Object format. This Personalizer object is created using factory method.
```
P13n Service (sap.ushell.Container.getService("Personalization")) <0..n>
    |=> Personalizer (using factory method) <1>
        |=> Item <1>
```

Personalizer object has following async methods that return _jQuery promise_ :
* getPersData - Used for reading the data from the storage
* setPersData - Used for storing the data to the storage
* delPersData - Used to delete the data from the storage

## Container Mode
It is used for complex use-case and is complex to implement as well. It can store data for multiple entities and data for different values for same entity (variants) or implement custom logic before save/load of the data.

Containers can contain items or mapped to a variant set adapter (contains variant sets). Following is the hierarchy of objects that are available to use:

```
P13n Service (sap.ushell.Container.getService("Personalization")) <0..n>
    |=> Container (PersoSrv.getContainer()) <1>
        |=>> Items (stores the data) <0..n>
        |=>> Variant Set Adapter (created using new) <1>
            |=>> Variant Set <0..n>
                |=>> Variant <0..n>
                    |=>> Item (stores the data) <0..n>
``` 

The start point for this approach is the **Container** Object.
_Good practice is to use one container for one app and names should be given accordingly. And name can't exceed **40 characters**_

## P13n API
### P13n Service:
* getPersonalizer - returns a personalizer used in direct mode
* getTransientPersonalizer - returns a transient personalizer used in container mode table p13n scenario
* getContainer - get a promise for container used in container mode
* createEmptyContainer - gets a promise for created empty container
* delContainer - delete a container, the local object and it's data stored on front-end server

### Container:
* load - loads the p13n data from storage (async)
* save - saves the p13n data to storage (also async)
* getItemKeys
* containsItem
* getItemValue
* setItemValue
* getValidity
* clear
* flush
* delItem

### Variant Set Adapter:
* getVariantSetKeys
* containsVariantSet
* getVariantSet
* addVariantSet
* delVariantSet

### Variant Set:
* setCurrentVariantKey - stores the currently selected variant by user
* getCurrentVariantKey - 
* getVariantKeys
* containsVariant
* getVariantKeyByName
* getVariant
* addVariant
* delVariant

### Variant
* getVariantKey
* getVariantName
* getItemKeys
* containsItem
* getItemValue
* setItemValue
* delItem


## Method Parameters for factory methods
[getPersonalizer, getContainer]
