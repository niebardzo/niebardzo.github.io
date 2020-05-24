---
layout: post
title: Gynvael's web security challenge - part 1.
subtitle: Quick hacking at the weekend.
gh-repo: daattali/beautiful-jekyll
tags: [ctf, hacking, challenges]
comments: true
---

Over the weekend I have decided to play with Gynvael's web security challenges. The post presents the write-up of challenge 1.
The challenge is located under the following URL:
http://challenges.gynvael.stream:5001/

The code returned is the simple Express.js application. We can copy the code and run it locally, so we would be able to debug.
The following code implements only the / endpoint and expects the secret parameter.

```
app.get('/', (req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain;charset=utf-8')

  if (req.query.secret.length > 5) {
    res.end("I don't allow it.")
    return
  }

  if (req.query.secret != "GIVEmeTHEflagNOW") {
    res.end("Wrong secret.")
    return
  }

  res.end(FLAG)
})

```

Again, there are two checks which must be bypassed. From the code analysis, we know that to get the flag, we would have to make the secret parameter with the length shorter or equal to 5. Also the second loose comparison (!=) to the "GIVEmeTHEflagNOW" must return false.

At this stage we know, that sending the string GIVEmeTHEflagNOW as the secret parameter would hit the second condition. But luckily, we know the vulnerability called HTTP parameter pollution. It is mostly used to bypass WAFs where the frontend server processes parameters different than the backend server. Let's add additional console.log to display the secret in our local instance and try to send the secret parameter twice.

![gyn_1](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn1_1.png)

We send the secret twice.

![gyn_1](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn1_2.png)

As we can see nodejs interpreted the parameters as the array.

![gyn_1](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn1_3.png)

Unfortunately, during the loose comparison the Javascript Object - Array would be converted to its primitive - string. We can see results of that by doing the join() on array objects.
![gyn_1](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn1_4.png)

Which builds the string with a comma (,).

![gyn_1](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn1_5.png)

With that method we would not be able to build "GIVEmeTHEflagNOW" string as there always will be a comma in that string. For parameter pollution, we may try the old PHP technique by adding the array[] to the name of the parameters.

![gyn_1](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn1_6.png)

Luckily, it is interpreted as one element array Object, so during the conversion to primitive, it would be interpreted as a valid string. 

![gyn_1](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn1_7.png)

Now, we can send our secret code to get the flag.

![gyn_1](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn1_8.png)

