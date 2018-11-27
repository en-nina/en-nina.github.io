---
title: 在vuejs 中使用axios不能获取属性data的解决方法
date: 2018-10-17
tags: vue
---

**在vuejs 中使用axios不能获取属性data的解决方法**

axios中的this.labels,未被定义

```javascript
data(){
        return {
            labels: null,
        }
    },
```



```javascript
fetchData() {
      axios.get('/static/api/form.json')
      .then(function (response) {
        this.labels = response.data
        console.log("axios=="+ JSON.stringify(response.data));
      })
      .catch(function (error) {
        console.log("axios=="+error);
      });
    },
```



```javascript
created () {
    this.fetchData()
   // this.getFormJson()
  },
```

 

谷歌报错:

axios==TypeError: Cannot set property 'labels' of undefined

解答

出错问题：
在`then` 这个里边的赋值方法`this.followed = response.data.followed`会出现报错，什么原因呢？在Google上查了下，原来是在 `then`的内部不能使用Vue的实例化的`this`, 因为在内部 `this` 没有被绑定。
可以看下 [Stackoverflow](http://stackoverflow.com/questions/40996344/axios-cant-set-data) 的解释：

> In option functions like `data` and `created`, vue binds `this` to the view-model instance for us, so we can use `this.followed`, but in the function inside `then`, `this` is not bound.

**So** you need to preserve the view-model like (`created` means the component's data structure is assembled, which is enough here, `mounted` will delay the operation more):

 

解决方法

```javascript
fetchData() {
      var self = this ;
      axios.get('/static/api/form.json')
      .then(function (response) {
        self.labels = response.data
        console.log("axios=="+ JSON.stringify(response.data));
      })
      .catch(function (error) {
        console.log("axios=="+error);
      });
    },
```

或者我们可以使用 `ES6` 的 [箭头函数](https://babeljs.io/learn-es2015/)`arrow function`，箭头方法可以和父方法共享变量 ：

```javascript
axios.get('/static/api/form.json')
      .then((response) =>{
        this.labels = response.data
        console.log("axios=="+ JSON.stringify(response.data));
      })
      .catch((error)=> {
        console.log("axios=="+error);
      });
```

 