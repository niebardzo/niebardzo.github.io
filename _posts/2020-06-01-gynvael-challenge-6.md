---
layout: post
title: Gynvael's web security challenge - part 6.
subtitle: Quick hacking at the weekend.
gh-repo: daattali/beautiful-jekyll
tags: [ctf, hacking, challenges]
comments: true
---

Over the weekend I have decided to play with Gynvael's web security challenges. The post presents the write-up of challenge 6.
The challenge is located under the following URL:
[http://challenges.gynvael.stream:5006](http://challenges.gynvael.stream:5006)

So far it is the easiest of the challenges, but it may look a little bit confusing at first glance. Below, is the code of target endpoint

```

app.get('/flag', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')

  if (!req.query.secret1 || !req.query.secret2) {
    res.end("You are not even trying.")
    return
  }

  if (`<${checkSecret(req.query.secret1)}>` === req.query.secret2) {
    res.end(FLAG)
    return
  }

  res.end("Lul no.")
})

```

As you can see it is GET endpoints that require secret1 and secret2 params. The important check that we need to bypass is the second if statement where magic checkSecret() function is called. Let's check the code of it:

```
const checkSecret = (secret) => {
  return
    [
      secret.split("").reverse().join(""),
      "xor",
      secret.split("").join("-")
    ].join('+')
}
```

Also, let's add the console.log to see how the left side of the comparison would look like based on the input.
![gyn_6](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn6_1.png)

![gyn_6](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn6_1.png)

From the first try we can assume that on every input in secret1 we would get the same value on the left side of comparison **<undefined>**

![gyn_6](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-06-01-gyn6_1.png)

It is because the function checkSecret() has broken EcmaScript6 syntax, which the interpreter tries to fix by adding the semicolon at the end of the return.
```
const checkSecret = (secret) => {
  return **;**
    [
      secret.split("").reverse().join(""),
      "xor",
      secret.split("").join("-")
    ].join('+')
}
```

This behavior is called the Automatic Semicolon Injection.
