## 网络连接模块

#### 版本更新
TrafficStats自动说明

| 版本  | 更新说明                      |
| ------- | ------------------------- |
| 1.0.3  | 1.增加默认密码设置 <br>              |

##  1. 依赖项配置

指定maven仓库地址

```
maven { url 'http://maven.shlsnetwork.com/nexus/content/repositories/releases/' }
```

添加SDK依赖
```
implementation 'com.android.socks:NetProxy:1.0.3'
```



## 2. 初始化

    NwProxy.init(appliction, new ProxyConfig.Builder()
                    //数据接口回调，用来获取节点信息
                    //这边实现remote config的数据接口就行
                    .remoteCallback(new IRemoteCallback() {
                        @Override
                        public String getString(String key) {
                            return FireBaseRemoteConfig.getString(key);
                        }
                    })
                    //通知栏点击后需要跳转的activity。如果设置为空，不显示默认通知栏
                    .launchClass(MainActivity.class)
                    //连接的默认密码
                    .defaultPw("aaaxdddsffddd")
                    //显示日志开关
                    .showLog(BuildConfig.DEBUG)
                    .build());

## 3. 接口调用
#### 3.1 获取节点列表

    NwProxy.getNodeList(new INodeCallback() {
        @Override
        public void onResponse(List<ServerNode> list) {
            //获得回调的节点列表            
        }
    });

ServerNode相关字段说明

| 字段名 | 数据类型| 说明        |
| ------ | ------- | -------------- |
| id      | long| 节点ID             |
| name      | String| 节点名称             |
| host      | String| 节点IP             |
| port      | int| 端口号             |
| order      | int| 排序顺序             |


#### 3.2 启动网络代理服务

```
//自动选择最优节点
NwProxy.start();
//选择指定节点启动
NwProxy.start(nodeId);
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

