## DNS
- **DNS Setup:** Full
- **DNSSEC**: Disable
- **Name Servers:** Cloudflare
	- Если NS записи будут от хостера на котором зарегистрирован домен, то CloudFlare не сможет использовать свою систему проксирования. Поэтому важно перевести все NS записи на те, что предоставляет CF.
	- Документация: https://developers.cloudflare.com/dns/nameservers/update-nameservers/#your-domain-uses-a-different-registrar
	- Сервисы для проверки NS записей
		- https://dnschecker.org/#NS/appenvisions.com 
		- https://www.whatsmydns.net/#NS/appenvisions.com
- **CNAME**
	- **Proxy Status:** Proxied
	- **Name:** webgl
	- **Target:** https://webgl-cdn.plain-dew-83df.workers.dev/
		- Здесь нужно указать на orgin сервер, с которого будут стягиваться необходимые статичные ресурсы. Все CDN эджи работают именно с этим origin сервером (если у ближайшего эджа нет закешированных ресурсов)
		- Мы использовали worker от CF, потому что нам потребовалось указать дополнительные настройки кеширования на эджах.

## Worker


## Caching
#### SSL/TLS
- **TLS 1.3:** Enable
- **Opportunistic Encryption:** Enable
- **Encryption mode:** Full
#### Configuration
- **Caching Level**: Standard
- **Browser Cache TTL**: Respect Existing Headers
- **Development Mode:** Disable
#### Cache Rules
**No Cache for index.html**
- **Custom filter expression**
```js
(http.host eq "webgl.appenvisions.com" and http.request.uri.path eq "/") or (http.request.uri.path eq "/index.html")
```
- **Cache eligibility:** Bypass cache
- **Place at:** 1

**Origin Cache Everything**
- **All incoming requests**
- **Cache eligibility:** Eligible for cache
- **Edge TTL:** Use cache-control header if present, cache request with Cloudflare's default TTL for the response status if not
- **Browser TTL:** Respect origin TTL
- **Serve stale content while revalidating**
	- Do not serve stale content while updating: Disable
- **Respect strong ETags**
	- Use strong ETag headers: Enable
- **Place At:** 2

**Cache .br compressed files**
- **Custom filter expression**
```js
(http.host eq "webgl.appenvisions.com" and ends_with(http.request.uri.path, ".br"))
```
- **Cache eligibility:** Eligible for cache
- **Edge TTL:** Use cache-control header if present, cache request with Cloudflare's default TTL for the response status if not
-  **Serve stale content while revalidating**
	- Do not serve stale content while updating: Disable
- **Respect strong ETags**
	- Use strong ETag headers: Enable
- **Place At:** 3

**WebGL Build Cache**
- **Custom filter expression**
```js
(http.host eq "webgl.appenvisions.com")
```
- **Cache eligibility:** Eligible for cache
- **Edge TTL:** Use cache-control header if present, cache request with Cloudflare's default TTL for the response status if not
- **Serve stale content while revalidating**
	- Do not serve stale content while updating: Disable
- **Respect strong ETags**
	- Use strong ETag headers: Enable
- **Place At:** 4