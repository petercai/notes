# Convert JWK public key to PEM



## JWK  format of spring reset service



Spring Security, a part of the Spring Framework, allows you to integrate security features, including JWT (JSON Web Token) authentication, into your applications. When configuring JWT in a Spring Security application, you typically provide the public key in the form of a JSON Web Key (JWK) set. This set contains the public keys used to verify JWT signatures.

Here's an example of how the JWK JSON format might look in a Spring Security application:


`{
  "keys": [
    {
      "kid": "1",
      "kty": "RSA",
      "alg": "RS256",
      "n": "your_modulus_base64_encoded",
      "e": "AQAB"
    }
  ]
}` 

In this example:

*   `kid`: Key ID, a unique identifier for the key.
*   `kty`: Key Type, which indicates that it's an RSA key.
*   `alg`: Algorithm, specifying the algorithm used for signing (e.g., RS256 for RSA-SHA256).
*   `n`: Modulus, the Base64-encoded value of the RSA key's modulus.
*   `e`: Exponent, the exponent of the RSA key.

Replace `"your_modulus_base64_encoded"` and `"AQAB"` with the actual values of your RSA public key modulus and exponent, respectively.

## how to convert the response to pem format file


To convert the JSON Web Key (JWK) set response to a PEM format file, you'll need to extract the individual public key components (modulus and exponent) and then create an RSA public key in PEM format. Here's how you can do that:

Assuming you have the JWK set response as a JSON object like the one you mentioned earlier:


`{
  "keys": [
    {
      "kid": "1",
      "kty": "RSA",
      "alg": "RS256",
      "n": "your_modulus_base64_encoded",
      "e": "AQAB"
    }
  ]
}` 

Follow these steps:

1.  Extract the values of `"n"` (modulus) and `"e"` (exponent) from the JSON response.
2.  Decode the Base64-encoded modulus and convert it to a raw byte array.
3.  Decode the Base64-encoded exponent and convert it to a raw byte array.
4.  Combine the modulus and exponent to create an RSA public key.
5.  Convert the RSA public key to PEM format.

Here's a sample Python script that demonstrates this process using the `cryptography` library:



`from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.serialization import Encoding, PublicFormat
import base64
import json

jwk_response = """
{
  "keys": [
    {
      "kid": "1",
      "kty": "RSA",
      "alg": "RS256",
      "n": "your_modulus_base64_encoded",
      "e": "AQAB"
    }
  ]
}
"""

jwk_data = json.loads(jwk_response)
modulus_base64 = jwk_data["keys"][0]["n"]
exponent_base64 = jwk_data["keys"][0]["e"]

modulus_bytes = base64.urlsafe_b64decode(modulus_base64 + "===")
exponent_bytes = base64.urlsafe_b64decode(exponent_base64 + "===")

public_numbers = rsa.RSAPublicNumbers(
    int.from_bytes(modulus_bytes, "big"),
    int.from_bytes(exponent_bytes, "big")
)

public_key = public_numbers.public_key(default_backend())

pem = public_key.public_bytes(
    encoding=Encoding.PEM,
    format=PublicFormat.SubjectPublicKeyInfo
)

with open("jwt_public_key.pem", "wb") as pem_file:
    pem_file.write(pem)` 

Replace `"your_modulus_base64_encoded"` with the actual Base64-encoded modulus and `"AQAB"` with the actual Base64-encoded exponent from your JWK response. After running this script, you should have a `jwt_public_key.pem` file containing the RSA public key in PEM format.

## read public key from url



here's an improved version of the Python script that reads the JWK set from a URL and converts the public key to a PEM file:



`import requests
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.serialization import Encoding, PublicFormat
import base64
import json


jwk_url = "https://example.com/jwk-set-url"

response = requests.get(jwk_url)
jwk_data = response.json()

modulus_base64 = jwk_data["keys"][0]["n"]
exponent_base64 = jwk_data["keys"][0]["e"]

modulus_bytes = base64.urlsafe_b64decode(modulus_base64 + "===")
exponent_bytes = base64.urlsafe_b64decode(exponent_base64 + "===")

public_numbers = rsa.RSAPublicNumbers(
    int.from_bytes(modulus_bytes, "big"),
    int.from_bytes(exponent_bytes, "big")
)

public_key = public_numbers.public_key(default_backend())

pem = public_key.public_bytes(
    encoding=Encoding.PEM,
    format=PublicFormat.SubjectPublicKeyInfo
)

with open("jwt_public_key.pem", "wb") as pem_file:
    pem_file.write(pem)` 

Replace `"https://example.com/jwk-set-url"` with the actual URL of your JWK set. This script fetches the JWK set from the URL, extracts the modulus and exponent, and converts them to an RSA public key in PEM format just like before. The resulting `jwt_public_key.pem` file will contain the public key in PEM format.

Make sure to have the `cryptography` library installed by running:



`pip install cryptography` 

Additionally, consider error handling and security measures when fetching data from URLs, as well as handling potential exceptions that might occur during network requests.

