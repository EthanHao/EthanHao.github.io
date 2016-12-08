---
layout: post
title:  "A trouble when installing Jekyll on Windows"
date:   2016-12-02 13:45:20 -0600
categories: jekyll installation
---
Trying to use Jekyll to windows.But after installation of Ruby of newest version.I get a trouble to install the Jekyll using  

> `gem instal jekyll`  
> Unable to download data from https://rubygems.org/ - SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://api.rubygems.org/specs.4.8.gz)  


**Soulution** is we need update the gem package manually by the following command:  

> `gem install  --local the path of your local rubygems package you downloaded`   
> `update_rubygems --no-ri --no-rdoc`

Done. thenn you can install the Jekyll successfully.


After installation of Jekyll, You possibly will encounter another issue when you try to run the jekyll server. 

> Your bundle is locked to kramdown (1.13.0), but that version could not be found in any of the sources listed in your Gemfile. If you haven't changed sources, that means the author of kramdown (1.13.0) has removed it. You'll need to update your bundle to a different version of kramdown (1.13.0) that hasn't been removed in order to install.
Run `bundle install` to install missing gems.

As the warning message indicated, we just need to run `Bundle Install` to solve this issue of dependency.