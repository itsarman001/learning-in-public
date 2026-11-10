## 1. Browser components

A browser is a set of cooperating parts, not one program.

| Component | What it does | Interview one-liner |
| --- | --- | --- |
| **UI Layer** | Address bar, back/forward, refresh/stop, bookmarks. Everything outside the page content. | The browser chrome. It never touches page content. |
| **Browser Engine** | Bridge between the UI and the Rendering Engine. Passes user actions (e.g. URL entered) to the rendering engine. Also talks to the Data layer. | The coordinator. |
| **Rendering Engine** | Turns HTML, CSS, and JS into pixels. Blink (Chrome, Edge), WebKit (Safari), Gecko (Firefox). | The part that draws the page. |
| **Networking** | HTTP(S) requests, WebSockets, WebRTC. WebRTC media uses UDP (speed over guaranteed delivery). | Handles all communication. |
| **JS Interpreter** | Parses and executes JavaScript. V8 (Chrome), SpiderMonkey (Firefox), JavaScriptCore (Safari). | The JS runtime. |
| **UI Backend** | Draws basic widgets (dropdowns, combo boxes, windows) using the OS's UI toolkit. | Native widget drawing. |
| **Data Storage** | Cookies, cache, localStorage, IndexedDB, bookmarks. | Persistent browser-side data. |

---

## 2. How the browser renders a page (rendering engine)

**Memory trick: Parse → Style → Layout → Paint → Composite.**

### Step by step

1. **Request → Response.** The browser requests a URL and receives HTML. While reading it, the browser discovers more resources (CSS, JS, images) and requests them too.
2. **Build the DOM (parse HTML).** Bytes → characters → tokens → nodes → tree. `<html>` is the root. The DOM is built **incrementally** while bytes are still arriving.
3. **Build the CSSOM (parse CSS).** CSS rules become a tree of styles. Children inherit from parents. The CSSOM must be **complete** before use, because later rules can override earlier ones.
4. **Execute JS.** Scripts can read and modify both the DOM and the CSSOM.
5. **Render Tree = DOM + CSSOM.** Contains only **visible** nodes with their computed styles.
6. **Layout (reflow).** Computes the exact size and position of each box in pixels (viewport, box model, flexbox, grid). The diagram calls this "wireframes".
7. **Paint.** Decides how to draw each box: text, colors, borders, shadows, images. Produces paint records (the "Paint Tree").
8. **Compositing (layering).** The page is split into layers, rasterized (often on the GPU), and stacked in the correct order.

### What goes into the Render Tree?

| Element | In render tree? | Why |
| --- | --- | --- |
| `display: none` | No | Not rendered at all |
| `<head>`, `<script>`, `<meta>` | No | Not visual |
| `visibility: hidden` | **Yes** | Invisible but still takes up space |
| `opacity: 0` | **Yes** | Still takes up space and can be interacted with |
| Pseudo-elements (`::before`, `::after`) | Yes | They have no DOM node, but they do appear in the render tree |

### Reflow vs repaint vs composite

| Term | Meaning | Triggered by | Cost |
| --- | --- | --- | --- |
| **Reflow (layout)** | Recalculate geometry | Changing `width`, `height`, `margin`, `top`, `font-size`, adding or removing DOM nodes, resizing the window | Highest. Also triggers repaint. |
| **Repaint** | Redraw pixels without changing geometry | Changing `color`, `background`, `box-shadow`, `visibility` | Medium |
| **Composite only** | Move or blend existing layers | `transform`, `opacity` (and `will-change` promotes an element to its own layer) | Cheapest |

**Practical takeaway:** animate `transform` and `opacity`, not `top`/`left`/`width`.

**Layout thrashing:** reading a layout property (e.g. `offsetHeight`) right after writing a style forces the browser to reflow synchronously. Batch your reads, then your writes.

### Blocking

| Resource | Blocking type | Why |
| --- | --- | --- |
| **CSS** | **Render-blocking** | The browser won't paint until the CSSOM is ready. This avoids a flash of unstyled content. |
| **JS** (plain `<script>`) | **Parser-blocking** | HTML parsing stops while the script downloads and executes, because the script might change the DOM. |
| JS that reads styles | Waits for CSSOM | The script needs the computed styles first. |

Blocking can happen at any point. It takes effect as soon as the browser reaches that line of code.

### Script loading attributes

| Attribute | Download | Execution | Order preserved? | Use for |
| --- | --- | --- | --- | --- |
| *(none)* | Blocks the parser | Immediately | Yes | Rarely ideal |
| `defer` | Parallel | After HTML is parsed, before `DOMContentLoaded` | **Yes** | App scripts that need the DOM |
| `async` | Parallel | As soon as downloaded (pauses the parser) | **No** | Independent scripts (analytics, ads) |

### Optimization tips (common interview follow-ups)

- Inline critical CSS, lazy-load the rest.
- Use `defer` for scripts, or place them at the end of `<body>`.
- Minimize DOM size and depth.
- Avoid forced synchronous layouts.
- Use `transform`/`opacity` for animations.
- Use `<link rel="preload">` for key resources.

### Important nuance

Rendering is **not a strict one-way sequence**. The DOM builds while the network is still downloading, JS can trigger reflow and repaint again, and the browser may show partial content before everything has loaded.

---

## 3. Network protocols

### Overview

| Protocol | How it works | Use cases | Default port |
| --- | --- | --- | --- |
| **HTTP** | Client-server request/response over a TCP connection | Web browsing | 80 |
| **HTTP/3 (QUIC)** | HTTP over QUIC, which runs on **UDP** | Faster web, mobile-heavy usage, VR, IoT | 443 |
| **HTTPS** | HTTP + TLS encryption | Secure web browsing | 443 |
| **WebSocket** | Starts with an HTTP Upgrade, then **full duplex** over TCP | Live chat, real-time data (trading, gaming) | 80/443 |
| **TCP** | Reliable, ordered delivery using a 3-way handshake | Web, email, file transfer | n/a |
| **UDP** | Connectionless; send without a handshake | Video conferencing, voice, streaming | n/a |
| **SMTP** | Sender → SMTP server → receiver | Sending email | 25 / 587 |
| **FTP** | Separate control and data channels | Uploading and downloading files | 21 (control) |

### Details

**HTTP**

- Protocol for fetching resources such as HTML documents; the foundation of data exchange on the web.
- Client-server and **stateless** (each request is independent; cookies and sessions add state).

**HTTP/3**

- Runs on **QUIC**, a transport protocol designed for mobile-heavy internet usage.
- Uses **UDP instead of TCP**, which gives faster page responsiveness.
- Avoids TCP's head-of-line blocking, because streams are independent.
- VR apps need more bandwidth for detailed scenes and benefit from HTTP/3.

**HTTPS**

- HTTP with encryption for secure communication (TLS).
- Flow from the diagram: TCP connection → server sends **public key** → client and server agree on a **session key** → **encrypted data** is exchanged.
- Asymmetric encryption (public/private keys) is used to securely exchange a **symmetric session key**, which is then used for fast encryption of the actual data.

**WebSocket**

- Provides **full-duplex** communication over TCP.
- Begins as an HTTP request with an `Upgrade: websocket` header; the server replies **101 Switching Protocols**.
- Unlike REST, which always "pulls" data, WebSocket lets the server **push** data.
- Used in online gaming, stock trading, and messaging apps.

**TCP**

- Designed to send packets across the internet and ensure **successful delivery**.
- Three-way handshake: **SYN → SYN+ACK → ACK**.
- Reliable, ordered, with retransmission of lost packets.
- Many application-layer protocols (HTTP, SMTP, FTP) build on TCP.

**UDP**

- Sends packets directly to the target **without establishing a connection**.
- Used in time-sensitive communication where dropping the occasional packet is better than waiting.
- Voice and video traffic are often sent over UDP.

**SMTP**

- Standard protocol for transferring electronic mail from one user to another.
- Used for **sending** email (receiving uses IMAP or POP3).

**FTP**

- Transfers files between client and server.
- Separate **control channel** (commands) and **data channel** (file contents).

### TCP vs UDP

| Feature | TCP | UDP |
| --- | --- | --- |
| Connection | Connection-oriented (handshake) | Connectionless |
| Reliability | Guaranteed delivery, retransmits | No guarantee |
| Ordering | Ordered | Not ordered |
| Speed | Slower (overhead) | Faster |
| Use cases | Web, email, file transfer | Video calls, gaming, streaming, DNS |

### HTTP versions

| Version | Transport | Key features |
| --- | --- | --- |
| HTTP/1.1 | TCP | Persistent connections, but head-of-line blocking at the request level |
| HTTP/2 | TCP | Multiplexing (many requests on one connection), header compression, binary framing |
| HTTP/3 | QUIC over UDP | Faster connection setup, no TCP head-of-line blocking, built-in encryption |

### HTTP vs WebSocket

|  | HTTP (REST) | WebSocket |
| --- | --- | --- |
| Direction | Client requests, server responds | Both sides can send any time |
| Connection | New request each time (or kept alive) | One persistent connection |
| Best for | Fetching resources, CRUD APIs | Real-time updates, chat, live feeds |

---

## 4. HTTP status codes

### The five classes

| Class | Meaning | Description |
| --- | --- | --- |
| **1XX** | Informational | The server acknowledges the request and is still processing it |
| **2XX** | Success | The request was received, understood, and processed |
| **3XX** | Redirection | The resource is somewhere else; further action is needed |
| **4XX** | **Client error** | The request is wrong or can't be fulfilled (bad request, not found, no permission) |
| **5XX** | **Server error** | The request was valid, but the server failed to complete it |

> Note: the slide's wording for 4XX is a little confusing. The rule to remember is simple: **4XX = the client's fault, 5XX = the server's fault.**

### Codes you must know

| Code | Name | Meaning / when used |
| --- | --- | --- |
| **100** | Continue | Initial part received; client can continue |
| **101** | Switching Protocols | Server agrees to switch protocol (e.g. WebSocket upgrade) |
| **200** | OK | Standard success |
| **201** | Created | New resource created (typical for POST) |
| **202** | Accepted | Request accepted, processing not finished (async jobs) |
| **204** | No Content | Success with no response body (typical for DELETE) |
| **206** | Partial Content | Range request (video streaming, resumable downloads) |
| **301** | Moved Permanently | Resource moved for good; browsers and search engines cache this |
| **302** | Found | Temporary redirect |
| **304** | Not Modified | Cached copy is still valid (conditional request with ETag or Last-Modified) |
| **307** | Temporary Redirect | Temporary redirect that **keeps the HTTP method** |
| **308** | Permanent Redirect | Permanent redirect that **keeps the HTTP method** |
| **400** | Bad Request | Malformed request or invalid input |
| **401** | Unauthorized | Not authenticated (missing or invalid credentials) |
| **403** | Forbidden | Authenticated, but **not allowed** |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | Method not supported on this endpoint |
| **408** | Request Timeout | Server timed out waiting for the request |
| **409** | Conflict | Conflicts with current state (e.g. duplicate) |
| **422** | Unprocessable Entity | Well-formed but fails validation |
| **429** | Too Many Requests | Rate limit exceeded |
| **500** | Internal Server Error | Generic server failure |
| **501** | Not Implemented | Server doesn't support the feature |
| **502** | Bad Gateway | A gateway or proxy got an invalid response from upstream |
| **503** | Service Unavailable | Server overloaded or down for maintenance |
| **504** | Gateway Timeout | A gateway or proxy timed out waiting for upstream |

### Commonly confused pairs

| Pair           | Difference                                                                         |
| -------------- | ---------------------------------------------------------------------------------- |
| **401 vs 403** | 401 = "who are you?" (not logged in). 403 = "I know who you are, but no access."   |
| **301 vs 302** | 301 = permanent (cached, SEO passes to new URL). 302 = temporary.                  |
| **302 vs 307** | 302 may change POST to GET in practice; 307 guarantees the method is preserved.    |
| **400 vs 422** | 400 = malformed syntax. 422 = valid syntax but fails business or validation rules. |
| **502 vs 504** | 502 = bad response from upstream. 504 = no response in time.                       |
| **200 vs 204** | 200 returns a body; 204 intentionally returns none.                                |

---

## 5. Quick-fire interview Q&A

**What happens when you type a URL and press Enter?** UI layer captures the URL → browser engine passes it on → DNS lookup → TCP connection (+ TLS for HTTPS) → HTTP request → server responds with HTML → rendering engine builds DOM and CSSOM → render tree → layout → paint → composite.

**What is the DOM? What is the CSSOM?** The DOM is the tree representation of the HTML. The CSSOM is the tree representation of the CSS. Both combine into the render tree.

**Difference between `display: none` and `visibility: hidden`?** `display: none` removes the element from the render tree (no space). `visibility: hidden` keeps it in the render tree (space is preserved, just invisible).

**Why is CSS render-blocking and JS parser-blocking?** Without the full CSSOM, the browser can't know correct styles and would flash unstyled content. JS can modify the DOM, so the parser must wait for it to finish.

**`async` vs `defer`?** Both download in parallel. `defer` runs after parsing, in order. `async` runs as soon as it's downloaded, in no guaranteed order.

**Reflow vs repaint?** Reflow recalculates geometry (expensive). Repaint redraws pixels with no geometry change. Compositing only moves layers (cheapest).

**Why is animating `transform` faster than `left`?** `transform` only needs compositing on the GPU. `left` triggers layout and paint on every frame.

**HTTP vs HTTPS?** HTTPS is HTTP over TLS: encrypted, authenticated server identity, protected against tampering and eavesdropping.

**Why does HTTP/3 use UDP?** QUIC builds its own reliability, ordering, and encryption on top of UDP. This avoids TCP's head-of-line blocking and speeds up connection setup.

**When would you use WebSocket instead of REST?** When you need real-time, bidirectional, server-pushed data (chat, live scores, collaborative editing).

**TCP vs UDP?** TCP is reliable and ordered with a handshake. UDP is fast and connectionless with no delivery guarantee.

**What does a 304 mean and how does it help performance?** The cached copy is still valid, so the server sends no body. Saves bandwidth and time.

**401 vs 403?** 401 = not authenticated. 403 = authenticated but not authorized.

---

## 6. One-page cheat sheet

- **Browser parts:** UI Layer, Browser Engine, Rendering Engine, Networking, JS Interpreter, UI Backend, Data Storage.
- **Rendering:** Parse HTML → DOM, Parse CSS → CSSOM, merge → Render Tree → Layout → Paint → Composite.
- **Blocking:** CSS blocks rendering. Sync JS blocks parsing. Use `defer`/`async`.
- **Performance:** avoid layout thrashing, animate `transform`/`opacity`.
- **Protocols:** HTTP (stateless, TCP), HTTP/3 (QUIC, UDP), HTTPS (TLS), WebSocket (full duplex, push), TCP (reliable, 3-way handshake), UDP (fast, no connection), SMTP (send email), FTP (control + data channels).
- **Status codes:** 1XX info, 2XX success, 3XX redirect, 4XX client error, 5XX server error.
- **Key codes:** 200, 201, 204, 301, 302, 304, 400, 401, 403, 404, 429, 500, 502, 503, 504.
