---
title: Staticman...The Journey Continues
url: "staticman-the-journey-continues"
date: "2019-08-20T11:39:00-0500"
draft: false
tags:
  - Staticman
  - comments
  - JAMstack
---

_In this post I will do a quick dive on how I setup my own staticman instance in [Heroku](https://dashboard.heroku.com/apps) since the [public API](https://api.staticman.net) and [Dev API](https://dev.staticman.net/) are no longer responding._

<!--more-->

First, [this site](https://yasoob.me/posts/running_staticman_on_static_hugo_blog_with_nested_comments/) from [yasoob](https://github.com/yasoob) does a great job running down how to get this setup in Github and on Heroku.  I will just add af ew changes I made based on the content of [Issue #299](https://github.com/eduardoboucas/staticman/issues/299).

As pointed out in yasoob's post, and in a [ton of issues](https://github.com/eduardoboucas/staticman/issues) on the repo, there are a few things amiss with the master branch.  And some of the fixes mentioned in the post did not work for me, but here is what did...

As pointed out by the trusty [VincentTam](https://github.com/VincentTam) in his [comment on Issue #299](https://github.com/eduardoboucas/staticman/issues/299#issuecomment-508029359) he points out that in his deploy release he is uing [`55d1430`](https://github.com/eduardoboucas/staticman/commit/55d14306d851059a2a27d24b5eb4cb17c5009477).  So, that is what I used as well.  And after some testing everything seems to work fine.  I am using `v3` on both of my sites and the `Mailgun` integration and all is working as expected.  I have not tested GitLab integration as I do not have a GitLab setup, but I may give it a shot in the future.

So, here is the quick and dirty on how I did it:


1.  Fork the [Staticman Repo](https://github.com/eduardoboucas/staticman)
2.  Create a Heroku app as mentioned above
3.  Run `git checkout -b workaround 55d1430` to create a branch named `workaround` from the commit mentioned above.
4.  Push that branch to Heroku using `git push -f heroku workaround:master`.  _I add the -f because I felt like living dangerously_

If you configured all of your environment variables properly you should be good to go!  I am running this instance on both [NetworkHobo](https://networkhobo.com) and [Dan C Williams](https://dancwilliams.com) using the `v3` API without issue.

I know this is quick, but I needed to get it out of my head.  Please comment below if there are any questions.  Thanks!