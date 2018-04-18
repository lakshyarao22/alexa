### Register your product and create a security profile.

After you've registered for an Amazon developer account, you'll need to create an Alexa device and security profile. Make note of the following parameters as you go through setup, **ProductID** and **ClientID** -- you'll need these later.

1. Login to Amazon Developer Portal - [developer.amazon.com](https://developer.amazon.com/login.html)
2. Navigate to the [AVS Console](https://developer.amazon.com/avs/home.html#/avs/home)
3. Click **Create Product**.
	![](assets/avs-choose-device.png)
4. Follow these steps:

**Product Information**

On this page you'll provide information about your product.

1. Product Name - This is what gets displayed to users when they register an instance of the product with Amazon.
2. Product ID - A simple identifier for your product.
3. Choose product type **Alexa-Enabled Device**.  
4. When asked if your product will use a companion app, select **No**.
5. For Product Category, select **Other**, and enter "Rapsberry Pi Project on GitHub". **Note**: For a commercial product, you should select the best option available.
   ![](https://github.com/alexa/alexa-avs-sample-app/wiki/assets/avs-device-type-info.png)
6. Enter a brief description. For example: My first Pi project.  
7. When asked how users will interact with your product, select **Touch-initiated** and **Hands-free**. **Note**: For a commercial product, you should select the options that best describe your product's functionality.  
8. You can skip uploading an image for now. This content is used on the manage your content and devices screen on amazon.com.  
9. When asked if you will distribute this project, select **No**. **Note**: For a commercial product, you should select **Yes**.  
10. When asked if this product is for children, select **No**.   
11. Click **Next**.  
    ![](https://github.com/alexa/alexa-avs-sample-app/wiki/assets/avs-device-type-info-2.png)

**Security Profile**  

On this page we'll create a new Login with Amazon (LWA) security profile. This associates user data and security credentials with one or more products.

1. Click **Create New Profile**.  
	![](https://github.com/alexa/alexa-avs-sample-app/wiki/assets/avs-create-new-security-profile.png)

2. Enter a name and description for your security profile, then click **Next**. For example:
	 - **Security Profile Name**: Alexa Voice Service Sample App Security Profile
	 - **Security Profile Description**: Alexa Voice Service Sample App Security Profile Description

   **Note:** These are suggested values. You provide custom information for **Security Profile Name** and **Security Profile Description**.

3. Now you'll generate a **Client ID**. Navigate to **Platform Information** > **Other devices and platforms** tab. Enter a Client ID name, then click Generate ID.

![](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/docs/avs-code-based-linking-other-generate-clientid._TTH_.png)

**IMPORTANT:** Save *this* **Client ID**, you'll need this later. Using the wrong **Client ID**, such as from the Product Information Page, these are different elements but have the same name. Using the wrong one will result in authorization failures.

4. Review and agree to the [AVS Agreement](https://developer.amazon.com/support/legal/alexa/alexa-voice-service/terms-and-agreements#Alexa%20Voice%20Service%20Agreement) and [AVS Program Requirement](https://developer.amazon.com/support/legal/alexa/alexa-voice-service/terms-and-agreements#Alexa%20Voice%20Service%20Program%20Requirements), and then select **Finish**.

5. You should see pop up confirmation that "Your Product has been created". Select **OK**.

You are now ready to generate self-signed certificates.

---

### Enable Security Profile

1. Open a web browser, and visit [https://developer.amazon.com/lwa/sp/overview.html](https://developer.amazon.com/lwa/sp/overview.html).  
2. Near the top of the page, select the security profile you created earlier from the drop down menu and click **Confirm**.
![](https://github.com/alexa/alexa-avs-sample-app/wiki/assets/avs-lwa-choose-security-profile.png)
3. Enter a privacy policy URL beginning with http:// or https://. For this example, you can enter a fake URL such as http://example.com.
4. [Optional] You may upload an image as well. The image will be shown on the Login with Amazon consent page to give your users context.
5. Click Save.
   ![](https://github.com/alexa/alexa-avs-sample-app/wiki/assets/avs-privacy-url.png)
