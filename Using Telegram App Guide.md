# Using Telegram App Guide
The `login_hint` parameter is used in OpenID Connect (OIDC) to provide a user identifier to the authorization server to pre-populate the user's email or username in the login form.

Here's how you can use `login_hint` in your OIDC authentication flow:

1.  When redirecting the user to the authorization server's authorization endpoint, include the `login_hint` parameter in the URL, with the value set to the user identifier you want to pre-populate in the form.

Example URL with `login_hint` parameter:

perlCopy code

`https://YOUR_AUTH0_DOMAIN/authorize?
    response_type=code
    &client_id=YOUR_CLIENT_ID
    &redirect_uri=YOUR_REDIRECT_URI
    &scope=openid profile
    &state=YOUR_STATE
    &login_hint=user@example.com` 

2.  On the authorization server, check for the `login_hint` parameter in the request and use its value to pre-populate the login form.
    
3.  After the user logs in and grants permission, the authorization server will redirect the user back to the `redirect_uri` specified in the initial request, along with the authorization code and state.
    

Note: The use of `login_hint` is optional and depends on the authorization server's implementation. Also, be sure to use the `login_hint` parameter securely, as it may contain sensitive information such as the user's email address or username.