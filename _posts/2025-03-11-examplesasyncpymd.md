---
title: examples_async.py.md
---
\---

title: examples\_[async.py](http://async.py)

date: 2024-12-14T10:31:57.773Z

\---

import asyncio

from pathlib import Path

from typing import List

from torequests.dummy import Requests

from ichrome import AsyncChromeDaemon, ChromeEngine

from ichrome.async\_utils import AsyncChrome, AsyncTab, Tag, logger

\# logger.setLevel('DEBUG')

\# AsyncTab.\_log\_all\_recv = True

\# headless = False

headless = True

async def test\_chrome(chrome: AsyncChrome):

assert str(chrome) == "<Chrome(connected): [http://127.0.0.1:9222](http://127.0.0.1:9222)\>"

assert chrome.server == "[http://127.0.0.1:9222](http://127.0.0.1:9222)"

version = await chrome.version

assert isinstance(version, dict) and "Browser" in version

ok = await chrome.check()

assert ok is True, ok

ok = await chrome.ok

assert ok is True, ok

resp = await chrome.get\_server("json")

assert isinstance(resp.json(), list)

await [chrome.new](http://chrome.new)\_tab()

tabs1: List\[AsyncTab\] = await chrome.get\_tabs()

tabs2: List\[AsyncTab\] = await chrome.tabs

assert tabs1 == tabs2 and tabs1 and tabs2, (tabs1, tabs2)

tab0: AsyncTab = await chrome.get\_tab(0)

tab0\_by\_getitem = await chrome\[0\]

assert tab0 == tab0\_by\_getitem

assert tabs1\[0\] == tab0, (tabs1, tab0)

tab1: AsyncTab = await [chrome.new](http://chrome.new)\_tab()

assert isinstance(tab1, AsyncTab)

await asyncio.sleep(0.2)

await chrome.activate\_tab(tab0)

\# test batch connect multiple tabs

async with chrome.connect\_tabs(\[tab0, tab1\]):

assert tab0.status == "connected", (tab0.status, tab0)

assert tab1.status == "connected", (tab1.status, tab1)

\# watch the tabs switch

await tab1.activate\_tab()

await asyncio.sleep(0.2)

await tab0.activate\_tab()

await asyncio.sleep(0.2)

await tab1.activate\_tab()

\# test connect single tab

async with chrome.connect\_tabs(tab0):

assert tab0.status == "connected"

await chrome.close\_tab(tab1)

async def test\_tab\_ws(tab: AsyncTab):

\# test msg\_id auto increase

assert tab.msg\_id == tab.msg\_id - 1

assert tab.status == "disconnected", tab.status

assert [tab.ws](http://tab.ws) is None or (tab.flatten and not tab.\_session\_id)

async with tab():

assert [tab.ws](http://tab.ws)

assert tab.status == "connected"

\# assert no connection out of async context.

assert tab.status == "disconnected"

assert [tab.ws](http://tab.ws) is None or (tab.flatten and not tab.\_session\_id)

async def test\_send\_msg(tab: AsyncTab):

\# test send msg

assert (

tab.get\_data\_value(

await tab.send("Network.enable"), value\_path="value", default={}

)

\== {}

)

\# disable Network

await tab.disable("Network")

async def test\_tab\_cookies(tab: AsyncTab):

await tab.clear\_browser\_cookies()

assert len(await tab.get\_cookies(urls="[http://httpbin.org/get](http://httpbin.org/get)")) == 0

assert await tab.set\_cookie("test", "test\_value", url="[https://github.com/](https://github.com/)")

assert await tab.set\_cookie("test2", "test\_value", url="[https://github.com/](https://github.com/)")

assert await tab.set\_cookies(

\[{"name": "a", "value": "b", "url": "[http://httpbin.org/get](http://httpbin.org/get)"}\]

)

assert len(await tab.get\_cookies(urls="[https://github.com/](https://github.com/)")) == 2

assert len(await tab.get\_cookies(urls="[http://httpbin.org/get](http://httpbin.org/get)")) == 1

assert await tab.delete\_cookies("test", url="[https://github.com/](https://github.com/)")

assert len(await tab.get\_cookies(urls="[https://github.com/](https://github.com/)")) == 1

\# get all Browser cookies

assert len(await tab.get\_all\_cookies()) > 0

async def test\_browser\_context(

tab1: AsyncTab, chrome: AsyncChrome, chromed: AsyncChromeDaemon

):

\# test chrome.create\_context

assert len(await tab1.get\_all\_cookies()) > 0

async with chrome.create\_context(proxyServer=None) as context:

async with [context.new](http://context.new)\_tab() as tab:

assert len(await tab.get\_all\_cookies()) == 0

await tab.set\_cookie("a", "1", "[http://httpbin.org](http://httpbin.org)")

assert len(await tab.get\_all\_cookies()) > 0

\# test chrome\_daemon.create\_context

assert len(await tab1.get\_all\_cookies()) > 0

async with chromed.create\_context(proxyServer=None) as context:

async with [context.new](http://context.new)\_tab() as tab:

assert len(await tab.get\_all\_cookies()) == 0

await tab.set\_cookie("a", "1", "[http://httpbin.org](http://httpbin.org)")

assert len(await tab.get\_all\_cookies()) > 0

\# test chrome.incognito\_tab

assert len(await tab1.get\_all\_cookies()) > 0

async with chrome.incognito\_tab(proxyServer=None) as tab:

assert len(await tab.get\_all\_cookies()) == 0

await tab.set\_cookie("a", "1", "[http://httpbin.org](http://httpbin.org)")

assert len(await tab.get\_all\_cookies()) > 0

\# test chrome\_daemon.incognito\_tab

assert len(await tab1.get\_all\_cookies()) > 0

async with chromed.incognito\_tab(proxyServer=None) as tab:

assert len(await tab.get\_all\_cookies()) == 0

await tab.set\_cookie("a", "1", "[http://httpbin.org](http://httpbin.org)")

assert len(await tab.get\_all\_cookies()) > 0

async def test\_tab\_set\_url(tab: AsyncTab):

\# set new url for this tab, timeout will stop loading for timeout\_stop\_loading defaults to True

assert not (await tab.set\_url("[https://postman-echo.com/delay/5](https://postman-echo.com/delay/5)", timeout=1))

assert await tab.set\_url("[https://postman-echo.com/delay/1](https://postman-echo.com/delay/1)", timeout=5)

assert await tab.set\_url("[https://postman-echo.com/ip](https://postman-echo.com/ip)", timeout=5)

mhtml\_len = len(await tab.snapshot\_mhtml())

assert mhtml\_len in range(1, 1000), f"test snapshot failed {mhtml\_len}"

await tab.goto("[https://bing.com?setlang=en&cc=US](https://bing.com?setlang=en&cc=US)")

assert await tab.wait\_tag(

"#sb\_form\_q", max\_wait\_time=10

), "wait\_tag failed for #sb\_form\_q"

async def test\_tab\_js(tab: AsyncTab):

await tab.goto("[https://staticfile.org/about\_en.html](https://staticfile.org/about_en.html)", timeout=5)

\# test js update title

await tab.js("document.title = 'abc'")

\# test js\_code

assert (await tab.js\_code("return document.title")) == "abc"

\# test findall

await tab.js("document.title = 123456789")

assert (await tab.findall("<title>(.\*?)</title>")) == \["123456789"\]

assert (await tab.findall("<title>.\*?</title>")) == \["<title>123456789</title>"\]

assert (await tab.findall("<title>(1)(2).\*?</title>")) == \[\["1", "2"\]\]

assert (await tab.findall("<title>(1)(2).\*?</title>", "body")) == \[\]

new\_title = await tab.current\_title

\# test refresh\_tab\_info for tab meta info

assert (await tab.title) == new\_title

assert tab.\_title != new\_title

assert await tab.refresh\_tab\_info()

assert tab.\_title == new\_title

\# inject JS timeout return None

if not headless:

temp = await tab.js("alert()")

assert temp is None

\# close the alert dialog

assert await tab.handle\_dialog(accept=True)

\# inject js url: vue.js

\# get window.Vue variable before injecting

vue\_obj = await tab.js("window.Vue", "result.result.type")

\# {'id': 22, 'result': {'result': {'type': 'undefined'}}}

assert vue\_obj == "undefined"

assert await tab.inject\_js\_url(

"[https://cdn.staticfile.org/vue/2.6.10/vue.min.js](https://cdn.staticfile.org/vue/2.6.10/vue.min.js)", timeout=3

)

vue\_obj = await tab.js("window.Vue", value\_path=None)

\# {'id': 23, 'result': {'result': {'type': 'function', 'className': 'Function', 'description': 'function wn(e){this.\_init(e)}', 'objectId': '{"injectedScriptId":1,"id":1}'}}}

assert "Function" in str(vue\_obj)

tag = await tab.querySelector("#not-exist")

assert not tag

\# querySelectorAll with JS, return list of Tag object

tags = await tab.querySelectorAll("#logo")

assert tags, f"{\[tags, type(tags)\]}"

assert isinstance(tags\[0\], Tag), f"{\[tags\[0\], type(tags\[0\])\]}"

assert tags\[0\].tagName in {"a"}, f"{\[tags\[0\], tags\[0\].tagName\]}"

\# querySelectorAll with JS, index arg is Not None, return Tag or None

one\_tag = await tab.querySelectorAll("#logo", index=0)

assert isinstance(one\_tag, Tag), type(one\_tag)

await tab.stop\_loading\_page()

assert await tab.set\_html("")

current\_html = await tab.current\_html

assert current\_html == "<html><head></head><body></body></html>"

\# reload the page

for \_ in range(2):

assert await tab.reload()

await tab.wait\_loading(5, timeout\_stop\_loading=True)

current\_html = await tab.html

if current\_html:

break

\# print(current\_html)

assert len(current\_html) > 100, repr(current\_html)

\# test wait tags

result = await tab.wait\_tags("abcdabcdabcdabcd", max\_wait\_time=1)

assert result == \[\]

assert await tab.wait\_tag("#logo", max\_wait\_time=3)

assert await tab.includes("This repo is open-source. Licensed under MIT.")

assert not (await tab.includes("abcdabcdabcdabcd"))

assert await tab.wait\_includes("This repo is open-source. Licensed under MIT.")

assert (await tab.wait\_includes("abcdabcdabcdabcd", max\_wait\_time=1)) is False

assert await tab.wait\_findall("This repo is open-source. Licensed under MIT.")

assert (await tab.wait\_findall("abcdabcdabcdabcd", max\_wait\_time=1)) == \[\]

\# test wait\_console\_value

await tab.js("setTimeout(() => {console.log(123)}, 2);")

assert (await tab.wait\_console\_value()) == 123

async def test\_wait\_response(tab: AsyncTab):

\# wait\_response with filter\_function

\# raw response: {'method': 'Network.responseReceived', 'params': {'requestId': '1000003000.69', 'loaderId': 'D7814CD633EDF3E699523AF0C4E9DB2C', 'timestamp': 207483.974238, 'type': 'Script', 'response': {'url': '[https://www.python.org/static/js/libs/masonry.pkgd.min.js](https://www.python.org/static/js/libs/masonry.pkgd.min.js)', 'status': 200, 'statusText': '', 'headers': {'date': 'Sat, 05 Oct 2019 08:18:34 GMT', 'via': '1.1 vegur, 1.1 varnish, 1.1 varnish', 'last-modified': 'Tue, 24 Sep 2019 18:31:03 GMT', 'server': 'nginx', 'age': '290358', 'etag': '"5d8a60e7-6643"', 'x-served-by': 'cache-iad2137-IAD, cache-tyo19928-TYO', 'x-cache': 'HIT, HIT', 'content-type': 'application/x-javascript', 'status': '200', 'cache-control': 'max-age=604800, public', 'accept-ranges': 'bytes', 'x-timer': 'S1570263515.866582,VS0,VE0', 'content-length': '26179', 'x-cache-hits': '1, 170'}, 'mimeType': 'application/x-javascript', 'connectionReused': False, 'connectionId': 0, 'remoteIPAddress': '151.101.108.223', 'remotePort': 443, 'fromDiskCache': True, 'fromServiceWorker': False, 'fromPrefetchCache': False, 'encodedDataLength': 0, 'timing': {'requestTime': 207482.696803, 'proxyStart': -1, 'proxyEnd': -1, 'dnsStart': -1, 'dnsEnd': -1, 'connectStart': -1, 'connectEnd': -1, 'sslStart': -1, 'sslEnd': -1, 'workerStart': -1, 'workerReady': -1, 'sendStart': 0.079, 'sendEnd': 0.079, 'pushStart': 0, 'pushEnd': 0, 'receiveHeadersEnd': 0.836}, 'protocol': 'h2', 'securityState': 'unknown'}, 'frameId': 'A2971702DE69F008914F18EAE6514DD5'}}

\# listening response

def filter\_function(r):

url = r\["params"\]\["response"\]\["url"\]

ok = "[cdn.staticfile.org/vue/2.6.10/vue.min.js](http://cdn.staticfile.org/vue/2.6.10/vue.min.js)" in url

logger.warning("get response url: %s %s" % (url, ok))

return ok

task = asyncio.ensure\_future(

tab.wait\_response(

filter\_function=filter\_function, response\_body=True, timeout=10

)

)

await tab.set\_url("[https://cdn.staticfile.org/vue/2.6.10/vue.min.js](https://cdn.staticfile.org/vue/2.6.10/vue.min.js)")

result = await task

ok = "Released under the MIT License" in result\["data"\]

logger.warning(f"check wait\_response callback, get\_response {ok}")

assert ok

\# test wait\_response\_context

def filter\_function2(r):

try:

url = r\["params"\]\["response"\]\["url"\]

ok = "[cdn.staticfile.org/vue/2.6.10/vue.min.js](http://cdn.staticfile.org/vue/2.6.10/vue.min.js)" in url

return ok

except KeyError:

pass

async with tab.wait\_response\_context(

filter\_function=filter\_function2, timeout=5

) as r:

await tab.goto("[https://cdn.staticfile.org/vue/2.6.10/vue.min.js](https://cdn.staticfile.org/vue/2.6.10/vue.min.js)")

result = (await r) or {}

assert "Released under the MIT License" in result.get("data", ""), result

async def test\_tab\_js\_onload(tab: AsyncTab):

\# add js onload

js\_id = await tab.add\_js\_onload(source="window.title=123456789")

assert js\_id

await tab.set\_url("[https://bing.com](https://bing.com)")

assert (await tab.get\_variable("window.title")) == 123456789

\# remove js onload

assert await tab.remove\_js\_onload(js\_id)

await tab.set\_url("[https://bing.com](https://bing.com)")

assert (await tab.get\_variable("window.title")) != 123456789

assert (await tab.get\_variable("\[1, 2, 3\]")) != \[1, 2, 3\]

assert (await tab.get\_variable("\[1, 2, 3\]", jsonify=True)) == \[1, 2, 3\]

current\_url = await tab.current\_url

url = await tab.url

assert "[bing.com](http://bing.com)" in url and "[bing.com](http://bing.com)" in current\_url, (url, current\_url)

async def test\_tab\_current\_html(tab: AsyncTab):

html = await tab.get\_html()

assert "Customer name:" in html

\# alias current\_html

assert html == (await tab.current\_html) == (await tab.html)

async def test\_tab\_screenshot(tab: AsyncTab):

\# screenshot

screen = await tab.screenshot()

part = await tab.screenshot\_element("fieldset")

assert screen

assert part

assert len(screen) > len(part)

async def test\_tab\_set\_ua\_headers(tab: AsyncTab):

\# test set\_ua

await tab.set\_ua("Test UA")

\# test set\_headers

await tab.set\_headers({"A": "1", "B": "2"})

for \_ in range(3):

if await tab.set\_url("[http://httpbin.org/get](http://httpbin.org/get)", timeout=10):

break

html = await tab.get\_html()

assert (

'"A": "1"' in html and '"B": "2"' in html and '"User-Agent": "Test UA"' in html

)

async def test\_tab\_keyboard\_mouse(tab: AsyncTab):

if "[httpbin.org/forms/post](http://httpbin.org/forms/post)" not in (await tab.current\_url):

await tab.set\_url("[http://httpbin.org/forms/post](http://httpbin.org/forms/post)", timeout=5)

rect = await tab.get\_bounding\_client\_rect('\[type="tel"\]')

await tab.mouse\_click(rect\["left"\], rect\["top"\], count=1)

await tab.keyboard\_send(text="1")

await tab.keyboard\_send(text="2")

await tab.keyboard\_send(text="3")

await tab.keyboard\_send(string="123")

await tab.mouse\_click(rect\["left"\], rect\["top"\], count=2)

selection = await tab.get\_variable("window.getSelection().toString()")

assert selection == "123123"

\# test mouse\_click\_element

await tab.mouse\_click\_element\_rect('\[type="tel"\]')

selection = await tab.get\_variable("window.getSelection().toString()")

assert selection == ""

\# test mouse\_drag\_rel\_chain draw a square, sometime witeboard load failed.....

await tab.set\_url("[https://zhoushuo.me/drawingborad/](https://zhoushuo.me/drawingborad/)", timeout=5)

await tab.mouse\_click(5, 5)

\# drag with moving mouse, released on each `await`

walker = (

await tab.mouse\_drag\_rel\_chain(320, 145)

.move(50, 0, 0.2)

.move(0, 50, 0.2)

.move(-50, 0, 0.2)

.move(0, -50, 0.2)

)

await walker.move(50 _1.414, 50_ 1.414, 0.2)

async def test\_iter\_events(tab: AsyncTab):

async with tab.iter\_events(\["Page.loadEventFired"\], timeout=8) as events\_iter:

await tab.goto("[http://bing.com](http://bing.com)", timeout=0)

data = await events\_iter

assert data, data

await tab.goto("[http://bing.com](http://bing.com)", timeout=0)

data = await events\_iter.get()

assert data, data

await tab.goto("[http://bing.com](http://bing.com)", timeout=0)

async for data in events\_iter:

assert data

break

def cb(event, tab, buffer):

return tab

async with tab.iter\_events({"Page.loadEventFired": cb}, timeout=8) as events\_iter:

await tab.goto("[http://bing.com](http://bing.com)", timeout=0)

data = await events\_iter

assert type(data) == type(tab), data

\# test iter\_fetch

async with tab.iter\_fetch(

patterns=\[{"urlPattern": "\*[postman-echo.com](http://postman-echo.com)\*", "resourceType": "Document"}\]

) as f:

url = "[https://postman-echo.com/get](https://postman-echo.com/get)"

await tab.goto(url, timeout=0)

data = await f

\# print(data, flush=True)

assert data

\# test continueRequest

await f.continueRequest(data)

assert await tab.wait\_includes('"host"', max\_wait\_time=5), await tab.html

await tab.goto(url, timeout=0)

data = await f

assert data

\# test modify response

await f.fulfillRequest(data, 200, body=b"hello world.")

assert await tab.wait\_includes("hello world.", max\_wait\_time=3)

await tab.goto(url, timeout=0)

data = await f

assert data

await f.failRequest(data, "AccessDenied")

assert (await tab.url).startswith("chrome-error://")

\# response fetch

url = "[https://postman-echo.com/get](https://postman-echo.com/get)"

RequestPatternList = \[

{"urlPattern": "\*[postman-echo.com](http://postman-echo.com)\*", "requestStage": "Response"}

\]

async with tab.iter\_fetch(RequestPatternList) as f:

await tab.goto(url, timeout=0)

\# only one request could be catched

event = await f

assert f.match\_event(event, RequestPatternList\[0\]), event

\# print('request event:', json.dumps(event), flush=True)

response = await f.get\_response(event, timeout=5)

\# print('response body:', response)

assert '"host"' in response\["data"\], response

async def test\_init\_tab(chromed: AsyncChromeDaemon):

\# test init tab from chromed, create new tab and auto\_close=True

async with chromed.connect\_tab(index=None, auto\_close=True) as tab:

TEST\_DEFAULT\_CB\_OK = False

def test\_default\_cb(tab, data\_dict):

nonlocal TEST\_DEFAULT\_CB\_OK

TEST\_DEFAULT\_CB\_OK = True

tab.default\_recv\_callback.append(test\_default\_cb)

\# await tab.goto("[https://staticfile.org/about\_en.html](https://staticfile.org/about_en.html)", timeout=5)

\# title = await tab.current\_title

\# assert TEST\_DEFAULT\_CB\_OK, "test default\_recv\_callback failed"

\# assert title == "About - StaticFile CDN", repr(title)

tab.default\_recv\_callback.clear()

async def test\_fetch\_context(tab: AsyncTab):

async with tab.iter\_fetch(patterns=\[{"urlPattern": "\*[bing.com](http://bing.com)\*"}\]) as f:

await tab.goto("[http://bing.com](http://bing.com)", timeout=0)

async for r in f:

assert "[bing.com](http://bing.com)" in r\["params"\]\["request"\]\["url"\]

await f.continueRequest(r)

break

async def cb(event, tab, buffer):

assert "[bing.com](http://bing.com)" in r\["params"\]\["request"\]\["url"\]

await buffer.continueRequest(event)

async with tab.iter\_fetch(

patterns=\[{"urlPattern": "\*[bing.com](http://bing.com)\*"}\],

callback=cb,

) as f:

await tab.goto("[http://bing.com](http://bing.com)", timeout=0)

async for r in f:

break

async def test\_port\_forwarding(host, port):

from ichrome.utils import PortForwarder

dst\_port = port + 100

async with PortForwarder((host, port), (host, dst\_port)):

r = await Requests().get(f"http://{host}:{dst\_port}/json/version", timeout=5)

assert "webSocketDebuggerUrl" in r.text, r.text

r = await Requests().get(f"http://{host}:{dst\_port}/json/version", timeout=5)

assert "webSocketDebuggerUrl" not in r.text, r.text

async def test\_duplicated\_key\_error(tab: AsyncTab):

_task = asyncio.create_task(tab.wait\_loading(timeout=3))

try:

await asyncio.create\_task(tab.wait\_loading(timeout=3))

except ValueError:

return await \_task

raise AssertionError

async def test\_tab\_new\_tab(chromed: AsyncChromeDaemon):

async with chromed.incognito\_tab() as tab:

url = "[http://www.bing.com/](http://www.bing.com/)"

await tab.goto(url, timeout=3)

MUIDB = (await tab.get\_cookies\_dict(\[url\])).get("MUIDB")

new\_tab = await [tab.new](http://tab.new)\_tab()

tab\_exist = bool(await [tab.chrome](http://tab.chrome).get\_tab(new\_[tab.tab](http://tab.tab)\_id))

assert tab\_exist

async with new\_tab(auto\_close=True) as tab:

\# same context, so same cookie

MUIDB2 = (await tab.get\_cookies\_dict(\[url\])).get("MUIDB")

\# print(MUIDB, MUIDB2, MUIDB == MUIDB2)

assert MUIDB == MUIDB2

\# the new\_tab auto closed

tab\_exist = bool(await [tab.chrome](http://tab.chrome).get\_tab(new\_[tab.tab](http://tab.tab)\_id))

assert not tab\_exist

async def test\_examples():

def on\_startup(chromed):

chromed.started = 1

def on\_shutdown(chromed):

chromed.\_\_class\_\_.bye = 1

host = "127.0.0.1"

port = 9222

async with AsyncChromeDaemon(

host=host,

port=port,

headless=headless,

on\_startup=on\_startup,

on\_shutdown=on\_shutdown,

clear\_after\_shutdown=True,

) as chromed:

\# test utils port forwarding

await test\_port\_forwarding(host, port)

await test\_init\_tab(chromed)

[logger.info](http://logger.info)("test init tab from chromed OK.")

\# test on\_startup

assert chromed.started

[logger.info](http://logger.info)("test on\_startup OK.")

\# test [tab.new](http://tab.new)\_tab

await test\_tab\_new\_tab(chromed)

[logger.info](http://logger.info)("test\_tab\_new\_tab OK.")

\# ===================== Chrome Test Cases =====================

async with AsyncChrome(host=host, port=port) as chrome:

\# memory = chrome.get\_memory()

\# assert memory > 0

\# [logger.info](http://logger.info)("get\_memory OK. (%s MB)" % memory)

\# await test\_chrome(chrome)

\# [logger.info](http://logger.info)("test\_chrome OK.")

\# # ===================== Tab Test Cases =====================

\# # Duplicate, use async with chrome.connect\_tab(None) instead

tab: AsyncTab = await [chrome.new](http://chrome.new)\_tab()

\# assert not tab.\_target\_info

\# await test\_tab\_ws(tab)

\# # same as: async with tab.connect():

async with tab():

\# assert [tab.info](http://tab.info)

\# # test duplicated event key issue

\# await test\_duplicated\_key\_error(tab)

\# [logger.info](http://logger.info)("test\_duplicated\_key\_error OK.")

\# # test send raw message

\# await test\_send\_msg(tab)

\# [logger.info](http://logger.info)("test\_send\_msg OK.")

\# # test cookies operations

\# await test\_tab\_cookies(tab)

\# [logger.info](http://logger.info)("test\_tab\_cookies OK.")

\# # test\_browser\_context

\# await test\_browser\_context(tab, chrome, chromed)

\# [logger.info](http://logger.info)("test\_browser\_context OK.")

\# # set url

\# await test\_tab\_set\_url(tab)

\# [logger.info](http://logger.info)("test\_tab\_set\_url OK.")

\# # test js

\# await test\_tab\_js(tab)

\# [logger.info](http://logger.info)("test\_tab\_js OK.")

\# # test wait\_response

await test\_wait\_response(tab)

[logger.info](http://logger.info)("test\_wait\_response OK.")

\# # test fetch\_context

\# await test\_fetch\_context(tab)

\# [logger.info](http://logger.info)("test\_fetch\_context OK.")

\# # test add\_js\_onload remove\_js\_onload

\# await test\_tab\_js\_onload(tab)

\# [logger.info](http://logger.info)("test\_tab\_js\_onload OK.")

\# # test iter\_events iter\_fetch

\# await test\_iter\_events(tab)

\# [logger.info](http://logger.info)("test\_iter\_events OK.")

\# # test set ua and set headers

\# await test\_tab\_set\_ua\_headers(tab)

\# [logger.info](http://logger.info)("test\_tab\_set\_ua\_headers OK.")

\# # load url for other tests

\# await tab.set\_url("[http://httpbin.org/forms/post](http://httpbin.org/forms/post)")

\# # test current\_html

\# await test\_tab\_current\_html(tab)

\# [logger.info](http://logger.info)("test\_tab\_current\_html OK.")

\# # test screenshot

\# await test\_tab\_screenshot(tab)

\# [logger.info](http://logger.info)("test\_tab\_screenshot OK.")

\# # test double click some positions. test keyboard\_send input

\# await test\_tab\_keyboard\_mouse(tab)

\# [logger.info](http://logger.info)("test\_tab\_keyboard\_mouse OK.")

\# # clear cache

\# assert await tab.clear\_browser\_cache()

\# [logger.info](http://logger.info)("clear\_browser\_cache OK.")

\# # close tab

\# await tab.close()

\# test chrome.connect\_tab

\# async with chrome.connect\_tab(chrome.server + "/json", True) as tab:

\# await tab.wait\_loading(2)

\# assert "webSocketDebuggerUrl" in (await tab.current\_html)

\# [logger.info](http://logger.info)("test connect\_tab OK.")

\# # close\_browser gracefully, I have no more need of chrome instance

\# await chrome.close\_browser()

\# # await chrome.kill()

\# sep = f'\\n{"=" \* 80}\\n'

\# logger.critical(

\# f"{sep}Congratulations, all test cases passed(flatten={AsyncTab.\_DEFAULT\_FLATTEN}).{sep}"

\# )

assert AsyncChromeDaemon.bye

\# test clear\_after\_shutdown

_user_dir\_path = AsyncChromeDaemon.DEFAULT\_USER\_DIR\_PATH / "chrome\_9222"

dir\_cleared = not _user_dir\_[path.is](http://path.is)\_dir()

assert dir\_cleared

await asyncio.sleep(1)

async def test\_chrome\_engine():

async def _test_chrome\_engine():

tab\_callback1 = r"""async def tab\_callback(self, tab, url, timeout):

await tab.set\_url(url, timeout=5)

return 'Bing' in (await tab.title)"""

async def tab\_callback2(self, tab, url, timeout):

await tab.set\_url(url, timeout=5)

return "Bing" in (await tab.title)

async with ChromeEngine(

max\_concurrent\_tabs=5,

headless=True,

disable\_image=True,

after\_shutdown=lambda cd: cd.clear\_dir\_with\_shutil(cd.user\_data\_dir),

) as ce:

\# test normal usage

tasks = \[

asyncio.create\_task(

[ce.do](http://ce.do)("[https://bing.com](https://bing.com)", tab\_callback1, timeout=10)

)

for \_ in range(3)

\] + \[

asyncio.create\_task(

[ce.do](http://ce.do)("[https://bing.com](https://bing.com)", tab\_callback2, timeout=10)

)

for \_ in range(3)

\]

for task in tasks:

assert (await task) is True

\# test screenshot full screen and partial tag range.

tasks = \[

asyncio.create\_task(

ce.screenshot("[https://bing.com](https://bing.com)", "#sbox", timeout=10)

),

asyncio.create\_task(ce.screenshot("[https://bing.com](https://bing.com)", timeout=10)),

\]

results = \[await task for task in tasks\]

assert 1000 < len(results\[0\]) < len(results\[1\])

\# test download

tasks = \[

asyncio.create\_task(

[ce.download](http://ce.download)("[https://bing.com](https://bing.com)", "#sbox", timeout=10)

),

asyncio.create\_task([ce.download](http://ce.download)("[https://bing.com](https://bing.com)", timeout=10)),

\]

results = \[await task for task in tasks\]

assert 1000 < len(results\[0\]\["tags"\]\[0\]) < len(results\[1\]\["html"\])

\# test connect\_tab

async with ce.connect\_tab() as tab:

await tab.goto("[http://pypi.org](http://pypi.org)")

title = await tab.title

assert "PyPI" in title, title

logger.critical("test\_chrome\_engine OK")

_user_dir\_path = (

AsyncChromeDaemon.DEFAULT\_USER\_DIR\_PATH

/ f"chrome\_{ChromeEngine.START\_PORT}"

)

dir\_cleared = not _user_dir\_[path.is](http://path.is)\_dir()

#assert dir\_cleared

await _test_chrome\_engine()

def test\_all():

import time

AsyncChromeDaemon.DEFAULT\_USER\_DIR\_PATH = Path("./ichrome\_user\_data")

loop = [asyncio.new](http://asyncio.new)\_event\_loop()

try:

for flatten in \[True, False, True\]:

logger.critical("Start testing flatten=%s." % flatten)

AsyncTab.\_DEFAULT\_FLATTEN = flatten

\# [loop.run](http://loop.run)\_until\_complete(test\_chrome\_engine())

\# time.sleep(1)

[loop.run](http://loop.run)\_until\_complete(test\_examples())

time.sleep(1)

finally:

loop.close()

if **name** == "\_\_main\_\_":

test\_all()