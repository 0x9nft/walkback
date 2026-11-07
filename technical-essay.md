# dev.to 文章 · 英文全文（可直接发布）

> 发布平台：https://dev.to/new
> 发布前把最下面的 **Canonical URL** 填成 `https://walkback.live/notes/what-changes-when-you-tell-the-ending-first`，
> 避免搜索引擎把这篇文章当成重复内容。
> 标签（tags，最多 4 个，逗号分隔）：`webdev`, `gamedev`, `javascript`, `datascience`

---

## 标题（复制到 Title 栏）

```
I built a life simulator that runs backwards — here's what broke
```

## 正文（复制到 Body 栏，dev.to 用 Markdown）

```markdown
A few weeks ago I got obsessed with a small question: what if you knew how a
life ends before you saw any of it?

Most life simulators run birth to death. The ending is a reveal, and the
player only ever sees the middle. I wanted to invert that — hand over the
year and cause of death on the first screen, then walk backwards, one year at
a time, until you reach the day the person was born.

That is [walkback](https://walkback.live/en). It is free, runs in the browser,
needs no account, and covers 173 countries in 13 languages. This post is about
the parts that turned out to be harder than expected.

## The data is not a random number generator

The tempting implementation is `age = 60 + Math.random() * 25`. That would be
wrong in a way that matters.

Age at death is sampled from each country's life table, split by sex. A life
table gives the probability of dying in each year of life *given survival to
that year*. So the distribution has a sharp early peak in some countries, a
very low plateau through childhood almost everywhere, and a steep climb from
midlife onward.

Concretely, in the data I used, under-five mortality is 115.6 per 1,000 live
births in Nigeria and 2.4 per 1,000 in Japan. That single line explains most
of the 30-year gap in life expectancy at birth between the longest and
shortest entries in the dataset. It also means some runs end at four years
old. Players report those as bugs. They are not — they are the distribution
being honest.

Country selection is weighted by annual births, not population. India comes
up about 17.7% of the time, China about 6.8%, Japan about 0.57%. That is a
different world from one where every country is equally likely, and it is the
whole point.

Sources: UN World Population Prospects 2024 revision (estimates through 2023),
World Bank World Development Indicators, WHO Global Health Observatory. The
site states on every page that this is a simplified model, not a prediction
and not health advice.

## Reversing the chronology is the hard part

Going forwards, an event generator can be causal: this year's family state,
health, work and income follow from last year's. You only ever need the
previous step.

Going backwards, the player sees the consequence first. If they are shown a
widow at 60, then every earlier year has to be consistent with a marriage
that happened and a partner who was still alive. If they are shown a
retirement at 65, the career events before that must line up. You cannot
lazy-generate backwards; the entire life has to exist before the first frame
is rendered.

So the model generates the whole life forwards once, stores it as a year-by-year
record, and then plays it in reverse. That sounds trivial, but it changes what
"a state" means: every year has to carry enough context to stand alone when
read out of order, because a player can stop, rewind, or jump to any year.

The other design consequence is quieter. Forward, a year where nothing
happens is filler — you click past it. Backward, the same year is a position:
you know exactly how many years remain. A line like "moved to another city for
a job" reads completely differently at 31 when you already know the ending is
47. No extra content is needed; the information is in the order.

## 13 languages is mostly a URL problem

Each language has its own path (`/ja`, `/de`, `/pt`, …), its own `<title>` and
description, and a full hreflang set including `x-default`. The part that
bit me: the root layout decides `<html lang>` from the user's cookie, so two
English-only content pages I added later were being served as
`lang="zh-Hans"` with English body text. I fixed it by pinning the locale in
the middleware for those routes, and by making them dynamically rendered —
`force-static` had pre-rendered the wrong language at build time, so the
middleware never even ran.

If you take one thing from this: when you mix a multi-locale app with a few
single-language pages, verify the rendered `<html lang>` in production, not
just the component that sets it.

## Sharing is the whole distribution loop

The end of a run produces a page with the age and cause of death, plus a
shareable link, an image sheet, and a short film. The share text includes the
actual numbers — "I died at 47, of pancreatic cancer. How long will you live?"
— because a brand slogan tells a stranger nothing.

I also replaced the social preview image this week. The old one was a pretty
night skyline with no text at all. In a timeline it looked like a broken link.
The new one says what the thing is in six words.

## What I would do differently

I spent the first days polishing the model and almost none on distribution.
By the time the site was ready, it had no external links, the domain was nine
days old, and nothing was indexed. Google eventually started crawling — the
first indication was a `Googlebot` hit on the sitemap, then image requests for
the film thumbnails, then Applebot — but that took days of doing nothing.

If you are building a small thing alone: put one real, crawlable page with a
clear title out on day one, and get one external link before you polish the
fourth feature.

## Try it

It is free, no signup, no download: https://walkback.live/en

Four longer notes on the model (why countries differ by 30 years, why some
lives end at four, what reversing the chronology changes):
https://walkback.live/notes

I would genuinely like to hear whether the backwards reading works for other
people, or whether it just feels like a gimmick after the first run.
```

## 发布后立刻做

1. 在这篇文章底部发**第一条自己的评论**（dev.to 允许自评），放一句：
   `Happy to go into detail on the life-table sampling or the reverse-consistency problem if anyone's curious.`
2. 把文章链接贴到 X / Reddit 相关讨论里（不是刷屏，是别人问「怎么做」时给）
3. 一天后看 dev.to 的阅读数；它通常会被 Google 在几小时内收录，给站点带来一条真实外链
