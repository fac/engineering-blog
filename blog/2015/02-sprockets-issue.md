---
:title: Complexity will bite you in production
:created_at: 2015-03-04 15:00:00.000000000 +00:00
:kind: article
:tags:
- deployment
- debugging
:author: alberto
:slug: complexity-will-bite-you-in-production
---

Here at FreeAgent we deploy updates to our app several times a day, and each deployment used to take between 10 and 15 minutes. This meant that if we introduced a severe bug into production and we needed to rollback, it would take us at least 10 minutes. This situation was not ideal, so we've been working to improve it.

A quick look at our deployment process indicated that the most expensive step was asset precompilation, so we started with this. There is no need to precompile assets on every deploy, we only need to precompile assets whenever they change. A common solution is to compare with git the revision we are deploying to the currently deployed version. If there are no changes to assets, gems or config files, we can skip the precompilation step.

So we created our own `assets:precompile` deploy task that only precompiles assets if this is actually needed. With just that change we were able to shave a few minutes from our deploys. After QAing these changes and making sure assets were precompiled when needed, we deployed the change to our production servers.

The new deploy took less than three minutes and we were pretty excited.

## Rollback!

This excitement only lasted a few seconds, as almost immediately we started to see 500 errors for some of our customers:

    Sass::SyntaxError: File to import not found or unreadable:
    talaria/practices/whitelabel. Load path: /releases/20150213142743
    (in /releases/20150213142743/app/assets/stylesheets/whitelabel.css.scss)

What? Sass syntax errors on production? We quickly rolled back the deploy, this time forcing precompilation, but because of the slow deploys it took us a few minutes to solve the issue.

After the dust from the deployment had settled we started to investigate the issue to see what had happened. And what we found was pretty surprising.

## Hidden complexity

Why was Sass throwing exceptions in production? We thought that Sass was only invoked at precompilation time. The answer was in the following method:

    def has_stylesheet?
      Rails.application.assets.find_asset("#{stylesheet_name}.css").present?
    end

At FreeAgent we have a growing number of channel partners. Some of these partners want the FreeAgent app to show custom styles to their clients. The styles depend on the particular partner, and not all the partners needs custom styles. That code was checking if a particular partner had a custom stylesheet we need to include.

The real problem is that there is a lot going on in that line. `Rails.application.assets` is really an instance of `Sprockets::Environment`. In our development and test environments, Sprockets is properly configured and it does the right thing. But in production the asset path is wrong and Sprockets can not find the stylesheet.

That seems to explain why the app started to fail after we deployed the conditional precompilation change. But it also creates another puzzling question: if the asset path is wrong in production, how was that line working before?

To answer that question we had to dig deeper into the error stack trace. The actual `Sass::SyntaxError` was thrown 20 calls deep inside the Sass stack. Following the stack we found code like this:

    # Cache asset building in memory and in persisted cache.
    def build_asset(path, pathname, options)
      # Memory cache
      key = cache_key_for(pathname, options)
      if @assets.key?(key)
        @assets[key]
      else
        @assets[key] = begin
          # Persisted cache
          cache_asset(key) do
            super
          end
        end
      end
    end

Each time Sprockets needs to find an asset it looks first in the file system cache, located in `tmp/assets`. Even if the asset path was wrong in production, Sprockets would still find the asset in the cache and everything would work. When we skipped precompilation, we created a release without an asset cache and when the production app tried to find an asset it blew up.

We were able to reproduce this issue by deleting the asset cache from a working deploy, so at that point we were pretty confident we had found the cause of the issue.

## Lessons learned

As with every fail, the first step is to fix it and afterwards try to use it to learn something. There are a couple of lessons we learned from this:

1. We were using Sprockets to do the simple task of finding a stylesheet. Under the hood Sprockets was executing the whole Sass stack, and that behaviour is dependent on non-obvious configuration settings like the load path. We don't need all that complexity in production! A plain old `File.exist?` call would have been much simpler and easier to understand than calling `Rails.application.assets.find_asset`.

2. We actually had this same issue a few months ago when we tried to use the `turbo-sprockets-rails3` gem. When we saw these errors we just reverted the change without fully understanding what had happened. But when a bad bug hits in production, a good post-mortem investigation can really help to prevent similar issues in the future. It's time worth investing.

After finally understanding all these issues we were able fix our code and reduce our deploy time. All's well that ends well :)
