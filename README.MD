## 网络连接模块

#### 版本更新

| 版本  | 更新说明                      |
| ------- | ------------------------- |
| 1.0.7  | 1.增加上报事件统一前缀np_ <br>2.增加通知栏内容设置<br>3.移除多余的依赖项，及部分冗余代码       |
| 1.0.6  | 1.更新libsslocal.so，去掉包名验证逻辑        |
| 1.0.5  | 1.移除节点管理，不再返回节点数据<br>2.初始化增加连接方式、dns两个参数。defaultPw、dataKeyremoteCallback设为过期，不再使用<br>3.修改启动方式，改为传入host、端口、密码启动，不再支持传入id启动        |
| 1.0.4  | 1.修改节点列表获取接口，返回分组列表<br>2.服务器节点增加flag图标标识、type服务器类型标识          |
| 1.0.3  | 1.增加默认密码设置            |

##  1. 依赖项配置

指定maven仓库地址

```
maven { url 'http://maven.shlsnetwork.com/nexus/content/repositories/releases/' }
```

添加SDK依赖
```
implementation 'com.android.socks:NetProxy:1.0.7'
```



## 2. 初始化

    NwProxy.init(appliction, new ProxyConfig.Builder()
         //可选参数，通知栏点击后需要跳转的activity。如果设置为空，不显示默认通知栏
         .launchClass(MainActivity.class)
         //可选参数,连接方式
         .connectMethod("chacha20-ietf")
         //可选参数，连接dns
         .remoteDns("dns.google")
          //可选参数,调试模式，会显示日志及调试模式下的设置，如通知栏显示节点IP信息等
         .showLog(BuildConfig.DEBUG)
         //已过期，不再使用（可选参数，连接的默认密码）
         .defaultPw("")
         //已过期，不再使用（可选参数，获取remote config节点配置的key值）
         .dataKey("")
        //已过期，不再使用（必选参数，实现remote config的数据接口）
        .remoteCallback(new IRemoteCallback() {
             @Override
             public String getString(String key) {
                  return FireBaseRemoteConfig.getString(key);
             }
         })
         .build());

## 3. 接口调用
#### ~~3.1 获取节点分组列表~~

    NwProxy.getGroupList(new IGroupCallback() {
        @Override
        public void onResponse(List<NodeGroup> list) {
            for (NodeGroup nodeGroup : list) {
                LogUtils.e(nodeGroup.toString());
            }
        }
    });

~~NodeGroup字段说明~~

| ~~字段名~~  | ~~数据类型~~         | ~~说明~~                           |
| ----------- | -------------------- | ---------------------------------- |
| ~~groupId~~ | ~~String~~           | ~~分组id~~                         |
| ~~name~~    | ~~String~~           | ~~分组名称~~                       |
| ~~flag~~    | ~~String~~           | ~~分组标记，用来失败显示的图标等~~ |
| ~~order~~   | ~~int~~              | ~~排序~~                           |
| ~~servers~~ | ~~List<ServerNode>~~ | ~~该分组下的服务器节点列表~~       |

~~ServerNode字段说明~~

| ~~字段名~~ | ~~数据类型~~ | ~~说明~~    |
| ------ | ------- | -------------- |
| ~~id~~  | ~~long~~ | ~~客户端本地生成的节点ID,用来设置连接节点的参数~~         |
|~~serverId~~     | ~~int~~ | ~~服务器id,标识唯一的服务器值~~         |
| ~~name~~  | ~~String~~ | ~~节点名称~~         |
| ~~flag~~  | ~~String~~ | ~~节点标识，用来定义显示的图标等参数~~         |
| ~~type~~  | ~~int~~ |~~服务器类型，对应常量NodeType。<br>   FREE : 0,免费节点；<br>       VIP：1,vip专用节点 ；<br>VIP_AD：2，vip或者看广告之后，可以使用的节点~~            |
| ~~host~~  | ~~String~~ | ~~节点IP~~         |
| ~~port~~  | ~~int~~ | ~~端口号~~         |
| ~~order~~  | ~~int~~ | ~~排序顺序~~         |


#### 3.2 启动网络代理服务

```
//选择指定节点启动
//参数分别为ip、端口号、密码
NwProxy.start("92.38.163.87",9500,"K3MMEj49HOP2Xw6RIcCF9haxwT5tvT4SFTVPZFYEQsg=");
```

#### 3.3 停止网络代理服务

```
 NwProxy.stop();
```

#### 3.4 连接状态监听
注册监听

       //网络状态监听回调
       IStateObserverListener listener = new IStateObserverListener() {
       		//连接状态发生改变
           @Override
           public void stateChanged(BaseService.@NotNull State state, @Nullable String nodeInfo) {
    			
            }
    		//实时网速回调
            @Override
            public void trafficUpdated(long nodeId, @NotNull TrafficStats trafficStats) {
    			
            }
        };
        //注册监听
        NwProxy.registerListener(listener);


取消监听   
```
    //取消监听
    NwProxy.unRegisterListener(listener);
```
State枚举值说明

| 字段名 | 说明        |
| ------  | -------------- |
| Idle      | 初始状态             |
| Connecting      | 正在连接             |
| Connected      | 已连接             |
| Stopping      | 正在停止             |
| Stopped      | 已停止             |

TrafficStats自动说明

| 字段名  | 说明                      |
| ------- | ------------------------- |
| txRate  | 实时上行流量              |
| rxRate  | 实时下行流量              |
| txTotal | 本次连接Seesion上行总流量 |
| rxTotal | 本次连接Seesion下行总流量 |

#### 3.5 节点测试

测速涉及到网络请求，方法调用需在子线程执行

```
//传入对应节点ip,返回节点每秒下载速度
//返回结果为0时，表示节点不可用，或测速失败
long speed = NwProxy.getNodeSpeed("1.1.1.1");
```

#### 3.6 设置通知栏内容文字

调用以下方法设置通知栏的内容，调用设置后，下一次更新通知栏生效

初始化时，设置了 .showLog(true)，视作调试默认，会加上host+端口号

```
NwProxy.setNotifyText("Welcome to Use ^_^");
```



