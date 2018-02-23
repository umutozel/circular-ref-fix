# circular-ref-fix

[![Build Status](https://travis-ci.org/umutozel/circular-ref-fix.svg?branch=master)](https://travis-ci.org/umutozel/circular-ref-fix)
[![Coverage Status](https://coveralls.io/repos/github/umutozel/circular-ref-fix/badge.svg?branch=master)](https://coveralls.io/github/umutozel/circular-ref-fix?branch=master)

:ferris_wheel: Fixes circular dependencies using $id identifiers and $ref pointers. Produces Json.NET friendly result.

---

Let's model Simpsons family (Maggie hasn't born yet),

```JavaScript
    var child1 = {
        name: 'Bart'
    };
    var child2 = {
        name: 'Lisa'
    };
    var mother = {
        name: 'Marge'
    };
    var father = {
        name: 'Homer'
    };

    child1.siblings = [child2];
    child1.mother = mother;
    child1.father = father;

    child2.siblings = [child1];
    child2.mother = mother;
    child2.father = father;

    mother.spouse = father;
    mother.children = [child1, child2];

    father.spouse = mother;
    father.children = [child1, child2];
    
    JSON.stringify([mother, father]); // error! could not serialize
```
We create referential links between objects;

```JavaScript
var circularFix = require('../index');
var createRefs = circularFix.createRefs;

var fixed = createRefs(couple);
console.log(fixed);
```
Produces below json;
```JavaScript
[  
   {  
      "$id":"1",
      "name":"Marge",
      "spouse":{  
         "$id":"2",
         "name":"Homer",
         "spouse":{  
            "$ref":"1"
         },
         "children":[  
            {  
               "$id":"3",
               "name":"Bart",
               "siblings":[  
                  {  
                     "$id":"4",
                     "name":"Lisa",
                     "siblings":[  
                        {  
                           "$ref":"3"
                        }
                     ],
                     "mother":{  
                        "$ref":"1"
                     },
                     "father":{  
                        "$ref":"2"
                     }
                  }
               ],
               "mother":{  
                  "$ref":"1"
               },
               "father":{  
                  "$ref":"2"
               }
            },
            {  
               "$ref":"4"
            }
         ]
      },
      "children":[  
         {  
            "$ref":"3"
         },
         {  
            "$ref":"4"
         }
      ]
   },
   {  
      "$ref":"2"
   }
]
```
Also, we can restore circular structure back;

```JavaScript
var circularFix = require('../index');
var restored = restoreRefs(fixed, true); // with true option, we tell circular-ref-fix to delete $id fields
```
