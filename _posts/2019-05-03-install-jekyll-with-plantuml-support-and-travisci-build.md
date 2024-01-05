---
title: "Install Jekyll with PlantUML support and TravisCI build"
date: 2019-05-03
categories: [Blogging,Jekyll]
tags: [plantuml,travisci]
---

# Install dependencies

## Install _PlantUML_

~~~ bash
$ sudo apt install -y graphviz plantuml
$ plantuml -testdot
~~~

You should see something like that:
~~~
The environment variable GRAPHVIZ_DOT has not been set
Dot executable is /usr/bin/dot
Dot version: dot - graphviz version 2.38.0 (20140413.2041)
Installation seems OK. File generation OK
~~~

## Install _Ruby_

~~~ bash
$ sudo apt install ruby-full build-essential zlib1g-dev
$ tee -a ~/.bashrc > /dev/null <<'EOF'
# Install Ruby Gems to ~/opt/gems
export GEM_HOME="$HOME/opt/gems"
export PATH="$HOME/opt/gems/bin:$PATH"
EOF
$ source ~/.bashrc
~~~

## Install _Jekyll_

~~~ bash
$ gem install jekyll bundler
~~~

# Configure environment

Create local directory for template source:

~~~ bash
$ mkdir blog && cd blog
~~~

Download jekyll template source and place inside created 'blog' directory.

# Configure site

Create _blog/Gemfile_ for jekyll bunler:
~~~
source "https://rubygems.org"
ruby RUBY_VERSION

gem "rake"
gem "jekyll"

group :jekyll_plugins do
    gem "jekyll-feed"
    gem "jekyll-paginate"
    gem "jekyll-plantuml"
    gem "jekyll-sitemap"
end
~~~

Create _blog/_config.yml_ to configure jekyll :
~~~
title: Just my notes
description: My Personal notes
author: <author-name>
github: <author-github>
about: 
google_analytics: UA-XXXXXXXX-Y

enableTags: true
tagarray: []

collections:
  project:
    output: true
    permalink: /project/:path/

social:
    - icon: fa-facebook
      link: "<author_facebook>"
    - icon: fa-linkedin
      link: "<author_linkedin>"
    - icon: fa-github
      link: "<author_github>"

urls:
    - text: About Me
      url: /about/
    - text: My Projects
      url: /projects/
    - text: XML Feed
      url: /feed.xml

url: "$USER.github.io (replace $USER by your github username)"
baseurl: ""
paginate: 20
per_page: 20
paginate_path: "/page/:num/"
markdown: kramdown
plugins:
  - jekyll-paginate
  - jekyll-gist

compress_html:
  clippings: all
  comments: ["<!-- ", " -->"]
  endings: all
  ignore:
    envs: ['development']
~~~

Create _blog/.gitignore_ to exclude files:
~~~
_site
uml
.sass-cache
.jekyll-metadata
Gemfile.lock
~~~

Create _blog/.travis.yml_ to configure Travis-CI build container (go to [site](https://docs.travis-ci.com/user/languages/ruby/) for more information):
~~~
dist: xenial
language: ruby
rvm:
  - 2.4.5
before_install:
  - sudo apt-get install -y graphviz plantuml
install:
  - bundle install
script:
  - rake
deploy:
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN
  keep_history: true
  local_dir: _site
  target_branch: master
  on:
    branch: source
~~~

Create _blog/Rakefile_ to configure travis-ci build task:
~~~
task :default do
  puts "Running CI tasks..."
  sh("JEKYLL_ENV=production bundle exec jekyll build")
  puts "Jekyll successfully built"
end
~~~

# Run site locally

~~~ bash
$ bundle install
$ bundle exec jekyll serve
~~~

# Upload site to GitHub

Create repository on github with name `$USER.github.io`.

Initialize git repository locally and upload source to github:
~~~ bash
$ git init
$ git add --all
$ git commit -m "initial commit"
$ git remote add origin git@github.com:$USER/$USER.github.io.git
$ git push -u origin master
$ git checkout -b source
$ git push -u origin source
~~~

Prepare created repository on github:
* Open github and open particular repo;
* Go to "Settings" => "Branches" change default branch to "source" and update;
* Go to GitHub profile settings => "Developer Settings" => "Personal access tokens";
* Click "Generate new token", mark all "repo" checkboxes and apply (make sure you are copied generated token).

Prepare travis-ci autobuild:
* Go to https://travis-ci.org/ and log in;
* Go to profile settings and sync account, refresh page by F5;
* Activate particular repo using checkbox on the right and click "Settings";
* Append environment variables with name "GITHUB_TOKEN" and value from clipboard (copied generated token);
* Open "Dashboard", click "Trigger a build";
* Wait until build will be success.
