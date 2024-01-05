---
title: "Install Jekyll with third-party plugins support"
date: 2019-05-03
categories: [Blogging,Jekyll]
tags: [plantuml]
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

# Create site

Follow instructions in [jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) starter page.

# Configure site

Edit _blog/source/Gemfile_ file and add PlantUML supporting:
~~~
...
group :plugins do
  gem "jekyll-plantuml"
end
...
~~~

Edit _blog/source/_config.yml_ file:
~~~
#--------------------------
theme: jekyll-theme-chirpy
timezone: Europe/Kiev
baseurl: ''
lang: en-US
#--------------------------
title: ZEON
tagline: Just my notes.
url: 'https://karz0n.github.io'
author: Denys Asauliak

github:
  username: karz0n
social:
  name: Denys Asauliak
  email: d.asauliak@gmail.com
  links:
    - https://github.com/karz0n
    - https://www.linkedin.com/in/karz0n
#--------------------------
...
plugins:
  - jekyll-plantuml
...
~~~
Change other properties on your own. The most important properties are:
* url
* avatar
* timezone
* theme_mode

After, `avatar` propery has been changed don't forget to place particular file.

# Configure deployment

Edit _.github/workflows/pages-deploy.yml_ file:
~~~
...
    steps:
      - name: Update apt cache
        run: sudo apt-get update

      - name: Install PlantUML Support
        run: sudo apt-get install -y graphviz plantuml
...
~~~

# Run site locally

To run site locally follow up this instruction:
```bash
$ cd <TOPDIR>
$ bundle install
$ bash tools/run.sh -r
```

# Deployment

1. Push any commit to the repository to trigger build workflow. Once the build is complete successfuly, a new remote brach named `gh-pages` will apear.
2. Configure GitHub pages to use `gh-pages` branch in `Settings → Options → GitHub Pages`.


