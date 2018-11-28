---
title: Release It!
date: '2018-04-28'
tags: [ops, book]
---

![Book cover](/post/release-it_files/release-it.jpeg)

This book is **a bible for any professional who wants to deploy a solution in production**--it's the goal normally, not building throwable POC. It is a recognized reference since it has helped to popularize certain patterns such as the [circuit breaker](https://back2code.quora.com/Circuit-breaker) and it is **at the top of all the must read lists in the domain**. It’s full of good advices and feedbacks since **Michael T. Nygard** has worked in the field in question, which is now called **operations (and even SRE)**, on critical applications--mainly, but not only, big e-commerce sites.

His book is **more about experience and the application of good practices than about theory and dogmatism**--it's a good thing. He is also endowed with a true talent of writer--I’m speaking about technical books and not about literature. This book is extremely well written and humorous which incredibly spices the reading--which might not be fun at all according to the subject. On this aspect it reminds me of another great technical author, **Joel Spolsky**. He talked exactly about the capability of technical writers to write well--or rather not so well**in hist book *The Best Software Writing*[^1]. This is how he introduces the subject.

> Yeah, I know: programmers hate writing. And when, for some reason, they are somehow forced to write down something, it reads like the repair manual of a DC-3, or the result of consolidating all the "do not do this" examples from an English style guide into one page.

And here in my opinion is an example of a **good writing style according** to Spolsky in *Release It !*. The object of this humour is not only the pleasure of the reader, it also serves to make a deep impression on people with strong images.

> It should not be possible for an administrator to break objects associations inside the application [he's talking about Spring dependency injection]. That's just wearing your guts on the outside. Whenever possible, keep production configurations separate from the basic wiring and plumbing of the application. They should be in separate files administrators do not accidentally edit internals. [...] Mixing them is the equivalent of putting the ejection seat button next to the radio tuner. Sooner or later, something bad will happen.

And other little phrases that you will certainly have the opportunity to use at your office--it's so true.

> First, nothing is as permanent as a temporary fix. Most of these remain in place for the next year or two.

The only complaint we can make is that it is a bit dated--fortunately there is less XML and more JSON now--and that's normal since it was written there are more 10 years, at a time when application servers reigned supreme. But let us be happy **because a fresh new edition has just come out in early 2018**--and also because this era is well and truly over. **A book to read absolutely if you work in the field**--do not read it only for humour.

***

**Michael T. Nygard**, *[Release It](https://www.goodreads.com/book/show/1069827.Release_It_)* (Pragmatic Bookshelf, 2007)

[^1]:  Joel Spolsky, *[The Best Software Writing I](https://www.goodreads.com/book/show/41787.The_Best_Software_Writing_I)* (Apress, 2005)

