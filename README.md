# PicACG Web App

<div align="center">

Deploy your own

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2FFreeNowOrg%2FPicaComicNow&demo-title=PicACG%20Web%20App&demo-url=https%3A%2F%2Fpica-comic.vercel.app)

想用的话点上面的按钮自己部署一个，一直点下一步就行了。别让我的被封了，我还要看本子呢，谢谢。

</div>

## 这是来自本fork的改编者的提示
请尽量将`/api/utils.ts`中的`picaimg.pro-ivan.cn`更改为自己的图床转发端口，因为若出现恶意使用requests次数的情况，我们会随时加以reference限制；为避免站点故障请尽量自己搭建图床转发，推荐的方法是使用Cloudflare Workers。

下面是Workers需要的代码，仅供参考，不代表绝对不会出现bug<del>（倒不如说我不知道为啥这坨玩意儿不会报错）</del>：
```
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)
  const path = url.pathname

  if (path.startsWith('/static/tobs/')) {
    const targetUrl = 'https://storage-b.picacomic.com/static/' + path.substring('/static/tobs/'.length)
    return fetch(targetUrl, request)
  }

  if (path.startsWith('/static/tobeimg/')) {
    const targetUrl = 'https://img.picacomic.com/' + path.substring('/static/tobeimg/'.length)
    return fetch(targetUrl, request)
  }

  const targetUrl = 'https://s3.picacomic.com' + path
  const modifiedHeaders = new Headers(request.headers)
  modifiedHeaders.set('Host', 's3.picacomic.com')
  modifiedHeaders.set('X-Real-IP', request.headers.get('CF-Connecting-IP'))
  modifiedHeaders.set('X-Forwarded-For', request.headers.get('CF-Connecting-IP'))
  modifiedHeaders.set('REMOTE-HOST', request.headers.get('CF-Connecting-IP'))
  modifiedHeaders.set('Upgrade', request.headers.get('Upgrade'))
  modifiedHeaders.set('Connection', request.headers.get('Connection'))
  modifiedHeaders.set('Accept-Encoding', '')
  modifiedHeaders.delete('Cookie')
  modifiedHeaders.delete('Cache-Control')
  modifiedHeaders.delete('expires')

  const modifiedRequest = new Request(targetUrl, {
    method: request.method,
    headers: modifiedHeaders,
    body: request.body,
    redirect: 'manual'
  })

  const response = await fetch(modifiedRequest)
  const modifiedResponse = new Response(response.body, response)
  modifiedResponse.headers.set('X-Cache', response.headers.get('CF-Cache-Status'))

  if (path.match(/\.(gif|png|jpg|css|js|woff|woff2)$/i)) {
    modifiedResponse.headers.set('Expires', new Date(Date.now() + 86400000).toUTCString())
  }

  return modifiedResponse
}
```

## Attention please

This is a fan made website. We are NOT PicACG official. Please DO NOT share this website anywhere.

这是一个粉丝向网站，我们与 PicACG 官方没有任何关系。请勿在任何地方传播本网站。

---

_For communication and learning only._

**All data & pictures from query:** &copy;PICA & Illusts' authors

> Copyright 2021 FreeNowOrg
>
> Licensed under the Apache License, Version 2.0 (the "License");<br>
> you may not use this file except in compliance with the License.<br>
> You may obtain a copy of the License at
>
> http://www.apache.org/licenses/LICENSE-2.0
>
> Unless required by applicable law or agreed to in writing, software<br>
> distributed under the License is distributed on an "AS IS" BASIS,<br>
> WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.<br>
> See the License for the specific language governing permissions and<br>
> limitations under the License.

