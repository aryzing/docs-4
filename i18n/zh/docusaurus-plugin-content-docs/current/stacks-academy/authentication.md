---
title: Authentication
description: Register and sign in users with identities on the Stacks blockchain
sidebar_position: 8
---

## Introduction

This guide explains how authentication is performed on the Stacks blockchain.

Authentication provides a way for users to identify themselves to an app while retaining complete control over their credentials and personal details. Authentication provides a way for users to identify themselves to an app while retaining complete control over their credentials and personal details. It can be integrated alone or used in conjunction with [transaction signing](https://docs.hiro.so/get-started/transactions#signature-and-verification) and [data storage](https://docs.stacks.co/build-apps/references/gaia), for which it is a prerequisite.

Users who register for your app can subsequently authenticate to any other app with support for the [Blockchain Naming System](bns) and vice versa.

## How it works

The authentication flow with Stacks is similar to the typical client-server flow used by centralized sign in services (for example, OAuth). However, with Stacks the authentication flow happens entirely client-side. However, with Stacks the authentication flow happens entirely client-side. However, with Stacks the authentication flow happens entirely client-side. However, with Stacks the authentication flow happens entirely client-side.

An app and authenticator, such as [the Stacks Wallet](https://www.hiro.so/wallet/install-web), communicate during the authentication flow by passing back and forth two tokens. The requesting app sends the authenticator an `authRequest` token. Once a user approves authentication, the authenticator responds to the app with an `authResponse` token. The requesting app sends the authenticator an `authRequest` token. Once a user approves authentication, the authenticator responds to the app with an `authResponse` token. The requesting app sends the authenticator an `authRequest` token. Once a user approves authentication, the authenticator responds to the app with an `authResponse` token. The requesting app sends the authenticator an `authRequest` token. Once a user approves authentication, the authenticator responds to the app with an `authResponse` token.

These tokens are are based on [a JSON Web Token (JWT) standard](https://tools.ietf.org/html/rfc7519) with additional support for the `secp256k1` curve used by Bitcoin and many other cryptocurrencies. They are passed via URL query strings. They are passed via URL query strings. They are passed via URL query strings. They are passed via URL query strings.

When a user chooses to authenticate an app, it sends the `authRequest` token to the authenticator via a URL query string with an equally named parameter:

`https://wallet.hiro.so/...?authRequest=j902120cn829n1jnvoa...`

When the authenticator receives the request, it generates an `authResponse` token for the app using an _ephemeral transit key_ . The ephemeral transit key is just used for the particular instance of the app, in this case, to sign the `authRequest`. The ephemeral transit key is just used for the particular instance of the app, in this case, to sign the `authRequest`. The ephemeral transit key is just used for the particular instance of the app, in this case, to sign the `authRequest`. The ephemeral transit key is just used for the particular instance of the app, in this case, to sign the `authRequest`.

The app stores the ephemeral transit key during request generation. The public portion of the transit key is passed in the `authRequest` token. The app stores the ephemeral transit key during request generation. The public portion of the transit key is passed in the `authRequest` token. The authenticator uses the public portion of the key to encrypt an _app private key_ which is returned via the `authResponse`. The public portion of the transit key is passed in the `authRequest` token. The app stores the ephemeral transit key during request generation. The public portion of the transit key is passed in the `authRequest` token. The authenticator uses the public portion of the key to encrypt an _app private key_ which is returned via the `authResponse`.

The authenticator generates the app private key from the user's _identity address private key_ and the app's domain. The app private key serves three functions: The app private key serves three functions: The app private key serves three functions: The app private key serves three functions:

1. It is used to create credentials that give the app access to a storage bucket in the user's Gaia hub
2. It is used in the end-to-end encryption of files stored for the app in the user's Gaia storage.
3. It serves as a cryptographic secret that apps can use to perform other cryptographic functions.

Finally, the app private key is deterministic, meaning that the same private key will always be generated for a given Stacks address and domain.

The first two of these functions are particularly relevant to [data storage with Stacks.js](https://docs.stacks.co/docs/gaia).

## Key pairs

Authentication with Stacks makes extensive use of public key cryptography generally and ECDSA with the `secp256k1` curve in particular.

The following sections describe the three public-private key pairs used, including how they're generated, where they're used and to whom private keys are disclosed.

### Transit private key

The transit private is an ephemeral key that is used to encrypt secrets that need to be passed from the authenticator to the app during the authentication process. It is randomly generated by the app at the beginning of the authentication response. It is randomly generated by the app at the beginning of the authentication response. It is randomly generated by the app at the beginning of the authentication response. It is randomly generated by the app at the beginning of the authentication response.

The public key that corresponds to the transit private key is stored in a single element array in the `public_keys` key of the authentication request token. The authenticator encrypts secret data such as the app private key using this public key and sends it back to the app when the user signs in to the app. The transit private key signs the app authentication request. The authenticator encrypts secret data such as the app private key using this public key and sends it back to the app when the user signs in to the app. The transit private key signs the app authentication request. The authenticator encrypts secret data such as the app private key using this public key and sends it back to the app when the user signs in to the app. The transit private key signs the app authentication request. The authenticator encrypts secret data such as the app private key using this public key and sends it back to the app when the user signs in to the app. The transit private key signs the app authentication request.

### Identity address private key

The identity address private key is derived from the user's keychain phrase and is the private key of the Stacks username that the user chooses to use to sign in to the app. It is a secret owned by the user and never leaves the user's instance of the authenticator. It is a secret owned by the user and never leaves the user's instance of the authenticator. It is a secret owned by the user and never leaves the user's instance of the authenticator. It is a secret owned by the user and never leaves the user's instance of the authenticator.

This private key signs the authentication response token for an app to indicate that the user approves sign in to that app.

### App private key

The app private key is an app-specific private key that is generated from the user's identity address private key using the `domain_name` as input.

The app private key is securely shared with the app on each authentication, encrypted by the authenticator with the transit public key. Because the transit key is only stored on the client side, this prevents a man-in-the-middle attack where a server or internet provider could potentially snoop on the app private key. Because the transit key is only stored on the client side, this prevents a man-in-the-middle attack where a server or internet provider could potentially snoop on the app private key. Because the transit key is only stored on the client side, this prevents a man-in-the-middle attack where a server or internet provider could potentially snoop on the app private key. Because the transit key is only stored on the client side, this prevents a man-in-the-middle attack where a server or internet provider could potentially snoop on the app private key.