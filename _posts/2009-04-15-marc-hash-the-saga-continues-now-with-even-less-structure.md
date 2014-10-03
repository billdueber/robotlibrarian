---
title: "MARC-HASH: The saga continues (now with even less structure)"
date: 2009-04-15
---

After a medium-sized discussion on #code4lib, we've collectively decided that...well, ok, no one really cares all that much, but a few people weighed in. 

The new format is: A list of arrays. If it's got two elements, it's a control field; if it's got four, it's a data field.

SO....it's like this now.


~~~javascript

{
  "type" : "marc-hash",
  "version" : [1, 0],
  
  "leader" : "leader string"
  "fields" : [
     ["001", "001 value"]
     ["002", "002 value"]
     ["010", " ", " ", 
      [
        ["a", "68009499"]
      ]
    ],
    ["035", " ", " ",
      [
        ["a", "(RLIN)MIUG0000733-B"]
      ],
    ],
    ["035", " ", " ",
      [
        ["a", "(CaOTULAS)159818014"]
      ],
    ],
    ["245", "1", "0", 
      [
        ["a", "Capitalism, primitive and modern;"],
        ["b", "some aspects of Tolai economic growth" ],
        ["c", "[by] T. Scarlett Epstein."]
      ]
    ]
  ]
}

~~~
