---
layout: post
title: "Jekyll Compile/Serve Error After Ruby Update"
description: "Dependency error on Jekyll, saying jemoji or other dependencies not installed"
tags: [web, note]
---
After Ruby updated on the end of March, some may found ```bundle exec jekyll serve``` not worked,  and come the following error messages:

```text
  Dependency Error: Yikes! It looks like you don't have jemoji or one of its
  dependencies installed. In order to use Jekyll as currently configured, 
  you'll need to install this gem. The full error message from Ruby is: 
  'dlopen(/usr/local/lib/ruby/gems/2.4.0/gems/nokogiri-1.7.1/lib/nokogiri/nokogiri.bundle, 9): 
  Library not loaded: 
  /usr/local/opt/ruby/lib/libruby.2.4.0.dylib
  Referenced from: 
  /usr/local/lib/ruby/gems/2.4.0/gems/nokogiri-1.7.1/lib/nokogiri/nokogiri.bundle
  Reason:
  image not found - /usr/local/lib/ruby/gems/2.4.0/gems/nokogiri-1.7.1/lib/nokogiri/nokogiri.bundle'
  If you run into trouble, you can find helpful resources at
  https://jekyllrb.com/help/!
```

To fix this, just execute `gem install nokogiri` to re-install nokogiri.
