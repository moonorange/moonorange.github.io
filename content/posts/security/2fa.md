---
title: 'Implementing TOTP in Go'
date: '2023-04-01'
categories: ["Security"]
tags: ["2FA", "English Article"]
---

## Introduction

In this article, I would like to try implementing a code that generates a one-time password, which is often used in many two-factor authentication functions, called `Time-based One-time Password (TOTP)`, using Go.

## What is Two-Factor Authentication?

Before writing the TOTP generation code, let's briefly review what two-factor authentication is.

There are mainly three types of factors for authentication: `knowledge, possession, and biometrics`.

- `Knowledge:` Something only the user knows or remembers, such as a login password.
- `Possession:` Something only the user has, such as a smartphone.
- `Biometrics:` A physical characteristic unique to the user, such as a fingerprint.

Two-factor authentication refers to using two of these factors for authentication.

A common pattern is to install an app such as Google Authenticator on your own device, which uses both the knowledge factor of the login password and the possession factor for two-factor authentication.

## What is HOTP (HMAC-based One-time Password)?

`HOTP` is an OTP (One-time Password) generation algorithm based on HMAC, as its name suggests.

`HMAC` (Hashed Message Authentication Code) is a message authentication code that uses a shared secret key and a hash function.

In other words, it is a code used for detecting tampering when messages are sent and received between clients and servers.

HOTP generates OTP using a value called a   `counter`, which changes for each generation. TOTP is an extension of HOTP that uses a value that changes with time.

Therefore, to implement TOTP, it is necessary to know how HOTP is implemented.

HOTP is generated through the following operations:

1. Generate a secret key of 16 bytes or more that is shared between the client and the server (in this article, we use an arbitrary value for simplicity).
2. Generate a counter that increments each time a new HOTP is created (in this article, we use an arbitrary value for simplicity).
3. Use a hash function(SHA-1) to generate an HMAC from the secret key and the counter.
4. Truncate MAC for the user to input easily
   1. The last 4 bits of the HMAC are used as an offset.
   2. Get 4 bytes starting from the offset while masking the most significant bit (this is done to avoid the difference in the result of the mod calculation between signed and unsigned on different processors).
   3. Perform a modulo calculation with the obtained 32-bit value and 10^d (d is the number of digits of HOTP, which is often 6 digits).

This is what it looks like when implemented in Go:

[The Go Playground](<https://go.dev/play/p/ijDdkIZI2Dz>)

```go
package main

import (
    "crypto/hmac"
    "crypto/sha1"
    "encoding/binary"
    "fmt"
    "math"
)

func main() {
    // Generate a counter
    // Define as uint64 to handle as 8-byte integer.
    // It needs to be common between server and client. Here, a random value is used.
    counter := uint64(11111111)

    // Secret key, a random value is used here
    secretKey := []byte("secretkey")

    generateHOTP(secretKey, counter)
}

func generateHOTP(secretKey []byte, counter uint64) {
    // Initialize an HMAC object using the SHA-1 algorithm and secret key
    mac := hmac.New(sha1.New, secretKey)
    // Write the counter to the HMAC object in BigEndian (from higher byte to lower byte) format
    binary.Write(mac, binary.BigEndian, counter)
    // Calculate the MAC of the written message. The result of HMAC-SHA-1 is 20 bytes.
    sum := mac.Sum(nil)

    // Use the last 4 bits as an offset, ranging from 0 to 15 decimal.
    offset := sum[len(sum)-1] & 0x0f
    // Obtain the value of a contiguous 4-byte block from the offset while masking the most significant bit to avoid confusion about signed vs. unsigned modulo computations.
    // The length of sum is 20 bytes, ensuring that the offset is within bounds.
    bin_code := binary.BigEndian.Uint32(sum[offset:offset+4]) & 0x7fffffff

    // To get a 6-digit HOTP, calculate the remainder when divided by 10^6
    hotp := bin_code % uint32(math.Pow10(6))

    fmt.Println(hotp)
}
```

## What is TOTP?

TOTP is a variant of HOTP that uses a changing value `T` based on the current time instead of the counter used in HOTP.

It is defined as follows:

```
TOTP = HOTP(K, T)

T = (current Unix time - T0) / X (where T0 is usually 0 seconds and X is 30 seconds)
K refers to the shared secret key.
```

In other words, by simply using T instead of the counter used in HOTP, TOTP can be implemented.

[The Go playground](https://go.dev/play/p/2SP2b_I9vQx)

```go
func main() {
    // Define T, which varies depending on the time
    t := uint64(time.Now().Unix() / 30)
    // Encode the secret key with base32, passing a random string that can be encoded here
    // The shared secret key needs to be encoded in base32
    secretBytes, _ := base32.StdEncoding.DecodeString("ONXW2ZJAMRQXIYJAO5UXI2BAAAQGC3TEEDX3XPY=")
    // Call generateHOTP with the secret key and T
    generateHOTP(secretBytes, t)
}
```

With that, the code for generating TOTP in Go has been implemented!

## Reference

[RFC 4226: HOTP: An HMAC-Based One-Time Password Algorithm](<https://www.rfc-editor.org/rfc/rfc4226>)

[RFC 6238: TOTP: Time-Based One-Time Password Algorithm](<https://www.rfc-editor.org/rfc/rfc6238>)
