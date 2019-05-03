## WoLive 文档

### 安装

- wolive分为两个部分，wolive-pusher服务部分和web部分。wolive-pusher服务用来做websocket实时通讯，web部分用做api接口和页面展现。

#### wolive-pusher服务部分安装
1. 安装websocket服务所需workerman环境，参考http://www.workerman.net。

2. 打开wolive-pusher/config.php(win系统的话是wolive-pusher-win/config.php)，将$domain设置成实际客服系统要用的域名。
   安装后，检查9090和2080端口是否在防火墙中打开！

![输入图片说明](https://gitee.com/uploads/images/2017/1129/153722_07922fea_1288445.png "TIM截图20171129153704.png")

3. 进入到wolvie-pusher(-for-win)目录，命令行运行 php start.php start -d 启动websocket服务，界面类似如下。

![输入图片说明](https://gitee.com/uploads/images/2017/1129/153834_6644a656_1288445.png "TIM截图20171129153820.png")


#### wolvie-web部分安装

1. wolive-web 可以用nginx+php-fpm或者apache来运行(二者选其一即可)，配置参考如下。
nginx配置

```
server {
    listen 80;
    server_name  你的域名;
    root 实际磁盘路径/wolive-web/public;
    client_max_body_size 18M;
    index index.php index.html;
    location / {
        index index.html index.php;
    }
    location ~ \/upload\/.*\.php {
      deny all;
      return 404;
   }
    if (!-e $request_filename) {
        rewrite ^/index.php(.*)$ /index.php?s=$1 last;
        rewrite ^(.*)$ /index.php?s=$1 last;
        break;
    }
    location ~ \.php$ {
       add_header Access-Control-Allow-Origin *;
       fastcgi_pass    127.0.0.1:9000;
       include fastcgi_params;
       fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```
2.apache配置(需要httpd.conf配置文件中加载了mod_rewrite.so模块)

```
<Directory "实际磁盘路径/wolive-web/public">
     Options Indexes MultiViews
     AllowOverride All
     Require all granted
 </Directory>
 <VirtualHost *:80>
     DocumentRoot "实际磁盘路径/wolive-web/public"
     ServerName 你的域名
 </VirtualHost>

```

3.IIS配置

如果你的服务器环境支持ISAPI_Rewrite的话，可以配置httpd.ini文件，添加下面的内容：

```
RewriteRule (.*)$ /index\.php\?s=$1 [I]
```

在IIS的高版本下面可以配置web.Config，在中间添加rewrite节点：

```
<rewrite>
 <rules>
 <rule name="OrgPage" stopProcessing="true">
 <match url="^(.*)$" />
 <conditions logicalGrouping="MatchAll">
 <add input="{HTTP_HOST}" pattern="^(.*)$" />
 <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
 <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
 </conditions>
 <action type="Rewrite" url="index.php/{R:1}" />
 </rule>
 </rules>
 </rewrite>

```

2. 配置好后访问 http://你的域名/install.php ，然后根据步骤提示安装即可。
第一步、系统会自动检查环境是否ok。

![输入图片说明](https://gitee.com/uploads/images/2017/1129/155009_5a424bb5_1288445.png "TIM截图20171129154404.png")

第二步、配置数据库相关信息，其中服务器端口配置用默认值即可

![输入图片说明](https://gitee.com/uploads/images/2017/1129/155058_b208c104_1288445.png "TIM截图20171129154514.png")


第三步、安装成功

![输入图片说明](https://gitee.com/uploads/images/2017/1129/154616_9fb7e35d_1288445.png "TIM截图20171129154607.png")

### 数据库结构

1. wolive_service表
 - 客服列表 
```
 CREATE TABLE `wolive_service` (
  `service_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(255) NOT NULL COMMENT '用户名',
  `nick_name` varchar(255) NOT NULL COMMENT '昵称',
  `password` varchar(255) NOT NULL COMMENT '密码',
  `phone` varchar(255) DEFAULT '' COMMENT '手机',
  `email` varchar(255) DEFAULT '' COMMENT '邮箱',
  `business_id` varchar(255) NOT NULL COMMENT '商家id',
  `avatar` varchar(1024) NOT NULL DEFAULT '/assets/images/admin/avatar-admin2.png' COMMENT '头像',
  `level` enum('super_manager','manager','service') NOT NULL DEFAULT 'service' COMMENT 'super_manager: 超级管理员，manager：商家管理员 ，service：普通客服',
  `parent_id` int(11) NOT NULL DEFAULT '0' COMMENT '所属商家管理员id',
  `state` enum('online','offline') NOT NULL DEFAULT 'offline' COMMENT 'online：在线，offline：离线',
  PRIMARY KEY (`service_id`),
  UNIQUE KEY `se` (`service_id`) USING BTREE,
  KEY `pid` (`parent_id`) USING BTREE,
  KEY `web` (`business_id`) USING BTREE
) ENGINE=MyISAM  DEFAULT CHARSET=utf8;

```

2.wolive_visiter表

```
CREATE TABLE `wolive_visiter` (
  `vid` int(11) NOT NULL AUTO_INCREMENT,
  `visiter_id` varchar(200) NOT NULL COMMENT '访客id',
  `visiter_name` varchar(255) NOT NULL COMMENT '访客名称',
  `channel` varchar(255) NOT NULL COMMENT '用户游客频道',
  `avatar` varchar(1024) NOT NULL COMMENT '头像',
  `ip` varchar(255) NOT NULL COMMENT '访客ip',
  `from_url` varchar(255) NOT NULL COMMENT '访客浏览地址',
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '访问时间',
  `business_id` varchar(100) NOT NULL COMMENT '商户id',
  `state` enum('online','offline') NOT NULL DEFAULT 'offline' COMMENT 'offline：离线，online：在线',
  PRIMARY KEY (`vid`),
  UNIQUE KEY `id` (`visiter_id`,`business_id`) USING BTREE,
  KEY `visiter` (`visiter_id`) USING BTREE,
  KEY `time` (`timestamp`) USING BTREE
) ENGINE=MyISAM  DEFAULT CHARSET=utf8;
```

3.wolive_business表

```
CREATE TABLE `wolive_business` (
  `wid` int(11) NOT NULL AUTO_INCREMENT,
  `business_id` varchar(100) NOT NULL COMMENT '商家id',
  `distribution_rule` enum('auto','claim') DEFAULT 'claim' COMMENT 'claim:认领，auto:自动分配',
  `video_state` enum('close','open') NOT NULL DEFAULT 'close' COMMENT '是否开启视频',
  `audio_state` enum('close','open') NOT NULL DEFAULT 'close' COMMENT '是否开启音频',
  `voice_state` enum('close','open') NOT NULL DEFAULT 'open' COMMENT '是否开启提示音',
  `voice_address` varchar(255) NOT NULL DEFAULT '/upload/voice/default.mp3' COMMENT '提示音文件地址',
  `state` enum('close','open') NOT NULL DEFAULT 'open' COMMENT '''open'': 打开该商户 ，‘close’：禁止该商户',
  PRIMARY KEY (`wid`),
  UNIQUE KEY `bussiness` (`business_id`) USING BTREE
) ENGINE=MyISAM  DEFAULT CHARSET=utf8;
```
4.wolive_chats表

```
CREATE TABLE `wolive_chats` (
  `cid` int(11) NOT NULL AUTO_INCREMENT,
  `visiter_id` varchar(200) NOT NULL COMMENT '访客id',
  `service_id` int(11) NOT NULL COMMENT '客服id',
  `business_id` varchar(100) NOT NULL COMMENT '商家id',
  `content` mediumtext NOT NULL COMMENT '内容',
  `timestamp` int(11) NOT NULL,
  `state` enum('readed','unread') NOT NULL DEFAULT 'unread' COMMENT 'unread 未读；readed 已读',
  `direction` enum('to_visiter','to_service') DEFAULT NULL,
  PRIMARY KEY (`cid`),
  KEY `visiter` (`visiter_id`) USING BTREE,
  KEY `service` (`service_id`) USING BTREE,
  KEY `time` (`timestamp`) USING BTREE,
  KEY `chat` (`business_id`) USING BTREE
) ENGINE=MyISAM  DEFAULT CHARSET=utf8;
```
5.wolive_message表


```
CREATE TABLE `wolive_message` (
  `mid` int(11) NOT NULL AUTO_INCREMENT,
  `content` text NOT NULL COMMENT '留言内容',
  `name` varchar(255) NOT NULL COMMENT '留言人姓名',
  `moblie` varchar(255) NOT NULL COMMENT '留言人电话',
  `email` varchar(255) NOT NULL COMMENT '留言人邮箱',
  `business_id` varchar(100) DEFAULT NULL COMMENT '商家id',
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`mid`),
  KEY `timestamp` (`timestamp`),
  KEY `web` (`business_id`) USING BTREE
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

```

6.wolive_question表

```
CREATE TABLE `wolive_question` (
  `qid` int(11) NOT NULL AUTO_INCREMENT,
  `business_id` varchar(225) NOT NULL,
  `question` longtext NOT NULL,
  `answer` longtext NOT NULL,
  `answer_read` longtext NOT NULL,
  PRIMARY KEY (`qid`)
) ENGINE=MyISAM AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

```

7.wolive_queue表

```
CREATE TABLE `wolive_queue` (
  `qid` int(11) NOT NULL AUTO_INCREMENT,
  `visiter_id` varchar(200) NOT NULL COMMENT '访客id',
  `service_id` int(11) NOT NULL COMMENT '客服id',
  `business_id` varchar(100) NOT NULL COMMENT '商户id',
  `state` enum('normal','complete','in_black_list') NOT NULL DEFAULT 'normal' COMMENT 'normal：正常接入,‘complete’:已经解决，‘in_black_list’:黑名单',
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`qid`),
  UNIQUE KEY `id` (`visiter_id`,`business_id`) USING BTREE,
  KEY `se` (`service_id`) USING BTREE,
  KEY `vi` (`visiter_id`) USING BTREE,
  KEY `business` (`business_id`) USING BTREE
) ENGINE=MyISAM  DEFAULT CHARSET=utf8;

```

8.wolive_sentence表

```
CREATE TABLE `wolive_sentence` (
  `sid` int(11) NOT NULL AUTO_INCREMENT,
  `content` text NOT NULL COMMENT '内容',
  `service_id` int(11) NOT NULL COMMENT '所属客服id',
  `state` enum('using','unuse') DEFAULT 'unuse' COMMENT 'unuse: 未使用 ，using：使用中',
  PRIMARY KEY (`sid`),
  UNIQUE KEY `se` (`service_id`) USING BTREE
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```
9.wolive_tablist表

```
CREATE TABLE `wolive_tablist` (
  `tid` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL COMMENT 'tab的名称',
  `content` text NOT NULL,
  `content_read` text NOT NULL,
  `business_id` varchar(2555) NOT NULL,
  PRIMARY KEY (`tid`)
) ENGINE=MyISAM  DEFAULT CHARSET=utf8;

```

10.wolive_weixin表

```
CREATE TABLE `wolive_weixin` (
  `wid` int(11) NOT NULL AUTO_INCREMENT,
  `service_id` int(11) NOT NULL COMMENT '客服id',
  `open_id` varchar(255) NOT NULL COMMENT '微信用户id',
  PRIMARY KEY (`wid`),
  KEY `service_id` (`service_id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

```

### Pusher 的基本运用

#### 服务端

```php

<?php
// 引入pusher ,在 app\extra\push\Pusher.php
use app\extra\push\Pusher;
// 如果 $ahost 是http的
$options = array(
    'encrypted' => false
 );

// 如果 $ahost 是https的
$options = array(
    'encrypted' => true
 );

// 初始化服务端
 $pusher = new Pusher(
            $app_key,
            $app_secret,
            $app_id,
            $options,
            $ahost,
            $aport
        );


// 向my-chanel的seedmsg事件推送消息
 $pusher->trigger('my-channel', 'seedmsg', array('message' =>"hello"));

```


#### 客户端

```javascript
<script src="https://你的域名/assets/libs/push/pusher.min.js"></script>
<script>
  // app_key 就是安装步骤中的app_key
  var pusher = new Pusher(app_key, {
                encrypted: false
                , enabledTransports: ['ws']
                , wsHost: '你的域名'
                , wsPort: 9090
        });

// 如果websocket 地址是 wss 加密协议的，则如下
   var pusher = new Pusher(app_key, {
                encrypted: true
                , enabledTransports: ['wss']
                , wsHost: '你的域名'
                , wssPort: 9090
        });

// 订阅频道 my-channel
var channel = pusher.subscribe('my-channel');

// 监听该频道中 sendmsg 事件
channel.bind('sendmsg', function(data) {
        // 这里会接受到 data：{message:'hello'}
        alert(data.message);
    });

</script>

```

在wolive中 客服后台监听了3个频道
- 第一是 'kefu'+ 客服的id，监听以下事件：
  1. cu-event  接受访客的发送的消息
  2. logout    监听访客的离线状态，并把头像至灰
  3. geton     监听访客的在线状态，并把头像高亮
  4. video     监听视频申请请求
  5  video-refuse  监听对方拒绝视频的请求

- 第二是 'ud' + 客服的id，监听以下事件：
  1. on_notice 监听并获取被分配过来客服的信息

- 第三是 'all'+ 该客服所负责的wed
  1.on_notice 监听并告知所有该web下的客服，有访客在等待交谈

在wolive中 访客监听2个频道
- 第一个 "cu" + 访客的channel，监听以下事件：
  1. my-event  接受客服的发送的消息
  2. video     监听视频申请请求
  3. video-refuse  监听对方拒绝视频的请求
  4. cu_notice  监听访客被认领并且获取客服数据
- 第二个 "se" + 客服的id，监听以下事件：
  1. logout    监听客服的离线状态，并把头像至灰
  2. geton     监听客服的在线状态，并把头像高亮

### WoLive 公用API

#### 1.访客分配接口

- 访客自动分配方法在wolive项目中app/admin/controller/event.php中的notice方法。
- url访问 http://你的域名/admin/event/notice 。
- 需要传入参数，如下

```
{channel: channel, visiter_id: visiter_id, business_id: business_id, record: record, avatar: pic}
channel : 访客订阅的频道 channel ;
visiter_id : 访客的名称;
business_id  : 商户id;
record  : 是访客从那个网页开始寻求交谈的;
avatar  : 访客的头像

```
- 这些参数是通过<a href='你的域名' visiter_id='' visiter_name='' avatar='' business_id='admin' id='workerman-kefu'></a>来传入的。

```
visiter_id  : 传入访客的id，可以被传入，没有这自动生成，后台会被转化为 channel；
visiter_name : 传入访客的姓名，就是visiter_name
avatar      : 传入访客头像。
business_id : 的值就是商户id。

如果以上三个属性值都空，则项目会自动产生默认值。

```
-成功返回{code：0，msg:''，data:客服的数据json}
-返回{code：1，msg:'该网站已经停止客服咨询！'},表示该web已经停止客服咨询
-返回{code：2，msg:'等待认领'}，表示等待认领，排队中
-返回{code:3,msg:'请留言'}，表示所有客服下线，跳转留言页面

#### 2.发送消息接口

- 发送消息有两个接口。
- 第一个是客服发送消息，url: 你的域名/admin/set/chats。
- 需要传入数据,如下


```
{visiter_id:sid,content: msg, avatar: img}

visiter_id : 用户的id ；
avatar     :访客的头像;
content    :发消息的内容 ;

```
- 成功返回{code：0，msg:'success'}。
- 失败返回{code：1，msg：'错误信息'}。

- 第二个是客服发送消息
- 客服发送消息，url: 你的域名/admin/event/chats。
- 需要传入数据,如下

```

{visiter_id:visiter_id,service_id:service_id,content: msg,business_id: business_id, avatar: pic,record: record}
visiter_id : 用户的id ；
service_id : 客服的id；
content    :发消息的内容 ;
business_id:就是商户id;
avatar     :访客的头像;
record    ：记录访客需要咨询的地址情况


```
- 需要的参数和上面的一样
- 成功返回{code：0，msg：'success'}。
- 失败返回{code：3，msg："错误消息"}。

#### 3.上传图片

- url: 你的域名/admin/event/upload
- 需要传入个name="upload"的文件表单域
- 返回值为

```
         {
            "code":0,     // 表示上传成功
            "msg":"",
            "data":newimgpath  // 新图片路径
           }

```

#### 4.上传文件

- url: 你的域名/admin/event/uploadfile
- 需要传入个name="folder"的文件表单域
- 返回值为

```
         {
            "code":0,     // 表示上传成功
            "msg":"",
            "data":newpath  // 文件下载路径
           }

```

#### 5.离线访问
- 客服离线或访客离线，服务器访问的地址 
- url： 你的域名/admin/event 

```
<?php


 $webhook_signature = $_SERVER ['HTTP_X_PUSHER_SIGNATURE'];

 $body = file_get_contents('php://input');

 $expected_signature = hash_hmac('sha256', $body, $app_secret, false);

 if ($webhook_signature == $expected_signature) {

      $payload = json_decode($body, true);
     foreach ($payload['events'] as $event) {
        
               // 通知离线
                if ($event['name'] == 'channel_removed') {
                  // 处理自己的逻辑
                }

               // 通知在线
                if ($event['name'] == 'channel_added') {
                   // 处理自己的逻辑
                }
   }

}

```


### WoLive 后台Api

#### 对话平台Api

1.获取认领的对话列表
- url: 你的域名/admin/set/getchats
- 返回的是通过查询wolive_queue获取被认领的数据
- 成功返回格式：{code：0，data：“对话列表数据”}

2.获取排队的列表
- url:你的域名/admin/set/getwait
- 通过查询wolive_queue获取的是未被认领等待通话的数据
- 成功返回格式：{code：0，data:'排队人员数据'}

3.即使获取被删除的访客信息
- url: 你的域名/admin/set/getchatnow
- 需要传入访客的所有信息josn格式，信息值按照wolive_queue表
- 不小心删除对话表中的通话对象，如果对方继续发消息，会恢复在对话表中

4.获取黑名单列表
- url: 你的域名/admin/set/getblackdata 
- 通过查询wolive_queue获取被设定为黑名单的数据
- 成功返回格式：{code：0，data：“黑名单列表数据”}

5.获取未读消息数
- url：你的域名/admin/set/getmessage
- 需要传入访客的channel值
- 通过查询wolive_chats获取未读消息数

6.标记已看消息
- url: 你的域名/admin/set/getwatch
- 需要传入访客的channel值
- 修改wolive_chats中消息的状态
- 成功返回{code：0，msg：'success'}

7.标记某个访客为黑名单
- url: 你的域名/admin/set/blacklist
- 需要传入访客的channel值
- 修改wolive_queue中访客的状态
- 成功返回{code：0，msg：'success'}

显示最近一条消息是通过监听自己kefu频道储存在local storage中

#### 客服管理Api

1. 新增客服。
  - url: 你的域名/admin/manager/registForService
  - 需要传入数据，如下：

```
{
     user_name: user,  // 用户名
     nick_name: nick,  // 昵称
     password: pass1, // 密码
     password2: pass2,// 确认密码
     phone: phone,    // 手机号
     email: email,    // 邮箱
     avatar: path   // 客服头像
 }

```

- 成功，返回{code:0,msg:'成功'}
- 失败，返回{code:1,msg:'错误信息'}

2.头像上传
  - url : 你的域名/admin/manager/uploadpic
  - 需要传入个name="img_head"的文件表单域
  - 成功，返回{code:0,msg:''，data：'图片的地址'}
  - 失败，返回{code:1,msg:'错误信息'}

3.修改客服信息
  - url: 你的域名/admin/manager/update
  - 需要传入数据，如下：

```
{
     user_name: user,  // 用户名
     nick_name: nick,  // 昵称
     phone: phone,    // 手机号
     email: email,    // 邮箱
     company:company  // 公司
     avatar: path   // 客服头像
 }

```
- 成功，返回{code:0,msg:'成功'}
- 失败，返回{code:1,msg:'错误信息'}

1. 修改密码
  - url: 你的域名/admin/manager/modify
  - 需要传入数据，如下：

```
{
   oldpass: oldpass,    // 旧密码
   newpass: newpass,    // 新密码
   newpass2: newpass2   // 确认新密码
 }

```  
- 成功，返回{code:0,msg:'成功'}
- 失败，返回{code:1,msg:'错误信息'}

#### 查看历史消息Api

1. 查询与该客服交谈过的所有的访客。
  - url: 你的域名/admin/set/getvisiters
  - 需要传入数据，如下：

```
{service: user}
service : 客服的昵称 
```
- 成功返回{code:0,msg:'',data:'获取的数据'}

2. 查询客服和访客的对话。
 - 第一个url:  你的域名/admin/set/getviews
 - 需要传入数据，如下：

```
{channel: cha, puttime: times}
channel : 访客的channel
time ： 最近一周或最近一个月

```
- 第二个url:  你的域名/admin/set/getdesignForViews
- 需要传入数据，如下：

```
{channel: cha, start: s_time, end: e_time}
channel : 访客的channel
s_time  : 开始时间
end     : 结束时间
```
- 成功，返回{code:0,msg:'',data:'获取的数据'}

#### 问候语添加Api

1. 添加问候语
 - url : 你的域名/admin/manager/cmtalk
 - 需要传入数据，如下：
```
 {content：content}
  content ： 输入的问候语
```
- 成功，返回{code:0,msg:'成功'}

#### 添加常用问题

- url：你的域名/admin/manager/addquestion
- 需要传入，如下：

```
{question:q,answer:a,answer_read:ra,qid:qid}
```

#### 添加tab的编辑面版
- url:你的域名/admin/set/gettab
- 需要传入，如下:

```
{title:title,content:content,content_read:rcontent,tid:id}
```
#### 更换‘在线咨询’的图像
- 在wolive中public/assets/index/kefu_online.js中

```
$(".kefu_box").css({"z-index":"9999","text-align":"center","width":"60px","height": "60px","cursor": "pointer",'background': 'url("'+web+'/assets/images/index/im.png") no-repeat',"position": "fixed","top":"60%","right":"5%","background-size": "cover"});

url: 就是咨询的图像
```


