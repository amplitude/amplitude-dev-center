If your web app configures the strict Content Security Policy (CSP) for security concerns, adjust the policy to whitelist the Amplitude domains:

* When using ["Script Loader"](https://github.com/amplitude/Amplitude-TypeScript/tree/main/packages/analytics-browser#installing-via-script-loader), add `https://*.amplitude.com` to `script-src`.
* Add `https://*.amplitude.com` to `connect-src`.
