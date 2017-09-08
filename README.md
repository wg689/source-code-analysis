# source-code-analysis
source code analysis(AFN源码解析)
从AFN学习装逼技能
#文件架构Architecture
NSURLSession
AFURLSessionManager
AFHTTPSessionManager
Serialization
<AFURLRequestSerialization>
AFHTTPRequestSerializer
AFJSONRequestSerializer
AFPropertyListRequestSerializer
<AFURLResponseSerialization>
AFHTTPResponseSerializer
AFJSONResponseSerializer
AFXMLParserResponseSerializer
AFXMLDocumentResponseSerializer (Mac OS X)
AFPropertyListResponseSerializer
AFImageResponseSerializer
AFCompoundResponseSerializer
Additional Functionality
AFSecurityPolicy
AFNetworkReachabilityManager
Usage
AFURLSessionManager
AFURLSessionManager creates and manages an NSURLSession object based on a specified NSURLSessionConfiguration object, which conforms to <NSURLSessionTaskDelegate>, <NSURLSessionDataDelegate>, <NSURLSessionDownloadDelegate>, and <NSURLSessionDelegate>.
#创建下载任务
Creating a Download Task
```objective-c NSURLSessionConfiguration configuration = [NSURLSessionConfiguration defaultSessionConfiguration]; AFURLSessionManager manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:configuration];

NSURL URL = [NSURL URLWithString:@"http://example.com/download.zip"]; NSURLRequest request = [NSURLRequest requestWithURL:URL];

NSURLSessionDownloadTask downloadTask = [manager downloadTaskWithRequest:request progress:nil destination:^NSURL (NSURL targetPath, NSURLResponse response) { NSURL documentsDirectoryURL = [[NSFileManager defaultManager] URLForDirectory:NSDocumentDirectory inDomain:NSUserDomainMask appropriateForURL:nil create:NO error:nil]; return [documentsDirectoryURL URLByAppendingPathComponent:[response suggestedFilename]]; } completionHandler:^(NSURLResponse response, NSURL filePath, NSError error) { NSLog(@"File downloaded to: %@", filePath); }]; [downloadTask resume]; ```

#创建上传任务

Creating an Upload Task
```objective-c NSURLSessionConfiguration configuration = [NSURLSessionConfiguration defaultSessionConfiguration]; AFURLSessionManager manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:configuration];

NSURL URL = [NSURL URLWithString:@"http://example.com/upload"]; NSURLRequest request = [NSURLRequest requestWithURL:URL];

NSURL filePath = [NSURL fileURLWithPath:@"file://path/to/image.png"]; NSURLSessionUploadTask uploadTask = [manager uploadTaskWithRequest:request fromFile:filePath progress:nil completionHandler:^(NSURLResponse response, id responseObject, NSError error) { if (error) { NSLog(@"Error: %@", error); } else { NSLog(@"Success: %@ %@", response, responseObject); } }]; [uploadTask resume]; ```
#创建上传任务使用多个请求,包含进度
Creating an Upload Task for a Multi-Part Request, with Progress

```objective-c NSMutableURLRequest *request = [[AFHTTPRequestSerializer serializer] multipartFormRequestWithMethod:@"POST" URLString:@"http://example.com/upload" parameters:nil constructingBodyWithBlock:^(id formData) { [formData appendPartWithFileURL:[NSURL fileURLWithPath:@"file://path/to/image.jpg"] name:@"file" fileName:@"filename.jpg" mimeType:@"image/jpeg" error:nil]; } error:nil];

AFURLSessionManager *manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];

NSURLSessionUploadTask *uploadTask; uploadTask = [manager uploadTaskWithStreamedRequest:request progress:^(NSProgress * Nonnull uploadProgress) { // This is not called back on the main queue. // You are responsible for dispatching to the main queue for UI updates dispatch_async(dispatch_get_main_queue(), ^{ //Update the progress view [progressView setProgress:uploadProgress.fractionCompleted]; }); } completionHandler:^(NSURLResponse * Nonnull response, id Nullable responseObject, NSError * Nullable error) { if (error) { NSLog(@"Error: %@", error); } else { NSLog(@"%@ %@", response, responseObject); } }];

[uploadTask resume]; ```

#创建一个数据任务

Creating a Data Task
```objective-c NSURLSessionConfiguration configuration = [NSURLSessionConfiguration defaultSessionConfiguration]; AFURLSessionManager manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:configuration];

NSURL URL = [NSURL URLWithString:@"http://httpbin.org/get"]; NSURLRequest request = [NSURLRequest requestWithURL:URL];

NSURLSessionDataTask dataTask = [manager dataTaskWithRequest:request completionHandler:^(NSURLResponse response, id responseObject, NSError *error) { if (error) { NSLog(@"Error: %@", error); } else { NSLog(@"%@ %@", response, responseObject); } }]; [dataTask resume]; ```

#请求结果序列化
Request Serialization
Request serializers create requests from URL strings, encoding parameters as either a query string or HTTP body.

objective-c NSString *URLString = @"http://example.com"; NSDictionary *parameters = @{@"foo": @"bar", @"baz": @[@1, @2, @3]};

Query String Parameter Encoding
objective-c [[AFHTTPRequestSerializer serializer] requestWithMethod:@"GET" URLString:URLString parameters:parameters error:nil];

GET http://example.com?foo=bar&baz[]=1&baz[]=2&baz[]=3
URL Form Parameter Encoding
objective-c [[AFHTTPRequestSerializer serializer] requestWithMethod:@"POST" URLString:URLString parameters:parameters error:nil];

POST http://example.com/
Content-Type: application/x-www-form-urlencoded

foo=bar&baz[]=1&baz[]=2&baz[]=3
JSON Parameter Encoding
objective-c [[AFJSONRequestSerializer serializer] requestWithMethod:@"POST" URLString:URLString parameters:parameters error:nil];

POST http://example.com/
Content-Type: application/json

{"foo": "bar", "baz": [1,2,3]}
Network Reachability Manager
AFNetworkReachabilityManager monitors the reachability of domains, and addresses for both WWAN and WiFi network interfaces.

Do not use Reachability to determine if the original request should be sent.
You should try to send it.
You can use Reachability to determine when a request should be automatically retried.
Although it may still fail, a Reachability notification that the connectivity is available is a good time to retry something.
Network reachability is a useful tool for determining why a request might have failed.
After a network request has failed, telling the user they're offline is better than giving them a more technical but accurate error, such as "request timed out."
See also WWDC 2012 session 706, "Networking Best Practices.".

#网络连接状态
Shared Network Reachability
```objective-c [[AFNetworkReachabilityManager sharedManager] setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) { NSLog(@"Reachability: %@", AFStringFromNetworkReachabilityStatus(status)); }];

[[AFNetworkReachabilityManager sharedManager] startMonitoring]; ```
#安全策略
Security Policy
AFSecurityPolicy evaluates server trust against pinned X.509 certificates and public keys over secure connections.

Adding pinned SSL certificates to your app helps prevent man-in-the-middle attacks and other vulnerabilities. Applications dealing with sensitive customer data or financial information are strongly encouraged to route all communication over an HTTPS connection with SSL pinning configured and enabled.
#允许SSL证书
Allowing Invalid SSL Certificates
objective-c AFHTTPSessionManager *manager = [AFHTTPSessionManager manager]; manager.securityPolicy.allowInvalidCertificates = YES; // not recommended for production
#单元测试
Unit Tests
AFNetworking includes a suite of unit tests within the Tests subdirectory. These tests can be run simply be executed the test action on the platform framework you would like to test.

#身份
Credits
AFNetworking is owned and maintained by the Alamofire Software Foundation.

AFNetworking was originally created by Scott Raymond and Mattt Thompson in the development of Gowalla for iPhone.

AFNetworking's logo was designed by Alan Defibaugh.

And most of all, thanks to AFNetworking's growing list of contributors.
#安全问题
Security Disclosure
If you believe you have identified a security vulnerability with AFNetworking, you should report it as soon as possible via email to security@alamofire.org. Please do not post it to a public issue tracker.

License
AFNetworking is released under the MIT license. See LICENSE for details.