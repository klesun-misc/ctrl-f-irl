## find-source.codes

#### (Github Crawler for Code Fragment Search)

See https://stackoverflow.com/questions/13321936/code-search-on-github-com

I always get sad when I need to find the source code given a known class name or a function name and fail because google does not index github code,
and github itself only allows a [limited search](https://docs.github.com/en/github/searching-for-information-on-github/searching-code) by file name.

The idea from user perspective should be dead simlpe: you open a webpage, there is a text area input - you paste code fragment, hit "Search" and get
the list of repositories that have this code occurences.

Will need to implement a crawler for that:
- Iteratively collect a list of repositories, most popular to be processed first
    - (https://github.com/search/advanced should greatly assist in that. When done with super popular repos, can iteratively get less popular by selecting small date windows)
    - (https://pygithub.readthedocs.io/en/latest/github.html#github.MainClass.Github.get_repos may come in handy as well)
    - I think it will make sense to start with something like 100k+ stars, fetch all repos, then make it 50k+ stars, do same, then 25k+ stars and so on. There will be repetitions after each threshold decrease, but that's ok, it's the only way I can think of to guarantee that between iterations some repo does not get a star and moves from 999 to 1000 that we already processed. And they will grow exponentially in number while we move down anyway.
    - We should also specify some "max creation date" and stick to it, saving for each processed group so that we could get newly appearing repos without re-indexing _everything_ again.
- Clone each of them (shallow of course and probably better via vpn), collect text files of less than `X` KiB (for example 300, it's about 10k lines of code) to exclude minified `/dist/` files and stuff. Files above `X` size should be stored as name list somewhere for analyzing and adjusting the `X` value.
    - ([sparse-checkout](https://stackoverflow.com/a/13738951/2750743) may be extremely useful here, as many repos tend to include large binary files we'd better not download to not get banned by github for traffic)
    - (and [this](https://stackoverflow.com/questions/6119956/how-to-determine-if-git-handles-a-file-as-binary-or-as-text) may be useful for filtering out binary files right on the clone step)
    - Maybe apply some heuristics, like instantly skipping files with extensions like `.png`, `.jpg`, etc... Also probably skip `/out/`, `/dist/`, `/bin/` directories and files ending with `.min.js`, `.bundle.js`, `.vendors.js` - would be nice to look at all options we'll have before that though.
- Build an index for exact text search. Most full-text search engines I managed to google seem to specialize on word search, though I believe elasticsearch provided an option to search by exact text. I remember we were working on an optimized text search in progmeistars, maybe that will help: https://github.com/klesun/Progmeistars_tasks/tree/master/SpecB_C-Data_SergeyIlich/g12-substring-partial-match. There was some simple, but effective concept, though it probably had to scan whole text file, so a 3rd party engine should be still beter...
    - https://www.sqlitetutorial.net/sqlite-full-text-search/ maybe can just use sqlite `FTS()`
- Implement the API for the web page to communicate with the text index engine


### Use Cases
- Finding sources from generated/minified code, like decoded `.jar`, generated `.d.ts` in `/node_modules/`, etc...
- Finding sources of code posted in Stack Overflow answers
- Finding sources of error messages that you get either when library throws an exception, or in your operating system dialog, or such...
- Finding comments and documentation related to a piece of code, usage examples
- Exposing these files for google robot (at least for popular repos with 500+ stars) - very good SEO + karma. Possibly it would even become the main usage for the web app: not being used directly, but just contributing to google.
- Finding real world examples of some API usages by searching stuff like `require('some-lib-with-bad-readme')` or `new SomeHardToGoogleLibClass()`
- ...

I'm really surprised google did not make this possible, this is, like, super useful, I did not even thought about how much when I started writing this doc...

Upd.: I tried googling code parts from some popular projects with 1000+ stars, like Mono, linux kernel, and google did find them, even though it did not work for special characters intense text queries, like curly brackets, parentheses, etc... Though it did not find code fragment from my 198 stars repo. Guess google relies on it's usual number of links algorithm when it decides what should be prioritized while indexing. So the few advantages of this tool over google will be:
- coverage: while I can't be positive that _all_ repos will be covered, at least 100+ or maybe even 20+ stars should be I think.
- text match precision: exact text search including non-word characters
- results precision: won't include random irrelevant hosters of the code in question including countless code analysis tools and forums
- results are ordered by stars rather than search relevance (since _everything_ is relevant as it is exact text search)
- small quality-of-life features specific for source file search
    - ability to show text in question right away without redirect (as will have to store it anyway for contributing SEO)
    - show some metadata (commit sha5, md5 of file contents with duplicates grouped)
    - link to specific line in github file viewer

Domain options:
- code-fragment-search.site
- code-search.run
- code-search.world
- code-search.app
- code-search.org
- search-source.codes
- find-source.codes

> Again, why not googe?

When you grep some code in your IDE, you don't use some fussy word search that ignores non-word characters. Why? Because it's useless and awfully inconvenient. So that's the thing - searching for code in this web app is intended to be as easy as you would in the IDE.

_____________________________________________

## Upd. never mind, github _does_ have it's own code fragment search:

![image](https://user-images.githubusercontent.com/5202330/105596740-74c0b600-5d9c-11eb-8f88-03d8fa1d3109.png)
