# js-for-apache-dubbo

---

![love dubbo](https://raw.githubusercontent.com/dubbo/dubbo2.js/master/resources/dubbo-love.png)

[![NPM](https://nodei.co/npm/dubbo2.js.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/dubbo2.js)

js-for-apache-dubbo = Nodejs connected Java Dubbo service by dubbo protocol (dubbo head + hessian body)

After all these years, js-for-apache-dubbo comes finally~ We love dubbo!👏

With core module [js-to-java](https://github.com/node-modules/js-to-java), and [hessian.js](https://github.com/node-modules/hessian.js), dubbo2 is finished. And many thanks to [fengmk2](https://github.com/fengmk2)和[dead-horse](https://github.com/dead-horse).

# Features

---

1. Keep it Simple (build tools for humans).

2. Support zookeeper as register center

3. TCP Dubbo Native protocol （Dubbo Header + Hessian Body）

4. Socket Container (ServerAgent -> SocketWorker)

5. Support Directly Dubbo (const Dubbo = DirectlyDubbo({..}))

6. Middleware the same as Koa middleware, Easy to extend.

7. Tracing (runtime info, call stack)

8. Supported Dubbox

9. Typescript type definition

10. Convert java dubbo interface to typescript module by interpret tools

11. SocketWorker was disconnected auto retry

# Oops

Oops, please dont's use dubbo2.js@2.2.2 dubbo2.js@2.3.5.
When zookeper session expires, it causes the dubbo2.js's zookeeper client to attempt to reconnect indefinitely.

# Tip

dubbo2.6.3:Support implicit delivery of attachments from provider to consumer, [#889](https://github.com/apache/incubator-dubbo/issues/889)

dubbo2.js@2.3.7 support the feature.

# Getting Started

---

yarn add dubbo2.js

# How to Usage?

---

## dubbo2.js@2.0.4+

```typescript
//=====================service.ts==================
//generated by interpret tools
import {BasicTypeProvider} from './providers/com/alibaba/dubbo/demo/BasicTypeProvider';
import {DemoProvider} from './providers/com/alibaba/dubbo/demo/DemoProvider';
import {ErrorProvider} from './providers/com/alibaba/dubbo/demo/ErrorProvider';

export default {
  BasicTypeProvider,
  DemoProvider,
  ErrorProvider,
};

//===============dubbo.ts========================
import {Dubbo} from 'dubbo2.js';
import service from './service';

//Create dubbo object
const dubbo = new Dubbo<typeof service>({
  application: {name: 'node-dubbo'},
  //zookeeper address
  register: 'localhost:2181',
  service,
});

//main method
(async () => {
  let {res, err} = await dubbo.service.DemoProvider.sayHello('node');
  //print {err: null, res:'hello node from dubbo service'}
  ({res, err} = await dubbo.service.DemoProvider.echo());
  //print {err: null, res: 'pang'}
  ({res, err} = await dubbo.service.DemoProvider.getUserInfo());
  //status: 'ok', info: { id: '1', name: 'test' }
})();
```

### If the typescript code is not automatically generated from java, how to inject it into dubbo object?

---

```typescript
//Create the service to be injected
import {Dubbo} from 'dubbo2.js';
const demoProvider = dubbo =>
  dubbo.proxyService({
    dubboInterface: 'com.alibaba.dubbo.demo.DemoProvider',
    version: '1.0.0',
    methods: {
      sayHello(name) {
        return [java.String(name)];
      },

      echo() {},

      test() {},

      getUserInfo() {
        return [
          java.combine('com.alibaba.dubbo.demo.UserRequest', {
            id: 1,
            name: 'nodejs',
            email: 'node@qianmi.com',
          }),
        ];
      },
    },
  });

//Integrate the service in demoProvider and dubbo object constructor
const service = {
  demoProvider,
};

const dubbo = new Dubbo<typeof service>({
  //  ....other parameters
  service,
});
```

## [dubbo2.js@1.xxx](https://github.com/dubbo/dubbo2.js/blob/master/README.md#dubbo2js1xxx)

```typescript
import {Dubbo, java, TDubboCallResult} from 'dubbo2.js';
//generated by interpret tools
import {BasicTypeProvider} from './providers/com/alibaba/dubbo/demo/BasicTypeProvider';
import {DemoProvider} from './providers/com/alibaba/dubbo/demo/DemoProvider';
import {ErrorProvider} from './providers/com/alibaba/dubbo/demo/ErrorProvider';

//Create dubbo object
const dubbo = new Dubbo({
  application: {name: 'node-dubbo'},
  //zookeeper address
  register: 'localhost:2181',
  interfaces: [
    'com.alibaba.dubbo.demo.DemoProvider',
    'com.alibaba.dubbo.demo.BasicTypeProvider',
    'com.alibaba.dubbo.demo.ErrorProvider',
  ],
});

const basicTypeProvider = BasicTypeProvider(dubbo);
const demoProvider = DemoProvider(dubbo);
const errorProvider = ErrorProvider(dubbo);

//main method
(async () => {
  let {res, err} = await demoProvider.sayHello('node');
  //print {err: null, res:'hello node from dubbo service'}
  ({res, err} = await demoProvider.echo());
  //print {err: null, res: 'pang'}
  ({res, err} = await demoProvider.getUserInfo());
  //status: 'ok', info: { id: '1', name: 'test' }
})();
```

# as developer

---

```sh
brew install zookeeper
brew services start zookeeper

#Run test example in java/dubbo-demo-provider
yarn run test

#Full link log tracking
DEBUG=dubbo\* yarn run test
```

# dubbo flow graph

![dubbo-flow](https://github.com/chaoshimeng/dubbo/blob/master/image/1234.jpg)

# API

---

## create dubbo object

```typescript
const dubbo = new Dubbo({
  isSupportedDubbox //Support dubbox or not: selectable, type:Boolean, false as default
  application //Name of the application: selectable, write the consumer type when zookeeper is called: {name: string}
  dubboInvokeTimeout //Dubbo timeout: selectable, 10s as default, type: number
  dubboSocketPool //Set the size of socket pool: selectable, 4 as default, type: number
  register //Set zookeeper registration center address: required, type: string
  zkRoot //Default root path of zk: ‘ /dubbo’ as default, type: string
  interfaces //Set the interface identifier for zk montoring: required, type: Array<string>, no longer exist in version dubbo2.js@2.0.4+
  service //Dubbo service injected in dubbo container: type: Object, used in version dubbo2.js@2.0.4+
});

// Or( Same as above)
const dubbo = Dubbo.from({
  isSupportedDubbox //Support dubbox or not: selectable, type:Boolean, false as default
  application //Name of the application: selectable, write the consumer type when zookeeper is called: {name: string}
  dubboInvokeTimeout //Dubbo timeout: selectable, 10s as default, type: number
  dubboSocketPool //Set the size of socket pool: selectable, 4 as default, type: number
  register //Set zookeeper registration center address: required, type: string
  zkRoot //Default root path of zk: ‘ /dubbo’ as default, type: string
  interfaces //Set the interface identifier for zk montoring: required, type: Array<string>, no longer exist in version dubbo2.js@2.0.4+
  service //Dubbo service injected in dubbo container: type: Object, used in version dubbo2.js@2.0.4+
})

//Agent service in dubbo
const demoSerivce = dubbo.proxService({
  //Agent service interface: string, required
  dubboInterface: 'com.alibaba.dubbo.demo.DemoService',
  //Version of the service interface: string, selectable
  version: '1.0.0',
  //Timeout: number, selectable
  timeout: 10
  //Group: string, selectable
  group: 'qianmi',
  //Methods within the interface: Array<Function>, required
  methods: {
  //method name
  sayHello(name) {
  //Only for parameter conversion(hessian)
  return [java.String(name)];
  },
  //method name
  getUserInfo() {
    //Only for parameter conversion(hessian)
    return [java.combine('com.alibaba.dubbo.demo.UserRequest', {
      id: 1,
      name: 'nodejs',
      email: 'node@qianmi.com',
      })];
    },
  },
})
```

## connect dubbo directly

---

```typescript
import {DirectlyDubbo, java} from 'dubbo2.js';
import {
  DemoProvider,
  DemoProviderWrapper,
  IDemoProvider,
} from './providers/com/alibaba/dubbo/demo/DemoProvider';
import {UserRequest} from './providers/com/alibaba/dubbo/demo/UserRequest';

const dubbo = DirectlyDubbo.from({
  dubboAddress: 'localhost:20880',
  dubboVersion: '2.0.0',
  dubboInvokeTimeout: 10,
});

const demoService = dubbo.proxyService<IDemoProvider>({
  dubboInterface: 'com.alibaba.dubbo.demo.DemoProvider',
  methods: DemoProviderWrapper,
  version: '1.0.0',
});
```

# When dubbo was ready?

---

```typescript
const dubbo = Dubbo.from(/*...*/);

(async () => {
  await dubbo.ready();
  //TODO dubbo was ready
})();

//for example, in egg.js
app.beforeStart(async () => {
  await dubbo.ready();
  app.logger.info('dubbo was ready...');
});
```

# How to trace dubbo2.js runtime system info?

---

```typescript
const dubbo = Dubbo.from(/*...*/);

//Trace by ‘subscribe’
dubbo.subcribe({
  onTrace(msg: ITrace) {
    //logger msg
  },
});
```

You will get all runtim system info just like this.

```text
{ type: 'INFO', msg: 'dubbo:bootstrap version => 2.1.5' }
    { type: 'INFO', msg: 'connected to zkserver localhost:2181' }
    { type: 'INFO',
      msg: 'ServerAgent create socket-pool: 172.19.6.203:20880' }
    { type: 'INFO',
      msg: 'socket-pool: 172.19.6.203:20880 poolSize: 1' }
    { type: 'INFO',
      msg: 'new SocketWorker#1 |> 172.19.6.203:20880' }
    { type: 'INFO',
      msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
    { type: 'INFO',
      msg: 'SocketWorker#1 <=connected=> 172.19.6.203:20880' }
    { type: 'INFO', msg: 'scheduler is ready' }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.DemoProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.ErrorProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.BasicTypeProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'ERR',
      msg: Error: Can not be found any agents
    at Object.Scheduler._handleZkClientOnData [as onData] (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/scheduler.js:68:29)
    at EventEmitter.<anonymous> (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/zookeeper.js:275:30)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:118:7) }
    { type: 'ERR',
      msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 6
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
    { type: 'INFO',
      msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
    { type: 'ERR',
      msg:
       { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
    { type: 'ERR',
      msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 5
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.DemoProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.BasicTypeProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.ErrorProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.ErrorProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.DemoProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.BasicTypeProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'ERR',
      msg: Error: Can not be found any agents
    at Object.Scheduler._handleZkClientOnData [as onData] (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/scheduler.js:68:29)
    at EventEmitter.<anonymous> (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/zookeeper.js:275:30)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:118:7) }
    { type: 'INFO',
      msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
    { type: 'ERR',
      msg:
       { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
    { type: 'ERR',
      msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 4
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
    { type: 'INFO',
      msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
    { type: 'ERR',
      msg:
       { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
    { type: 'ERR',
      msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 3
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
    { type: 'INFO',
      msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
    { type: 'ERR',
      msg:
       { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
    { type: 'ERR',
      msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 2
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
    { type: 'INFO',
      msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
    { type: 'ERR',
      msg:
       { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
    { type: 'ERR',
      msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 1
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
    { type: 'INFO',
      msg: 'SocketWorker#1 =connecting=> 172.19.6.203:20880' }
    { type: 'ERR',
      msg:
       { Error: connect ECONNREFUSED 172.19.6.203:20880
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1173:14)
     errno: 'ECONNREFUSED',
     code: 'ECONNREFUSED',
     syscall: 'connect',
     address: '172.19.6.203',
     port: 20880 } }
    { type: 'ERR',
      msg: Error: SocketWorker#1 <=closed=> 172.19.6.203:20880 retry: 0
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:78:29)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
    { type: 'ERR',
      msg: Error: 172.19.6.203:20880's pool socket-worker had all closed. delete 172.19.6.203:20880
    at ServerAgent._clearClosedPool (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/server-agent.js:66:33)
    at Object.onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/server-agent.js:51:34)
    at SocketWorker._onClose (/Users/hufeng/Github/dubbo2.js/packages/dubbo/es7/socket-worker.js:97:34)
    at Socket.emit (events.js:180:13)
    at TCP._handle.close [as _onclose] (net.js:541:12) }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.DemoProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'INFO',
      msg: 'ServerAgent create socket-pool: 172.19.6.203:20880' }
    { type: 'INFO',
      msg: 'socket-pool: 172.19.6.203:20880 poolSize: 1' }
    { type: 'INFO',
      msg: 'new SocketWorker#2 |> 172.19.6.203:20880' }
    { type: 'INFO',
      msg: 'SocketWorker#2 =connecting=> 172.19.6.203:20880' }
    { type: 'INFO',
      msg: 'SocketWorker#2 <=connected=> 172.19.6.203:20880' }
    { type: 'INFO', msg: 'scheduler is ready' }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.BasicTypeProvider/providers, type: NODE_CHILDREN_CHANGED' }
    { type: 'INFO',
      msg: 'trigger watch /dubbo/com.alibaba.dubbo.demo.ErrorProvider/providers, type: NODE_CHILDREN_CHANGED' }
```

# middleware

---

Use calling link abstractly and the same middleware mechanism as koa. It’s convenient to customize interceptors, such as loggers.

```typescript
//cost-time middleware
dubbo.use(async (ctx, next) => {
  const startTime = Date.now();
  await next();
  const endTime = Date.now();
  console.log('invoke cost time->', endTime - startTime);
});
```

dubbo2.6.3:Support implicit delivery of attachments from provider to consumer, [#889](https://github.com/apache/incubator-dubbo/issues/889)
How to get providerAttachments?. Because we have middleware, it's so easy :)

```typescript
dubbo.use(async (ctx, next) => {
  await next();
  //yes, we get it.
  console.log(ctx.providerAttachments);
});
```

# dubbo-invoker

---

Set dynamic parameters in the interface interchange of dubbo, such as version, group, timeout, retry, etc.

Assign these parameters precisely when calling consumer.

Abstract a dubbo-invoker as the middleware for seting parameters in order to set various runtime parameters dynamically convenientely.

```typescript
import {dubboInvoker, matcher} from 'dubbo-invoker';

//init
const dubbo = Dubbo.from(/*....*/);

const matchRuler = matcher
  //match interface precisely
  .match('com.alibaba.demo.UserProvider', {
    version: '1.0.0',
    group: 'user',
  })
  //match thunk
  .match(ctx => {
    if (ctx.dubboInterface === 'com.alibaba.demo.ProductProvider') {
      ctx.version = '2.0.0';
      ctx.group = 'product-center';
      //inform dubboInvoker of successful match
      return true;
    }
  })
  //regular matching
  .match(/^com.alibaba.dubbo/, {
    version: '2.0.0',
    group: '',
  });

dubbo.use(dubboInvoke(matchRuler));
```

[dubbo2.js@2.3.5 we add dubboSetting](https://github.com/dubbo/dubbo2.js/releases/tag/dubbo2.js%402.3.5)

# Translator => Cool.

Believing the experience of development is as important as that of users, some innovations and cool practices are made.

[Translator](https://github.com/dubbo/dubbo2.js/blob/master/packages/interpret-cli) is designed and implemented to make the call between node and dubbo is the same simple and transparent as java calling dubbo.

Analyze bytecode in java jar package and extract interface information in dubbo. Typescript type definition file and callable code is generated automatically.

More examples, Please visit [interpret-example](https://github.com/creasy2010/interpret-example)

## _Duty_

1. translate Interface code for node-side calling
2. convert the parameters to recognizable objects for hessian.js automatically
3. interface method and parameter type promption

[a detailed description of translator](https://github.com/dubbo/dubbo2.js/blob/master/packages/interpret-cli)

# Performance

```text
❯ loadtest -t 20 -c 200 http://localhost:3000/dubbo -k
[Wed Jun 20 2018 15:10:16 GMT+0800 (CST)] INFO Requests: 0, requests per second: 0, mean latency: 0 ms
[Wed Jun 20 2018 15:10:21 GMT+0800 (CST)] INFO Requests: 37305, requests per second: 7484, mean latency: 26.9 ms
[Wed Jun 20 2018 15:10:26 GMT+0800 (CST)] INFO Requests: 79187, requests per second: 8371, mean latency: 23.9 ms
[Wed Jun 20 2018 15:10:31 GMT+0800 (CST)] INFO Requests: 120374, requests per second: 8247, mean latency: 24.2 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Target URL: http://localhost:3000/dubbo
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Max time (s):20
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Concurrency level: 200
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Agent: keepalive
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Completed requests: 161828
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Total errors:0
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Total time: 20.000902374000002 s
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Requests per second: 8091
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Mean latency:24.7 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO Percentage of the requests served within a certain time
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO 50% 22 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO 90% 30 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO 95% 34 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO 99% 49 ms
[Wed Jun 20 2018 15:10:36 GMT+0800 (CST)] INFO 100% 134 ms (longest request)
```

# FAQ

---

```typescript
import {Dubbo} from 'dubbo2.js';
```

The default compiling language: es2017 (support node7.10+)

For lower version of node:

```typescript
import {Dubbo} from 'dubbo2.js/es6';
```

# How to participate in development？

---

1. welcome pr

2. welcome issue

# Sharing

---

[Click here!](https://github.com/hufeng/iThink/tree/master/talk)
