# What was the bug?

The `request` method failed to handle the case where `oauth2Token` was a plain object (a valid type according to `TokenState`). It only refreshed if the token was `null` or an expired `OAuth2Token` instance.

# Why did it happen?

The logic used `!this.oauth2Token` to check for emptiness. A plain object like `{}` is **truthy**, so it passed that check. Because it also wasn't an `instanceof OAuth2Token`, the code skipped the refresh logic entirely, resulting in no `Authorization` header being added for API calls using plain object tokens.

# Why does your fix actually solve it?

By checking `!(this.oauth2Token instanceof OAuth2Token)`, we ensure that any state that isn't a "ready-to-use" class instance (including `null` and plain objects) triggers a refresh. This guarantees that by the time we reach the header assignment, we have a valid instance.

# What’s one realistic edge case your tests still don’t cover?

The tests don't cover **Clock Skew**. If the client's system clock is significantly behind the server's clock, `this.oauth2Token.expired` might return false locally even though the server will reject the token as expired.