[![Build Status](https://travis-ci.org/mimani/mongoose-diff-tracking.svg?branch=master)](https://travis-ci.org/mimani/mongoose-diff-tracking)
[![Downloads](https://img.shields.io/npm/dt/mongoose-diff-tracking.svg)](https://www.npmjs.com/package/mongoose-diff-tracking)
[![Codacy](https://api.codacy.com/project/badge/grade/bf1936538af048ac8d104a6c2ecd71ca)](https://www.codacy.com/app/mimani-saurabh/mongoose-diff-tracking)
[![Code Climate](https://codeclimate.com/github/mimani/mongoose-diff-tracking/badges/gpa.svg)](https://codeclimate.com/github/mimani/mongoose-diff-tracking)
[![Test Coverage](https://codeclimate.com/github/mimani/mongoose-diff-tracking/badges/coverage.svg)](https://codeclimate.com/github/mimani/mongoose-diff-tracking/coverage)

mongoose-diff-tracking
=============

Stores and Manages all the differences and versions, any Mongo collection goes through it's lifecycle.

THIS IS A FORK FROM [mongoose-diff-tracking](https://www.npmjs.com/package/mongoose-history)

## Installation
---------------
### npm
``` sh
npm install mongoose-diff-tracking
```

## Operation
---------------
Each update will create a history record with [jsonDiff](https://github.com/benjamine/jsondiffpatch) of the change. This helps in tracking all the changes happened to an object from the beginning.

Following will be the structure of the diff history being saved:


diff Collection schema:

```
_id : mongo id of the diff object
collectionName: Name of the collection for which diff is saved
collectionId : Mongo Id of the collection being modified
diff: diff object
user: User who modified
reason: Why the collection is modified
createdAt: When the collection is modified
_v: version
```

## Usage
---------------
Use as you would any Mongoose plugin:

```js
var mongoose = require('mongoose'),
    diffHistory = require('mongoose-diff-tracking/diffHistory'),
    schema = new mongoose.Schema({ ... });
    schema.plugin(diffHistory.plugin);
```

The plugin also has an omit option which accepts either a string or array. This will omit the given
keys from history. Follows dot syntax for deeply nested values.

```js
const mongoose = require('mongoose');
const diffHistory = require('mongoose-diff-tracking/diffHistory');

const schema = new mongoose.Schema({
    someField: String,
    ignoredField: String,
    some: {
        deepField: String
    }
});

schema.plugin(diffHistory.plugin, { omit: ['ignoredField', 'some.deepField'] });
```

## Send history to another database
---------------

You can keep your history collection far away from your primary database or replica set. This can be useful for improve the architecture of your system.

Just create another connection to the new database and link the reference in options.uri:

```js
let opts = {
    uri : config.mongo.path
}
var diffHistory = require("mongoose-diff-history/diffHistory");
EmployeeSchema.plugin(diffHistory.plugin,opts,opts);
```


## Helper Methods
---------------
You can get all the histories created for an object using following method:

```js
const diffHistory = require('mongoose-diff-tracking/diffHistory');
const expandableFields = ['abc', 'def'];

diffHistory.getHistories('modelName', ObjectId, expandableFields, function (err, histories) {

});

// or, as a promise
diffHistory.getHistories('modelName', ObjectId, expandableFields)
    .then(histories => {})
    .catch(console.error);
```

If you just want the raw histories return with json diff patches:

```js
const diffHistory = require('mongoose-diff-tracking/diffHistory');

diffHistory.getDiffs('modelName', ObjectId, function (err, histories) {

});

// or, as a promise
diffHistory.getDiffs('modelName', ObjectId).then(histories => {}).catch(console.error);

// with optional query parameters:
diffHistory.getDiffs('modelName', ObjectId, { select: 'diff user' })
    .then(histories => {})
    .catch(console.error);
```

You can get an older version of the object using following method:
```js
const diffHistory = require('mongoose-diff-tracking/diffHistory');

diffHistory.getVersion(mongooseModel, ObjectId, version, function (err, oldObject) {

});

// or, as a promise
diffHistory.getVersion(mongooseModel, ObjectId, version)
    .then(oldObject => {})
    .catch(console.error);
```

You can also use Mongoose query options with getVersion like so:
```js
const diffHistory = require('mongoose-diff-tracking/diffHistory');

diffHistory.getVersion(mongooseModel, ObjectId, version, { lean: true }, function (err, oldObject) {

});

// or, as a promise
diffHistory.getVersion(mongooseModel, ObjectId, version, { lean: true })
    .then(oldObject => {})
    .catch(console.error);
```



## Example
---------------
I have created an [example](https://github.com/mimani/mongoose-diff-tracking/tree/master/example) express service (documentation [here](https://github.com/mimani/mongoose-diff-tracking/blob/master/example/README.md)), demonstrating this plugin via an simple employee schema, checkout `example` directory in this repo.
