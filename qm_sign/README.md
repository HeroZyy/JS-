# 某Q音乐Sign参数解析与破解

## 声明
本文仅供学习交流，因使用本文内容而产生的任何风险及后果，作者不承担任何责任。

## 目标网站
```
aHR0cHM6Ly95LnFxLmNvbS9uL3J5cXEvc29uZ0RldGFpbC8wMDJxVTVhWTNRdTI0eQ==
```

## 破解过程

### 1. 定位加密点
查看sign参数的生成过程，使用xhr定位sign的生成位置，发现来源于`P(n.data)`，而P函数来源于S函数，随即抠取相关代码。

![定位加密点1](https://github.com/user-attachments/assets/9611c4d1-2b60-45ee-afd5-f7b9a1d86a14)

![定位加密点2](https://github.com/user-attachments/assets/e862af5b-ed31-48ad-a816-ad4f51708845)

![定位加密点3](https://github.com/user-attachments/assets/d184280b-289b-4cc4-9f0c-42e3dda53a8a)

### 2. 代码执行问题
代码在node.js运行时，会卡在以下位置：

![代码卡点](https://github.com/user-attachments/assets/16232ced-2b3c-40cb-9b5d-c4a8c5b1b367)

取消捕获异常后代码会死循环，这个问题通常与浏览器环境有关。

## 补环境方法

### 方法一：代理补环境
```javascript
window=global
function getEnvs(proxyObjs) {
    for (let i = 0; i < proxyObjs.length; i++) {
        const handler = `{
      get: function(target, property, receiver) {
        console.log("方法:", "get  ", "对象:", "${proxyObjs[i]}", "  属性:", property, "  属性类型：", typeof property, ", 属性值：", target[property], ", 属性值类型：", typeof target[property]);
        return target[property];
      },
      set: function(target, property, value, receiver) {
        console.log("方法:", "set  ", "对象:", "${proxyObjs[i]}", "  属性:", property, "  属性类型：", typeof property, ", 属性值：", value, ", 属性值类型：", typeof target[property]);
        return Reflect.set(...arguments);
      }
    }`;
        eval(`try {
            ${proxyObjs[i]};
            ${proxyObjs[i]} = new Proxy(${proxyObjs[i]}, ${handler});
        } catch (e) {
            ${proxyObjs[i]} = {};
            ${proxyObjs[i]} = new Proxy(${proxyObjs[i]}, ${handler});
        }`);
    }
}
 
window=global
proxyObjs = ['window', 'document', 'location', 'navigator', 'history', 'screen']
getEnvs(proxyObjs);
```

运行上述代码后，根据参数属性和属性值类型补充信息。例如：补充location的host信息和navigator的userAgent信息：

![补充环境信息](https://github.com/user-attachments/assets/b233f181-f7e4-436a-954a-e7a767633d30)

### 方法二：使用v_tools生成临时环境

将DOM相关选项都勾选：

![v_tools设置](https://github.com/user-attachments/assets/93f3e2f8-267a-4225-9e60-094e69b177ec)

最终生成环境信息后复制到代码中，这也是一种补充临时环境的有效方式。

## 结果

通过上述方法补充环境后，即可成功生成sign值，完成接口调用。

---

## 项目结构
```
qm_sign/
  ├── sign.js        # 主要加密逻辑
  └── README.md      # 项目说明文档
```

## 使用方法
1. 克隆本仓库
2. 运行sign.js
3. 获取生成的sign值

## 贡献者
欢迎提交PR或Issues完善本项目
