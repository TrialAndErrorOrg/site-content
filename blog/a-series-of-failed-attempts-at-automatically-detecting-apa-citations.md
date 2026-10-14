---
title: A Series of Failed Attempts at Automatically Detecting APA Citations
date: 2026-10-14T18:32:00
authors:
  - jorna
category: behind-the-scenes
tags:
  - doi
  - typesetting
excerpt: |-
  Or: An Old Fool Reminisces About a Younger Fool's Attempts at Automatic Citation Conversion.
  What if you could automatically turn bad data into good data? *EXTREMELY LOUD INCORRECT BUZZER*
image: /uploads/blog/pasted-image-1791995521182.webp
imageAlt: ''
doi: ''
related: []
description: ''
---

_In "Behind the Scenes" members of JOTE's staff give some insight on what running an Open Access Journal entails._

Until the venerable Jip Prinsen joined JOTE, I used to be the unfortunate masochist-in-chief in charge of turning JOTE's articles from the unstructured Word documents we receive from authors into nice(-ish) typeset PDFs that exude that kind "yeah we know what we're doing" vibe we strive for as a Journal. As the head (and only) typesetter, I had to come up with all the systems, templates, and (partially) processes we now, for better or for worse, still use to produce your \~bi-annual serving of failed experiments. After recently giving our current conversion pipeline some MUCH needed T&C, I stumbled upon some _very_ old code of mine, which made me want to reminisce about a time which feels long past, where I still had some fun learning about how to ~~make my (and JOTE's) life more complicated~~ optimize/automate JOTE's production process. There are many things I regret in setting up and running those processes. I may get around to writing about more of them in the future after I have processed said failures by fixing them first and being able to present myself as someone slightly more responsible.

I want to talk about one of said failed experiments (get it??) which _has_ already been corrected for some time now namely **Automatic Conversion of unstructed Inline APA citations into structured citations**. That sounds a bit abstract, so it's easier to start at the end-goal of all that sentence. 

(For context: JOTE's typeset articles were, and still are, created by some custom conversion process of turning a submitted and copyedited Word document to LaTeX, after which it's corrected/made pretty by a human being.

## Biblatex, an interesting proposition

Anyone who has done any academic work in LaTeX that required any kind of citation is familiar with LaTeX useful citation system. I'm not that up to date on LaTeX anymore (thank ~~god~~ Jip), but last I checked the most reasonable solution to doing citations in LaTeX was `biblatex`. `biblatex` is nice because it allows you to do something like the following

```plain
\documentclass{article}
% this says: I want to use biblatex and do citation stuff
\usepackage[backend=biber,style=apa]{biblatex}

% this says: use the citaitons i have defined in this file
\addbibresource{bibliography.bib}

% usually you add a bibliography as separate files, but i've put them in here
% for ease of reading
\begin{filecontents}{bibliography.bib}
@article{JornaWise,
    title       = {An Mildly Plausible Idea},
    author      = {Jorna, T. F. K. and Gaillard, Stefan and Field, Sarahanne},
    url         = {https://blog.trialanderror.org/xxx},
    date        = {2026},
    number      = {1},
    volume      = {7},
    url         = {https://doi.org/10.36850/why-do-you-even-try},
    doi         = {10.36850/why-do-you-even-try},
    date        = {2026-09-30},
    journal     = {Journal of Relaxation and Success}
}

@article{JornaStupider,
    title       = {A Worse Idea},
    author      = {Jorna, T. F. K.},
    url         = {https://blog.trialanderror.org/xxx},
    date        = {2025-09-30},
    journal     = {Journal of Trial and Error}
}

@article{JornaStupid,
    title       = {A Bad Idea},
    author      = {Jorna, T. F. K.},
    url         = {https://blog.trialanderror.org/xxx},
    date        = {2025-09-29},
    journal     = {Journal of Trial and Error}
}
\end{filecontents}


\begin{document}

\noindent Following the experiments put forth by \textcites{JornaStupid}{JornaStupider}, we ended up doing the complete opposite of what those buffoons were suggesting \parencite{JornaWise}.

\printbibliography
\end{document}
```

Which produces this:

![file-9c74b042901d8aac4efc95ef8c9c38f3.png](https://cms.trialanderror.org/uploads/file_9c74b042901d8aac4efc95ef8c9c38f3_e8b43b2cac.png)

That's really nice! Look at all the things `biblatex` did for us here:

- It automatically disambiguated the same-author-same-name citations _Jorna 2025a_ and _Jorna 2025b_.
- It automatically truncated inline citation of the three authors of _Jorna 2026_ into _(Jorna et al., 2026)_
- It correctly prints the references in the most perfect APA.

That's a lot to like from a citation system! What's more, if you add a single extra line to the beginning of the LaTeX document

```plain
\usepackage[pagebackref=true]{hyperref}
```

we get the even nicer behavior of being able to click and jump to the full reference (and back!) in the PDF.

![jump-citation.gif](https://cms.trialanderror.org/uploads/jump_citation_30ee3b78c8.gif)

Isn't that nice? Relatedly, when we import our `.tex` files into our publishing platform [PubPub](https://pubpub.org), we get these nice previews of the full citation.  

![image.png](https://cms.trialanderror.org/uploads/image_f343845049.png)

## Hell is other authors

After _definitely_ conferring with the rest of the team about the long term viability of this, I decided that JOTE, in order to be taken seriously as a fledgling journal, simply _must_ have these structured/linked citations.

The issues however, are obvious (to me):

- The above requires typesetting our PDF in LaTeX and some correct use of `biblatex`
- Authors do not write their submissions in LaTeX but in Word. Often without any kind of citation management system to begin with, which means there is no structured citation information at all.
- I did not want to spend a lot of time per article manually converting citations into `biblatex` style.

The solution is of course obvious as well.

- ~~Accept that it may not be possible~~ Automate that shit.

The rest of this article is a recollection of the methods I've tried over the years, and what we do now.

## Manual-ish

The method I relied on at the beginning, unclouded by foolish notions of automating the entire typesetting process, was extremely simple:

1. Put all of the references at the end of the submission in a [Zotero](https://www.zotero.org/) collection. Zotero does a pretty good job of fetching the additional metadata for each reference, especially since most authors correctly included the DOI to each reference.
2. Export the references to a `biblatex` bibliography `.bib` file using the venerable [Zotero Bibtex](https://retorque.re/zotero-better-bibtex/).
3. Convert the plain Word document into an okay first-pass LaTeX document by... just importing the Word document into PubPub and then exporting it as a LaTeX document (PubPub uses [Pandoc](https://pandoc.org/) for this). Not really usable but it's something!
4. Manually change each inline APA citation `(Jorna, 2026)` to the correct `biblatex` command `\parencite{Jorna2026}` . 

Obviously, this took quite some time. "Luckily" this was during the start of the COVID-19 pandemic, so I had some time on my hands (my other main commitment at the time was my master's thesis, which..... I don't think I'm ready to talk about yet). 

This whole trick hinges on the _keys_ of the `.bib` exported from Zotero being named like `Jorna2026`. If they we're e.g. called `JornaWiser`, like I called it in the beginning, this process wouldn't work, because we'd have now way of telling what the key in the bibliography will look like.) However, already after our [first article](https://journal.trialanderror.org/pub/alcoholcues/release/11) I was itching for an easier way to do this, the automation-siren was calling my name. We will go into how I tried in vain over many years to do so, but first...

## APA is pretty hard

At this point we really need to talk about the [American Psychological Association's](https://apastyle.apa.org/style-grammar-guidelines/citations)inline citation style guide, specifically the 7th revision, referred to as APA 7. This is the style guide JOTE has been using since its inception. We are mostly concerned with _inline_ citations, rather than formatting the full reference, as `biblatex` can handle the latter part quite well. Our (self-imposed) job is to find a way to turn normal, in-line citations into structured ones.

There are two types: parenthetical and narrative. In the following I will **bold** the parts we want to match to create the key, and underline other relevant parts of the text we need.

#### Parenthetical

Appear after the part of the sentence the text references. Such as 

> This has shown to be patently untrue (**Jorna**, **2025**). `\parencite{Jorna2025}`

`\parencite` is the `biblatex` command to say "insert a parenthetical citation for the bibliography entry Jorna2025".

This is by far the most common kind of in-line citation: it doesn't require a lot of work by the author to write the text, it gets out of the way, it looks nice and professional_._ A good case to study we will use for the rest of this section is [Abma (2020)](https://journal.trialanderror.org/pub/experimentfail/release/10) (ironically I couldn't use a parenthetical citation there because I am directly referring to the article itself, rather than referring to some information in said article).

The article in question we call a "Reflection", which is one the ingredients in JOTE's special journal sauce. A reflection is written by a third party (usually a philosophor of science or meta-scientist) on one of our "normal" submissions, which we call "Empirical", and usually dives deeper into the broader context of what "went wrong." Our goal initially was to have _every_ Empirical article be paired with such a Reflection, which we did not manage to hit for a single issue.

Since it's an article written about a specific other article, you will of course want to refer to specific passages in the text. Abma does so within the first paragraph (not counting the abstract)

> and 15 are “aggression-related words of a sexual nature” (**Leboeuf** et al., **2020**, p. 10).`\parencite[p. 10]{Leboeuf2020}`

Suffixes are pretty easy to do in `biblatex` . So far so good. However, you can also do prefixes!

> in the area of priming research (see **Derksen**, **2017**).`\parencite[see][]{Derksen2017}`. 

Why do you need this empty `[]` ?? Well, otherwise, as we saw earlier, LaTeX (or `biblatex` to be specific) would interpret it as a suffix instead. Which, you know, I understand, but what I _don't_ understand is why it `\parencite[see][]` instead of `\parencite[][see]`. In normal programming languages, it's common to have a function be able to take multiple arguments, but normally the _order_ of the arguments stays consistent. So if `\parencite[<suffix>]` would produce a suffix, you would _think_ that the rule would be, if a bit confusing, `\parencite[<suffix>][<prefix>]`. But of course, you are thinking like you aren't building on top of a decades high tower of hacks! Of _course_ original author of the `biblatex` decided that the order of the arguments changes when you have more than one, so instead of `[<suffix>]` , it's now `[<prefix>][<suffix>]`. Which makes some kind of sense of course, you want the prefix to come after the suffix, but the suffix only case makes this terribly confusing!

_slow exhale_

Furthermore, you are allowed to combine multiple citations together! Isn't that fun. See here a beautiful such specimen from the same article

> the experiment was conducted (as elderly people are supposed to do; **Bargh** et al., **1996**; **Derksen**, **2017**, pp. 183-188).`\parencite`**`s`**`[as elderly people are supposed to do][]{Bargh1996}[pp. 183-188]{Derksen2017}`

Oof. We got our work cut out for us! And we haven't even gotten to the hard part yet!

### Narrative citations

To make things even more complicated, APA allows a second form of inline citing, namely by incorporating the citation in the text. These look like such

> in addition to these three varieties of reproduction type, **Radder (1996)** distinguishes four possible types`in addition to these three varieties of reproduction type, \textcite{Radder1996} distinguishes four possible types`

This already looks trickier than the parenthetical case, as these citations look much more similar to their surroundings.

You can also do suffixes in narrative citations.

> or, as **Radder** (**1996**, p. 84) puts it:`or, as \textcite[p. 84]{Radder1996} puts it:`

There's luckily no need for us to distinguish prefixes and multi-cites in narrative citations. Prefixes are just, part of the surrounding text, and therefore do not need special handling. There is one case we need to think of regarding multi-citations, and that is citing multiple works by the same author. This is a contrived example because thankfully no actual author of us tried to pull this:

> as noted in **Jorna** (**2025**, a decent one; **2026**; **2099**)`as noted in \textcites[a decent one]{Jorna2025}{Jorna2026}{Jorna2099}`

We only need to worry about the _same_ author case rather than the multi-author case, because multiple inline narrative citations are just the same as... multiple inline narrative citations. I actually found a _single_ use of this in our archives, in [Bennet (2023)](https://journal.trialanderror.org/pub/manifesto-rewarding-recognizing/release/1)

> see similar recommendations by **Teperek** et al. (**2022**) and **UKRI-Research England** (**2022**)`see similar recommendations by \textcites{Teperek2022}{UKRI-ResearchEngland2022}`

While neat that `biblatex` is able to turn `\textcites{A2020}{B2021}` into "A (2021) and B (2021)", this is equivalent to `\textcite{A2020} and \textcite{B2021}`.  Therefore we don't actually need to think about this.

### Other fun stuff

As a final note: APA also requires you to handle the following miscellaneous cases

- You can refer to non/not-yet published text, eg `the thesis he never finished (Jorna; n.d.)`, n.d. meaning No Date. Other common ones are `forthcoming`, `in press`, `tucked away in a drawer somewhere` , or  `in review`
- Works by the same author in the same year `the initial article, Jorna (2020a), was retracted and a correction was issued (Jorna, 2020b)`
- Authors with the same surnames require initials `(T. F. K. Jorna, n.d.) while his brother did finish his Master's (F. J. Jorna, 2022)`. 
 
So: how to do we ~~automate ourselves out of job~~ ~~burden ourselves with mountains of tech dept chaining you to a life of servitude~~ solve this nifty problem gang!!

## Stratum 1: Simple Regexes

Back then, before I grew wiser, I thought RegEx was _the_ shit, and came up with a bunch of handy shortcuts over the course of the articles in the first issue to speed up this tedious task.

Luckily they were still there in the abandoned Overleaf project of our first issue or two (no need to look in detail, I will go over an example)

```plain
### Author Year citations (APA)
##Find things like (Blah et al., 2019)
#Find :
\(([A-Z]\w*)[^,]*\, (\d\d\d\d)\)
#Replace: 
\parencite{$1$2}

```

For you unenlightened ones blissfully unaware of the power of Regular Expressions (RegEx, either pronounced as `/ˈɹɛɡˌɛks/` (rehg-ex) or `/ˈɹɛd͡ʒˌɛks/` (redz-yex) [1]), these arcane inscriptions allow you to specify

1. a pattern to search for (match) in the text
2. what to replace it with

Let us look at the first one in too much depth:

The expression \``\(([A-Z]\w*)[^,]*\, (\d\d\d\d)\)`\` does is it matches 

- `\(` : just an opening parenthesis, as we are matching parenthetical APA citations here. We need the `\` in front of it to not confuse it with the "match a group" syntax. This matches `(` (wow)
- `([A-Z]\w*)`: A capital letter A to Z (`[A-Z]`, followed by any number( `*` means "0 or more") of word characters `\w`  (letters, numbers, hyphens). They are enclosed in `()` which turns them into a captured group. This matches `Jorna`
- `[^,]*\,`: match any number (`*`) of "anything that's not a comma" (`[^,]`), then match a comma `,` . This matches ` et al.,`.
- `(\d\d\d\d)`: match a space `   ` (after the comma), then 4 numbers `\d` as a group again. This matches ` 2026`
- `\)`: final closing parenthesis. This matches `\)`

Putting it all together we have  matched

```plain
|(<Jorna> et al.|, <2026>)|
```

Now we just have to apply the, much more simple, substitution rule,

```plain
\parencite{$1$2}
```

Which will replace the entirety of the matched text with 

- `\parencite{` (the double `\` was needed to say "this is an actual backslash!")
- the content of the first group, `Jorna`
- the content of the second group, `2026`
- `}`

Here's the full result in one go using [regex101.com](https://regex101.com/substitution?regex=%5C%28%28%5BA-Z%5D%5Cw*%29%5B%5E%2C%5D*%5C%2C+%28%5Cd%5Cd%5Cd%5Cd%29%5C%29&testString=...what+those+buffoons+were+suggesting+%28Jorna+et+al.%2C+2026%29.%0A&flags=gm&flavor=javascript&delimiter=%2F&substitution=%5C%5Cparencite%7B%241%242%7D) (if you click the link you can see this example worked out). 

![regex101-1.png](https://cms.trialanderror.org/uploads/regex101_1_de6fcf54ef.png)

Pretty neat huh? With a couple such find-and-replaces I could go through the whole document in couple handfuls of minutes, which left some cleanup.

There were two other entries in my file

```plain
## Find things like: (Blah et al., 2009; Pieter, 2010 ....)
## Need to find a way to generalize this
#Find 
\(([A-Z]\w*)[^,]*\, (\d\d\d\d)\; (\w+)[^,]*\, (\d\d\d\d)\;\ (\w+)[^,]*\, (\d\d\d\d)\;\ (\w+)[^,]*\, (\d\d\d\d)\)
#Replace
\parencite{$1$2, $3$4, $5$6, $7$8}

##Inline: Lebeouf et al. (2020) -> Lebeouf2020
#Find: 
([A-Z]\w+)\ (?:[\w\.]*? )?\((\d\d\d\d)\)
#Replace: 
\textcite{$1$2}
```

These "handle" multi-parenthetical citations and narrative citations respectively.

The problem is of course that these does not handle

- Prefixes
- Suffixes
- More complex multi-citations
- Other evil combinations
- bell hooks
- The Dutch and the Spanish

The Dutch and the Spanish? 

### The Dutch and the Spanish

Let's look at that narrative regex

```plain
([A-Z]\w+)\ (?:[a-z \.]*? )?\((\d\d\d\d)\)
```

This tries to find an in-text narrative citation by looking for "something starting with a Capital letter, followed _maybe_ by some other junk (eg "et al.,) followed by a parenthesized 4-digit number". While of course not taking into account suffixes or more complex combinations, it does correctly work for basic-ass single word surnames like my own, but falls apart with more _custom_ last names as found with extremely common Dutch surnames, which start with some lowercased terms like `van de(r)` (meaning _of the_), as in one of the founders of [JOTE Martijn van der Meer (Martin of the Lake?)](https://blog.trialanderror.org/author/van-der-meer) or just plain `de`,  as in "de Jong" or "de Vries". 

It also fails with _another_ of [JOTE's co-founders (although he hates that term), Max Bautista Perpinya ](https://blog.trialanderror.org/author/bautista-perpinya), as of course the last name is actually _two_ different Capitalized Names!

![image.png](https://cms.trialanderror.org/uploads/image_2ffde4137b.png)

As you can see this leads to some gnarly results. For one, `Meer2020` would likely not match, nor would `Bautista2021`, as the keys would likely be called `vanderMeer2020` and `BautistaPerpinya2021`. The output of the LaTeX would be 

> I can't believe Jorna (2026) created such a convoluted system. Other culprits include **van der** van der Meer (2020) and **Bautista** Bautista Perpinya (2021)

Extremely messed up. 

So we should give up right.
 
## Stratum 2: Going further

Around this time, which must have been circa 2021, I started getting _ideas._ What if we could completely automate the typesetting process? While I initially liked learning and using LaTeX, I was also lazy and definitely did not want to spend more time than needed. I still had a thesis to write!

Around this time we also started a collaboration with the University of Tilburg to publish some of their books. Exciting! But more work, of course we needed to make that go faster!

While I won't go into detail in all the iterations of automations I've done over the years, it started with a very shoddy `.js` script (I wasn't even using Typescript back then! so cute). However, in that `.js` script lay the reason I wanted to write this blog post to begin with. In some ways some of my proudest work I've produced, just because of how insane it looks to me now!

Behold: 

```plain
// if you want to unedrstand how these things find things, plug them into
// regexr.com. Just trust me tho
// get rid of ands, make them \&s. Easier to parse

  /([A-Z]\w+,? )and( [A-Z][\w\u00E9\']+,?) (\(?\d{4}\)?)/gm

// find the ; Guy, 2021) part of a (Dude, 1990; Guy, 2021) citation

  /(?<=(\d ?))(;|,) ([A-Z]\. ?)*(([A-Z][\w\u00E9]*)((?:( |-|\'))[A-Z][\w\u00E9]*)*)((, (([A-Z][\w\u00E9]*)((?:( |-|\'))[A-Z][\w\u00E9]*)*)(?=,)))*(,? \\& )?(?<=\13)(([A-Z][\w\u00E9]*)((?:( |-|\'))[A-Z][\w\u00E9]*)*)?( et al\.)?,? (\d{4}|\d+ [A-Z\.]+),? ?(p+\. ?\d+ *)?(\))?/gm

// find the (Dude 1990; part. Handles A lot, such as (see Dude 1990), (ASHT; Dude, 1990), (Dude, 1990, pp.234)
// (D'Ude McOnnell 1990)

  /\(([\w\.\- ]*)(?=; ?)?(; ?)?([a-z, ]*)? ?(([A-Z]\. ?)*(([A-Z][\w\u00E9]*)((?:( |-|\'))[A-Z][\w\u00E9]*)*)((, (([A-Z][\w\u00E9]*)((?:( |-|\'))[A-Z][\w\u00E9]*)*)(?=,)))*(,? \\& )?(?<=\14)(([A-Z][\w\u00E9]*)((?:( |-|\'))[A-Z][\w\u00E9]*)*)?( et al\.)?),? (\d{4}|\d+ ?[A-Z\.]+)(\/\d{4})?( ?\{\[\}.+?\{\]\})?([,\:] ([^\;\,\)]*))?(\;|\))/gms

// Find in text citations. This one fucks up the most, because there's no way to determine whether to
// take the "Whereas" in "Whereas Bentham (1688) says you are evil" as a name or not, because the
// spanish have names like Bautista Perpinya. That's why there is the spanish toggle.

  /([A-Z]\. ?)*((([A-Z][\w\u00E9]*)((?:( |-|\'))[A-Z][\w\u00E9]*)*)((, (([A-Z][\w\u00E9]*)((?:( |-|\'))[A-Z][\w\u00E9]*)*)(?=,)))*(,? \\& )?(?<=\12)(([A-Z][\w\u00E9]*)((?:( |-|\'))[A-Z][\w\u00E9]*)*)?( et al\.)?)(\'s)? \(((\d{4})|(\d+) ([A-Z\.]+))(\/\d{4})?( ?\{\[\}.+?\{\]\})?([,\:] ([^\;\,\)]*))?\)/gms
```

I won't go into too much detail for these, as I too no longer understand these glyphs. However, plugging them [into regex101](https://regex101.com/substitution?regex=%28%5BA-Z%5D%5C.+%3F%29*%28%28%28%5BA-Z%5D%5B%5Cw%5Cu00E9%5D*%29%28%28%3F%3A%28+%7C-%7C%5C%27%29%29%5BA-Z%5D%5B%5Cw%5Cu00E9%5D*%29*%29%28%28%2C+%28%28%5BA-Z%5D%5B%5Cw%5Cu00E9%5D*%29%28%28%3F%3A%28+%7C-%7C%5C%27%29%0A+++++++%E2%94%82+%29%5BA-Z%5D%5B%5Cw%5Cu00E9%5D*%29*%29%28%3F%3D%2C%29%29%29*%28%2C%3F+%5C%5C%5C%26+%29%3F%28%3F%3C%3D%5C12%29%28%28%5BA-Z%5D%5B%5Cw%5Cu00E9%5D*%29%28%28%3F%3A%28+%7C-%7C%5C%27%29%29%5BA-Z%5D%5B%5Cw%5Cu00E9%5D*%29*%29%3F%28+et+al%0A+++++++%E2%94%82+%5C.%29%3F%29%28%5C%27s%29%3F+%5C%28%28%28%5Cd%7B4%7D%29%7C%28%5Cd%2B%29+%28%5BA-Z%5C.%5D%2B%29%29%28%5C%2F%5Cd%7B4%7D%29%3F%28+%3F%5C%7B%5C%5B%5C%7D.%2B%3F%5C%7B%5C%5D%5C%7D%29%3F%28%5B%2C%5C%3A%5D+%28%5B%5E%5C%3B%5C%2C%5C%29%5D*%29%29%3F%5C%29&testString=...I+can%27t+believe+Jorna+et+al.+%282020%29+created+such+a+convoluted+system.+Other+culprits+include+van+der+Meer+%282020%2C+2021%29+and+Bautista+Perpinya+%282021%29%0A&flags=gsm&flavor=javascript&delimiter=%2F&substitution=%5C%5Ctextcite%7B%241%242%7D) again...

![file-7064ff2aed1a3d6aeec03c0a74572475.png](https://cms.trialanderror.org/uploads/file_7064ff2aed1a3d6aeec03c0a74572475_22bee5cf51.png)

...

It doesn't even fucking work! Sure it handles the Spanish, but not the Dutch, and seemingly I left it in such a broken state that a simple `Jorna et al. (2020)` isn't even caught. I'm very disappointed in you mister.

Anyway, the parenthetical part _did_ work, but it need to be done iteratively over chunks.

This did mark the end of my year-long fling with "trying to regex my way out of a job." Shockingly, I learned what wiser ones likely already knew, namely that Regex isn't a parser.

So, obviously that was the next step: a parser.

### A parser

I won't go into too much detail, as, like always, I've ran out of steam going into too much detail above.

After typesetting my first book, [Second Thoughts](https://openpress.tilburguniversity.edu/projects/second-thoughts), for then-fledgling Tilburg University's Open Press using this convoluted `.js` script, I thought it was time to take a more structured approach to things.

I had fallen in love with the [unified,js](https://unifiedjs.com/) ecosystem of parsers/converters and wanted to try my hand at that.  I wrote a [whole bunch of those ](https://github.com/TrialAndErrorOrg/parsers)little packages dealing with converting from Word and to LaTeX.

Of course I hadn't yet learned my lesson, and _still_ had a thesis to procrastinate on, so I tried my hand at writing a parser for this using [nearley.js](https://nearley.js.org/). Not having an _actual_ CS education (only half of a minor), I of course didn't know the first thing about writing parsers.

The difference between the regex and parser approach is roughly as follows:

- With regex, you are trying to write _one_ big-ass magic word that applies to every permutation of your desired match.
- With a parser, the idea is to divide your text into valid and invalid chunks, and have specific rules as to how those chunks are allowed to follow each other.

Anyway: this sort of worked! This last iteration was the result of some complex cases in [Bennet (2023) ](https://journal.trialanderror.org/pub/manifesto-rewarding-recognizing/release/1). Stuff like `(Bossu & Brown, 2018; McInturff & Adenis, 2022)` it handled fine!

#### The Spanish, again

However, I realized at this point I could likely _never_ get a _perfect_ APA parser (even given perfect input!), because of, again, the Spanish.

While the number of Dutch surname prefixes is somewhat finite, there's no such limit to Spanish imagination. As a comment above already points out, it's not possible to fully disambiguate between a Spanish double-name and... the start of a sentence.

Consider the following examples
 

> Whereas Bentham (1688) says you are evil...Bautista Perpinya (2024) suggests you may be of poor character...

Without having a completely accurate "surname from any-other-word" discerner, I had sadly no way of knowing  that `Whereas` is just a capitalized word at the start of a sentence, and `Bautista Perpinya` was a person's last name.

Despite this set-back, we continued to use this system to some effect. However...
 
## Holes

Around 2023 is when I started more seriously avoiding writing my thesis by taking on a(nother) half-time job at PubPub (hey remember them from earlier?). This left me with less time to think about the Spanish, so I was content with slightly worse parsing for now.

However, at this time the _other_ end of the pipeline really started to show it's edges. Remember when I mentioned the "putting stuff in Zotero and then getting a `.bib`" part of the pipeline \~2500 words ago? Yeah me neither, but I had stopped doing that for a while. Manually doing actions? I only have time to create bad automations! So instead I was hoping to get correct structured _references_ (we had only been talking about citations before) as well from the raw Word doc. Luckily someone else had already thought of that.

[Anystyle](https://anystyle.io/) is a Ruby gem (package) that takes unstructured references and parses them into `biblatex` suitable output. 

For instance, it can turn

> Whitchurch, C. (2008). Shifting Identities and Blurring Boundaries: the Emergence of Third Space Professionals in UK Higher Education. Higher Education Quarterly, 62(4), 377–396. https://doi.org/10.1111/j.1468-2273.2008.00387.x

Into

```plain
@article{whitchurch2008a,
  author = {Whitchurch, C.},
  date = {2008},
  title = {Shifting Identities and Blurring Boundaries: the Emergence of Third Space Professionals in UK Higher Education},
  volume = {62},
  pages = {377–396},
  url = {https://doi.org/10.1111/j.1468-2273.2008.00387.x},
  doi = {10.1111/j.1468-2273.2008.00387.x},
  language = {en},
  journal = {Higher Education Quarterly},
  number = {4}
}
```

Nifty! 

So nifty I thought it was usable for everything! The issue with this approach is obvious however: authors do not present perfectly formatted APA references! Tragedy. We sadly have a bit of a "garbage-in-garbage-out" situation: while `biblatex` can format a reference list like nobodies business, it sadly can only format it as well as the data you input. So this _still_ required a lot of manual cleaning.

I tried to get around this by doing a little trick, namely: parse all the data with anystyle, then simultaneously _also_ search for all the citations using [Crossref's API](https://api.crossref.org/swagger-ui/index.html). Then do a lil comparison to see if it's roughly the same data, and then merge it?

Yeah, that worked about as badly as you'd imagine.
 
## Stratum 3: Accepting defeat

Around this time (mid-late 2023) I had been trying to onboard a good friend of mine to possibly help with the typesetting work. As you may realize now as well... this shit is a mess. While I could deal with the mess I made, explaining it to someone else and expecting them to roll with the mess-punches as well, not so great.

But still, we kept using it, all the way until 2024. At that time I was getting _really_ squeezed for time with now full-time working at PubPub and still, against all odds full-time avoiding my thesis. We onboarded another typesetter, the venerable Jip Prinsen I mentioned earlier, who is now the lead (and only) typesetter/production editor at JOTE.

After a while, we did start to realize that the amount of corrections we needed to make in the junk we got out of anystyle was no longer worth the time. We eventually just went back to the old fashioned way: having our extremely capable copy-editing team lead by Alex Visser just manually correct the references. This is what they had been doing all along, and my cursed anystyle/parsing pipeline introduced _more_ errors into their already corrected work. 

At that point it was time to hang up the hat, and JOTE articles published after 2024 no longer have the fun little hover links on the site, or the "jump to reference" links in the PDF. Alas. 

## Take Home Message

~~Never try anything ever!~~

No, that's not it. I've learned to be _slightly_ more discerning in what is worth a multi-month optimization pass. And that it's _probably_ more important to discuss this with the relevant people (stakeholders, if you will) first.

While I don't regret this little part of my adventure at JOTE, as I _did_ learn a lot, I do look back and wonder whether spending 5% of this energy on reaching out to people who would be interested in working with/on the journal might not have had a better return on investment.

Ah well. At least I got a blog post out of it.
 
 
### Mandatory AI section

_A bUt cOUldNT yoU uSE an LLM_ 

Yeah probably! But like I said, around the time those started popping up, I was extremely busy with not writing my thesis you see. 

Who knows, maybe we'll have another crack at it. If someone reading this feels the desire to burn some tokens on it, send patches!
 
 
 
 
[1]: [https://en.wiktionary.org/wiki/regex](https://en.wiktionary.org/wiki/regex)
