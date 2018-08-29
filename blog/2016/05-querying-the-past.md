---
:title: Querying the past
:created_at: 2016-06-01 13:00:00.000000000 +00:00
:kind: article
:tags:
- data
- analytics
- ruby
:author: alex
:slug: querying-the-past
---

<% excerpt do %>
I’ve been learning to love the ActiveRecord query interface over the past few months. Whilst I find it infuriating when I’m battling it to do what I _actually_ want, I also relish the power and convenience it gives me for many simple queries.
<% end %>

So, when it came to designing a query language for historical data in our systems, ActiveRecord was a natural choice. We can now do queries like:

<pre>
FA::Subscriptions.
  active.
  before(start_of_may).
  with_company_id(company_id).
  order_by_time.last
</pre>

(to find the last subscription event where status was set to “Active” for a certain company for a certain date)

In this blog post, I’d like to show you how we built this query system.

## In the Beginning

When we started this project, historical data was stored as CSV text files, one per database table per date. To query the past involved finding the file for approximately the right date and time, then parsing or loading it to find the required information. This was a pain.

We also didn’t have access to fine-grained historical activity-generated events (essentially records of users interactions with our application). The solution we had aggregated data after a short period, so it was impossible to make detailed statistical analyses.

It’s worth noting the old adage of data science, that a large part of the work involves munging your data into shape for analyses. However, with this solution, almost _all_ of the work was munging data.

This made our data team sad.

## A New Hope

We were able to instrument our ActiveRecord models of interest to emit events over our messaging system (RabbitMQ), upon either create, update or delete. Collecting these in our data warehouse meant that we could piece together a lifetime history for all these tables.

For each entity record (e.g. a User, Company, etc), we now store a set of events which record how the entity has changed over time. Each of these events stores all the original entity data (we prefix each field with `fa_` to show that it is recording an entity field), along with some metadata of our own.

Of course, we couldn’t rely entirely on the events, as we could experience slow drift from reality if we missed events for any reason. So, we also apply a backup mechanism to ensure that real data and the historical data are kept in synch.

At the same time, we started storing our user interaction events (generated when users interact with the application)  ourselves, in a compact but searchable format.

Together, these improvements gave us a data store with historical changes to our main tables, and fine-grained application engagement events going back to when we started storing. These stores then augmented each other because of the shared keys which we could use to correlate the data.

## But Who Was Where When?

That’s all very well, but how do you find stuff out? Maybe I’d like to know all the companies who were direct customers (rather than through an accountancy practice) and based in Glasgow during the month of April. And maybe I’d also like to know how many of those logged in to the application in May?

Well, because we store the time at which changes were made, it’s possible to reconstruct the state of a particular entity at any given time. So, to find out what state a User was in at the beginning of April, all we need to do is find the last event for that user before the beginning of April.

That’s fine for just one user, but we might like to find _all_ the Users who were in a given state at a given time. That means finding the last record for all users before that time, and then selecting only those in a given state.

You can see that querying code is going to have some gnarly SQL - absolutely not what we want for our poor data team!

## Nicing It Up

Well, it turns out that we can use the power of Active Record to abstract all this nasty SQL behind a nice Rails-y interface. This means our team now get to write nice queries like:

<pre>
glasgow_company_ids = FA::Companies.direct.
  with_town(“glasgow”).
  during(first_april, first_may).company_ids
</pre>

This query will find all the companies who started the month based in Glasgow, along with all the companies who moved to Glasgow during the month (or even any Glasgow-based companies who started with us in the month).

Oh, the other question above - “how many of the Glasgow-based companies in April then logged in to the application in May?”

<pre>
FA::EngagementEvents.with_company_id(glasgow_company_ids).
  with_event(USER_LOGGED_IN).
  between(first_may, first_june).company_ids.count
</pre>

Well, it might not be beautiful, but it’s a pretty easy way to query our historical data!

## Where’s the Code?

I’d like to show you some of the code we use to do this, but first it’s worth talking about the design of the query language. It all looks pretty simple now, but it took us a surprising number of iterations to get it there.

The main hurdle was realising that we needed to split out multi-dimensional queries. It was too easy to find ourselves writing scopes which combined time and another dimension, like `field_value_at_time` - now what does that mean?!

So, being strict with our rule that each scope could only address one point, we were able to come up with nice simple scopes like:

<pre>
base.scope:currently, lambda { base.at_time(Time.now) }

# Remember fa_updated_at_milliseconds is when the entity was updated
base.scope:before, lambda { |end_date = Time.now|
  base.where("fa_updated_at_milliseconds <= ?",
    Milliseconds.to_millis(end_date))
}

base.scope:order_by_time, lambda {
  base.order(:fa_updated_at_milliseconds,:created_at)
}

base.scope:with_company_id, lambda { |id|
  base.where(:fa_company_id => id)
}
</pre>

Of course, we can use the power of Ruby to generate all the field comparisons dynamically, to end up with scopes like:

<pre>
with_town(town)
</pre>

Because the data tables contain much more than just the copy of the FreeAgent data, we prefix all the data which is copied by “fa_”, so we know which data is which. We also don’t want people messing around with the data directly (as it can be confusing), so we override the “where” scope and ban its direct use, replacing it with constructs of the form “with_email”. In this way, we can utilise the power of ActiveRecord without exposing it for the use of foot-shooting.

However, we have to do a little hacking:

<pre>
base.columns_hash.keys.each do |column|
  next if column.include?"created_at"
  next if column.include?"updated_at"
  # Make new [name]_all method to return every single value,
  # not just distinct values. Make user account owner
  if column.start_with?("fa_")
    straight_name = column.sub("fa_", "")
    plural_straight_name = straight_name.pluralize
    base.scope plural_straight_name.to_sym, lambda {
      base.distinct(column).pluck(column)
    }
    all_name = "all_" + plural_straight_name
    base.scope all_name.to_sym, lambda {
      base.pluck(column)
    }
    with_name = "with_#{straight_name}"
    base.scope with_name.to_sym, lambda { |value|
      base.where(column.to_sym => value)
    }
  end
end
</pre>

This code dynamically generates all the methods required to interact with every mirrored column.

## Wrapping it all up

We don’t want users to mess around with the actual records, so we wrap everything up in an “FA” interface (FA is short for FreeAgent):

<pre>
module FA
  class BaseEntity
    UPDATES = [
      CREATED = 'created'.freeze,
      UPDATED = 'updated'.freeze,
      DELETED = 'deleted'.freeze
    ]

    def self.method_missing(name, *arguments, &block)
      event_class.send(name, *arguments, &block)
    end

    def self.respond_to?(name)
      event_class.respond_to?(name)
    end

    def self.event_class
      raise NotImplementedError.new("Subclass must implement!")
    end

    def self.where(*)
      raise "Please don't use where in FA entities!"
    end
  end
end
</pre>

And that’s it! Now we can do powerful queries like:

<pre>
FA::Subscriptions.with_company_id(FA::Companies.
  created_between(date1, date2).
  with_type(FA::Companies::UK_LIMITED_COMPANY).company_ids).
  direct.
  during(date3, date4).company_ids.count
</pre>

Oh - and the data team? Well, they’re “getting overly excited using the new scopes” - I think that means we have a happy data team!



