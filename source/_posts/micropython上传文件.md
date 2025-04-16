---
title: micropython上传文件
date: 2025-04-16 21:15:34
tags: micropython
---

micropython 使用 openmv 的 requests 库上传文件

## 问题

根据 microptyhon 官方 requests 库中描述，并不支持上传文件。

- File upload is not supported.

## 查找其他库

从此链接最下方得到 openmv的库已经支持上传文件 https://forums.openmv.io/t/send-files-to-google-storage/8338

openmv urequests库

文档：https://docs.openmv.io/library/urequests.html

源码：https://github.com/openmv/openmv/blob/master/scripts/libraries/requests.py



![1744529612789](1744529612789.png)

部分源码：

```python
        if files is not None:
            data = bytearray()
            boundary = b"37a4bcce91521f74142f1868e328a6b9"
            s.write(b"Content-Type: multipart/form-data; boundary=%s\r\n" % (boundary))
            for name, fileobj in files.items():
                data += b"--%s\r\n" % (boundary)
                data += b'Content-Disposition: form-data; name="%s"; filename="%s"\r\n\r\n' % (
                    name,
                    fileobj[0],
                )
                data += fileobj[1].read()
                data += b"\r\n"
            data += b"\r\n--%s--\r\n" % (boundary)

        if data:
            s.write(b"Content-Length: %d\r\n\r\n" % len(data))
            s.write(data)
        else:
            s.write(b"\r\n")
```

可以看到函数会根据用户传入files参数，将数据写入到data变量，再通过 socket.write函数发送出去。



## 搭建调试环境

### python + fastapi 作为文件服务器。

增加了存储功能 ，方便验证文件完整性。

```python
# https://fastapi.tiangolo.com/zh/tutorial/request-files/#file_1
# http://127.0.0.1:8000/docs 可在线测试

import shutil
from pathlib import Path
import uvicorn
from fastapi import FastAPI, File, UploadFile

app = FastAPI()


@app.post("/files/")
async def create_file(file: bytes = File(description="A file read as bytes")):
    return {"file_size": len(file)}


@app.post("/uploadfile/")
async def create_upload_file(
    file: UploadFile = File(description="A file read as UploadFile"),
):
    print(file.filename)
    p = Path('.')
    with open(p/file.filename, "wb") as buffer:
        shutil.copyfileobj(file.file, buffer)
    return {"filename": file.filename}

if __name__ == '__main__':
    uvicorn.run(app, host="0.0.0.0", port=int("8000"))
```



### postman 向 fastapi 传输文件

验证 fastapi 是否工作正常。只是为了找问题，正常开发可跳过。



![1744530177421](1744530177421.png)

注意：在 file 单元格右侧，将 Text 切换为 File 并选中一个文件。建议初期选择简单的文本文档。

![1744530218116](1744530218116.png)



### postman 向网络调试助手发送文件

为了直观的看到发送的所有数据，方便调试。这个步骤也只是为了找问题，正常开发可跳过。

![1744536371222](1744536371222.png)



### OpenMV 的 requests 库发送文件

既然原版不支持，就用 OpenMV 的库。

使用 OpenMV 的 requests 库向 fastapi 发送文本文件，fastapi提示错误  "POST /uploadfile/ HTTP/1.0" 422 Unprocessable Entity。

但是 postman 却可以和 fastapi 正常配合。此时就需要找到两者之间的差异。需要 OpenMV 的 requests 库也向网络调试助手发送同一个文本文件，对比 postman 的网络助手数据，发现少一行 `Content-Type: text/plain`。库中增加相关的代码，再去和fastapi 通讯，可以正常上传文件。

测试函数：

```python
def upload_img(url, f_path, f_name, f_type):
    pic_size = 0
    try:
        p = f_path+f_name
        with open(p, 'rb') as file_handle:

            print(p, "read finish")
            files = {f_type: (f_name, file_handle)}

            print("upload file...")
            r = up_requests.request('POST', url, files=files)
            if r.status_code == 200:
                print("upload success")
            else:
                print("upload failed")
    except UnicodeError as e:
        print("Unicode error:", e)
    except Exception as e:
        print("HTTP request error:", e)
```

修改后的 requests 库

```python
# The MIT License (MIT)
#
# Copyright (c) 2013, 2014 micropython-lib contributors
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

# Source: improved version of micropython-lib's requests.
# Some useful links for future updates:
# https://www.w3.org/TR/html401/interact/forms.html#h-17.13.4
# https://docs.python-requests.org/en/master/

import socket
import binascii


class Response:
    def __init__(self, code, reason, headers=None, content=None):
        self.encoding = "utf-8"
        self._content = content
        self._headers = headers
        self.reason = reason
        self.status_code = code

    @property
    def headers(self):
        return str(self._headers, self.encoding)

    @property
    def content(self):
        return str(self._content, self.encoding)

    def json(self):
        import json

        return json.loads(self._content)


def readline(s):
    l = bytearray()
    while True:
        try:
            l += s.read(1)
            if l[-1] == b"\n":
                break
        except:
            break
    return l


def socket_readall(s):
    buf = b""
    while True:
        recv = b""
        try:
            recv = s.recv(1)
        except:
            pass
        if len(recv) == 0:
            break
        buf += recv
    return buf


def request(method, url, data=None, json=None, files=None, headers={}, auth=None, stream=None):
    try:
        proto, dummy, host, path = url.split("/", 3)
    except ValueError:
        proto, dummy, host = url.split("/", 2)
        path = ""
    if proto == "http:":
        port = 80
    elif proto == "https:":
        import ssl

        port = 443
    else:
        raise ValueError("Unsupported protocol: " + proto)

    if ":" in host:
        host, port = host.split(":", 1)
        port = int(port)

    if auth:
        headers["Authorization"] = b"Basic %s" % (
            binascii.b2a_base64("%s:%s" % (auth[0], auth[1]))[0:-1]
        )

    resp_code = 0
    resp_reason = None
    resp_headers = []

    ai = socket.getaddrinfo(host, port)[0]
    s = socket.socket(ai[0], ai[1], ai[2])
    try:
        s.connect(ai[-1])
        s.settimeout(5.0)
        if proto == "https:":
            s = ssl.wrap_socket(s, server_hostname=host)

        s.write(b"%s /%s HTTP/1.0\r\n" % (method, path))

        if "Host" not in headers:
            s.write(b"Host: %s\r\n" % host)

        # Iterate over keys to avoid tuple alloc
        for k in headers:
            s.write(k)
            s.write(b": ")
            s.write(headers[k])
            s.write(b"\r\n")

        if json is not None:
            import json

            data = json.dumps(json)
            s.write(b"Content-Type: application/json\r\n")

        if files is not None:
            data = bytearray()
            boundary = b"37a4bcce91521f74142f1868e328a6b9"
            s.write(b"Content-Type: multipart/form-data; boundary=%s\r\n" % (boundary))
            for filetype, fileobj in files.items():
                data += b"--%s\r\n" % (boundary)
                data += b'Content-Disposition: form-data; name="%s"; filename="%s"\r\n' % (
                    'file',
                    fileobj[0]
                )
                data += b'Content-Type: %s\r\n\r\n' % (filetype)
                data += fileobj[1].read()
                data += b"\r\n"
            data += b"\r\n--%s--\r\n" % (boundary)

        if data:
            s.write(b"Content-Length: %d\r\n\r\n" % len(data))
            s.write(data)
        else:
            s.write(b"\r\n")

        response = socket_readall(s).split(b"\r\n")
        while response:
            l = response.pop(0).strip()
            if not l or l == b"\r\n":
                break
            if l.startswith(b"Transfer-Encoding:"):
                if b"chunked" in l:
                    raise ValueError("Unsupported " + l)
            elif l.startswith(b"Location:") and not 200 <= status <= 299:
                raise NotImplementedError("Redirects not yet supported")
            if "HTTPS" in l or "HTTP" in l:
                sline = l.split(None, 2)
                resp_code = int(sline[1])
                resp_reason = sline[2].decode().rstrip() if len(sline) > 2 else ""
                continue
            resp_headers.append(l)
        resp_headers = b"\r\n".join(resp_headers)
        content = b"\r\n".join(response)
    except OSError:
        raise
    finally:
        s.close()

    return Response(resp_code, resp_reason, resp_headers, content)


def head(url, **kw):
    return request("HEAD", url, **kw)


def get(url, **kw):
    return request("GET", url, **kw)


def post(url, **kw):
    return request("POST", url, **kw)


def put(url, **kw):
    return request("PUT", url, **kw)


def patch(url, **kw):
    return request("PATCH", url, **kw)


def delete(url, **kw):
    return request("DELETE", url, **kw)
```



## 其他

OpenMV 的库下载文件功能不能用。

现在是 micropython 库下载文件，OpenMV的库上传文件。
