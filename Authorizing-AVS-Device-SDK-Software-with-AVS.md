# Authorizing AVS Device SDK Software with AVS

This guide describes the steps required to implement the various methods used to authorize the Alexa Voice Service (AVS) Device SDK with AVS.

## AuthDelegateInterface

In order for the AVS Device SDK to access AVS on behalf of an end user, an access token must be provided with each request it makes to AVS. The AVS Device SDK uses `AuthDelegateInterface` to abstract the task of acquiring access tokens. This abstraction allows the SDK to work with any of the existing methods to acquire access tokens. It also allows applications to specialize their implementation of the `AuthDelegateInterface` without changing the code that relies on the `AuthDelegateInterface`.

### Inject your implementation into the SDK

If your application is based on the AVS sample app, you can inject your implementation of `AuthDelegateInterface` into the same places that the sample app does. The sample app creates an instance of `AuthDelegateInterface` as part of it's initialization and passes it to these methods:
* `CapabilitiesDelegate::create()` - Authorizes the `CapabilitiesDelegate` to publish the capabilities of the device.
* `DefaultClient::create()` - Authorizes the Alexa Communications Library (ACL) to send requests to AVS.

If you using the ACL, and have not based your application on the sample app, you'll need to pass your instance of `AuthDelegateInterface` in the `MessageRouter` constructor call used to create the `MessageRouter` instance passed in to `AVSConnectionManager::create()`.

### Abstract methods

`AuthDelegateInterface` defines a few abstract methods that all implementations must provide:

#### **`getAuthToken()`**
```
virtual std::string getAuthToken() = 0;
```
`getAuthToken()` returns the `current` access token. If a valid access token isn't available, it will return an empty string. Access tokens expire, so this method is intended to be called right before each message is sent to AVS. This is handled for you automatically by the ACL. Proper implementations of this interface will preemptively refresh access tokens before they expire, so that the caller does not need to wait while a new access token is acquired before a request can be sent to AVS.

#### **`addAuthObserver()` and `removeAuthObserver()`**
```
virtual void addAuthObserver(std::shared_ptr<avsCommon::sdkInterfaces::AuthObserverInterface> observer) = 0;
virtual void removeAuthObserver(std::shared_ptr<avsCommon::sdkInterfaces::AuthObserverInterface> observer) = 0;
```
`addAuthObserver()` and `removeAuthObserver()` manage the set of `AuthObserverInterface` instances that receive notification when the state of the authorization delegate changes.

State changes are reported to the observers by calling `onAuthStateChange()`.
This method takes two parameters:
  * The new `State` of the authorization delegate.
  * An `Error` code that provides more detail about how that `State` was reached.

## CustomerDataHandler interface

Implementations of `AuthDelegateInterface` must retain the current access token, as well as the other properties used to refresh the access token before it expires. These properties are customer-specific data, so their persistence must be handled by an implementation of the `CustomerDataHandler` interface. This interface has one abstract method:

```
virtual void clearData() = 0;
```

`clearData()` is called for each instance of `CustomerDataHandler` whenever there is an event that requires clearing user data from the product. These events include resetting the product, and changing the user associated with the product.

One simple way to incorporate `CustomerDataHandler` into your implementation of authorization is to use multiple inheritance, and derive your implementation from both `AuthDelegateInterface` and `CustomerDataHandler`.

For example:
```
class MyAuthDelegate
        : public avsCommon::sdkInterfaces::AuthDelegateInterface
        , public registrationManager::CustomerDataHandler {
    ...
};
```

## Authorization

There are several ways to authorize the AVS Device SDK to access AVS:
* [Companion App](https://developer.amazon.com/docs/alexa-voice-service/authorize-companion-app.html)
* [Companion Site](https://developer.amazon.com/docs/alexa-voice-service/authorize-companion-site.html)
* [Code Based Linking](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html)
* [On Product](https://developer.amazon.com/docs/alexa-voice-service/authorize-on-product.html)

The method you'll want to use depends upon the user experience that is the best fit for your product.

## Companion app

To implement the [Companion App](https://developer.amazon.com/docs/alexa-voice-service/authorize-companion-app.html) method within the AVS Device SDK, the product developer will need to provide a **companion app**. This companion app interacts with the product and Login With Amazon (LWA) to provide the product with an **Authorization Code** that enables the product to acquire access tokens from LWA. For more information on implementing a companion app, refer to the [Companion App documentation](https://developer.amazon.com/docs/alexa-voice-service/authorize-companion-app.html).

### Authorization steps

For your product to receive access tokens from LWA and provide them to the **AVS Device SDK**, you will need to provide an implementation of `AuthDelegateInterface` that can perform these steps:

<details><summary>Companion app data flow</summary>
<img src="https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/companion_app_data_flow.png" alt="companion app data flow">
</details>
<br>

1. Check if the **Client ID** and **refresh token** values were saved during the last execution. ***If so, you can skip to step 7***.
2. Load fixed configuration parameters: **Product ID** and **Device Serial Number**.
3. Generate a **Code Verifier** and **Code Challenge** pair.  These must be retained for use in later steps.
4. The **companion app** must be able to discover the product (via Bluetooth, Wifi, etc.) and request product metadata from the product. The product will return the **Product ID**, **Device Serial Number**, and **Code Challenge** values. How this discovery and exchange are implemented is up to the developer of the product.
5. The **companion app** must send **Authorization Code**, **Client ID**, and **Redirect URI** values to the product. As with the previous step, how this step is implemented is up to the developer of the product. These values received by the product must be retained for use in later steps.
6. Once the **Authorization Code**, **Client ID**, and **Redirect URI** are received by the product, the product should call LWA in order to acquire an **access token** and **refresh token**. When making the call, the product must send a `POST` request to https://api.amazon.com/auth/O2/token and pass in the following parameters:
  * `grant_type`: `authorization_code`
  * `code`: The **authorization code** received from the companion app.
  * `redirect_uri`: The **Redirect URI** received from the companion app.
  * `client_id`: The **Client ID** received from the companion app.
  * `code_verifier`: The **Code Verifier** that was initially generated by the product.

  The response is a JSON document that includes the following values:
  * `access_token`: The **access token**.
  * `refresh_token`: The **refresh token**.
  * `token_type`: bearer
  * `expires_in`: The number of seconds for which the access token is still valid.

  If this requests times out or fails due to a server error, retry after a delay until it succeeds or the product is shut down.

7. Once the initial **access token** and **refresh token** pair has been acquired, your implementation will repeatedly refresh the access token shortly before it will expire. The access token is refreshed by sending a `POST` request to https://api.amazon.com/auth/O2/token with the following parameters:
  * `grant_type`: refresh_token
  * `refresh_token`: The **refresh token**.
  * `client_id`: The **Client ID** received from the companion app.

  The response is a JSON document that includes the following values:
  * `access_token`: The access token.
  * `refresh_token`: The **refresh token**.
  * `token_type`: bearer
  * `expires_in`: The number of seconds for which the access token is still valid.

**Note:** Whenever a **refresh token** value is received (*see steps 6 and 7*), save it and the **Client ID** value for use in subsequent SDK sessions. This enables authorization of the product without requiring the user to re-authenticate each time the product restarts. Storage of these values must be secured against unwanted inspection or tampering.

Also note that at any point **LWA** may refuse to honor a **refresh token**.  As such, a companion service implementation of `AuthDelegateInterface` should be prepared to fall back to restarting the authorization process.

## Companion site

To implement the [companion site](https://developer.amazon.com/docs/alexa-voice-service/authorize-companion-site.html) method with the AVS Device SDK, the product developer will need to provide a **companion site**. This **companion site** interacts with **LWA** to acquire access tokens that are then transmitted securely to the product.  For more information on implementing a **companion site**, refer to the [Companion Site documentation](https://developer.amazon.com/docs/alexa-voice-service/authorize-companion-site.html).

### Authorization steps

For your product to receive **access tokens** from the **companion site** and provide them to the **AVS Device SDK**, you'll need to provide an implementation of _**AuthDelegateInterface**_ that can perform these steps:

<details><summary>Companion site data flow</summary>
<img src="https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/companion_site_data_flow.png" alt="Companion site data flow">
</details>
<br>

1. Check if **Session ID** value was saved during the last execution. ***If so, you can skip to step 5.***
2. Load fixed configuration parameters: **Product ID** and **Device Serial Number**.
3. Register the product with the product manufacturer's* **companion site** by sending the **Product ID** and **Device Serial Number** to it. The **companion site** replies to valid requests with a **Registration Code** and a **Session ID**.

  **Registration Code**

  The **Registration Code** is a value defined by the **companion site** that uniquely identifies the product to the **companion site**.  The product converts this value in to a URL that is used to direct the customer to the **companion site**.  How the product directs the customer to the resulting URL is up to the developer of the product and the **companion site**.  When the customer is directed to the **companion site** with that URL, the **companion site** redirects the customer to an **LWA** consent request page.

  **Session ID**

  The **Session ID** is a value that uniquely identifies the product to the **companion site**.  This value is used later by the product to identify itself when requesting Access tokens from the **companion site**.  To avoid requiring the customer to re-authenticate at each start-up, a product may choose to persist the **Session ID** whenever a new one is received.  If **Session ID** values are persisted, they must be secured against unwanted inspection or tampering.

  How the product communicates with the **companion site**, the specifics of **Registration Code** and **Session ID** values, and how the **companion site** maps them to products are up to the product developer.

4. At this point, the product waits until it receives confirmation that the consent request has completed. How this is to be done is up to the product developer.
5. The product's authorization delegate makes requests to the **companion site** for an **access token**, sending the **Session ID** in the request to identify itself. Access tokens have a limited life-span, so these requests will need to be repeated periodically to assure that the current access token has not expired. How these requests are made between the product and the **companion site** is up to the product developer.

Also note that at any point the **companion site** may refuses to honor a **Session ID**.  As such, a companion service  implementation of `AuthDelegateInterface` should be prepared to fall back to restarting the authorization process.

## Code based linking (CBL)

The AVS Device SDK provides an example implementation of the [Code Based Linking](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html) authorization method with the class `CBLAUthDelegate`, and code in sample app that uses that class.  With this method, the product handles all interactions with LWA and directs the customer to authorize the product by browsing to a specified URL (which typically requires authentication with the customer's credentials), entering a code, and confirming that the product should be authorized.  `CBLAUthDelegate` handles the interactions with LWA, but defers interacting with the customer to an implementation of CBLAuthRequesterInterface provided by the application.

### Authorization steps

Applications may use `CBLAUthDelegate` as-is, but if product developers wish to create their own implementation of the [Code Based Linking](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html) authorization method within the AVS Device SDK, the implementation of the `AuthDelegateInterface` must be able to perform these steps:

<details><summary>Companion app data flow</summary>
<img src="https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/cbl_data_flow.png" alt="CBL data flow">
</details>
<br>

1. Load fixed configuration parameters: **Client ID**, **Product ID** and **Device Serial Number**.
2. Check if **refresh token** was saved during the last execution. ***If so, you can skip to step 6***.
3. Send a **Device Authorization Request** to **LWA**. See [Device Authorization Request](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step2) for details.
4. Prompt the end user to navigate to **Verification URI** and enter **User Code**. See [Direct the user to log in](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step3) for details.
5. While the end user is entering **User Code**, poll **LWA** to acquire an initial access token by sending **Device Token Requests** to **LWA**. See [Device Token Request](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step4) for details.
6. Once the initial **access token** and **refresh token** pair has been acquired, your implementation will repeatedly refresh the access token shortly before it expires. See [Exchange the refresh token for a new access token](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step5) for details.

**Note**: Whenever a **refresh token** value is received (see steps 5 and 6), save it for use in subsequent SDK sessions. This enables authorization of the product without requiring the user to re-authenticate each time the product restarts. Storage of these values must be secured against unwanted inspection or tampering.

### On product

To create your own implementation of the [On Product](https://developer.amazon.com/docs/alexa-voice-service/authorize-on-product.html) authorization method within the AVS Device SDK, the product developer will need to provide an implementation of `AuthDelegateInterface` that uses the Android or iOS **LWA Library** operations described in the [On Product](https://developer.amazon.com/docs/alexa-voice-service/authorize-on-product.html) documentation to acquire access tokens.