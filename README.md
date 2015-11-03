# pouchy
[ ![Codeship Status for cdaringe/pouchy](https://codeship.com/projects/723a9160-4203-0133-3599-062894ba1566/status?branch=master)](https://codeship.com/projects/103658)

## what
Simple [PouchDB](https://github.com/pouchdb/pouchdb) wrapper, equipped with a few useful sugar methods.  Most methods provided are _very_ simple PouchDB-native method modifiers, but are targetted to save you frequent re-typing!  This library also proxies most of the PouchDB API directly, so you can use it like a Pouch itself!

## why
- because managing `_id` and `_rev` can be obnoxious with pouch.  Get your document back _in the same way it is represented in the store_ when using pouchy methods.
- because you need some frequently used, sugar methods.
    - ex `.all()`, to get all full documents in your store, in a simple array.
    - ex `.clear()/.deleteAll()` to purge your store of its docs.
- because you want `find` to return simply an array of docs!
- because it pre-loads the `pouchdb-find` plugin, which is super handy and regularly used!

## api
Most `fn`s below modify default PouchDB behavior.  For full options, make sure to check out the [PouchDB API](http://pouchdb.com/api.html)

### new Pouchy(opts)
Setup a new PouchDB wrapper!
- @constructor
- @param {object} opts {
    - name: {string=} name of db. calculated from derived url string if `conn` or `url` provided. otherwise, required
    - conn: {object=} creates `url` using the awesome and simple [url.format](https://www.npmjs.com/package/url#url-format-urlobj)
    - couchdbSafe {boolean=} [default: true] asserts that `name` provided or `url` provided will work with couchdb.  tests by asserting str conforms to [couch specs](https://wiki.apache.org/couchdb/HTTP_database_API#Naming_and_Addressing), minus the `/`.  This _may complain that some valid urls are invalid_.  Please be aware and disable if necessary.
    - path: {string=} path to store pouch on filesystem, if using on filesystem!  defaults to _PouchDB_ default of cwd
    - pouchConfig: {object=} PouchDB constructor [options](http://pouchdb.com/api.html#create_database)
    - replicate: {string=} [default: undefined] 'out/in/sync/both', where 'sync' and 'both' mean the same.  Shorthand for syncing a local datastore to a remote datastore.  **the local db name is extracted from the required url**.
    - replicateLive: {boolean=} [default: true] activates only if `replicate` is set
    - url: {string=} url to remote CouchDB

```js
var p = new Pouchy({ name: 'my-pouch-db' });
```


### all(opts)
- @param opts {object=} defaults to `include_docs: true`. In addition to the usual [PouchDB allDocs options](http://pouchdb.com/api.html#batch_fetch), you may also specify
`includeDesignDocs: true` to have CouchDB design documents returned.
- @return {promise}
- @resolve array of documents (excluding any design documents), vs an object
with a `docs` array per Pouch allDocs default

```js
p.all().then(function(docs) {
    console.log('i have a total of ' + docs.length + ' docs'!);
});

p.all({ includeDesignDocs: true }).then(function(docs) {
    console.log('this will include design docs as well');
});
```


### add(...args)
See `.save`

```js
// with _id
p.add({ _id: 'my-sauce', bbq: 'sauce' }).then(function(doc) {
    console.log(doc._id, doc._rev, doc.bbq); // 'my-sauce', '1-a76...46c', 'sauce'
});

// no _id
p.add({peanut: 'butter'}).then(function(doc) {
    console.log(doc._id, doc._rev, doc.peanut); // '66188...00BF885E', '1-0d74...7ac', 'butter'
});
```


### createIndicies(indicies)
Basic index creator.
- @param indices {array|string} 'an-index' or ['some', 'indicies']
- @return promise
- @resolves index meta.  see `pouch.createIndex`
```js
p.createIndicies('test')
    .then(function(indexResults) {
        console.dir(indexResults);
        ...
/*
{
    id: "_design/idx-28933dfe7bc072c94e2646126133dc0d"
    name: "idx-28933dfe7bc072c94e2646126133dc0d"
    result: "created"
}
 */
```


### delete(doc, opts, cb)
- @returns promise
- @alias `pouch.remove`
```js
p.delete(doc).then(function() { console.dir(arguments); });
/*
// same as pouch.remove
{
    id: "test-doc-1"
    ok: true
    rev: "2-5cf6a4725ed4b9398d609fc8d7af2553"
}
*/
```


### clear() / deleteAll()
Deletes all the documents in the store.
- @returns `promise`
- @resolves {array} [of, meta, === to `delete`'s @resolve meta]

### deleteDB()
- @returns `promise`
- @alias `pouch.destroy()`

### save(doc, opts)
Adds or updates a document.  If `_id` is set, a `put` is performed (basic add operation). If no `_id` present, a `post` is performed, in which the doc is added, and large-random-string is assigned as `_id`.
- @param doc {object} doc to store
- @param opts {object=} default `pouch.put/post` options
- @return {promise}
- @resolve doc, with updated `_id` and `_rev` properties

### update(doc[, _id, _rev])
- @returns promise
- @resolves doc, with updated _id, _rev


### find(opts)
Accept a find query, formatted per the [find plugin query options](https://github.com/nolanlawson/pouchdb-find#dbfindrequest--callback)
- @alias for `pouch.find`, with the `find` plugin loaded.  note that it is slated to be the default query mechanism over `pouch.query` in the long-term.

### proxied [pouch fns](http://pouchdb.com/api.html)
- destroy
- put
- post
- get
- remove
- bulkDocs
- allDocs
- changes
- replicate
- sync
- putAttachment
- getAttachment
- removeAttachment
- query
- viewCleanup
- info
- plugin
- compact
- revsDiff
- defaults

## instance properties
- name {string}
- db {PouchDB} ref to the PouchDB instance

## constructor properties
- PouchDB {function} ref to the Pouch constructor

Thanks! [cdaringe](http://cdaringe.com/)

# changelog
- 4.0.0 - major bump with PouchDB
- 3.0.0 - remove default `changes`, and associated `on/off`. didn't work out-of-the-box anyway.  may return in 4.x
- 2.0.1 - Don't modify constructor opts.name
- 2.0.0 - Fix synced db `fs` location. Previously was not honoring `path` option
- 1.0.0 - 2.0.1 pouchdb-wrapper => pouchy
