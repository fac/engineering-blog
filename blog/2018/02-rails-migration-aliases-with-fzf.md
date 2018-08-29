---
:title: Rails migration aliases with fzf
:created_at: 2018-06-12 09:51:00.000000000 +00:00
:kind: article
:tags:
- Rails
:author: matthutchinson
:slug: rails-migration-aliases-with-fzf
---
<% excerpt do %>
Ever find yourself re-running Rails migrations? Up, down, redo-ing etc. Have
you forgotten that long `VERSION` number again? Or what the migration
actually migrates? These handy aliases just might be for you.
<% end %>

I've been running individual migrations a lot recently, so I took some time to
set up these aliases with [fzf](https://github.com/junegunn/fzf) (a command-line
fuzzy finder).

<pre>
rdbm     # bundle exec rake db:migrate (no auto-completion)
rdbmu    # bundle exec rake db:migrate:up
rdbmd    # bundle exec rake db:migrate:down
rdbmr    # bundle exec rake db:migrate:redo
</pre>

With this script:

<pre>
#!/bin/bash

# helper to echo and execute a command
function _echo_and_execute() {
  echo $1
  eval $1
}

# fzf (with preview) a rails migration and pass it as the command VERSION
function _fzf_rails_migration() {
  echo $(ls ./db/migrate/ | fzf --preview 'head -20 ./db/migrate/{}' | xargs | cut -d '_' -f 1 | xargs | tr -d '\n')
}

function rdbm() {
  if [ ! -d "./db/migrate" ]; then
    echo "Opps, you're not in a Rails directory"
  else
    if [ $# -eq 0 ]; then
      _echo_and_execute "bundle exec rake db:migrate"
    else
      migrate_version=$(_fzf_rails_migration)
      if [ ! -z $migrate_version ]; then
        _echo_and_execute "$*$migrate_version"
      fi
    fi
  fi
}

# db migrate up/down/redo with a VERSION
alias rdbmu="rdbm bundle exec rake db:migrate:up VERSION="
alias rdbmd="rdbm bundle exec rake db:migrate:down VERSION="
alias rdbmr="rdbm bundle exec rake db:migrate:redo VERSION="
</pre>

For example running `rdbmr` to redo a migration:

<a href="https://asciinema.org/a/180507" target="_blank">
  <img src="https://asciinema.org/a/180507.png" alt="screencast of Rails migration aliases with fzf" />
</a>

Auto-completion for your migrations with a useful preview! To install this for
yourself:

<pre>
brew install fzf
curl https://gist.githubusercontent.com/matthutchinson/6c1bc7681b323d4d2ef5d5a55626a5cf/raw > ~/.fzf_migrations
source ~/.fzf_migrations
</pre>

If this has piqued your interest, I have [other fzf related
aliases](https://github.com/matthutchinson/workbench/blob/master/bash/fzf)
to help with common git and shell commands.
