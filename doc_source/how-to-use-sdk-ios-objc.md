# Use Generated iOS SDK \(Objective\-C\) to Call API<a name="how-to-use-sdk-ios-objc"></a>

Before beginning the following procedure, you must complete the steps in [Generate SDKs for an API Using the API Gateway Console](how-to-generate-sdk-console.md) for iOS in Objective\-C and download the \.zip file of the generated SDK\.

## Install the AWS Mobile SDK and an iOS SDK generated by API Gateway in an Objective\-C Project<a name="use-sdk-ios-objc-install-sdk"></a>

The following procedure describes how to install the SDK\.

**To install and use an iOS SDK generated by API Gateway in Objective\-C**

1. Extract the contents of the API Gateway\-generated \.zip file you downloaded earlier\. Using the [SimpleCalc API](simple-calc-lambda-api.md), you may want to rename the unzipped SDK folder to something like **sdk\_objc\_simple\_calc**\. In this SDK folder there is a `README.md` file and a `Podfile` file\. The `README.md` file contains the instructions to install and use the SDK\. This tutorial provides details about these instructions\. The installation leverages [CocoaPods](https://cocoapods.org) to import required API Gateway libraries and other dependent AWS Mobile SDK components\. You must update the `Podfile` to import the SDKs into your app's Xcode project\. The unarchived SDK folder also contains a `generated-src` folder that contains the source code of the generated SDK of your API\.

1. Launch Xcode and create a new iOS Objective\-C project\. Make a note of the project's target\. You will need to set it in the `Podfile`\.  
![\[\]](http://docs.aws.amazon.com/apigateway/latest/developerguide/images/use-sdk-in-ios-objc-project-find-target.png)

1. To import the AWS Mobile SDK for iOS into the Xcode project by using CocoaPods, do the following:

   1. Install CocoaPods by running the following command in a terminal window:

      ```
      sudo gem install cocoapods
      pod setup
      ```

   1. Copy the `Podfile` file from the extracted SDK folder into the same directory containing your Xcode project file\. Replace the following block:

      ```
      target '<YourXcodeTarget>' do
          pod 'AWSAPIGateway', '~> 2.4.7'
      end
      ```

      with your project's target name: 

      ```
      target 'app_objc_simple_calc' do
          pod 'AWSAPIGateway', '~> 2.4.7'
      end
      ```

      If your Xcode project already contains a file named `Podfile`, add the following line of code to it:

      ```
      pod 'AWSAPIGateway', '~> 2.4.7'
      ```

   1. Open a terminal window and run the following command:

      ```
      pod install
      ```

      This installs the API Gateway component and other dependent AWS Mobile SDK components\.

   1. Close the Xcode project and then open the `.xcworkspace` file to relaunch Xcode\.

   1. Add all of the `.h` and `.m` files from the extracted SDK's `generated-src` directory into your Xcode project\.  
![\[\]](http://docs.aws.amazon.com/apigateway/latest/developerguide/images/use-sdk-in-ios-objc-project-add-sdk-src.png)

   To import the AWS Mobile SDK for iOS Objective\-C into your project by explicitly downloading AWS Mobile SDK or using [Carthage](https://github.com/Carthage/Carthage#installing-carthage), follow the instructions in the *README\.md* file\. Be sure to use only one of these options to import the AWS Mobile SDK\.

## Call API Methods Using the iOS SDK generated by API Gateway in an Objective\-C Project<a name="use-sdk-ios-objc-call-sdk"></a>

When you generated the SDK with the prefix of `SIMPLE_CALC` for this [SimpleCalc API](simple-calc-lambda-api.md) with two models for input \(`Input`\) and output \(`Result`\) of the methods, in the SDK, the resulting API client class becomes `SIMPLE_CALCSimpleCalcClient` and the corresponding data classes are `SIMPLE_CALCInput` and `SIMPLE_CALCResult`, respectively\. The API requests and responses are mapped to the SDK methods as follows:
+ The API request of

  ```
  GET /?a=...&b=...&op=...
  ```

  becomes the SDK method of

  ```
  (AWSTask *)rootGet:(NSString *)op a:(NSString *)a b:(NSString *)b
  ```

  The `AWSTask.result` property is of the `SIMPLE_CALCResult` type if the `Result` model was added to the method response\. Otherwise, the property is of the `NSDictionary` type\.
+ This API request of

  ```
  POST /
      
  {
     "a": "Number",
     "b": "Number",
     "op": "String"
  }
  ```

  becomes the SDK method of

  ```
  (AWSTask *)rootPost:(SIMPLE_CALCInput *)body
  ```
+ The API request of

  ```
  GET /{a}/{b}/{op}
  ```

  becomes the SDK method of

  ```
  (AWSTask *)aBOpGet:(NSString *)a b:(NSString *)b op:(NSString *)op
  ```

The following procedure describes how to call the API methods in Objective\-C app source code; for example, as part of the `viewDidLoad` delegate in a `ViewController.m` file\.

**To call the API through the iOS SDK generated by API Gateway**

1. Import the API client class header file to make the API client class callable in the app:

   ```
   #import "SIMPLE_CALCSimpleCalc.h"
   ```

   The `#import` statement also imports `SIMPLE_CALCInput.h` and `SIMPLE_CALCResult.h` for the two model classes\.

1. Instantiate the API client class:

   ```
   SIMPLE_CALCSimpleCalcClient *apiInstance = [SIMPLE_CALCSimpleCalcClient defaultClient];
   ```

   To use Amazon Cognito with the API, set the `defaultServiceConfiguration` property on the default `AWSServiceManager` object, as shown in the following, before calling the `defaultClient` method to create the API client object \(shown in the preceding example\):

   ```
   AWSCognitoCredentialsProvider *creds = [[AWSCognitoCredentialsProvider alloc] initWithRegionType:AWSRegionUSEast1 identityPoolId:your_cognito_pool_id];
   AWSServiceConfiguration *configuration = [[AWSServiceConfiguration alloc] initWithRegion:AWSRegionUSEast1 credentialsProvider:creds];
   AWSServiceManager.defaultServiceManager.defaultServiceConfiguration = configuration;
   ```

1. Call the `GET /?a=1&b=2&op=+` method to perform `1+2`:

   ```
   [[apiInstance rootGet: @"+" a:@"1" b:@"2"] continueWithBlock:^id _Nullable(AWSTask * _Nonnull task) {
       _textField1.text = [self handleApiResponse:task];
       return nil;
   }];
   ```

   where the helper function `handleApiResponse:task` formats the result as a string to be displayed in a text field \(`_textField1`\)\.

   ```
   - (NSString *)handleApiResponse:(AWSTask *)task {
       if (task.error != nil) {
           return [NSString stringWithFormat: @"Error: %@", task.error.description];
       } else if (task.result != nil && [task.result isKindOfClass:[SIMPLE_CALCResult class]]) {
           return [NSString stringWithFormat:@"%@ %@ %@ = %@\n",task.result.input.a, task.result.input.op, task.result.input.b, task.result.output.c];
       }
       return nil;
   }
   ```

   The resulting display is `1 + 2 = 3`\.

1. Call the `POST /` with a payload to perform `1-2`:

   ```
   SIMPLE_CALCInput *input = [[SIMPLE_CALCInput alloc] init];
       input.a = [NSNumber numberWithInt:1];
       input.b = [NSNumber numberWithInt:2];
       input.op = @"-";
       [[apiInstance rootPost:input] continueWithBlock:^id _Nullable(AWSTask * _Nonnull task) {
           _textField2.text = [self handleApiResponse:task];
           return nil;
       }];
   ```

   The resulting display is `1 - 2 = -1`\.

1. Call the `GET /{a}/{b}/{op}` to perform `1/2`:

   ```
   [[apiInstance aBOpGet:@"1" b:@"2" op:@"div"] continueWithBlock:^id _Nullable(AWSTask * _Nonnull task) {
       _textField3.text = [self handleApiResponse:task];
       return nil;
   }];
   ```

   The resulting display is `1 div 2 = 0.5`\. Here, `div` is used in place of `/` because the [simple Lambda function](simple-calc-nodejs-lambda-function.md) in the backend does not handle URL encoded path variables\.