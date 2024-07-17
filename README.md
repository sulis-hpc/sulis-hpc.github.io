# Sulis HPC Facility Documentation

This is the website of the Sulis HPC Facility documentation, it is built using Jekyll. The main website for the Sulis facility may be found at https://sulis.ac.uk.

## Contributing to the Site

Please with raise an issue or fork this site and submit a pull request if you would like to contribute to this site.

## Editing the Site Locally

The web site may be built and previewed locally, enabling changes to be reviewed before they are committed. The following steps *may* work to setup an environment in which you can build and preview the web site; the steps have only been attempted on Ubuntu 20.04 so YMMV and they assume that Ruby and Bundler are already installed (see the Prerequisites of https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll).

### Installing Jekyll/Github-Pages

The following steps are based on https://jekyllrb.com/tutorials/using-jekyll-with-bundler/ and https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll.

Clone the repository and change into the directory. For example:

```bash
$ git clone git@github.com:sulis-hpc/sulis-hpc.github.io.git
Cloning into 'sulis-hpc.github.io'...
remote: Enumerating objects: 35, done.
remote: Counting objects: 100% (35/35), done.
remote: Compressing objects: 100% (24/24), done.
remote: Total 35 (delta 6), reused 27 (delta 4), pack-reused 0
Receiving objects: 100% (35/35), 10.97 KiB | 5.48 MiB/s, done.
Resolving deltas: 100% (6/6), done.
$ cd sulis-hpc.github.io.git
```

Start a new bundle project by running `bundle init`.
```bash
$ bundle init
Writing new Gemfile to <repopath>/Gemfile
```

It is recommended to configure bundle to install the Ruby gems into the subdirectory of the project (note that the .gitignore for this project will ignore the directory).

```bash
$ bundle config set --local path 'vendor/bundle'
```

The Gemfile in the directory should be relatively empty:

```ruby
# frozen_string_literal: true

source "https://rubygems.org"

git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

# gem "rails"
```

To the bottom of the Gemfile add

```ruby
gem "github-pages", group: :jekyll_plugins
```

or 

```ruby
gem "github-pages", "~> GITHUB-PAGES-VERSION", group: :jekyll_plugins
```

replacing `GITHUB-PAGES-VERSION` with the version number, if you want to lock to a specific versions (see https://pages.github.com/versions/).

Then install the Gems with:

```bash
$ bundle install
```

### Serving the Pages Locally

The pages can be served locally using:

```bash
$ bundle add webrick (if you are using Ruby version 3.0 or above)
$ bundle exec jekyll serve
Configuration file: <repopath>/_config.yml
            Source: <repopath>/
       Destination: <repopath>/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
      Remote Theme: Using theme pmarsceill/just-the-docs
                    done in 3.679 seconds.
<repopath>/vendor/bundle/ruby/2.7.0/gems/pathutil-0.16.2/lib/pathutil.rb:502: warning: Using the last argument as keyword parameters is deprecated
 Auto-regeneration: enabled for '<repopath>'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

Then go to http://127.0.0.1:4000/ in a browser on your local computer.

Note that you do not need to commit the local build of the site to Github as the pages will be automatically rebuilt and served.
