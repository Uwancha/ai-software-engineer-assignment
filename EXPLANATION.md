## What was the bug?

The HttpClient.request method failed to refresh the OAuth2 token when oauth2Token was a plain object instead of an OAuth2Token instance.

## Why did it happen?

The refresh logic relied on a truthiness check and an instanceof guard:
```typescript
    !this.oauth2Token ||
    (this.oauth2Token instanceof OAuth2Token && this.oauth2Token.expired) 
```

If oauth2Token was a plain object, it was truthy but not an instance of OAuth2Token. Therefore, the condition evaluated to false and refreshOAuth2() was not called. Since the object was also not an OAuth2Token instance, the Authorization header was never set.

## Why does your fix solve it?

The fix changes the condition to:
```typescript
    !(this.oauth2Token instanceof OAuth2Token) ||
    this.oauth2Token.expired
```

Now any non-OAuth2Token value (including plain objects or null) triggers a refresh. This guarantees that only a valid OAuth2Token instance can be used to set the Authorization header.

## One realistic edge case not covered

The tests do not cover the scenario where oauth2Token is an expired OAuth2Token instance. While the implementation checks the expired getter, there is no explicit test verifying that an expired instance triggers a refresh. If the expiration logic were incorrect, the current tests would not detect it.
