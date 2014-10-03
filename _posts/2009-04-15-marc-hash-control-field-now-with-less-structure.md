---
title: "MARC-HASH control field, now with less structure"
date: 2009-04-15
layout: post

---

Why do I ever, ever think that MARC might not rely on order? I don't know.

In any case, control fields will now be just an array of duples:


~~~javascript

control: [
  ['001', 'value of the 001'],
  ['006', 'value of the 006']
  ['006', 'another 006']
}

~~~
