# life time
[[API]], hosts a lot of clients, meaning **it is not a one client per usage** as a front side, meaning that global state (program scoped instances) is affecting all users.

# authentication
Authentication is implemented with [[JWT]]
- with security (https, bearer authentication + jwt)
- and with automation *for browser* with cookie setting, and setting parameters for mobile

It is a client specific state, meaning we need to compose an auth details uniquely to the user (its JWT etc).

> **NOTE:** we can share the instances down the stack, for example in a [[middleware]] case, to the actual handler to access info via [[context]]

- **Revocation**: Both approaches require mechanisms for revoking sessions/tokens, but JWTs can use a smaller "revoked tokens" table instead of storing complete session data.


# database
Several database pools placed in one struct `DB` that can be though of as a *holder*,  it is ==common to all usages and clients==, so it is lived entire time, to be global, **single** to all of the clients