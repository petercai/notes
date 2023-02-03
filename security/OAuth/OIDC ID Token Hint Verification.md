# OIDC ID Token Hint Verification
Step-Up authentication is a process where the user is prompted to provide additional authentication information to access a resource. Here's how ID Token Hint can be used in a Step-Up authentication scenario:

1.  In your client application, store the ID Token received from the authentication response after a successful login.
    
2.  When attempting to access a resource that requires a higher level of authentication, include the ID Token as a "id\_token\_hint" query parameter in the request to the OIDC provider.
    
3.  The OIDC provider will receive the request and validate the ID Token Hint. This may include checking the signature, expiration time, and issuer of the token.
    
4.  If the ID Token is valid, the OIDC provider can use the information in the token to determine which user is attempting to access the resource.
    
5.  Based on the user's authentication level, the OIDC provider can prompt the user for additional authentication information, such as a one-time password (OTP) or biometric authentication.
    
6.  Once the user has provided the additional authentication information, the OIDC provider will verify it and grant access to the resource if it is valid.
    

It is important to secure the ID Token Hint by transmitting it over a secure connection (HTTPS) and encrypting it if necessary.