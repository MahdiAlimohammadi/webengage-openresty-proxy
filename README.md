# WebEngage OpenResty Custom Proxy Domain

A lightweight, containerized OpenResty (Nginx + Lua) reverse proxy designed to set up a custom proxy domain for the WebEngage iOS, Android, and Web SDKs. 

This proxy enables you to redirect SDK traffic through your own domain (e.g., to bypass ISP-level blockages or improve latency) while securely validating and filtering requests so your proxy server cannot be abused.

## Why OpenResty?

Unlike a standard Squid or Nginx forward proxy, this project uses **Lua rewrite blocks** to validate request query parameters. It unescapes the target URL and verifies that the destination matches the official WebEngage endpoints before proxying the request. Any attempt to use the proxy for unauthorized domains will receive an `HTTP 403 Forbidden` response.

---

## Features

*   **Dockerized Deployment:** Fast and simple startup using Docker Compose with `openresty/openresty:alpine`.
*   **Lua-Based Target Validation:** Unescapes and validates the target URL parameter to prevent forward-proxy abuse.
*   **Domain Whitelist:** Only permits requests to official WebEngage endpoints (e.g., `c.webengage.com`, `p.webengage.com`, `api.webengage.com`, etc.).
*   **SSL/TLS Termination:** Ready for production with custom domain HTTPS configuration.

---

## Quick Start

### 1. Prerequisites
Ensure you have Docker and Docker Compose installed:
*   [Docker](https://docs.docker.com/get-docker/)
*   [Docker Compose](https://docs.docker.com/compose/install/)

### 2. Configuration
1. Clone this repository to your server:
   ```bash
   git clone https://github.com/MahdiAlimohammadi/webengage-openresty-proxy.git
   cd webengage-openresty-proxy
   ```
2. Open `proxy.conf` and update `server_name` to your actual custom proxy domain (e.g., `proxy.example.com`):
   ```nginx
   server {
       listen 443 ssl;
       server_name your-custom-proxy-domain.com;
       ...
   }
   ```
3. Place your SSL certificate chain (`cert.pem`) and private key (`key.pem`) in the `./certs` folder:
   *   `./certs/cert.pem`
   *   `./certs/key.pem`
4. Start the container:
   ```bash
   docker compose up -d
   ```

By default, the proxy will listen on host port **`443`**. You can modify this in `docker-compose.yml`:
```yaml
ports:
  - "443:443" # Change 443 to your preferred host port if needed
```

---

## How to Test the Proxy

You can verify that the proxy works and restricts access locally (before changing public DNS records) by using the `curl --resolve` flag.

### 1. Verify Allowed Destinations (Should return HTTP 200 OK)
Run this command from your terminal (replace `proxy.example.com` with your actual proxy domain):
```bash
curl -vk --resolve "proxy.example.com:443:127.0.0.1" \
  "https://proxy.example.com/?url=https://c.webengage.com/healthcheck"
```

### 2. Verify Blocked Destinations (Should return HTTP 403 Forbidden)
```bash
curl -vk --resolve "proxy.example.com:443:127.0.0.1" \
  "https://proxy.example.com/?url=https://google.com"
```

---

## Whitelisted Destinations
The Lua script whitelists the following WebEngage hosts:
*   `c.webengage.com`
*   `p.webengage.com`
*   `api.webengage.com`
*   `dashboard.webengage.com`
*   `ssl.widgets.webengage.com`
*   `msdk-files.webengage.com`
*   `wsdk-files.webengage.com`
*   `afiles.webengage.com`
*   `efiles.webengage.com`
*   `ofiles.webengage.com`
*   `afiles-io.webengage.com`
*   `efiles-io.webengage.com`
*   `mfiles.webengage.com`

---

## SDK Integration

Once deployed, configure your WebEngage SDK initialization to point to your custom proxy domain.

### Android SDK Example
Configure your proxy in `WebengageConfig`:
```java
WebChildChildConfiguration proxyConfig = new WebChildChildConfiguration.Builder()
    .setProxyHost("https://your-custom-proxy-domain.com")
    .build();

WebengageConfig config = new WebengageConfig.Builder()
    .setWebengageKey("your-webengage-license-code")
    .setProxyConfiguration(proxyConfig)
    .build();
```

### iOS SDK Example
Set the custom proxy URL in your `plist` file or configure it dynamically:
```swift
let config = WEGObjectiveCConfig()
config.proxyHost = "https://your-custom-proxy-domain.com"
```

### Web SDK Example
```javascript
webengage.init('your-webengage-license-code', {
    baseUrl: 'https://your-custom-proxy-domain.com'
});
```

---

## Author

Created by [Mahdi Alimohammadi](https://github.com/MahdiAlimohammadi).

## License

This project is licensed under the MIT License. See the [LICENSE](./LICENSE) file for details.
