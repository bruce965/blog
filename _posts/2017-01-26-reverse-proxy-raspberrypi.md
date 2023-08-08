---
title: Reverse Proxy on the Raspberry Pi
date: 2017-01-26 22:37:00 +0100
categories: [Software]
tags: [Raspberry Pi, Linux]
---

The goal of this guide is to setup a reverse proxy serving multiple virtual hosts from various sources on my network.

After configuring everything, I will be able to access, from the internet, various web applications running on my Raspberry Pi and other machines on my LAN. Port-forwarding my Raspberry Pi's port 8080 from my router's port 80 will thus be necessary.

The following steps assume Node.js and NPM to already be installed.

```bash
mkdir reverse-proxy
cd reverse-proxy

npm install express http-proxy-middleware

nano reverse-proxy.js
```

The following sample code can be customized to use different routes.

```javascript
const express = require('express');
const httpProxyMiddleware = require('http-proxy-middleware');

const app = express();
app.use(httpProxyMiddleware({
	// Unmapped requests forwarded to local port 80.
	target: "http://localhost:80",

	// This is the proxy-table, { "virtual host": "target destination" }.
	router: {
		"blog.fabioiotti.com": "http://localhost:2368",
		"sonerezh.fabioiotti.com": "http://localhost:80"reverse-proxy-raspberrypi
The proxy can be started with `node reverse-proxy.js` command.

A new job should be added to crontab to ensure the proxy is started with the system.

```bash
(crontab -l; echo "@reboot $(which node) $PWD/reverse-proxy.js >/dev/null 2>&1") | crontab -
```

Don't forget that virtual hosts must point to the IP of the machine running this code.

---

> References

> https://github.com/chimurai/http-proxy-middleware/blob/master/recipes/router.md
