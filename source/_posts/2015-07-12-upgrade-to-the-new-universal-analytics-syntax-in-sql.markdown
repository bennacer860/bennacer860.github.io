---
layout: post
title: "Upgrade to the new Universal Analytics Syntax in SQL"
date: 2015-07-12 15:14:18 -0400
comments: true
categories: 
---


Upgrade to the Universal Analytics Syntax in SQL

We are all struggling with upgrading our analytics tools from the old ga.js to the new universal.js plateform. 
Most of it is done automatically and would require just changing that ga.js file. But would would happen to event tracking? all that `_gaq.push(['_trackEvent', 'button', 'event'])` that should be turned into something like this `ga('send', 'event', 'button');` 
 
In this post we are going to update our syntax that is present in a code stored in Postgres. This is very common, just imagine you have a table `post` with a field `body` and you want to track some click on a specific post. so you would have something like `a href="#" onClick="_gaq.push(['_trackEvent', 'Videos', 'Play', 'Baby\'s First Birthday']);">Play</a>` . We need to figure out ways to track all the instances of this syntax and then replace it with the new syntax. We don't want to do it by hand if there are more than 2 :), that would take forever!! so let's figure out an automated way

Doing this with a Programming Language would take a long time, the best way seems to be SQL. We need to find all the `posts` where the tracking event is used:

```sql
 select id,regexp_matches(body,'_gaq.push\(\[\s*?''\s*?_trackEvent\s*?''\s*?,(.+?)\]\)', 'g') as match
  from  posts ;
```

Good, now let's make sure that this would capture multiple instances in a single post, so let's create a query that would display every post and how many matches it has:

```sql
SELECT id,count(*) FROM posts, regexp_matches(body,'_gaq.push\(\[\s*?''\s*?_trackEvent\s*?''\s*?,(.+?)\]\)', 'g') GROUP BY id HAVING count(*) > 0;
```

it would look like this:

            id   | count
       19911 |     1
       20894 |     6
       19717 |     8
       19178 |     1
       ..... | .....

Now that our regex capture all the instances of the old syntax, it is time to change it. Let's update our table with the new syntax:

```sql
begin;
  UPDATE posts SET DATA = regexp_replace(body,'_gaq.push\(\[\s*?''\s*?_trackEvent\s*?''\s*?,(.+?)\]\)', 'ga(''send'',''event'',\1)','gi' )    
        WHERE id IN ( SELECT id FROM posts 
                           WHERE DATA::text LIKE '%_gaq.push%' );
commit;
```

This looks scary but it is not!! I am capturing the parameter for `_gaq.push` and will create a new function call `ga(...)`  using the `\1` .

And you should be ready to go now. You can see the event tracking in real time, that is the best way to test it!



