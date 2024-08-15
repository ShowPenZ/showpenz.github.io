---
title: 依赖倒置解耦的简单事例(vue3)
date: 2024-08-15 10:34:02
tags:
    - vue3
    - 设计模式
---

# 场景1：没有依赖倒置的耦合代码
## 1. 定义具体的消息服务类
``` javascript
// services/EmailService.js
export default class EmailService {
  send(to, message) {
    console.log(`Sending email to ${to}: ${message}`);
  }
}

// services/SMSService.js
export default class SMSService {
  send(to, message) {
    console.log(`Sending SMS to ${to}: ${message}`);
  }
}
```
## 2. 在组件中直接依赖具体的服务
```html
<!-- components/NotificationComponent.vue -->
<template>
  <div>
    <button @click="sendNotification">Send Notification</button>
  </div>
</template>

<script setup>
// 直接引入具体的消息服务实现
import EmailService from '../services/EmailService';
// import SMSService from '../services/SMSService'; // 如果想使用 SMS 服务，需要手动修改这里

const emailService = new EmailService();
// const smsService = new SMSService(); // 如果要切换到 SMS 服务，需要手动创建实例

const sendNotification = () => {
  emailService.send('user@example.com', 'Hello without Dependency Inversion!');
  // smsService.send('123-456-7890', 'Hello via SMS!'); // 如果要使用 SMS 服务，需要手动修改这里
};
</script>
```
## 场景1结论

假设你想从 EmailService 切换到 SMSService，你需要在组件代码中手动进行以下修改：
- 修改 import 语句，切换到新的服务类。
- 修改实例化代码，使用新的服务类创建实例。
- 修改所有调用代码，确保使用新的服务实例发送消息。

# 场景2：使用依赖倒置解耦合代码
## 1. 定义 MessageService 抽象接口
``` javascript
// services/MessageService.js
export default class MessageService {
  send(to, message) {
    throw new Error('This method should be overridden');
  }
}
```
## 2. 定义具体的消息服务类
``` javascript
// services/EmailService.js
import MessageService from './MessageService';

export default class EmailService extends MessageService {
  send(to, message) {
    console.log(`Sending email to ${to}: ${message}`);
  }
}

// services/SMSService.js
import MessageService from './MessageService';

export default class SMSService extends MessageService {
  send(to, message) {
    console.log(`Sending SMS to ${to}: ${message}`);
  }
}
```
## 3. 在页面中提供消息服务
```html
<!-- page.vue -->
<template>
  <div>
    <NotificationComponent />
  </div>
</template>

<script setup>
import { provide } from 'vue';
import EmailService from './services/EmailService';
// import SMSService from './services/SMSService'; // 可以替换 EmailService

const messageService = new EmailService();
// const messageService = new SMSService(); // 如果想使用 SMS 服务，可以替换上面的实例化

provide('messageService', messageService);
</script>
```

## 4. 创建消息服务的组件
```html
<!-- components/NotificationComponent.vue -->
<template>
  <div>
    <button @click="sendNotification">Send Notification</button>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const messageService = inject('messageService');

const sendNotification = () => {
  messageService.send('user@example.com', 'Hello via Vue 3 and Dependency Inversion!');
};
</script>
```

## 场景2结论
解耦: 组件和服务实现之间是解耦的，组件只关心 MessageService 抽象接口，而不关心具体的实现。
灵活性: 可以轻松替换不同的服务实现而无需修改组件代码。
可测试性: 组件依赖于抽象接口，可以轻松地在测试中替换为模拟实现。

## 依赖倒置的核心思想是："高层模块不应该依赖于低层模块，而是应该依赖于抽象接口"。
