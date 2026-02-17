# Bug Explanation

## What is the bug?

When `oauth2_token` was set to a plain dictionary like `{"access_token": "stale", "expires_at": 0}` 
instead of a proper `OAuth2Token` object, the refresh never happened. The request went out with 
no `Authorization` header instead of `"Bearer fresh-token"`.

## Why did it happen?

The original condition was:

if not self.oauth2_token or (isinstance(self.oauth2_token, OAuth2Token) and self.oauth2_token.expired):

Python treats a non-empty dictionary as truthy — meaning "I have something". So `not self.oauth2_token` 
becomes `False`. The second part also fails because the dict is not an `OAuth2Token` instance. 
Both sides are `False`, so `refresh_oauth2()` is never called, and no header gets attached.

## Why does the fix solve it?

The replacement condition is:

if not isinstance(self.oauth2_token, OAuth2Token) or self.oauth2_token.expired:

Instead of asking "do I have something?", it now asks "do I have the RIGHT thing?". 
Anything that is not a proper `OAuth2Token` — whether `None`, a dict, or anything else — 
triggers a refresh. Only a valid, non-expired `OAuth2Token` passes through.

## One edge case the tests still don't cover

If two threads run at the same time, one thread could replace the token with a dict 
*after* the check passes but *before* the header is attached. The header would silently 
be omitted with no error raised.