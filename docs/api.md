## Single Session API

### login(options) => Promise:void
Kick off the login process for the user:

```typescript
import { Session } from '@inrupt/solid-client-authn-browser';

const session = new Session(/*...*/);

await session.login({
    oidcIssuer: 'https://identityprovider.com', 
    redirectUrl: 'https://mysite.com/redirect'
});
```

Options:
| Field Name | Required?                                          | Type          | Description                                                                                                         | Default   |
|------------|----------------------------------------------------|---------------|---------------------------------------------------------------------------------------------------------------------|-----------|
| `oidcIssuer`   | Yes | String or URL | The user's issuer                                                                                                   | undefined |
| `redirectUrl` | Yes                                                | String or URL | The URI within this application that the user should be redirected to after successful login. This can be either a web URL or a mobile URL scheme | undefined |
| `clientId` | Only if you don't want to do [dynamic registration](https://tools.ietf.org/html/rfc7591) | String or URL | The id of a statically registered application.                                                                                                     | undefined |
| `clientSecret` | Only if you don't want to do [dynamic registration](https://tools.ietf.org/html/rfc7591) | String or URL | The secret of a statically registered application. __Warning__: do not use this in a web browser environment.                                                                                                     | undefined |
| `popUp`    | No                                                 | Boolean       | If true, the login process will initiate via a popup. This only works on web clients.                              | false     |
| `handleRedirect`    | No                                                 | `(redirectUrl) => {}` or `"auto"`        | If a function is provided, the browser will not auto-redirect and will instead trigger that function to redirect. If "auto" or undefined, the browser will auto-redirect given it is in a browser environment. | "auto" |

### handleIncomingRedirect(url) => Promise:void
Handles redirects from the identity provider to wrap up the login process. 

```typescript
import { Session } from '@inrupt/solid-client-authn-browser';

const session = new Session(/*...*/);

session.handleIncomingRedirect(window.location.href);
```

### fetch(url, options) => Promise:result

Send an HTTP request to a Solid Pod:

```typescript
import { Session } from '@inrupt/solid-client-authn-browser';

const session = new Session(/*...*/);

session.fetch('https://example.com/resource', {
    method: 'POST',
    headers: {
        "Content-Type": "text/plain"
    },
    body: 'What a cool body!'
}).then((result) => {});
```
Fetch follows the [WHATWG Fetch Standard](https://github.github.io/fetch/).

### logout() => Promise:void

Log the user out:

```typescript
import { Session } from '@inrupt/solid-client-authn-browser';

const session = new Session(/*...*/);

session.logout().then(() => {});
```

### onLogin(callback) => Promise:void

Register a callback function to be called when a user completes login:

```typescript
import { Session } from '@inrupt/solid-client-authn-browser';

const session = new Session(/*...*/);

session.onLogin((sessionInfo) => {
  console.log(session.webId)
});
```

### onLogout(callback) => Promise:void

Register a callback function to be called when a user logs out:

```typescript
import { Session } from '@inrupt/solid-client-authn-browser';

const session = new Session(/*...*/);
session.onLogout(() => {});
```

## Multi Session API

### class SessionManager

A class that manages all sessions in your application.

#### constructor(options)

Creates a SessionManager object.

```typescript
import { SessionManager } from "@inrupt/solid-client-authn-browser";
import customStorage from "./myCustomStorage";

const sessionManager = new SessionManager({
  secureStorage: customStorage
});
```

Options:
| Field Name | Required? | Type     | Description                                              | Default           |
|------------|-----------|----------|----------------------------------------------------------|-------------------|
| `secureStorage`  | No        | [IStorage](IStorage) | A storage object to help you access your custom storage. A secure storage environment should not be accessible by a third party  | In Memory Storage |
| `insecureStorage`  | No        | [IStorage](IStorage) | A storage object to help you access your custom storage. An insecure storage environment could be accessible by a third party  | In Memory Storage |
| `clientId`  | No        | string | The client id of the application if it was statically registered | undefined |
| `clientSecret`  | No        | string | The client secret of the application if it was statically registered  | undefined |

#### getSessions() =>  Promise:[Session](#class-session)[]

Returns all sessions currently managed by the session manager.

```typescript
sessionManager.getSessions().then(sessions => {});
```

#### getSession(sessionId?) => Promise:[Session](#class-session)

Creates a new session and adds it to the session manager. If a session ID is not provided then
a random UUID will be assigned as the session ID. If the session of the provided ID already exists
then that session will be returned.

```typescript
sessionManager.getSession("mySessionid").then((session) => {});
```

#### hasSession(sessionId) => Promise:[Session](#class-session) | null

Returns true if the session has already been created.

```typescript
sessionManager.hasSession("mySessionId").then(doesSessionExist => {});
```

#### onSessionLogin(callback) => void or on("sessionLogin", callback) => void

Registers a callback to be called when a session is logged in.

```typescript
sessionManager.onSessionLogin((session) => {});
sessionManager.on("sessionLogin", (session) => {});
```

#### onSessionLogout(callback) => void or on("sessionLogout" callback)

Registers a callback to be called when a session is logged out.

```typescript
sessionManager.onSessionLogout((session) => {});
sessionManager.on("sessionLogout", (session) => {});
```

#### handleIncomingRedirect(url) => void

Part of the login flow is a redirect. Use `handleIncomingRedirect` at the redirect route for your app.

```typescript
app.get("/redirect", async (req, res) => {
  await sessionManager.handleInboundRedirect(req.url);
  res.redirect("/home");
})
```

### class Session

#### Properties

 - sessionId: string - an identifier for the session
 - webId: string - the WebID of the current user
 - isLoggedIn: boolean - true if the user is logged in

#### constructor(sessionManager, sessionId?)

Creates a Session object. If `sessionId` is not present, a random UUID will be generated.

**Warning**: you should either use `sessionManager.getSession()` or call `session.init()`
directly after calling the constructor.

```typescript
import { Session } from "@inrupt/solid-client-authn-browser-sessions";

const session = new Session(sessionManager, "mySessionId");
session.init().then(() => {});
```

#### login(options): => Promise:[NeededAction](#neededaction)

Kick off the login process for this session:

```typescript
session.login({
    oidcIssuer: 'https://identityprovider.com', 
    redirectUrl: 'https://mysite.com/redirect'
}).then((neededAction) => {});
```

Options:
| Field Name | Required?                                          | Type          | Description                                                                                                         | Default   |
|------------|----------------------------------------------------|---------------|---------------------------------------------------------------------------------------------------------------------|-----------|
| `oidcIssuer`   | Yes | String or URL | The user's OIDC issuer                                                                                              | undefined |
| `redirectUrl` | Yes                                                | String or URL | The URI within this application that the user should be redirected to after successful login. This can be either a web URL or a mobile URL scheme | undefined |
| `clientId` | Only if you don't want to do [dynamic registration](https://tools.ietf.org/html/rfc7591) | String or URL | The ID of a statically registered application.                                                                                                     | undefined |
| `clientSecret` | Only if you don't want to do [dynamic registration](https://tools.ietf.org/html/rfc7591) | String or URL | The secret of a statically registered application. __Warning__: do not use this in a web browser environment.                                                                                                     | undefined |
| `popUp`    | No                                                 | Boolean       | If true, the login process will initiate via a popup. This only works on web clients.                              | false     |
| `handleRedirect`    | Yes                                                 | `(redirectUrl) => {}` or `"auto"`        | If a function is provided the browser will not auto-redirect, and will instead trigger this function to perform the redirect. If "auto", the browser will auto-redirect given it is in a browser environment. | undefined |

#### fetch(url, requestInit) => Promise:response

Send an HTTP request to a Solid Pod:

```typescript
session.fetch('https://example.com/resource', {
    method: 'POST',
    headers: {
        "Content-Type": "text/plain"
    },
    body: 'What a cool body!'
}).then((result) => {});
```

Fetch follows the [WHATWG Fetch Standard](https://github.github.io/fetch/).

#### logout() => Promise:void

Log the user out:

```typescript
session.logout().then(() => {});
```

#### onLogin(callback) => void or on("login", callback) => void

Registers a callback to be called whenever this session logs in.

```typescript
sessionManager.onLogin((session) => {});
sessionManager.on("login", (session) => {});
```

#### onLogout(callback) => void or on("logout" callback)

Registers a callback to be called whenever this session logs out.

```typescript
sessionManager.onLogout((session) => {});
sessionManager.on("logout", (session) => {});
```

#### IStorage

If you'd like to configure custom storage, provide an object following this interface:

```typescript
{
  get: (key: string) => Promise<string | null>;
  set: (key: string, value: string) => Promise<void>;
  delete: (key: string) => Promise<boolean>;
  getEntries: () => Promise<Record<string, string>>;
}
```
