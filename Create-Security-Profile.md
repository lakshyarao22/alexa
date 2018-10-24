## Register your product and create a security profile

After you've registered for an Amazon developer account, you'll need to create a new Alexa product and security profile.

1. Login to [Amazon Developer Portal](https://developer.amazon.com/login.html)

2. Navigate to the Alexa > [Alexa Voice Service](https://developer.amazon.com/avs/home.html#/avs/home) console.
3. Choose [Create Product](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/create-product._CB1539978229_.png), and follow then instructions below to 1). [register a product](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile#product-information), 2). [create a security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile#create-a-security-profile), and 3). [authorize your security profile]((https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile#enable-your-security-profile) to use Login With Amazon (LWA).
	![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/create-product._CB1539978229_.png)

### Product information

On [this page](https://developer.amazon.com/avs/home.html#/avs/products/new), you'll provide information about your product.

1. **Product Name** - This is what gets displayed to users when they register an instance of the product with Amazon.

2. **Product ID** - A simple identifier for your product.

   ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/product-information.png)

3. Choose the product type **Device with Alexa built-in**.  

4. When asked if your product will use a companion app, select **No**.

   ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/type-and-app.png)

5. For **Product Category**, select **Other**, and enter a description of the product type, such as Raspberry Pi.

    **Note**: For a commercial product, you should select the best option available.

6. Enter a **Brief product description**.

   ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/product-category.png)

7. When asked how users will interact with your product, select **Touch-initiated** and **Hands-free**. **Note**: For a commercial product, you should select the options that best describe your product's functionality.

    ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/interact.png)

8. (*Optional*) You can skip uploading an image for now. This content is used on the manage your content and devices screen on amazon.com.

    ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/product_image.png)

9. When asked if you will distribute this project, select **No**. **Note**: For a commercial product, you should select **Yes**.

10. When asked if this product is for children, select **No**.

    ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/avs-product-info-3.png)

    11. Click **Next**.

### Create a Security Profile  

On this page we'll create a *new* Login with Amazon (LWA) security profile. This associates user data and security credentials with one or more products.

1. Select **Create New Profile**.  
	![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/create-new-security-profile.png)

2. Enter a name and description for your security profile, then click **Next**. For example:
	 - **Security Profile Name**: AVS Test Product
	 - **Security Profile Description**: This is for a reference build of the AVS Device SDK on Raspberry Pi.

   **Note:** These are suggested values. You provide *custom information* for **Security Profile Name** and **Security Profile Description**.

   ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/new-security-profile-fields.png)

3. Now you'll generate a **Client ID**. Navigate to **Platform Information** > **Other devices and platforms**. Enter a **Client ID name**, then select **Generate ID**.

    ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/security-profile-generate-id.png)

4. Now, select **Download**. This will save a **config.json** file to your computer that includes the **clientID** and **productID** associated with your security profile. This information will be used by the sample app as part of the setup process.
    ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/download.png)

5. Review and agree to the [AVS Developer Services Agreement](https://developer.amazon.com/support/legal/alexa/alexa-voice-service/terms-and-agreements#Alexa%20Voice%20Service%20Agreement), including the [Alexa Voice Services Program Requirements](https://developer.amazon.com/support/legal/alexa/alexa-voice-service/terms-and-agreements#Alexa%20Voice%20Service%20Program%20Requirements), and then select **Finish**.

    ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/avs_developer_agreement._CB1539968709_.png)

6. You should see pop up confirmation that "Your Product has been created". Select **OK**.

    ![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/product-created.png)

You are now ready to generate self-signed certificates.

### Enable your Security Profile

1. Open a web browser, and visit [https://developer.amazon.com/lwa/sp/overview.html](https://developer.amazon.com/lwa/sp/overview.html).

2. Near the top of the page, select the security profile you created earlier from the drop down menu and click **Confirm**.
![](https://github.com/alexa/alexa-avs-sample-app/wiki/assets/avs-lwa-choose-security-profile.png)

3. Enter a privacy policy URL beginning with http:// or https://. For this example, you can enter a fake URL such as http://example.com.

4. **Optional** You may upload an image. The image will be shown on the LWA consent page to give your users context.

5. Click Save.
   ![](https://github.com/alexa/alexa-avs-sample-app/wiki/assets/avs-privacy-url.png)
