---
layout: post
title: Gynvael's web security challenge - part 3.
subtitle: Quick hacking at the weekend.
gh-repo: daattali/beautiful-jekyll
tags: [ctf, hacking, challenges]
comments: true
---

Over the weekend I have decided to play with Gynvael's web security challenges. The post presents the write-up of challenge 3.
The challenge is located under the following URL:
[http://challenges.gynvael.stream:5003](http://challenges.gynvael.stream:5003)

The code returned is the simple Express.js application. We can copy the code and run it locally, so we would be able to debug.

```
app.get('/truecolors/:color', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')

  const color = ('color' in req.params) ? req.params.color : '???'

  if (color === 'red' || color === 'green' || color === 'blue') {
    res.end('Yes! A true color!')
  } else {
    res.end('Hmm? No.')
  }
})


```

This time the code uses req.params instead of req.query and the task is to cause the exception to read the path of the script. As we learned by the previous challenges, we should jump straight to the documentation:
[https://expressjs.com/en/api.html#req.params](https://expressjs.com/en/api.html#req.params)

We can see the Note for devs:

{: .box-note}
**Note:** Express automatically decodes the values in req.params (using decodeURIComponent).

So, the task is to crash during the URI decoding. To do that, it is enough to pass the % (percentage sign) to the params. As the % is used for URI encoding without value to be decoded after it, it would cause the express to crash.

![gyn_3](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-29-gyn3_1.png)
