---
:title: Revving the engine
:created_at: 2012-11-08 14:38:31.000000000 +00:00
:kind: article
:tags:
- Accounting
- Code
- Platform
- Ruby on Rails
:author: paul
:slug: revving-the-engine
---
At FreeAgent we deal with a lot of accounting data - every invoice, bank
transaction, expense and VAT return in the system must be processed, and
its tax and accounting effects calculated and recorded.

In the early days of FreeAgent this was something that was baked
directly into the business objects – reporting data was pulled directly
from the day-to-day business records. As the number of users grew and
the software became more complex this evolved into a fully fledged
double-entry bookkeeping system – the best practice for accounting that
has its [origins over 700 years
ago](http://en.wikipedia.org/wiki/Double-entry_bookkeeping_system).

This is the brain of our application, constantly working away behind the
scenes to generate profit and loss reports, balance sheets and tax
returns for over 25,000 small businesses, sole traders and partnerships
who use our accounting software, in the UK, USA and worldwide.

Recently we’ve picked this apart and separated it from our main code
base into a separate project – the FreeAgent Accounting Engine. This has
been designed to encapsulate all this logic and act as a black box,
working the same way every time - business data comes in, accounting
data is generated and cached for easy access and speedy calculations.

Define Your Interfaces
----------------------

The business data is used to generate plain old Ruby objects, simple
non-persisted classes with a set of attributes containing the bare
minimum of information required to model the accounting effect. These
are defined in the accounting engine and these definitions are included
back into FreeAgent – Super
[DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself)!

    class Invoice < FreeAgent::AccountingEngine::SourceItem::Base
      attr_reader :net_value, :sales_tax_value

      def initialize(attributes = {})
        @net_value       = attributes[:net_value]
        @sales_tax_value = attributes[:sales_tax_value]
      end

      def gross_value
        net_value + sales_tax_value
      end
    end

These lightweight objects can be passed into the accounting engine in a
number of ways – directly from the application, via background jobs,
over
[DRb](http://www.ruby-doc.org/stdlib-1.9.3/libdoc/drb/rdoc/DRb.html),
or, as is our ultimate goal, pushed onto a message queue for fully
asynchronous processing.

Ride on Time
------------

Our accounting engine is a classic black box which means we can test it
separately from the main FreeAgent code base. Modelling our simple
source items in memory and passing them into the accounting engine
produces a test suite that is lightning quick to run, and also allows us
to remove these accounting tests from the FreeAgent project, simplifying
and speeding up our main test suite.

Using our own custom [Shoulda](https://github.com/thoughtbot/shoulda)
matcher we can write simple, expressive tests that allow us to verify
that the correct accounts are generated for any given item.

    should_generate_ledgers_for Invoice.new(
      :net_value => BigDecimal('100.00'), 
      :sales_tax_value => BigDecimal('20.00')) do |ledger|

      ledger.count  3
      ledger.exists 'Trade Debtors', BigDecimal('-120.00')
      ledger.exists 'Sales', BigDecimal('100.00')
      ledger.exists 'VAT Outputs', BigDecimal('20.00')
    end

Slow, Slow, Quick Quick, Slow
-----------------------------

Some of the features that we’re adding to FreeAgent are spread between
the application and the accounting engine – recent improvements to the
way we handle foreign currency transactions and capital assets, for
example. Because of this we need to keep the versions in lockstep with
each other, something that [Bundler](http://gembundler.com/) allows us
to do easily by specifying a git SHA for every release we deploy.

    gem 'freeagent-accounting-engine', 
        :git => 'git@github.com:fac/freeagent-accounting-engine.git', 
        :ref => 'a74001c50a5099936cb17e4fdf4eca348ce70ad0'

Zooooooom
---------

The most exciting thing about these changes is the fact that it puts us
in a great position to supercharge FreeAgent, making it even faster,
more responsive and allowing us to efficiently handle the increased
demand caused by our rapid growth. As we gain more users we can simply
add more engines to FreeAgent, delegating the calculations to an array
of dedicated accounting servers.

Because the changes our users make to their business data don’t need to
be reflected in their accounts in absolute real-time (a second or two
delay is generally acceptable, provided we’re clear in the UI when the
underlying data is being recalculated), we can begin to decouple the
engine from our front end. This architectural separation will allow us
to keep FreeAgent as snappy as an angry crocodile, and our users happy
and productive for the foreseeable future.
