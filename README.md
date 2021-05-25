# Sulis HPC Facility Documentation

This is the website of the Sulis HPC Facility documentation, it is built using Jekyll. The main website for the Sulis facility may be found at https://sulis.ac.uk.

## Contributing to the Site

Please with raise an issue or fork this site and submit a pull request if you would like to contribute to this site.

## Editing the Site Locally

The web site may be built and previewed locally, enabling changes to be reviewed before they are committed. The following steps *may* work to setup an environment in which you can build and preview the web site; the steps have only been attempted on Ubuntu 20.04 so YMMV and they assume that Ruby and Bundler are already installed (see the Prerequisites of https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll).

### Installing Jekyll with Bundle

The following steps are based on https://jekyllrb.com/tutorials/using-jekyll-with-bundler/

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
Writing new Gemfile to ...path...to...repo/sulis-hpc.github.io/Gemfile
```
