---
layout: post
title: Gynvael's web security challenge - part 2.
subtitle: Quick hacking at the weekend.
gh-repo: daattali/beautiful-jekyll
tags: [ctf, hacking, challenges]
comments: true
---

Over the weekend I have decided to play with Gynvael's web security challenges. The post presents the write-up of challenge 2.
The challenge is located under the following URL:
http://challenges.gynvael.stream:5002/

The code returned is the simple Express.js application. We can copy the code and run it locally, so we would be able to debug.
The following code implements only the / endpoint and expects the secret parameter.

```
app.get('/', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')

  if (req.query.X.length > 800) {
    const s = JSON.stringify(req.query.X)
    if (s.length > 100) {
      res.end("Go away.")
      return
    }

    try {
      const k = '<' + req.query.X + '>'
      res.end("Close, but no cigar.")
    } catch {
      res.end(FLAG)
    }

  } else {
    res.end("No way.")
    return
  }
})

```

Again, there are two checks which must be bypassed. From the code analysis, we know that to get the flag, we would have to make the secret parameter with the length longer than 800 and in the second condition, the string concatenation must trow the exception to retrive the flag.

At this moment, we should do our lesson and refer to the Express documentation:
https://expressjs.com/en/api.html#req.query

The text in the box gives us the exact solution to the challenge.

![gyn_2](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-29-gyn2_1.png)

During the string concatenation, X object would be converted to a string to perform the concatenation. As we would overwrite the toString method the exception would be raised and we will get the flag. Also, the Express says that the user controls whole the object, so to bypass the first condition we can control the length parameter as well.

![gyn_2](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-29-gyn2_2.png)


Then, we overwrite the toString() method by setting there anything to cause the exception.

![gyn_2](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-29-gyn2_3.png)

Is Express.js drunk?
