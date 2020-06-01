---
layout: post
title: Gynvael's web security challenge - part 5.
subtitle: Quick hacking at the weekend.
gh-repo: daattali/beautiful-jekyll
tags: [ctf, hacking, challenges]
comments: true
---

Over the weekend I have decided to play with Gynvael's web security challenges. The post presents the write-up of challenge 5.
The challenge is located under the following URL:
[http://challenges.gynvael.stream:5005](http://challenges.gynvael.stream:5005)

As previously, we have a POST endpoint, but this time the middleware is ulencoded.

```

app.use(express.urlencoded({extended: false}))

app.post('/flag', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')

  if (req.body.secret !== 'ShowMeTheFlag') {
    res.end("Say the magic phrase!")
    return
  }

  if (req.youAreBanned) {
    res.end("How about no.")
    return
  }

  res.end(FLAG)
})

```

Let's now see, if we are able to send secret=ShowMeTheFlag

![gyn_5](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn5_1.png)

We hit the second condition, it is not that simple. The second if statements checks, if the request is banned. The variable is set by the proxy implemented few lines below. 

```
const proxy = function(req, res) {
  req.youAreBanned = false
  let body = ''
  req
    .prependListener('data', (data) => { body += data })
    .prependListener('end', () => {
      const o = new URLSearchParams(body)
      req.youAreBanned = o.toString().includes("ShowMeTheFlag")
    })
  return app(req, res)
}

```

The proxy checks, if the ShowMeTheFlag found in the params value, the proxy sets the youAreBanned boolen to true.

As we have already learned, Gynvael prepares his challenges based on the notes/warnings to the developers in the documentation. The middleware uses the express.urlencoded() function, let's see what documentation says about that function.
[http://expressjs.com/en/4x/api.html#express.urlencoded](http://expressjs.com/en/4x/api.html#express.urlencoded)

In the second paragraph, we may read:

{: .box-note}
Returns middleware that only parses urlencoded bodies and only looks at requests where the Content-Type header matches the type option. This parser accepts only UTF-8 encoding of the body and supports automatic inflation of gzip and deflate encodings.


The trick with the encoding won't work as UTF-8 is only supported by this middleware. But, the middleware supports the inflation of gzip, which is a good candidate to try smuggling **ShowMeTheFlag**. 

At the begining of previously mentioned read() function we see by the comments that there is decompression from content stream performed:
**node_modules/body-parser/lib/read.js**

![gyn_5](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn5_2.png)


Simple as that, let's try to send the gzip compressed urlencoded body params, with the usage of Burp Suite and Hackvertor. We also must add the HTTP Header; **Content-Encoding: gzip**

![gyn_5](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn5_3.png)
