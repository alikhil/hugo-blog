---
title: "Unintended Side Effects of Using http.DefaultClient in Go"
date: 2025-03-19T19:20:45+03:00
categories:
- programming
- opensource
tags:
- Go
- http

---

The Internet is plenty of articles that telling why you should not be using `http.DefaultClient` in Golang ([one](https://medium.com/@nate510/don-t-use-go-s-default-http-client-4804cb19f779), [two](https://vishnubharathi.codes/blog/know-when-to-break-up-with-go-http-defaultclient/)) but they refer to `Timeout` and `MaxIdleConns` settings.

Today I want to share with you another reason why you should avoid using `http.DefaultClient` in your code.

<!--more-->

## The Story

As an SRE at Criteo, I both read and write code. Last week, I worked on patching [Updatecli](https://github.com/updatecli/updatecli) — an upgrade automation tool written in Go.

The [patch](https://github.com/updatecli/updatecli/pull/4432) itself was just ~15 lines of code. But then I spent three days debugging a strange authorization bug in an unrelated part of the code.

It happened because of code like this:

```go
client := http.DefaultClient
client.Transport = &transport.PrivateToken{
    Token: s.Token,
    Base:  client.Client.Transport,
}
```

Since `http.DefaultClient` is a reference, not a value:

```go
var DefaultClient = &Client{}
```

The code above is effectively the same as:

```go
http.DefaultClient.Transport = &transport.PrivateToken{
    Token: s.Token,
    Base:  http.DefaultClient.Transport,
}
```

Later, in a third-party library, I found this:

```go
if opts.Client == nil {
    opts.Client = http.DefaultClient
}
```

## The Fix

To prevent this, I had to change the code to:

```go
client := &http.Client{}
client.Transport = &transport.PrivateToken{
    Token: s.Token,
    Base:  client.Transport,
}
```

As a result, the patched client with the authorization transport got injected into the third-party library, causing unexpected failures.

Bugs like this are hard to catch just by reading the code, since they involve global state mutation. But could they be detected by linters?

What do you think? How do you find or prevent such issues in your projects?
