Rancher API Store
===============

Storage adapter to [compatible APIs](http://github.com/rancher/api-spec).

## Installation


```bash
  npm install rancher-api-store
```

## Usage

### Store

The store performs all communication with the API service and maintains a single copy of all the resources that come back from it.  This ensures that changes to a resource in one place propagate propertly to other parts of your application that use the same resource.

A property named `store` is automatically injected into all routes, controllers, and models on initialization.

**Methods:**

* `find(type [,id] [,options])`: Query API for records of `type`, optionally with `id` and other `options` like `filter` and `include`.  Returns a promise.

* `getById(type, id)`: Get a record from the local cache by `type` and `id`.  Returns a resource or undefined synchronously.

* `hasRecordFor(type, id)`: Returns true if a record for `type` and `id` exists in cache synchronously.

* `all(type)`: Returns a "live" array of all the records for [type] in the store.  The array will be updated as records are added and removed from the store.

* `findAll(type)`: Calls `find(type)` if it hasn't been called before, then returns `all(type)` to give you back a live list of all the records in one call.  Convenient for a model hook.

* `createRecord(data)`: Create a record given fields `data`.  Returns a `Resource`.  Does **not** add the record to the store, call `resource.save()` on the response or `\_add()` on the store.

**More methods, that you shouldn't need often:**

* `_add(type, obj)`: Add a record to the store.  This is normally done automatically when reading objects, but you might have created one with `createRecord` manually want it added without `resource.save()`.

* `_remove(type, obj)`: Remove a record from the store.  This doesn't tell the server about it, so you probably want `resource.delete()`.

* `_bulkAdd(type, array)`: add a lot of instances of the same type at once.

  - There must be a model for the type already defined.
  - Instances cannot contain any nested other types (e.g. include or subtypes),
  - (they will not be deserialzed into their correct type.)
  - wasAdded hooks are not called
  - Basically this is just for loading schemas faster.

**Properties:**

* `removeAfterDelete: true`: Set to false to disable automatically removing from the store after `record.delete()`.  You might want this if your API has a 2-step deleted vs purged state.

### Resource
A resource is a model object representing a single resource in the API.

**Methods:**

* `.merge(data)`: Take the values in `data` and replace the corresponding values in this resource with them.  Returns the resource so you can chain calls.
* `.replaceWith(data)`: Replace all the values in this resource with the ones in `newData`.  Returns the resource so you can chain calls.
* `.clone()`: Returns a duplicate of this resource.  Changes to the clone will not initially affect the original.  If `.save()` is called on the clone, the response data will be merged into both the clone and original.
* `.hasLink(name)`: Returns a boolean for whether this resource has a link with the given `name` or not.
* `.followLink(name [,options])`: Retrieves the link with the given `name` and returns a promise that resolves with the response data.
* `.importLink(name [,options])`: Retrieves the link with the given `name` and assigns the response data as a property with the same `name`  (or you can use opttion.as to change the name) on the resource.  Returns a promise that resolves with the resource.

* `.hasAction(name)`: Returns a boolean for whether this resource has an action with the given `name` or not.
* `.doAction(name [,data])`: Performs the action given by `name`, optionally sending `data` and returns a promise that resolves with the response data.
* `.save()`: Sends the resource to the API to persist it.  On success, adds the resource to the store if it wasn't already in there.  Returns a promise that resolves to the resource, and also updates the store record with the response data if it is provided.
* `.delete()`: Sends a delete request to the API to remove a resource.  On success, the resource is removed from the store if it was in it.
* `.serialize()`: Returns a plain JavaScript object representation of the resource.
* `optionsFor(field)`: Returns an array with options of the `field`, only for enum fields.
* `isRequired(field)`: Returns a boolean, which indecates whether the `field` is required.
* `getDefault(field)`: Returns the field's default value.

**Static Properties:**

* `alwaysInclude: []`: An array of fields to always request be included when making requests for this tyep of resource.


### Collection
A collection is a model object representing an array of resources in the API.  It is an ArrayProxy that proxies array requests to the `data` elements of the collection, but collections are themselves resources that may have links and actions themselves that you can use (as a resource, above).

**methods**

* `length ()` Returns the length of the collection.

* `getById(id)` Returns the resource with the passed id or undefined if not found.

* `getAt(index)` Returns the resource at provided index.

* `call(nameOfmethod, [...args])`. Call any array method on the collection. such as `collection.call('map', function(item, idx) {...})`

* `.serialize()`: Returns a plain JavaScript array representation of the collection.

