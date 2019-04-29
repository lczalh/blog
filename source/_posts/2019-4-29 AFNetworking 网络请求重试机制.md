---
layout: post
title: "AFNetworking 网络请求重试机制"
date: 2019-04-29 13:44:24
tags: MySQL
---
使用递归方式
```
- (void)downloadFileRetryingNumberOfTimes:(NSUInteger)ntimes 
                                success:(void (^)(id responseObject))success 
                                failure:(void (^)(NSError *error))failure
{
    if (ntimes <= 0) {
        if (failure) {
        NSError *error = ...;
        failure(error);
        }
    } else {
        [self getPath:@"/path/to/file" parameters:nil success:^(AFHTTPRequestOperation *operation, id responseObject) {
            if (success) {
            success(...);
            }
        } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
            [self downloadFileRetryingNumberOfTimes:ntimes - 1 success:success failure:failure];
        }];
    }
}
```
摘录于[AFNetworking Issuse]("https://github.com/AFNetworking/AFNetworking/issues/393")




