s3Fileupload - A Light Weight PowerFull ANGULAR Directive For Uploading a File to AMAZON S3 Server.
===========

An AngularJS directive that allows you to simply upload files directly to AWS S3.

Features:
===========
* Ease of Use But PowerFull Angular Directive.
* Direct Upload to Amazon S3 Server.
* Open Source Free Light weight angular directive.
* Supports Call Back functions by detective the Status of file Upload ( pre-call , success-call , error-call ).
* Supports Dynamic changes for 'Target File Name' and 'Folder Name' with Angular 2-way Data Binding.
* Contains Status Block-elements inside directive which has auto Disable and enable for display.
* Contains Staus Object holds the information of the File uploading along with current upload status.

## Setup
1. Create AWS S3 bucket

2. Grant "put/delete" permissions to everyone
In AWS web interface, select S3 and select the destination bucket, then
expand the "Permissions" sections and click on the "Add more permissions" button. Select "Everyone" and "Upload/Delete" and save.

3. Add CORS configuration to your bucket

  In AWS web interface, select S3 and select the wanted bucket.
  Expand the "Permissions" section and click on the "Add CORS configuration" button. Paste the wanted CORS configuration, for example:
  ```XML
  <?xml version="1.0" encoding="UTF-8"?>
  <CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
      <CORSRule>
          <AllowedOrigin>*</AllowedOrigin>
          <AllowedMethod>GET</AllowedMethod>
          <AllowedMethod>POST</AllowedMethod>
          <AllowedMethod>PUT</AllowedMethod>
          <AllowedHeader>*</AllowedHeader>
      </CORSRule>
  </CORSConfiguration>
    ```

  In addition, create the following crossdomain.xml file and upload it to the root of your bucket.

  ```XML
  <?xml version="1.0"?>
  <!DOCTYPE cross-domain-policy SYSTEM
  "http://www.macromedia.com/xml/dtds/cross-domain-policy.dtd">
  <cross-domain-policy>
    <allow-access-from domain="*" secure="false" />
  </cross-domain-policy>
  ```

  Once the CORS permissions are updated, your bucket is ready for client side uploads.

4. Create a server side service that will return the needed details for uploading files to S3.
your service shall return a json in the following format:

  ```json
  {
   "policy": "XXXX",
   "signature": "YYY",
   "key": "ZZZ"
  }
  ```
XXX - A policy json that is required by AWS, base64 encoded.
YYY - HMAC and sha of your private key
ZZZ - Your public key
Here's a rails example, even if you're not a rails developer, read the code, it's very straight forward.

  For a php example, please refer to [this guide](https://github.com/asafdav/ng-s3upload/wiki/PHP-Creating-the-S3-Policy).
  ```ruby
      def s3_access_token
        render json: {
          policy:    s3_upload_policy,
          signature: s3_upload_signature,
          key:       GLOBAL[:aws_key]
        }
      end

      protected

        def s3_upload_policy
          @policy ||= create_s3_upload_policy
        end

        def create_s3_upload_policy
          Base64.encode64(
            {
              "expiration" => 1.hour.from_now.utc.xmlschema,
              "conditions" => [
                { "bucket" =>  GLOBAL[:aws_bucket] },
                [ "starts-with", "$key", "" ],
                { "acl" => "public-read" },
                [ "starts-with", "$Content-Type", "" ],
                [ "content-length-range", 0, 10 * 1024 * 1024 ]
              ]
            }.to_json).gsub(/\n/,'')
        end

        def s3_upload_signature
          Base64.encode64(OpenSSL::HMAC.digest(OpenSSL::Digest::Digest.new('sha1'), GLOBAL[:aws_secret], s3_upload_policy)).gsub("\n","")
        end
  ```

  The following code generates an upload policy that will be used by S3, in this example the maximum file size is limited to 10MB (10 * 1024 * 1024), update it to match your requirments. for a full list of S3's policy options, please refer to [AWS documentation](http://docs.aws.amazon.com/AmazonS3/latest/dev/HTTPPOSTExamples.html#HTTPPOSTExamplesTextAreaPolicy).


## How to get it ?

#### Manual Download
Download Zip from [here](https://github.com/vinayvnvv/s3FileUpload/releases) (v1.0)

```
npm and bower direct installation will available from v2.0
```


## Usage
1. Add s3-file-upload.js or s3-file-upload.min.js to your main file (index.html)


2. Set `s3FileUpload` as a dependency in your module
  ```javascript
  var myapp = angular.module('myapp', ['s3FileUpload'])
  ```

3. Add s3-file-upload directive to the wanted element, example:
  ```html
 <div 
     s3-file-upload="Bucket" 
     s3-folder="folder1/folder2" 
     s3-access-uri="/api/s3_access.json" 
     s3-pre-call="beforeUpload"
     s3-error-call="errorUpload"
     s3-succes-call="sucessUpload"
     s3-auto-upload="true" >
     <!-- Child elements  -->
       <input s3-file-model type="file"/> <!-- input File Holder -->
       <s3-progress>Progressing...</s3-progress> <!-- this block Visible when file is uploading -->
       <s3-success>SuccessFull!!</s3-success> <!-- this block Visible when file upload is success -->
       <s3-error>Error!</s3-error> <!-- this block Visible when error in file upload -->
  </div>
  ```

## Attributes
| Name             | value        | description                                         | type     | default     
| :---------------:|:-----------: | :-----:                                             |  :---:   |  :--:       
| `s3-file-upload` | bucket name  | Root atrribute to function s3 file upload           | required |  -           
| `s3-folder`      | Folder path  | Folder path in s3 server where file being uploaded  | required |  -           
| `s3-access-uri`  | s3 acceess API url | api path to access s3 access details          | required |  - |




attributes:
* bucket - Specify the wanted bucket
* s3-upload-options - Provide additional options:
  * getOptionsUri - The uri of the server service that is needed to sign the request (mentioned in section Setup#4) - Required if second option is empty.
  * getManualOptions - if for some reason you need to have your own mechanism of getting a policy, you can simply assign your scope variable to this option. Note it should be resolved on the moment of directive load.
  * folder - optional, specifies a folder inside the bucket the save the file to
  * enableValidation - optional, set to "false" in order to disable the field validation.
  * targetFilename - An optional attribute for the target filename. if provided the file will be renamed to the provided value instead of having the file original filename.

## Themes
ng-s3upload allows to customize the directive template using themes. Currently the available themes are: bootstrap2, bootstrap3.
You can also use your own template (if so, the provided path must start with a `/`).

#### How to?

```javascript
app.config(function(ngS3Config) {
  ngS3Config.theme = 'bootstrap3';
  ngS3Config.theme = '/path/to/custom/theme.html';
});
```

[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/asafdav/ng-s3upload/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

