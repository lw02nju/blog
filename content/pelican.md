Title: Pelican usage
Date: 2013-2-11 17:34
Category: tool
Slug: Pelican usage 
Author: Lu Wen

## install
* install virtualenv
* run 'pip install pelican Markdown Jinja2'

## setup
* setvirtualenvproject
* pelican-quickstart
* (write source files in contents/)
* make html/make regenerate
* make serve/make devserve(+regenerate)
* ./develop_server.sh stop
* make rsync_upload

## operations
* make -r/make --autoreload
* cd output && pythom -m SimpleHTTPServer

## mark down
* example
````markdown
Title: My super title
Date: 2010-12-03 10:20
Tags: thats, awesome
Category: yeah
Slug: my-super-post
Author: Alexis Metaireau
Summary: Short version for index and feeds

This is the content of my super blog post
````
* internal link: `[a link relative to current file](|filename|../article1.rst)`

## settings
* DEFAULT_DATE: use mtime as date
* USE_FOLDER_AS_CATAGORY:
* SUMMARY_MAX_LENGTH:
* FILENAME_METADATA: '(?P<date>\d{4}-\d{2}-\d{2})_(?P<slug>.*)'
* DEFAULT_LANG


[1]: [Pelican Getting Started](http://docs.getpelican.com/en/3.1.1/getting_started.html)
