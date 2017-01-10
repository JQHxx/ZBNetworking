# ZBNetworking
AFNetworking和NSURLSession 封装 添加了请求缓存,离线下载,显示缓存大小,删除缓存等功能 — 
低耦合，易扩展。

可见的缓存文件

![](http://a3.qpic.cn/psb?/V12I5WUv0Ual5v/uls*nG1YySR.EpyYI8*lFu9kW.lwzjgW.cnPbGMUBG8!/b/dPgAAAAAAAAA&bo=aAHwAAAAAAACDLE!&rf=viewer_4)

## 使用 AFNetworking 
```objective-c
//get请求方法 会默认创建缓存路径    
  [ZBNetworkManager requestWithConfig:^(ZBURLRequest *request){
        request.urlString=list_URL;
        request.methodType=ZBMethodTypeGET;//默认为GET
        request.apiType=ZBRequestTypeDefault;//默认为default
        request.timeoutInterval=10;
       // request.parameters=@{@"1": @"one", @"2": @"two"};
       // [request setValue:@"1234567890" forHeaderField:@"apitype"];
    }  success:^(id responseObj,apiType type){
        //如果是刷新的数据
        if (type==ZBRequestTypeRefresh) {
            [self.dataArray removeAllObjects];
            [_refreshControl endRefreshing];    //结束刷新
        }
        if (type==ZBRequestTypeLoadMore) {
            //上拉加载
        }
        NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:responseObj options:NSJSONReadingMutableContainers error:nil];
        NSArray *array=[dict objectForKey:@"authors"];
        
        for (NSDictionary *dic in array) {
            RootModel *model=[[RootModel alloc]init];
            model.name=[dic objectForKey:@"name"];
            model.wid=[dic objectForKey:@"id"];
            model.detail=[dic objectForKey:@"detail"];
            [self.dataArray addObject:model];
        }
        [self.tableView reloadData];
        
    } failed:^(NSError *error){
        if (error.code==NSURLErrorCancelled)return;
        if (error.code==NSURLErrorTimedOut){
            [self alertTitle:@"请求超时" andMessage:@""];
        }else{
            [self alertTitle:@"请求失败" andMessage:@""];
        }
    }];

```


## 使用 NSURLSession
1.添加#import "ZBNetworking.h"

2.添加 delegate
```objective-c
<ZBURLSessionDelegate>
```

3.使用简单:  一行代码调用 
```objective-c
//get请求方法 会默认创建缓存路径    
  [[ZBURLSessionManager shareManager] getRequestWithUrlString:URL target:self];
```

4.完成和失败俩个代理回调
```objective-c
//请求完成的代理方法里进行解析或赋值
- (void)urlRequestFinished:(ZBURLSessionManager *)request
{
    
    NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:request.downloadData options:NSJSONReadingMutableContainers error:nil];
    NSArray *array=[dict objectForKey:@"authors"];
    
    for (NSDictionary *dic in array) {
        RootModel *model=[[RootModel alloc]init];
        model.icon=[dic objectForKey:@"icon"];
        model.name=[dic objectForKey:@"name"];
        model.wid=[dic objectForKey:@"id"];
        model.detail=[dic objectForKey:@"detail"];
        [_dataArray addObject:model];
        
    }
    [_tableView reloadData];
    
}
//请求失败的方法里 进行异常判断 支持error.code所有异常
- (void)urlRequestFailed:(ZBURLSessionManager *)request
{
    if (request.error.code==-999)return;
    if (request.error.code==NSURLErrorTimedOut) {
        NSLog(@"请求超时");
    }else{
        NSLog(@"请求失败");
    }

}

```
## 使用 其他功能
1.离线下载

```objective-c
 [ZBNetworkManager requestWithConfig:^(ZBURLRequest *request)
        request.urlArray=offlineArray;
        request.apiType=ZBRequestTypeOffline;   //离线请求 apiType:ZBRequestTypeOffline
    }  success:^(id responseObj,apiType type){
        //如果是离线请求的数据
        if (type==ZBRequestTypeOffline) {
        
        } 
    } failed:^(NSError *error){
        if (error.code==NSURLErrorCancelled)return;
        if (error.code==NSURLErrorTimedOut){
            [self alertTitle:@"请求超时" andMessage:@""];
        }else{
            [self alertTitle:@"请求失败" andMessage:@""];
        }
    }];
   
 或是用URLSession的方法  
[[ZBURLSessionManager shareManager] offlineDownload:offlineArray target:self apiType:ZBRequestTypeOffline];

//具体演示看demo
```
![](http://a3.qpic.cn/psb?/V12I5WUv0Ual5v/cY8K3L2*GJ9RO3i*z1If9XTmzas0cylmafMXWqdFe4o!/b/dK0AAAAAAAAA&bo=aAHwAAAAAAACLJE!&rf=viewer_4)


2.缓存相关
```objective-c
//显示缓存大小
 [[ZBCacheManager shareCacheManager]getCacheSize];
 //删除缓存
[[ZBCacheManager shareCacheManager]clearCache];
 ```
