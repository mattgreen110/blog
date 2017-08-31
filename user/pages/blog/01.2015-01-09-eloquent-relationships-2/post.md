---
title: 'Complex Relationships Using Eloquent - Part 2'
published: false
date: '09-01-2015 00:00'
taxonomy:
    tag:
        - laravel
        - eloquent
        - doctrine
        - 'active record'
        - 'data mapper'
---

If you missed it and you did because no one reads this, [Part one](http://mattgreen.co/blog/eloquent-relationships)

No part 2 because I rewrote the entire thing using Doctrine after realizing I was starting to use Eloquent like a data mapper. Eloquent is awesome but it is not a data mapper.

I want to write about the process and exactly what moment I realized I needed to switch as the application grew, but I more than likely wont. [because I am lazy.](http://i1.kym-cdn.com/photos/images/original/000/516/329/9a0.gif) But the primary factor in that decision was due to the requirements that the API had for filtering and sorting. Eloquent does that fine until you start needing stuff like "filter product by status x AND y AND z as well as categories a OR b, also give me assets nested under each product but only with categories c and d, etc...

TGIF