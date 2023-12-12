Who Are You
===
 [X-Forwarded-For parsing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For) which lead me to this [RFC2616](https://datatracker.ietf.org/doc/html/rfc2616#section-4.2)
- slow server times
- iframes?
- _Cache-Control_ header
- caching
- slow HTTP attack?


The headers I edited +/ included:
- _User-Agent_: PicoBroswer
- _Referer_: http://mercury.picoctf.net:46199/
- _Date_: Sat, 29 Oct 2018 19:43:31 GMT
- _DNT_:1
- _X-Forwarded-For_:83.252.140.156
- _Accept-Language_:sv, sv-SE;q=0.6, en;q=0.5

### Thoughts
I found this one a quite fun one, as there was a lot of trial and error and researching + learning about different HTTP headers which can be (ab)used when sending a GET request. I had not thought about the possibilities of this so was rather eye opening. 