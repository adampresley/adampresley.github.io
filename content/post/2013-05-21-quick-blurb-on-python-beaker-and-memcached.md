---
author: "Adam Presley"
date: 2013-05-21T12:55:00Z
tags: ["python", "beaker", "memcached"]
title: "Quick Blurb on Python Beaker and Memcached"
slug: "quick-blurb-on-python-beaker-and-memcached"
---

Real quickly I wanted to post to show how one can configure Beaker for
session management using Memcached as the session storage. It took me a
minute and a few searches to put it all together, so hopefully this will
help someone searching the web.

{{< highlight python >}}
SESSION_OPTS = {
   "session.type": "ext:memcached",
   "session.cookie_expires": 14400,
   "session.auto": True,
   "session.url": "127.0.0.1:11211",
   "session.lock_dir": os.path.join(SESSION_PATH, "lock"),
   "session.data_dir": os.path.join(SESSION_PATH, "data")
}
{{< / highlight >}}

I'm still not sure if the lock and data directories are needed, and this
example only shows connecting to a single, localhost Memcached server.
Just the same it will help me remember how to do it in the future.
Cheers, and happy coding!
