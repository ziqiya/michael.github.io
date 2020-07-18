---
layout: posts
title: 智能回复机器人
date: 2020-07-18 16:27:05
cover: /covers/robot.jpg
category: 前端
tags: [人工智能, webSocket, 前端, 后端]
excerpt: 各种市面上的智能机器人回复系统介绍及使用
---

## 前言

&emsp;&emsp;人工智能是一个当代的热门话题，也会是下一个技术发展的趋势。预言帝凯文·凯利就曾经预言未来 25 年内未来人工智能、虚拟现实以及追踪和跟踪是三大发展趋势。而市面上也已经有了一些例如叮咚智能音响，Siri，小爱等等的智能机器人，有的可以语音操控，有的可以语音聊天，对话等。而今天我们的目标就是实现一个我们自己的可以智能聊天的机器人。

## 方法

### 一、用酷 q 机器人 qq /微信接入

&emsp;&emsp;第一种方法就是借助市面上已有的机器人直接接入 qq 或微信，这边推荐比较好的一款是酷 q 机器人，可以在官网上申请注册一个账号，可以免费体验三天时间普通版，但是三天后只能用免费版，一天只能回复 10 条消息，后续需要 12 块钱 1 个月，像我们都是向来白嫖的，这个就比较奢侈了哈，还因为需要 windows 环境，mac 上 还要借助虚拟机。但是，毕竟是付费的，功能和智能化我觉得都是最好的，也能用来玩一些游戏。效果图如下：

### 二、用 QRspeed 手动设定回复机器人

&emsp;&emsp;第二种方法，就是用在电脑上跑一个模拟器上跑一个 QRSpeed 搭配 QRSpeed 词库，给机器人词库里写对应的语句，匹配到就会回复指定的语句。

下图为 QRSpeed 配置，另外需要在群列表中选中需要机器人运作的群。

![QRSpeed](/images/posts/robot/QRspeed.jpg)

上图中有一个 QRSpeed 词库的插件也需要安装一下，配置下主人和管理员，再导入下词库。

![QRSpeed 词库](/images/posts/robot/QRSpeedWord.jpg)

在 QRSpeed 词库中导入下图的词库，如图为词库(语法可以在网上找到，门槛比较低)：

![机器人截图1](/images/posts/robot/robot1.jpg)

这样设置以后再给机器人发消息就会自动回复了：

![机器人截图2](/images/posts/robot/robot2.jpg)

也可以找网上例如小浅等智能机器人的词库，还可以实现一些小游戏。

### 三、用 webSocket 搭后台服务，接机器人接口

&emsp;&emsp;这种方法就是今天重点了，用 webSocket 搭服务，这个我在之前的博客里讲过，就不展开讲了，接茉莉机器人接口，茉莉机器人,广泛应用于各类网站客服、QQ 机器人和微信公众平台。

#### 步骤一：申请接口

&emsp;&emsp;前往[茉莉机器人官网](http://www.itpk.cn/profile.php)，注册申请自己的 API Key，可以在个人中心设置机器人的名字等信息，也可以设置词库，获取到接入 URL。如图：

![茉莉机器人API]](/images/posts/robot/moliRobot1.jpg)

#### 步骤二：写前端页面

用 React 写一个简单的页面，我这里就直接把最后的代码放上来了。

```tsx
// 主文件
import React, { useState, useEffect, useRef } from 'react';
import { message, Button, Input } from 'antd';
import { useRequest } from 'ahooks';
import { useImmer } from 'use-immer';
import moment from 'moment';
import styles from './index.module.less';
import { BASE_URL, MSG_TYPE } from '../constant';
import classnames from 'classnames';
import Websocket from '@/utils/webSocket';
import { ChatDataProps } from 'interfaces/common';

const { TextArea } = Input;

// 初始化聊天配置
const INITIAL_CHAT_CONFIG = {
  userName: '',
  userContent: '',
};

const HomePage = () => {
  const [webSocket, setWebSocket] = useState<Websocket>();
  const [chatConfig, setChatConfig] = useImmer(INITIAL_CHAT_CONFIG);
  const chatListRef = useRef<HTMLDivElement>(null);
  const { userName, userContent } = chatConfig;

  /** 渲染聊天内容 */
  const renderChatItem = (chatItem: ChatDataProps) => {
    const { id, time, userName, content, msgType } = chatItem;
    const timeString = moment(time).format('YYYY-MM-DD HH:mm:ss');
    return msgType === MSG_TYPE.user ? (
      <div
        className={classnames(
          styles.chatRow,
          userName === chatConfig.userName ? styles.myChatRow : {}
        )}
        key={id}
      >
        <div className={styles.releaseTime}>{timeString}</div>
        <div className={styles.userLine}>
          <div className={styles.nameWrap}>
            <span className={styles.userName}>{userName}</span>说:
          </div>
          <div className={styles.chatWord}>{content}</div>
        </div>
      </div>
    ) : (
      <div className={styles.systemMessage} key={id}>
        <span className={styles.userName}>{userName}</span>
        {content}
      </div>
    );
  };

  // 获取聊天室数据
  const { data: chatData, run: fetchData } = useRequest(
    `${BASE_URL}/chatData`,
    {
      formatResult: (result) => result.data,
      onError: () => {
        message.error('请求失败!');
      },
    }
  );

  // 连接服务器
  const handleConnect = () => {
    let webSocketIo;
    // 发送成功/回调
    const callback = async () => {
      // 获取聊天室数据
      await fetchData();
      const dom = chatListRef.current;
      if (dom) {
        const scrollHeight = dom.scrollHeight;
        dom.scrollTop = scrollHeight;
      }
    };

    // 断开连接回调
    const failCallback = () => {
      setChatConfig((config) => {
        config.userName = '';
        config.userContent = '';
      });
    };
    if (!webSocket) {
      webSocketIo = new Websocket();
      const userName = localStorage.getItem('chatRoom-userName');
      setWebSocket(webSocketIo);
      setChatConfig((config) => {
        config.userName = userName || '';
      });
    }
    webSocketIo && webSocketIo.handleConnect({ callback, failCallback });
  };

  // 修改聊天昵称
  const handleChangeName = (e: React.ChangeEvent<HTMLInputElement>) => {
    const userName = e.target.value;
    setChatConfig((config) => {
      config.userName = userName;
    });
  };

  // 修改了昵称
  const handleBlurName = (e: React.ChangeEvent<HTMLInputElement>) => {
    const oldUserName = localStorage.getItem('chatRoom-userName');
    const userName = e.target.value;
    if (userName !== '' && userName !== oldUserName) {
      webSocket?.handleSendSystemMessage({ userName, status: 'online' });
      localStorage.setItem('chatRoom-userName', userName);
    }
  };

  // 发送信息
  const handleSubmit = () => {
    if (userName === '') {
      alert('请先输入昵称!');
      return;
    }
    if (userContent.trim() === '') {
      alert('发送内容不可为空!');
      return;
    }
    webSocket?.handleSendMessage({
      userName: userName,
      userContent: userContent,
    });
    setChatConfig((config) => {
      config.userContent = '';
    });
  };

  useEffect(() => {
    document.onkeydown = function (e: KeyboardEvent) {
      // 兼容FF和IE和Opera
      var theEvent = (window.event as KeyboardEvent) || e;
      var code = theEvent.keyCode || theEvent.which || theEvent.charCode;
      // 若按下回车键
      if (code == 13) {
        e.preventDefault();
        handleSubmit();
      }
    };
  }, [userName, userContent]);

  useEffect(() => {
    // 连接至服务器
    handleConnect();
  }, []);

  return (
    <div className={styles.container}>
      <div className={styles.chatWrap}>
        <div className={styles.chatHeader}>智能聊天室</div>
        <div ref={chatListRef} className={styles.chatList}>
          {chatData &&
            chatData.map((item: ChatDataProps) => renderChatItem(item))}
        </div>
        <div className={styles.sendBar}>
          <div className={styles.nameBar}>
            <Input
              className={styles.nameInput}
              placeholder="请输入昵称"
              onChange={handleChangeName}
              value={userName}
              onBlur={handleBlurName}
            />
            <Button className={styles.sendBtn} onClick={handleSubmit}>
              发送
            </Button>
          </div>
          <TextArea
            className={styles.wordInput}
            onChange={(e) => {
              const value = e.target.value;
              setChatConfig((config) => {
                config.userContent = value;
              });
            }}
            placeholder="请输入要说的话"
            value={userContent}
          />
        </div>
      </div>
    </div>
  );
};

export default HomePage;
```

```less
// 样式文件
.container {
  background: #eee;

  .chatWrap {
    display: flex;
    flex-direction: column;
    position: relative;
    width: 50%;
    height: 100vh;
    margin: 0 auto;
    box-shadow: 10px 6px 10px #ccc;
    border-radius: 4px;
    border: 1px solid #b9b9b9;
    overflow: hidden;
    justify-content: space-between;
    background: #fff;

    .chatHeader {
      background: #f2f3f5;
      padding: 10px;
      font-size: 16px;
      color: #333;
      border-bottom: 1px solid #e8e8e9;
    }

    .chatList {
      display: flex;
      flex: 1;
      flex-direction: column;
      overflow: auto;
      max-height: calc(100vh - 110px);
      padding-bottom: 60px;
      margin-bottom: 160px;

      .systemMessage {
        width: 100%;
        text-align: center;
        font-size: 14px;
        color: #666;
        margin: 6px 0;
      }

      .chatRow {
        padding: 4px 12px;
      }

      .myChatRow {
        display: flex;
        flex-direction: column;
        align-items: flex-end;
      }

      .releaseTime {
        color: #999;
        font-size: 14px;
        margin-bottom: 6px;
      }

      .userLine {
        display: flex;
        color: #666;
        font-size: 14px;
      }

      .nameWrap {
        white-space: nowrap;
        margin-right: 10px;
      }

      .userName {
        color: #378ddf;
        font-size: 14px;
        margin-right: 4px;
      }
    }

    .sendBar {
      position: absolute;
      bottom: 0;
      width: 100%;
      background: #eee;
      box-shadow: 0 2px 10px #999;

      .nameBar {
        display: flex;
        justify-content: space-between;
        padding: 10px 2% 0;

        .nameInput {
          width: 200px;
          border: 0;
          padding: 4px 10px;
          font-size: 14px;
        }

        .sendBtn {
          background: rgba(55, 141, 223, 0.8);
          border-radius: 4px;
          color: #fff;
          padding: 0 10px;
          outline: none;
          cursor: pointer;
        }

        .sendBtn:hover {
          background: #378ddf;
        }
      }

      .wordInput {
        width: 96%;
        height: 100px;
        padding: 4px 10px;
        margin: 10px 2%;
        border: 0;
        font-size: 14px;
        box-sizing: border-box;
      }
    }
  }
}
```

这样就可以简单地搭出一个聊天室界面了，效果如下：

![聊天室1](/images/posts/robot/chatRoom1.jpg)

#### 步骤三：用 node.js 写一个服务端

这里我 `webSocket` 使用的是 `socket.io` 的框架，代码如下：

```js
// node端
//引入koa
const Koa = require('koa');
const app = new Koa();
const mysql = require('./mysql');
//创建http服务
var server = require('http').createServer(app.callback());

const request = require('request');

const Router = require('koa-router');

var config = require('./config/default.js');

const { robot, msgType } = config;

const { ROBOT_NAME, ROBOT_URL, API_KEY, API_SECRET } = robot;

// 跨域请求
const cors = require('koa2-cors');

const router = new Router();

// 跨域配置
app.use(
  cors({
    origin: function (ctx) {
      return '*'; // 允许来自所有域名请求
    },
    exposeHeaders: ['WWW-Authenticate', 'Server-Authorization'],
    maxAge: 5,
    credentials: true,
    allowMethods: ['GET', 'POST', 'DELETE'],
    allowHeaders: ['Content-Type', 'Authorization', 'Accept'],
  })
);

//给http封装成io对象
var io = require('socket.io')(server);
// 建立链接
io.on('connection', function (socket) {
  // io.emit代表广播，socket.emit代表私发
  socket.on('sendMessage', async function (content) {
    await mysql.addChatData(content);
    io.emit('getMessage', content);

    // 若不为系统消息，机器人自动回复（实现机器人自动回复主要就是这一段）
    if (content.msgType !== msgType.SYSTEM) {
      request(
        `${ROBOT_URL}?question=${encodeURI(
          content.userContent
        )}&api_key=${API_KEY}&api_secret=${API_SECRET}`,
        async function (error, response, body) {
          if (!error && response.statusCode == 200) {
            const robotContent = {
              userContent: body,
              userName: ROBOT_NAME,
              msgType: msgType.USER,
            };
            await mysql.addChatData(robotContent);
            io.emit('getMessage', robotContent);
          }
        }
      );
    }
  });
});

router.get('/chatData', async (ctx, next) => {
  const result = await mysql.getChatData();
  ctx.body = {
    code: 20000,
    data: result,
    message: '请求成功',
    success: true,
  };
});

app.use(router.routes());

app.use((ctx) => {
  ctx.response.body = '服务器运行中';
});

server.listen(3000, function () {
  console.log('Server listening on port 3000');
});
```

##### Socket.IO

这里介绍一下 `Socket.IO`，`Socket.IO` 是一个库，可用于在浏览器和服务器之间进行实时，双向和基于事件的通信。在客户端使用 `io.connect(BASE_URL)` 可以实例化一个 `socket` 对象，跟服务器端建立 WebSocket 连接，就可以用 `socket.on（'connect',func)` 监听 `connect` 事件，如果服务器开启，客户端会自动连接上服务器，执行一些在建立连接后的事件。

以下为 mysql 的连接池和接口请求配置，服务器端启动时需要启动 mysql 的服务：

```js
var mysql = require('mysql');
var config = require('../config/default.js');
var moment = require('moment');

var pool = mysql.createPool({
  host: config.database.HOST,
  user: config.database.USERNAME,
  password: config.database.PASSWORD,
  database: config.database.DATABASE,
});

class Mysql {
  constructor() {}
  getChatData() {
    return new Promise((resolve, reject) => {
      pool.query('SELECT * from chatWords', function (error, results, fields) {
        let counts = 0;
        if (error) {
          throw error;
        }
        results.forEach((item) => {
          const userName = pool.query(
            `SELECT username from user WHERE id=${item.userId}`,
            function (error, result) {
              try {
                if (error) {
                  throw error;
                }
                item.userName = result[0].username;
                counts++;
                if (counts === results.length) {
                  resolve(results);
                }
              } catch (error) {
                console.log('error: ', error);
                item.userName = '匿名';
              }
            }
          );
        });
      });
    });
  }
  addChatData(chatData) {
    const { userName, userContent, msgType = 1 } = chatData;
    const nowTime = moment(new Date()).format('YYYY-MM-DD HH-mm-ss');
    return new Promise((reslove, reject) => {
      pool.query(`SELECT id from user WHERE username="${userName}"`, function (
        error,
        results,
        fields
      ) {
        try {
          if (error) {
            throw error;
          }
          if (results.length === 0) {
            pool.query(
              `INSERT INTO user (username) VALUES ("${userName}")`,
              function (error, result) {
                if (error) {
                  throw error;
                }
                pool.query(
                  `INSERT INTO chatWords (content,time,userId,msgType) VALUES ("${userContent}","${nowTime}",${result.insertId},${msgType})`,
                  function (error, result) {
                    if (error) {
                      throw error;
                    }
                    reslove(true);
                  }
                );
              }
            );
          } else {
            pool.query(
              `INSERT INTO chatWords (content,time,userId,msgType) VALUES ("${userContent}","${nowTime}",${results[0].id},${msgType})`,
              function (error, result) {
                if (error) {
                  throw error;
                }
                reslove(true);
              }
            );
          }
        } catch (error) {
          console.log('error: ', error);
        }
      });
    });
  }
}

module.exports = new Mysql();
```

以下是服务端配置文件,在 robot 的配置里把 API_KEY 和 API_SECRET 换成自己的：

```js
// ./config/default.js
const config = {
  // 启动端口
  port: 3000,

  // 数据库配置
  database: {
    DATABASE: 'first',
    USERNAME: 'root',
    PASSWORD: '123456',
    PORT: '3306',
    HOST: 'localhost',
  },

  // 机器人配置
  robot: {
    ROBOT_NAME: '汐汐AI',
    ROBOT_URL: 'http://i.itpk.cn/api.php',
    API_KEY: 'xxxxx',
    API_SECRET: 'xxxxxx',
  },

  // 消息类型枚举
  msgType: {
    USER: 1,
    SYSTEM: 2,
  },
};

module.exports = config;
```

#### 步骤四：在客户端写一个 WebSocket 类

在客户端写一个 WebSocket 类，代码如下，因为我服务端的 `webSocket` 用的是 `socket.io` 的框架，所以客户端就要用 `socket.io-client` 插件并引入。

```ts
// @/utils/webSocket

import io from 'socket.io-client';
import { MSG_TYPE, BASE_URL } from '@/pages/constant';

export interface MsgContentProps {
  userName: string;
  userContent: string;
  msgType?: number;
}

/**
 * @功能描述: Websocket类
 * @参数:
 * @返回值:
 */
export default class Websocket {
  private socket: any;
  // 连接到服务器
  handleConnect({
    callback,
    failCallback,
  }: {
    callback: () => void;
    failCallback: () => void;
  }) {
    try {
      const _this = this;
      this.socket = io.connect(BASE_URL);
      this.socket.on('connect', function () {
        const userName = localStorage.getItem('chatRoom-userName');
        const userStatus = localStorage.getItem('chatRoom-userStatus');
        if (userName && userStatus !== 'online') {
          _this.handleSendSystemMessage({ userName, status: 'online' });
        }
        // 执行回调
        callback && callback();
      });
      this.socket.on('getMessage', function () {
        // 服务端收到信息回调
        callback && callback();
      });
      this.socket.on('disconnect', function () {
        alert('服务器已断开连接');
        _this.socket = null;
        failCallback && failCallback();
      });
    } catch (err) {
      console.log('err: ', err);
      alert('服务器未启动!');
    }
  }
  // 断开服务器
  handleDisconnect() {
    if (this.socket) {
      this.socket.disconnect(true);
    } else {
      alert('还未连接到服务器！');
    }
  }
  // 点击发送内容给服务器
  handleSendMessage(content: MsgContentProps) {
    if (this.socket) {
      this.socket.emit('sendMessage', content);
    } else {
      alert('请先连接到服务器！');
    }
  }
  /**
   * @功能描述: 发送系统消息
   * @参数: @param userName:用户名 @param status:用户状态(online/offline)
   * @返回值:
   */
  handleSendSystemMessage({
    userName,
    status,
  }: {
    userName: string;
    status: string;
  }) {
    if (status === 'online') {
      this.handleSendMessage({
        userName,
        userContent: '加入了聊天室',
        msgType: MSG_TYPE.system,
      });
    } else {
      this.handleSendMessage({
        userName,
        userContent: '离开了聊天室',
        msgType: MSG_TYPE.system,
      });
    }
    localStorage.setItem('chatRoom-userStatus', status);
  }
}
```

以下为客户端常量配置：

```js
// @/pages/constant

// 消息类型映射
export const MSG_TYPE = Object.freeze({
  user: 1,
  system: 2,
});
export const BASE_URL = 'http://localhost:3000';
```

配置完成后就可以在客户端发送信息的时候，服务器端通过 webSocket 拿到前端调用接口的参数，通过判断信息类型以及内容，调用机器人回复接口，进行对应的回复。效果如下：

![聊天室截图](/images/posts/robot/chatRoom2.jpg)

这样，一个简单的机器人就完成啦。
