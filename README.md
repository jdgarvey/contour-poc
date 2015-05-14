README - Contour S3 File Upload PoC
=====

This small Ruby/Rails/AngularJS app uses the aws-sdk-v1 gem to sign POST request variables for client direct uploads to S3. Signing request variables on the service side keeps AWS secret keys secret. On the client side AngularJS and ng-file-upload (https://github.com/danialfarid/ng-file-upload) use those POST form variables in a request direct to S3 with a file.

Requirements and Frameworks
-----
####Service Side

* Ruby 2.1
* Rails 4
* Gem 2.2
* gem aws-sdk-v1

####Client Side

* AngularJS 1.3
* ng-file-upload 4.2 - https://github.com/danialfarid/ng-file-upload

####AWS S3

* An AWS account
* A dedicated IAM user
* A destination bucket
* An appropriate CORS policy configured on the bucket
* An IAM policy granting s3:* on the bucket for the user

Main Components
-----
* Credentials
* aws-sdk initializer
* Controller
* View
* IAM policy
* CORS bucket policy

####Credentials

Credentials and the name of the target S3 bucket are provided to the service as environment variables. The variable names are:
* AWS_ACCESS_KEY_ID
* AWS_SECRET_ACCESS_KEY
* AWS_REGION
* S3_BUCKET

####aws-sdk initializer

The aws-sdk initializer is located at `config/initializers/aws.rb`. It creates the S3 client and pulls the target bucket using the credentials and bucket name provided by the environment.

####Controller

The new controller is located at `app/controllers/file_upload_controller.rb`. You can extend the validation or criteria for the file uploads by adding attributes to the map in the call to `presigned_post`. Docs for the presigned_post method are at http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/S3/Bucket.html#presigned_post-instance_method.

####View

The main view is located at `app/views/file_upload/upload.html.erb`. It injects the signed request variables created by the controller into the currently inline javascript. This file also defines the Angular app. AngularJS is brought into the view by its presence in the app/assets/javascripts directory. The rails scaffolding will walk the tree and pull in each of those resources automagically. 

####IAM Policy

Please know that IAM policies are particularly fickle. Many generated policies will not work the way you'd expect. AWS's documentation is poor and response codes from service calls are terrible. The only way I could get this to work was by granting full S3 access on the target bucket. If you want, you could continue to iterate getting as close to s3:PutObject only as possible. But the following policy already limits the scope of access to the target resource. If this policy is used in production, you will want to move content out of that bucket quickly. AWS provides event driven tools for doing so. Do not use bucket policies, use the IAM policy tools. 

Create a new policy under IAM and attach it to your new user:
<pre>
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::contour-poc-uploads",
                "arn:aws:s3:::contour-poc-uploads/*"
            ]
        }
    ]
}
</pre>

#### CORS Bucket Policy

Under the permissions properties of your new destination bucket, click on the button labeled, "Edit CORS Configuration." Paste in the following XML, replacing 192.241.199.122 with the IP address or hostname where the upload page is served:
<pre>
&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;
&lt;CORSConfiguration xmlns=&quot;http://s3.amazonaws.com/doc/2006-03-01/&quot;&gt;
    &lt;CORSRule&gt;
        &lt;AllowedOrigin&gt;http://192.241.199.122:3000&lt;/AllowedOrigin&gt;
        &lt;AllowedMethod&gt;GET&lt;/AllowedMethod&gt;
        &lt;AllowedMethod&gt;POST&lt;/AllowedMethod&gt;
        &lt;AllowedMethod&gt;PUT&lt;/AllowedMethod&gt;
        &lt;AllowedHeader&gt;*&lt;/AllowedHeader&gt;
    &lt;/CORSRule&gt;
&lt;/CORSConfiguration&gt;
</pre>
