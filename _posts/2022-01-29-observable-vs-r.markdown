---
layout: post
title:  "Observable vs. R for data journalism"
date:   2022-01-29 19:24:00 +0000
image:  /images/observable-notebooks.png
---

I've been using [Observable](https://observablehq.com/) at work for the best part of a year now, since being introduced to it properly by my colleague [Ben](https://twitter.com/BenAyre). It's an incredibly well-designed tool with a growing (and very friendly) community of developers and users. In this respect, it resembles my other favoured tool(s) for smaller-scale data analysis: R, and particularly the world of RStudio/[the tidyverse](https://www.tidyverse.org/).

The similarities are pretty striking, actually. Both tools allow you tap into a huge ecosystem of existing code ([npm](https://www.npmjs.com/); [CRAN](https://cran.r-project.org/)); both include a charting library based on Leland Wilkinson's *Grammar of Graphics* (1999) ([Plot](https://observablehq.com/@observablehq/plot); [ggplot2](https://ggplot2.tidyverse.org/)); both put an emphasis on notebook-style literate programming (Observable's [web interface](https://observablehq.com/@observablehq/how-observable-runs); [RMarkdown](https://rmarkdown.rstudio.com/)); both include collections of UI components that can be easily used for prototyping ([Inputs](https://observablehq.com/@observablehq/inputs); [Shiny](https://shiny.rstudio.com/)); perhaps most important, both were spearheaded by the developer of a wildly popular package for their host language, who now seems somewhat uneasy with his newfound fame ([Mike Bostock](https://twitter.com/mbostock); [Hadley Wickham](https://twitter.com/hadleywickham/)).

Despite all these similarities, I have encountered a few pain points. Rather than touting Observable's advantages (ease of sharing notebooks in a team, working directly with JSON, etc.), I thought it might be worth running through my top three frustrations and giving the solutions I've come up with, in case they're useful to others making the same move.

## Browser security rules

![An Observable cell showing a networking error]({{site.baseurl}}/images/http-error.png)

Without a doubt, the biggest pain in the proverbial when switching from local R scripts to Observable's web interface has been dealing with modern web browsers' (sensible) rules around networking and security. Want to quickly scrape something from a Brazilian provincial government website that doesn't use HTTPS? No can do! How about an API that doesn't serve CORS headers? You're out of luck.

I've found two ways around this. The first is workflow-based: split up your data collection and data analysis and do the [ETL](https://en.wikipedia.org/wiki/Extract,_transform,_load) work server-side, leaving Observable for the fun stuff. This is what I do for larger projects, taking advantage of Observable's excellent (and [getting excellenter](https://observablehq.com/@observablehq/sql-cells)) tools for working with databases. At Global Witness we currently favour a combination of [dbt](https://docs.getdbt.com/) models and GitHub Actions feeding a central Postgres (and, crucially, PostGIS) database on RDS.

The second is simpler but less 'clean': set up your own reverse proxy to serve whatever you like over HTTPS, adding the appropriate CORS headers in transit. I've done this too, adapting [this simple Docker set-up](https://github.com/maximillianfx/docker-nginx-cors), but generally only use it for accessing Maxar's WMS endpoint for high-resolution satellite imagery.

## Working with tabular data

The approach pioneered by [dplyr](https://dplyr.tidyverse.org/) is rightly held up as a gold standard for working with tabular data in a reproducible way: it's really well-designed, easy to learn, and works beautifully with the rest of the R ecosystem. And while it's spawned a few imitators, of which Peter Beshai's [tidy.js](https://observablehq.com/@pbeshai/tidy-js-intro-demo) is the best, nothing comes close to providing a comprehensive and coherent 'grammar of data manipulation'...

...except SQL. I've found sticking with a database infinitely preferable to working with a JavaScript-based dplyr clone, particularly as for smaller projects a [SQLite DB](https://observablehq.com/@observablehq/sqlite) can be attached directly to an Observable notebook. There will always come a point---e.g. right before plotting---at which it's more sensible to manipulate JSON directly, but when you reach that stage D3.js's [array functions](https://github.com/d3/d3-array), along with new additions to the ECMAScript spec like [Array.prototype.flatMap()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap), are more than good enough.

Here's my favourite new pattern for manipulating an array of objects, analogous to a dplyr `mutate()` call. The flexibility afforded by defining your output as a JavaScript object really helps with things like updating deeply nested GeoJSON properties. 

```js
data = [
  { id: 1, width: 200, height: 100 },
  { id: 2, width: 260, height: 130 },
  { id: 3, width: 500, height: 70 }
];

data.map(d => ({
  ...d,
  area: d.width * d.height
}));
```

## Reproducibility and the web

Observable is built around the browser, and this design choice influences how it's used. While there are many advantages to this approach, it doesn't make it easy to cache or persist data. Let's say you're scraping some data from a web page and manipulating it: in R, I might cache this data as a .Rds file the first time the script runs and re-use this cache (if it exists) subsequently, leaving me with a script that's clear about the original source of its data (it'll still contain the relevant [httr](https://cran.r-project.org/web/packages/httr/index.html) calls) but that won't break or spit out incorrect results if the website changes.

I've rolled [my own solution](https://observablehq.com/@ltrgoddard/sqlite-http-cache) to this in the form of the little tool which 'remembers' any `fetch()` calls in your notebook and caches them in a SQLite database. You'll still to need to download the DB file after first run and attach it to the notebook before being able to take advantage of the cache (Observable's [File Attachments](https://observablehq.com/@observablehq/file-attachments) are immutable, for good reason), but it solves the problem of stale API endpoints or unreliable websites, and should be pretty fast too. Props to [Toph Tucker](https://twitter.com/tophtucker) for the inspiration! 

<iframe width="100%" height="500" frameborder="0" src="https://observablehq.com/embed/@ltrgoddard/sqlite-http-cache?cell=*"></iframe>
<br>
<hr>
<br>

I hope these quick reflections are of use. If you're someone on a similar technical journey, I'd love to hear about it [on Twitter](https://www.twitter.com/ltrgoddard). I'm not a full convert by any means---R still forms a significant part of my day-to-day, and for introducing less technical types to reproducible data workflows it can't be beat, but I'm looking forward to what's coming next from Observable HQ!
