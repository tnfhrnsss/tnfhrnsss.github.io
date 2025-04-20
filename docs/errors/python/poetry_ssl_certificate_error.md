---
layout: post
title: python-poetry SSLCertVerificationError
date: 2025-04-21 00:27:00
last_modified_at : 2025-04-21 00:27:00
parent: Python
has_children: false
nav_order: 1
nav_exclude: true
---

# Problem
- poetry 설치하다가 "ssl.SSLCertVerificationError: certificate verify failed: unable to get local issuer certificate error" 발생한 현상

# Error log

```java
sudo curl -sSL https://install.python-poetry.org | python3.13 -
Retrieving Poetry metadata
Traceback (most recent call last):
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/urllib/request.py", line 1319, in do_open
    h.request(req.get_method(), req.selector, req.data, headers,
    ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
              encode_chunked=req.has_header('Transfer-encoding'))
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/http/client.py", line 1338, in request
    self._send_request(method, url, body, headers, encode_chunked)
    ~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/http/client.py", line 1384, in _send_request
    self.endheaders(body, encode_chunked=encode_chunked)
    ~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/http/client.py", line 1333, in endheaders
    self._send_output(message_body, encode_chunked=encode_chunked)
    ~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/http/client.py", line 1093, in _send_output
    self.send(msg)
    ~~~~~~~~~^^^^^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/http/client.py", line 1037, in send
    self.connect()
    ~~~~~~~~~~~~^^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/http/client.py", line 1479, in connect
    self.sock = self._context.wrap_socket(self.sock,
                ~~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^
                                          server_hostname=server_hostname)
                                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/ssl.py", line 455, in wrap_socket
    return self.sslsocket_class._create(
           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~^
        sock=sock,
        ^^^^^^^^^^
    ...<5 lines>...
        session=session
        ^^^^^^^^^^^^^^^
    )
    ^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/ssl.py", line 1076, in _create
    self.do_handshake()
    ~~~~~~~~~~~~~~~~~^^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/ssl.py", line 1372, in do_handshake
    self._sslobj.do_handshake()
    ~~~~~~~~~~~~~~~~~~~~~~~~~^^
ssl.SSLCertVerificationError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1028)

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "<stdin>", line 959, in <module>
  File "<stdin>", line 937, in main
  File "<stdin>", line 538, in run
  File "<stdin>", line 800, in get_version
  File "<stdin>", line 861, in _get
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/urllib/request.py", line 189, in urlopen
    return opener.open(url, data, timeout)
           ~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/urllib/request.py", line 489, in open
    response = self._open(req, data)
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/urllib/request.py", line 506, in _open
    result = self._call_chain(self.handle_open, protocol, protocol +
                              '_open', req)
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/urllib/request.py", line 466, in _call_chain
    result = func(*args)
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/urllib/request.py", line 1367, in https_open
    return self.do_open(http.client.HTTPSConnection, req,
           ~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                        context=self._context)
                        ^^^^^^^^^^^^^^^^^^^^^^
  File "/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/urllib/request.py", line 1322, in do_open
    raise URLError(err)
urllib.error.URLError: <urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1028)>
```

# Cause

- 어느 버전 이후부터 certificate가 기본 설치가 안되어있다고 한다..(정확하진 않음)
- 수동으로 설치 필요하다.
- 파이썬 홈디렉토리로 이동햐서 Install Certificates.command 파일 실

```java
Python 3.13 % ./Install\ Certificates.command 
 -- pip install --upgrade certifi
Collecting certifi
  Downloading certifi-2025.1.31-py3-none-any.whl.metadata (2.5 kB)
Downloading certifi-2025.1.31-py3-none-any.whl (166 kB)
Installing collected packages: certifi
Successfully installed certifi-2025.1.31

[notice] A new release of pip is available: 24.3.1 -> 25.0.1
[notice] To update, run: python3.13 -m pip install --upgrade pip
 -- removing any existing file or link
 -- creating symlink to certifi certificate bundle
 -- setting permissions
 -- update complete

```

# Reference

[https://github.com/python-poetry/poetry/issues/3528#issuecomment-878902520](https://github.com/python-poetry/poetry/issues/3528#issuecomment-878902520)