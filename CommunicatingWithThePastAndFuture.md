# Communicating with the past and future
J. M. F. Tsang (j.m.f.tsang@cantab.net)

---

In his dialogue _Phaedrus_, Plato has unflattering things to say about
the development of writing, arguing among other things that it weakens
the memory:

> And so it is that you by reason of your tender regard for the writing
> that is your offspring have declared the very opposite of its true
> effect. If men learn this, it will implant forgetfulness in their
> souls. They will cease to exercise memory because they rely on that
> which is written, calling things to remembrance no longer from within
> themselves, but by means of external marks.
>
> What you have discovered is a recipe not for memory, but for reminder.
> And it is no true wisdom that you offer your disciples, but only the
> semblance of wisdom, for by telling them of many things without
> teaching them you will make them seem to know much while for the most
> part they know nothing. And as men filled not with wisdom but with the
> conceit of wisdom they will be a burden to their fellows.

What might have been applicable to the classical philosophers, however,
is not applicable to a modern society. (What does that say about us?) In
our world there is a tendency to regard the oral tradition as a remnant
of an ancient time, preserved only in 'primitive' cultures (_yikes_) or
among the illiterate – but it is much more prevalent than we care to
admit. In many organizations, knowledge and know-how are often held by a
single 'elder' or 'wise person', spread informally by the coffee machine
or over the shoulder in a pair programming session. For good reason:
this is usually the fastest and most direct way to communicate with
someone, as well as the warmest and the most interactive; whereas
writing complete documentation takes time and is usually not as fun.

However, this word-of-mouth approach soon becomes unsustainable as an
organization gets larger and older, and its collective memory fades:
individuals leave, or develop other interests or priorities, and those
who remain simply cannot be expected to remember the details of projects
long past. They may have taken care to document at the time, but this
information might be on a lost hard drive, or buried deep inside a
filing cabinet.

This amnesia costs the organization time and money, and severely
undermines its operations. For scientific computing, data engineering
and machine learning, this means results become irreproducible,
experience is lost, and the providence of data becomes forgotten -
possibly leading to the use of flawed data, or of data that has legal or
ethical restrictions.

As software engineers, one of the first things we learn is the value of
comments, especially well-written API documentation (docstrings); we are
also quickly introduced to version control systems such as Git, and
issue trackers (GitHub Issues, Bugzilla, Jira, _etc._).  Writing useful
comments and docs, commit messages, user stories and bug reports is an
art that takes time to learn, but one that software engineers spend
plenty of time practising. Writing docstrings is a largely mechanical
process that IDEs can automate, and user stories usually follow a
prescribed format ('As..., so that..., I want...').

However, knowledge and documentation of project-level information,
as well as operational practices, are much more unstructured and require
more creativity. I include such things as data collection or generation,
data processing steps, results and status reports; as well as 'savvy
know-how', such as shell commands (magic spells) that do not necessarily
fit into a version control system (for example, if they are too
specialized for a particular application). This sort of writing is a key
skill for the natural or social scientist, but overlooked by software
engineers.

For this sort of information, many organizations have therefore adopted
the use of a 'knowledge base' system, and wiki-style systems such as
Confluence or MediaWiki have become extremely popular (each with its
downsides and limitations). But a poorly-maintained knowledge base soon
becomes a swamp, impossible to search or read.

In what follows, I propose some techniques and best practices for using
these tools effectively.


## Cross-linking

A primary feature of wiki systems such as Confluence and MediaWiki –
indeed, the draw of the WWW – is the ability to link between pages. This
should be taken advantage of.

On Wikipedia, the 'See also' section at the bottom of articles

Their [Manual of
Style](https://en.wikipedia.org/wiki/Wikipedia:Manual_of_Style/Layout#%22See_also%22_section)
has guidelines of what should be included in such a section.

Confluence has good support for linking to Jira tickets,


## Categorization and heirarchy

MediaWiki supports filing articles into categories (_e.g._
https://en.wikipedia.org/wiki/Category:Coffee). Using this filing system
helps readers find articles related to the topic that they seek, even if
the pages may not directly link to each other
