# appstoreconnectapi
In this article i'll show how i generated and signed token to make requests to https://developer.apple.com/documentation/appstoreconnectapi

Following this article https://developer.apple.com/documentation/appstoreconnectapi/generating_tokens_for_api_requests

I used this library to create token 
```
        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.16.0</version>
        </dependency>
```

```
package com.example.simplesspringbootapp;

import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.interfaces.ECDSAKeyProvider;
import io.jsonwebtoken.Header;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.security.KeyFactory;
import java.security.NoSuchAlgorithmException;
import java.security.PrivateKey;
import java.security.interfaces.ECPrivateKey;
import java.security.interfaces.ECPublicKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import static java.util.Base64.getDecoder;

@SpringBootApplication
public class SimplesspringbootappApplication {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SimplesspringbootappApplication.class, args);
        create();
    }

    private static PrivateKey generatePrivateKey() throws IOException, NoSuchAlgorithmException, InvalidKeySpecException {
        StringBuilder content = new StringBuilder();
        List<String> strings = Files.readAllLines(Paths.get("src/main/resources/AuthKey_YOUR_KID.p8"));
        strings.forEach(content::append);
        String pkcs8Pem = content.toString();
        pkcs8Pem = pkcs8Pem.replace("-----BEGIN PRIVATE KEY-----", "");
        pkcs8Pem = pkcs8Pem.replace("-----END PRIVATE KEY-----", "");
        pkcs8Pem = pkcs8Pem.replaceAll("\\s+", "");

        // Base64 decode the result
        return KeyFactory.getInstance("EC").generatePrivate(new PKCS8EncodedKeySpec(getDecoder().decode(pkcs8Pem)));
    }

    private static void create() throws IOException, NoSuchAlgorithmException, InvalidKeySpecException {
        final PrivateKey privateKey = generatePrivateKey();
        ECDSAKeyProvider ecdsaKeyProvider = new ECDSAKeyProvider() {
            @Override
            public ECPublicKey getPublicKeyById(String s) {
                return null;
            }

            @Override
            public ECPrivateKey getPrivateKey() {
                return (ECPrivateKey) privateKey;
            }

            @Override
            public String getPrivateKeyId() {
                return "YOUR_KID";
            }
        };
        Algorithm algorithmHS = Algorithm.ECDSA256(null, ecdsaKeyProvider.getPrivateKey());
        Date expiresAt = new Date(System.currentTimeMillis() + 1200000);
        System.out.println(expiresAt);

        String token = JWT.create()
                .withHeader(header())
                .withExpiresAt(expiresAt)
                .withIssuer("YOUR_ISSUER_ID")
                .withAudience("appstoreconnect-v1")
                .sign(algorithmHS);
        System.out.println(token);
    }

    private static Map<String, Object> header() {
        Map<String, Object> map = new HashMap<>();
        map.put("alg", "ES256");
        map.put("kid", "YOUR_KID");
        map.put("typ", Header.JWT_TYPE);
        return map;
    }


}
```
