---
layout: post
title: "encodeURIComponent in ruby"
description: "how to perform encodeURIComponent in ruby"
tags: [web, note]
---
## Purpose
For my project [jsdomrenderer](https://github.com/zenixls2/jsdomrenderer) deployed in AWS lambda, recently I met some bug caused by the ruby client. The url parameter passed in was wrong, and looks like being encoded twice. (ex: '%' should be encoded as '%25', but ruby's URI::encode_www_form_component returns '%2525'. The javascript's encodeURIComponent works fine in this case, that's why I have this title)

## Solution
Based on the [stackoverflow discussion](https://stackoverflow.com/questions/2834034/how-do-i-raw-url-encode-decode-in-javascript-and-ruby-to-get-the-same-values-in-b), it gives out 
```ruby
URI.escape(foo, Regexp.new("[^#{URI::PATTERN::UNRESERVED}]"))
```
to replace encode_www_form_component.
The regular expression gives out the following pattern for match: `/\[^\-\_.!~\*'()a-zA-Z\d\]/`. Where does this pattern comes from? Well, let's look into [encodeURIComponent's implementation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent). It escapes everything except `A-Z a-z 0-9 - _ . ! ~ * ' ( )`. Also notice that other language's default encode/decode uri component functions might have slight differences. Be sure to check some corner cases before your service goes to the production.

