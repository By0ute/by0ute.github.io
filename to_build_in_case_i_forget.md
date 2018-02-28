See [nandomoreira.me](http://nandomoreira.me/nandomoreira-jekyll-theme/)

## Setup

### In the terminal run the commands

```
$ sudo npm i -g gulp bower browser-sync
$ sudo gem install bundler
$ bundle install
$ npm install
```

## Deploy in Github pages in 2 steps

1. Change the variables `GITHUB_REPONAME` and `GITHUB_REPO_BRANCH` in `Rakefile`
2. Run `rake` or `rake publish` for build and publish on Github


PS: Be sure the Gemfile and 8config.yml are up to date in case of conflict