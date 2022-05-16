# Communicating with the past and future

J. M. F. Tsang (j.m.f.tsang@cantab.net)

---


## Introduction

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
becomes a swamp, impossible to search or read. Readers give up and go
back to the oral tradition.

In what follows, I propose some techniques and best practices for using
these tools effectively.

Some context about myself: I'm a software engineer of about three years;
my day job title is 'Senior Data Engineer', meaning I spend a lot of
time thinking about data pipelines and processes. I moved into the world
of software engineering from a stint in academia; my research background
was in mathematical and computational physics.  I'm a longtime Wikipedia
reader and sometime Wikipedia contributor, and spent too much time
reading the Wikipedia policies and [manual of style](https://en.wikipedia.org/wiki/Wikipedia:Manual_of_Style).
We use the Atlassian products (Jira, Confluence) at work, and Slack as
the primary means of live communication.


## The Golden Rule

Ignore all rules.

This is an [official policy of Wikipedia](https://en.wikipedia.org/wiki/Wikipedia:Ignore_all_rules).

> If a rule prevents you from improving or maintaining Wikipedia, ignore
> it.

These are _proposals_, not rules. They reflect _my_ opinions, shaped by
my background and experience. Your mileage may vary.


## Using wiki software

### Is it the right tool?

A knowledge base should aspire to be the 'single point of truth' that
future readers can refer to. Its contents and style should therefore be
timeless, and impersonally or objectively written. While an article
might describe the current state of play for an ongoing project, it
should identify this as a snapshot taken at a certain time. Likewise, it
may express the opinions of individuals (usually the author), but these
should be identified as opinions.

For dynamic information following the progress of a project, an issue
tracker such as Jira is usually more appropriate.

### Crosslinking

A primary feature of wiki systems such as Confluence and MediaWiki –
indeed, the draw of the WWW in the first place – is the ability to link
between pages. This should be taken advantage of as much as possible.

Where links fit well into the main text, they should be used. Where
possible, the link text should not significantly differ from the title
of the page being linked to. Nobody googles for 'click here'.

On Wikipedia, the 'See also' section at the bottom of articles is a
place for linking to related articles that do not fit into the main
text. The [Manual of Style](https://en.wikipedia.org/wiki/Wikipedia:Manual_of_Style/Layout#%22See_also%22_section)
has guidelines of what should be included in such a section.

Crosslinking is not restricted to pages within a wiki.  Confluence has
good support for linking to Jira tickets, and _vice versa_. Jira has the
concept of linking tickets to one another.  Maintaining these links
makes it much easier for a user to find related information from one
page.

Backlinking can be useful. MediaWiki offers a search for pages that link
to a specific article (example:
[https://en.wikipedia.org/wiki/Special:WhatLinksHere/Coffee](https://en.wikipedia.org/wiki/Special:WhatLinksHere/Coffee)),
but backlinking needs to be done manually in Confluence.

### Permalinking

Web resources are fickle. External websites may change their layouts
without properly redirecting; or change the contents of pages without
notice.

To avoid [link rot](https://en.wikipedia.org/wiki/Link_rot), consider
linking to an archived version of a webpage using the [Wayback
Machine](https://web.archive.org/), rather than directly linking to the
URL.

Most academic publications have a DOI
([Wikipedia](https://en.wikipedia.org/wiki/Digital_object_identifier))
which uniquely identifies the publication. The DOI link (example:
[https://doi.org/10.1017/jfm.2018.2](https://doi.org/10.1017/jfm.2018.2))
is succinct and will redirect to the page for the journal article, even
if the journal website changes its layout and breaks a URL. Many
bibliography management software are able to automatically extract
information about an article, such as title, author, journal and year,
from a DOI.

Websites such as Wikipedia and Stack Overflow, where information is
mutable by design, have 'permalinks' that allow a link to a specific
version of the content (example:
[https://stackoverflow.com/a/7969052/](https://stackoverflow.com/a/7969052/)).

### Dating

Statements written in the present tense or using relative time
references soon become obsolete. They should be tagged with the date
on which the statement was made or last checked:

> Our progress is hindered by the availability of the lab, which is
> currently being repaired after recent explosions _(2022-05-16)_.

or with an issue tracker reference:

> We are blocked until we fix the tests in MagicSoftware (MT-1337).

provided that the ticket MT-1337 contains relevant information.

This can be sidestepped by referring to version numbers:

> As of v1.5, TheBestIDE is incapable of detecting unmatched brackets.


### Page notices

Page notices go at the top of the page and should indicate the
maintenance state of the page. They should usually include a date.

Examples:

> This page is work in progress. _(2022-05-16)_

> This page was last updated for DeepMagic v3.6.3. _(2022-05-16)_

> The information on this page is obsolete. Please refer instead to
> __This Other Page__. _(2022-05-16)_

Page notices should usually be concerned with the page itself, not with
the thing being described in the page. A page on MagicSoftware should
not have a notice like this:

> ~~MagicSoftware is obsolete. _(2022-05-16)_~~

That should instead be given in the main text – preferably with links to
relevant issue trackers. However, the following may be appropriate:

> This page describes a project in active development, and may change
> rapidly asthe project progresses. _(2022-05-16)_

Depending on the wiki software, it may be possible to standardize these
notices as templates, as well as to search for pages with a particular
tag.


### Categorization and hierarchy

MediaWiki supports filing articles into categories (_e.g._
https://en.wikipedia.org/wiki/Category:Coffee). Using this filing system
helps readers find articles related to the topic that they seek, even
though the pages may not directly link to each other.

Categories are a many-to-many relationship (each page may be in multiple
categories), and categories can be nested.

'Meta' categories that describe the state of a page can also be useful.

Confluence offers poor support for page organization. Each page belongs
to a workspace and is either a root page in that workspace or belongs to
exactly one parent page. Parent pages should give an overview of the
topic but leave details to the child pages. Child pages can be used to
contain supplementary information that does not fit into the main text
of the parent.
