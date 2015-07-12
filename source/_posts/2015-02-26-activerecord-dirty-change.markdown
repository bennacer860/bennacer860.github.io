---
layout: post
title: "ActiveRecord dirty change"
date: 2015-02-26 22:47:14 -0500
comments: true
categories: 
---


Today, After chatting with my colleague and try to fix some [ActiveRecord Store bug](http://api.rubyonrails.org/classes/ActiveRecord/Store.html), he showed me some very neat [ActiveRecord:Dirty](http://api.rubyonrails.org/classes/ActiveModel/Dirty.html) method called changed.

I went to Rails documentation and found out the module that allows all this magic. It will tell you if anything has changed on the object.

```
2.0.0-p598 :109 > c = Content.first
  Content Load (0.8ms)  SELECT  "content".* FROM "content"   ORDER BY "content"."id" ASC LIMIT 1
 => #<Content id: 1, content_type_id: 2, name: "culture page", url_name: "culture_page", options: {}, creator_id: 123, created_at: "2014-04-21 17:02:23", updated_at: "2014-05-30 18:44:41", published_at: "2014-04-21 17:02:23", removed_at: "2014-05-30 18:44:41">
2.0.0-p598 :110 > c.changed?
 => false
2.0.0-p598 :111 > c.name = "hi"
 => "hi"
2.0.0-p598 :112 > c.changed?
 => true
2.0.0-p598 :113 > c.changed
 => ["name"]
2.0.0-p598 :115 > c.name_change
 => ["culture page", "hi"]
 2.0.0-p598 :118 > c.changes
 => {"name"=>["culture page", "hi"]}
 2.0.0-p598 :114 > c.name_was
 => "culture page"
```

If you really to avoid saving and making useless DB call. you can always use this module.
