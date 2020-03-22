---
title: "Netscape, the rewrite big mistake"
date: "2016-11-10"
categories: ['dev']
---

Back in the mid-1990s **Netscape** was one of the first successful startup of the Internet era. The company had made with success it's IPO (Initial Public Offering) and trusted more dans 90% of browser usage. In 2008 the company had lost the browser war and its owner AOL has announced the end of the support of Netscape products. What happened?

![Netscape logo](/post/netscape-rewrite_files/netscape.png)

In these cases there is often more than one reason, but one of them is certainly “the single worst strategic mistake that any software company can make: **They decided to rewrite the code from scratch**”.[^fn1]

Here are 2 comments of 2 of the 5 programming superstars who did the original version of Navigator:

> This one decision cost Netscape 3 years. That's three years in which the company couldn't add new features, couldn't respond to the competitive threads from Internet Explorer, and had to sit on their hands while Microsoft completely ate their lunch.
-- Lou Montulli[^fn1]

> [the rewrite] is one of the biggest software disasters there has ever been. It basically killed the company
-- Jamie Zawinski (a.k.a. jwz)[^fn2]

In his book[^fn1], Joel Spolsky explains why it is such a **big mistake**. And these are lessons learned that can be applied for almost every software.

# You will loose the experiences capitalized in the code

> Each of these bugs took weeks of real-world usage before they were found. The programmer might have spent a couple of days reproducing the bug in the lab and fixing it. [...]

> When you throw away code and start from scratch, you are throwing away all that knowledge. All those collected bug fixes. Years of programming work.

# You will loose time and money

> You are throwing away your market leadership. You are giving a gift of two or three years to your competitors, and believe me, that is a long time in software years. [...]

> You are wasting an outlandish amount of money writing code that already exists.

# You will not actually make a better job

> It's important to remember that when you start from scratch there is absolutely no reason to believe that you are going to do a better job than you did the first time. [...]

> You're just going to make most of the old mistakes again, and introduce some new problems that weren't in the original version.
Let's learn from history!

[^fn1]: Joel Spolsky, *[Joel on Software] (https://www.goodreads.com/book/show/41786.Joel_on_Software)* (Apress, 2004). Articles are also available on his blog [Things You Should Never Do, Part I](http://www.joelonsoftware.com/articles/fog0000000069.html).

[^fn2]: Peter Seibel, *[Coders at Work] (https://www.goodreads.com/book/show/6713575-coders-at-work)* (Apress, 2009).
