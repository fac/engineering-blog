---
:title: Passphrase generation using awk
:created_at: 2016-10-14 10:00:00.000000000 +00:00
:kind: article
:tags:
- awk
- security
:author: caius
---

<% excerpt do %>
Given a requirement of generating a temporary passphrase that can be communicated over the phone to another person, I thought of [XKCD #936][] which suggests using four random words together as a passphrase. Then there's just the question of how to generate that easily.
<% end %>

[XKCD #936]: https://xkcd.com/936/

On each system there's a file containing a list of words somewhere, on OS X it's located at `/usr/share/dict/words`. This contains a good ~236,000 words on my machine so that seems like a good enough corpus to pull our four words from. We could take the easy route and lean on gnu `sort` to sort the file randomly & then take the top four words using `head`:


    $ gsort -R /usr/share/dict/words | head -n4 | xargs
    fountained irretraceable unoil barylalia

This works, but isn't that performant (takes nearly 5 seconds on my machine.) There should be a more performant way to do this, we're just reading a file in as a list & picking four random elements from said list. How about we lean on awk instead:

    $ awk '
    BEGIN { srand() }
    { words[NR] = $0 }
    END {
      for (i = 1; i <= 4; i++) {
        f = "%s "
        if (i == 4) { f = "%s" }
        printf f, words[int(rand() * NR)]
      }
    }' /usr/share/dict/words
    swording memorability sneakingness readily

This runs in around 350ms on the same machine, which is a nice speedup and quick enough it feels near-instant.

Next time you need to generate a random passphrase, remember our old friend `awk`!
