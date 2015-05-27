runtime-types
=============

Use flow type information at runtime. Useful for validations, mapping to an ORM, and more.

Installation
------------

    npm install --save runtime-types

Runtime Example
---------------

If this file is in ./types.js

    // @flow
    export type PhoneNumber = string;

    export type User = {
      username: string;
      age: number;
      phone: PhoneNumber;
      created: ?Date;
    }

You can import the type information as follows:

    var types = require('runtime-types')

    // read the file into a runtime type description
    var MyTypes = types.readFile(path.join(__dirname, '../test/example-types.js'))

    // MyTypes is now equal to:
    {
      PhoneNumber: { name: 'string' },

      User: {
        name: 'Object',
        properties: [
          { key: 'username', type: { name: 'string' } },
          { key: 'age', type: { name: 'number' } },
          { key: 'phone', type: { name: 'PhoneNumber' } },
          { key: 'created', type: { name: 'Date', nullable: true } } 
        ]
      }
    }

Validation
----------

You can use this description to create validators for your types

    var types = require('runtime-types')
    var validate = require('runtime-types/validate')

    var MyTypes = types.readFile(path.join(__dirname, '../test/example-types.js'))

    var VALIDATORS:ValidatorMap = {
      PhoneNumber: validate.validateRegex(/^\d{10}$/),
    }

    var validators = createAll(VALIDATORS, MyTypes)

Then you can check various objects for errors

    var errs = validators.User({
      username: "bobby",
      age: 23,
      phone: "8014114399",
      created: null
    })

    // ==> []

Checks if fields are all set

    var errs = validators.User({
      age: 23,
      phone: "8014114399"
    })

    // ==> [ { key: 'username', value: undefined, error: 'missing' } ]

Checks correct typeof

    var errs = validators.User({
      username: "bobby",
      age: "not an age",
      phone: "8014114399",
    })

    // ==> [ { key: 'age', value: 'not an age', error: 'expected typeof number' } ]

Checks instances

    var errs = validators.User({
      username: "bobby",
      age: 23,
      phone: "8014114399",
      created: 1432757991843 // was supposed to be date, not a timestamp
    })

    // ==> [ { key: 'created',
    //  value: 1432757991843,
    //  error: 'expected instance of function Date() { [native code] }' } ]

Provided Validators: regex

    var VALIDATORS:ValidatorMap = {
      PhoneNumber: validate.validateRegex(/^\d{10}$/),
    }

    var errs = validators.User({
      username: "bobby",
      age: 23,
      phone: "801-443-8899", // should be 10 digits without hyphens
    })

    // [ { key: 'phone',
    //     value: '801-411-4399',
    //     error: 'did not match /^\\d{10}$/' }, ]

Custom Validators: anything

    var VALIDATORS:ValidatorMap = {
      PhoneNumber: function(value) {
        if (value.length == 10) {
          return true
        }
        else {
          return "wrong length!"
        }
      }
    }

It does not try to guess validators for your type aliases. If you forget to provide one it will throw an error when you generate the validators

    var VALIDATORS:ValidatorMap = {}

    var validators = createAll(Validators, MyTypes)

    // Error: Could not find validator for type: PhoneNumber
