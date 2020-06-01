---
layout: post
title: Gynvael's web security challenge - part 4.
subtitle: Quick hacking at the weekend.
gh-repo: daattali/beautiful-jekyll
tags: [ctf, hacking, challenges]
comments: true
---

Over the weekend I have decided to play with Gynvael's web security challenges. The post presents the write-up of challenge 4.
The challenge is located under the following URL:
[http://challenges.gynvael.stream:5004](http://challenges.gynvael.stream:5004)

This time the code is more advanced. At first, we see a simple POST endpoint.

```
app.post('/flag', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')
  if ((typeof req.body) !== 'string') {
    res.end("What?")
    return
  }

  if (req.body.includes('ShowMeTheFlag')) {
    res.end(FLAG)
    return
  }

  res.end("Say the magic phrase!")
})


```

Based on that endpoint only, the tasks look very simple - we must only send ShowMeTheFlag as the body of POST request. Let's try this out...

![gyn_4](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn4_1.png)

Unfortunately, this time there is also the middleware - app.use() which protects us for sending the right value.

```
app.use(express.text({
  verify: (req, res, body) => {
    const magic = Buffer.from('ShowMeTheFlag')

    if (body.includes(magic)) {
      throw new Error("Go away.")
    }
  }
}))

```

As we have already learned, Gynvael prepares his challenges based on the notes/warnings to the developers in the documentation. The middleware uses the express.text() function, let's see what documentation says about that function.
https://expressjs.com/en/api.html#express.text

This time we have got the following note:

{: .box-note}
**Note:** As req.body’s shape is based on user-controlled input, all properties and values in this object are untrusted and should be validated before trusting. For example, req.body.trim() may fail in multiple ways, for example stacking multiple parsers req.body may be from a different parser. Testing that req.body is a string before calling string methods is recommended.

But, unfortunetly when we try to play with the Content-Type, we hit this check:
```
  if ((typeof req.body) !== 'string') {
    res.end("What?")
    return
  }
```

We should also take a look at the table below the note:
![gyn_4](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn4_2.png)

The first row in the table describes the default charset, that can be specified by the programmer. The important part is that "if the Charset is not specified in the Content-Type header of the request". So the idea is to send the value encoded as different charset which would not be stoped on middleware but succeed with the flag. Let's see the Express implementation of the text/plain middleware.

**node_modules/body-parser/lib/types/text.js**

![gyn_4](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn4_3.png)

At line 83, the charset is retrieved from the request, then it is read using the provided encoding. 

In the file that implements read() function: **node_modules/body-parser/lib/read.js**
![gyn_4](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn4_4.png)

Inline 68 we find that the encoding is checked using the iconv library, which is used by Express for encoding and decoding.
Iconv is the popular GNU project which converts one encoding to another and it supports multiple encodings. I have found the whole list here:
https://gist.github.com/hakre/4188459

At this point, we may probably use any of those encodings to send our payload. I have chosen the UTF-16. With the usage of Burp Suite and the Hackvertor, we send our payload utf-16 encoded.

![gyn_4](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn4_5.png)
