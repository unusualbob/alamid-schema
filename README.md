alamid-schema
========================================================================
**Extendable mongoose-like schemas for node.js and the browser**

[![npm status](https://nodei.co/npm/alamid-schema.svg?downloads=true&stars=true)](https://npmjs.org/package/alamid-schema)<br>
[![build status](https://travis-ci.org/peerigon/alamid-schema.svg)](http://travis-ci.org/peerigon/alamid-schema)
[![dependencies](https://david-dm.org/peerigon/alamid-schema.svg)](http://david-dm.org/peerigon/alamid-schema)

If you like [mongoose](http://mongoosejs.com/) schemas and you want to use them standalone, **alamid-schema** is the right module for you.

```javascript
var Schema = require("alamid-schema");

var Panda = new Schema("Panda", {
    name: String,
    age: {
        type: Number,
        required: true
    },
    mood: {
        type: String,
        enum: ["grumpy", "happy"]
    },
    birthday: Date
});
```

## Examples

### Schema Definition

You can defined your Schema with concrete values...

```javascript
var PandaSchema = new Schema({
    name: "panda",
    age: 12,
    friends: {
        type: []
    }
});
```

..or with abstract types... 

```javascript
var PandaSchema = new Schema({
    name: String,
    age: Number,
    friends: {
        type: Array
    }
});
```

### Extend 

Sometimes you want to extend your Schema and add new properties.


```javascript
var PandaSchema = new Schema({
    name: String,
    age: Number,
    friends: {
        type: Array
    }
});

var SuperPanda = PandaSchema.extend("SuperPanda", {
    xRay: true,
    canFly: {
        type: Boolean
    }
});


/*
//we have a superpanda now... which can fly and has xray eyes!
//same as...
var SuperPanda = new Schema({
    name: String,
    age: Number,
    friends: {
        type: Array
    },
    xRay: true, //added
    canFly: {   //added 
        type: Boolean
    }
});
*/
```


__Overwriting properties__ 

If you define a property in the schema you are extending with, the extending schema takes precedence. 


```javascript
var Animal = new Schema({
    name: String,
    age: String
});

var Panda = Animal.extend("Panda", {
    age: Number
    color: String
});

/*
//equals 
var Panda = new Schema("Panda", {
    name: String,
    age: Number,   //overwritten
    color: String  //added 
});
*/
```

### Schema Subsets

If you need a subset of a Schema containing only certain properties, you can use `Schema.fields()` . 

```javascript
var Panda = new Schema("Panda", {
    name: String,
    age: Number,
    color: String 
});

var SubsetPanda = Panda.fields("name", "age"); 

/*
//equals
var Panda = new Schema("Panda", {
    name: String,
    age: Number
});
*/
```

### Plugin: Validation

The validation plugins adds - *suprise!* - validation support.  

```javascript
"use strict";

var Schema = require("alamid-schema");
Schema.use(require("alamid-schema/plugins/validation"));

var PandaSchema = new Schema({
    name: {
        type: String,
        required: true
    },
    age: {
        type: Number,
        min: 9,
        max: 99
    },
    mood: {
        type: String,
        enum: ["happy", "sleepy"]
    },
    birthday: Date
});

var panda = {
    name: "Hugo",
    age: 3,
    mood: "happy"
};

PandaSchema.validate(panda, function(validation) {

    if(!validation.result) {
        console.log(validation);
        return;
    }

    console.log("happy panda");
});

/*
//outputs...
{
    result: true,
    failedFields: {
        age: [ 'min' ] 
    }
}
*/
```

_Included validators:_  

- required
- (Number) min
- (Number) max
- enum 

_Writing custom validators:_
 
You can write sync and async validators.. 

```javascript

//sync
function oldEnough(age) {
    return age > 18 || "too-young";
}

//async
function nameIsUnique(name, callback) {

    fs.readFile(__dirname + "/names.json", function(err, names) {
        if(err) {
            throw err;
        }

        names = JSON.parse(names);

        callback(names.indexOf(name) === -1 || "name-already-taken");
    });
}

var PandaSchema = new Schema({
    name: {
        type: String,
        required: true,
        validate: nameIsUnique
    },
    age: {
        type: Number,
        validate: oldEnough,
        max: 99
    }
});

var panda = {
    name: "hugo",
    age: 3,
    mood: "happy"
};

PandaSchema.validate(panda, function(validation) {

    if(!validation.result) {
        console.log(validation.failedFields);
        return;
    }

    /**
     * { name: [ 'name-already-taken' ], age: [ 'too-young' ] }
     */

    console.log("happy panda");
});
``` 

_Note:_ validators will be called with `this` bound to `model`. 

## API 

### Core 

#### Schema(name?: String, definition: Object)

Creates a new schema. 

#### .fields(key1: Array|String, key2?: String, key3?: String, ...)

Returns a subset with the given keys of the current schema. You may pass an array with keys or just the keys as arguments.

#### .extend(name?: String, definition: Object)

Creates a new schema that inherits from the current schema. Field definitions are merged where appropriate. 
If a definition conflicts with the parent definition, the child's definition supersedes.

### Plugin: Validation

#### .validate(model: Object, callback: Function)

Validate given model using the schema-definitions. 

Callback will be called with a validation object with `result` (Boolean) and `failedFields` (Object).
