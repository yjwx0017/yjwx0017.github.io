title: Vue 笔记
categories: 日志
tags: [Vue]
date: 2023-08-24 14:22:00
---
[尚硅谷Vue2.0+Vue3.0全套教程丨vuejs从入门到精通][1]

## 初识Vue

``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>初识Vue</title>
    <!-- 引入Vue -->
    <script type="text/javascript" src="../js/vue.js"></script>
  </head>
  <body>
    <!-- 准备好一个容器 -->
    <div id="demo">
      <h1>Hello，{{name.toUpperCase()}}，{{address}}</h1>
    </div>

    <script type="text/javascript" >
      Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

      //创建Vue实例
      new Vue({
        el:'#demo', //el用于指定当前Vue实例为哪个容器服务，值通常为css选择器字符串。
        data:{ //data中用于存储数据，数据供el所指定的容器去使用，值我们暂时先写成一个对象。
          name:'atguigu',
          address:'北京'
        }
      })

    </script>
  </body>
</html>
```

<!--more-->

## 模板语法

插值语法：

```
{{js表达式}}
```

指令语法：

```
v-bind:href='js表达式' // 简写为:href='xxx'
```

v-开头的都是Vue指令。

``` html
<a v-bind:href="school.url.toUpperCase()">点我去{{school.name}}学习1</a>
```

## 数据绑定

单向绑定(v-bind)：数据只能从data流向页面。

双向绑定(v-model)：数据不仅能从data流向页面，还可以从页面流向data。

双向绑定一般都应用在表单类元素上（如：input、select等）

v-model:value 可以简写为 v-model，因为v-model默认收集的就是value值。


``` html
<!-- 普通写法 -->
单向数据绑定：<input type="text" v-bind:value="name"><br/>
双向数据绑定：<input type="text" v-model:value="name"><br/>

<!-- 简写 -->
单向数据绑定：<input type="text" :value="name"><br/>
双向数据绑定：<input type="text" v-model="name"><br/>
```

## el与data的两种写法


``` js
//el的两种写法
//第一种写法
const v = new Vue({
  el:'#root',
  data:{
    name:'尚硅谷'
  }
})
//第二种写法
const v = new Vue({
  data:{
    name:'尚硅谷'
  }
})
v.$mount('#root')

//data的两种写法
//第一种写法，对象式
new Vue({
  el:'#root',
  data:{
    name:'尚硅谷'
  }
})
//第二种写法，函数式，学习到组件时，data必须使用函数式，否则会报错
new Vue({
  el:'#root',
  data(){
    return{
      name:'尚硅谷'
    }
  }
})
```
## 理解MVVM

MVVM模型

- M：模型(Model) ：data中的数据
- V：视图(View) ：模板代码
- VM：视图模型(ViewModel)：Vue实例

## 事件处理

使用v-on:xxx 或 @xxx 绑定事件，其中xxx是事件名

``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>事件的基本使用</title>
    <!-- 引入Vue -->
    <script type="text/javascript" src="../js/vue.js"></script>
  </head>
  <body>
    <!-- 准备好一个容器-->
    <div id="root">
      <h2>欢迎来到{{name}}学习</h2>
      <button v-on:click="showInfo1">点我提示信息</button>
      <button @click="showInfo1">点我提示信息1（不传参）</button>
      <button @click="showInfo2($event,66)">点我提示信息2（传参）</button>
    </div>
  </body>

  <script type="text/javascript">
    Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

    const vm = new Vue({
      el:'#root',
      data:{
        name:'尚硅谷',
      },
      methods:{
        showInfo1(event){
          // console.log(event.target.innerText)
          // console.log(this) //此处的this是vm
          alert('同学你好！')
        },
        showInfo2(event,number){
          console.log(event,number)
          // console.log(event.target.innerText)
          // console.log(this) //此处的this是vm
          alert('同学你好！！')
        }
      }
    })
  </script>
</html>
```

Vue中的事件修饰符：

1. prevent：阻止默认事件（常用）；
2. stop：阻止事件冒泡（常用）；
3. once：事件只触发一次（常用）；
4. capture：使用事件的捕获模式；
5. self：只有event.target是当前操作的元素时才触发事件；
6. passive：事件的默认行为立即执行，无需等待事件回调执行完毕；

键盘事件：

```
@keydown.enter="showInfo"
```

Vue中常用的按键别名：

- 回车 => enter
- 删除 => delete (捕获“删除”和“退格”键)
- 退出 => esc
- 空格 => space
- 换行 => tab (特殊，必须配合keydown去使用)
- 上 => up
- 下 => down
- 左 => left
- 右 => right

## 计算属性

要用的属性不存在，要通过已有属性计算得来。

优势：与 methods 实现相比，内部有缓存机制（复用），效率更高，调试方便。

``` js
const vm = new Vue({
  el:'#root',
  data:{
    firstName:'张',
    lastName:'三',
    x:'你好'
  },
  computed:{
    fullName:{
      //get有什么作用？当有人读取fullName时，get就会被调用，且返回值就作为fullName的值
      //get什么时候调用？1.初次读取fullName时。2.所依赖的数据发生变化时。
      get(){
        console.log('get被调用了')
        // console.log(this) //此处的this是vm
        return this.firstName + '-' + this.lastName
      },
      //set什么时候调用? 当fullName被修改时。
      set(value){
        console.log('set',value)
        const arr = value.split('-')
        this.firstName = arr[0]
        this.lastName = arr[1]
      }
      //简写
      /*
      fullName(){
        console.log('get被调用了')
        return this.firstName + '-' + this.lastName
      }
      */
    }
  }
})
```

## 监视属性

监视属性 watch

1. 当被监视的属性变化时, 回调函数自动调用, 进行相关操作
2. 监视的属性必须存在，才能进行监视
3. 监视的两种写法：(1).new Vue时传入watch配置 (2).通过vm.$watch监视

``` js
const vm = new Vue({
  el:'#root',
  data:{
    isHot:true,
  },
  // 第一种写法
  watch:{
    isHot:{
      immediate:true, //初始化时让handler调用一下
      //handler什么时候调用？当isHot发生改变时。
      handler(newValue,oldValue){
        console.log('isHot被修改了',newValue,oldValue)
      }
    }
    //简写
    /* isHot(newValue,oldValue){
      console.log('isHot被修改了',newValue,oldValue,this)
    } */
  }
})

// 第二种写法
vm.$watch('isHot',{
  immediate:true, //初始化时让handler调用一下
  //handler什么时候调用？当isHot发生改变时。
  handler(newValue,oldValue){
    console.log('isHot被修改了',newValue,oldValue)
  }
})
//简写
/* vm.$watch('isHot',(newValue,oldValue)=>{
  console.log('isHot被修改了',newValue,oldValue,this)
}) */
```


`deep:true` 启用深度监视，可监视对象深层值的改变。

## 绑定样式

``` html
<!-- 绑定class样式--字符串写法，适用于：样式的类名不确定，需要动态指定 -->
<div class="basic" :class="mood" @click="changeMood">{{name}}</div> <br/><br/>

<!-- 绑定class样式--数组写法，适用于：要绑定的样式个数不确定、名字也不确定 -->
<div class="basic" :class="classArr">{{name}}</div> <br/><br/>

<!-- 绑定class样式--对象写法，适用于：要绑定的样式个数确定、名字也确定，但要动态决定用不用 -->
<div class="basic" :class="classObj">{{name}}</div> <br/><br/>

<!-- 绑定style样式--对象写法 -->
<div class="basic" :style="styleObj">{{name}}</div> <br/><br/>

<!-- 绑定style样式--数组写法 -->
<div class="basic" :style="styleArr">{{name}}</div>
```

``` js
const vm = new Vue({
  el:'#root',
  data:{
    name:'尚硅谷',
    mood:'normal',
    classArr:['atguigu1','atguigu2','atguigu3'],
    classObj:{
      atguigu1:false,
      atguigu2:false,
    },
    styleObj:{
      fontSize: '40px',
      color:'red',
    },
    styleObj2:{
      backgroundColor:'orange'
    },
    styleArr:[
      {
        fontSize: '40px',
        color:'blue',
      },
      {
        backgroundColor:'gray'
      }
    ]
  },
  methods: {
    changeMood(){
      const arr = ['happy','sad','normal']
      const index = Math.floor(Math.random()*3)
      this.mood = arr[index]
    }
  },
})
```

## 条件渲染

v-if 适用于切换频率较低的场景，不展示的DOM元素直接被移除

v-show 适用于切换频率较高的场景，不展示的DOM元素未被移除，仅仅是使用样式隐藏掉

``` html
<!-- 使用v-show做条件渲染 -->
<h2 v-show="false">欢迎来到{{name}}</h2>
<h2 v-show="1 === 1">欢迎来到{{name}}</h2>

<!-- 使用v-if做条件渲染 -->
<h2 v-if="false">欢迎来到{{name}}</h2>
<h2 v-if="1 === 1">欢迎来到{{name}}</h2>

<!-- v-else和v-else-if -->
<div v-if="n === 1">Angular</div>
<div v-else-if="n === 2">React</div>
<div v-else-if="n === 3">Vue</div>
<div v-else>哈哈</div>

<!-- v-if与template的配合使用 -->
<template v-if="n === 1">
  <h2>你好</h2>
  <h2>尚硅谷</h2>
  <h2>北京</h2>
</template>
```

## 列表渲染

``` html
<!-- 遍历数组 -->
<h2>人员列表（遍历数组）</h2>
<ul>
  <li v-for="(p,index) of persons" :key="index">
    {{p.name}}-{{p.age}}
  </li>
</ul>

<!-- 遍历对象 -->
<h2>汽车信息（遍历对象）</h2>
<ul>
  <li v-for="(value,k) of car" :key="k">
    {{k}}-{{value}}
  </li>
</ul>

<!-- 遍历字符串 -->
<h2>测试遍历字符串（用得少）</h2>
<ul>
  <li v-for="(char,index) of str" :key="index">
    {{char}}-{{index}}
  </li>
</ul>

<!-- 遍历指定次数 -->
<h2>测试遍历指定次数（用得少）</h2>
<ul>
  <li v-for="(number,index) of 5" :key="index">
    {{index}}-{{number}}
  </li>
</ul>
```

``` js
new Vue({
  el:'#root',
  data:{
    persons:[
      {id:'001',name:'张三',age:18},
      {id:'002',name:'李四',age:19},
      {id:'003',name:'王五',age:20}
    ],
    car:{
      name:'奥迪A8',
      price:'70万',
      color:'黑色'
    },
    str:'hello'
  }
})
```

开发中如何选择key：

1. 最好使用每条数据的唯一标识作为key, 比如id、手机号、身份证号、学号等唯一值。
2. 如果不存在对数据的逆序添加、逆序删除等破坏顺序操作，仅用于渲染列表用于展示，使用index作为key是没有问题的。

更新时的一个问题：

``` js
const vm = new Vue({
  el:'#root',
  data:{
    persons:[
      {id:'001',name:'马冬梅',age:30,sex:'女'},
      {id:'002',name:'周冬雨',age:31,sex:'女'},
      {id:'003',name:'周杰伦',age:18,sex:'男'},
      {id:'004',name:'温兆伦',age:19,sex:'男'}
    ]
  },
  methods: {
    updateMei(){
      // this.persons[0].name = '马老师' //奏效
      // this.persons[0].age = 50 //奏效
      // this.persons[0].sex = '男' //奏效
      // this.persons[0] = {id:'001',name:'马老师',age:50,sex:'男'} //不奏效
      this.persons.splice(0,1,{id:'001',name:'马老师',age:50,sex:'男'})
    }
  }
})
```

动态添加响应式属性：

``` js
const vm = new Vue({
  el:'#root',
  data:{
    school:{
      name:'尚硅谷',
      address:'北京',
    },
    student:{
      name:'tom',
      age:{
        rAge:40,
        sAge:29,
      },
      friends:[
        {name:'jerry',age:35},
        {name:'tony',age:36}
      ]
    }
  },
  methods: {
    addSex(){
      // Vue.set(this.student,'sex','男')
      this.$set(this.student,'sex','男')
    }
  }
})
```

在Vue修改数组中的某个元素一定要用如下方法：

1. 使用这些API:push()、pop()、shift()、unshift()、splice()、sort()、reverse()
2. Vue.set() 或 vm.$set()

特别注意：Vue.set() 和 vm.$set() 不能给vm 或 vm的根数据对象 添加属性！

## 收集表单数据

``` html
<div id="root">
  <form @submit.prevent="demo">
    账号：<input type="text" v-model.trim="userInfo.account"> <br/><br/>
    密码：<input type="password" v-model="userInfo.password"> <br/><br/>
    年龄：<input type="number" v-model.number="userInfo.age"> <br/><br/>
    性别：
    男<input type="radio" name="sex" v-model="userInfo.sex" value="male">
    女<input type="radio" name="sex" v-model="userInfo.sex" value="female"> <br/><br/>
    爱好：
    学习<input type="checkbox" v-model="userInfo.hobby" value="study">
    打游戏<input type="checkbox" v-model="userInfo.hobby" value="game">
    吃饭<input type="checkbox" v-model="userInfo.hobby" value="eat">
    <br/><br/>
    所属校区
    <select v-model="userInfo.city">
      <option value="">请选择校区</option>
      <option value="beijing">北京</option>
      <option value="shanghai">上海</option>
      <option value="shenzhen">深圳</option>
      <option value="wuhan">武汉</option>
    </select>
    <br/><br/>
    其他信息：
    <textarea v-model.lazy="userInfo.other"></textarea> <br/><br/>
    <input type="checkbox" v-model="userInfo.agree">阅读并接受<a href="http://www.atguigu.com">《用户协议》</a>
    <button>提交</button>
  </form>
</div>
```

``` js
new Vue({
  el:'#root',
  data:{
    userInfo:{
      account:'',
      password:'',
      age:18,
      sex:'female',
      hobby:[],
      city:'beijing',
      other:'',
      agree:''
    }
  },
  methods: {
    demo(){
      console.log(JSON.stringify(this.userInfo))
    }
  }
})
```

## 过滤器

``` html
<div id="root">
  <h2>显示格式化后的时间</h2>
  <!-- 计算属性实现 -->
  <h3>现在是：{{fmtTime}}</h3>
  <!-- methods实现 -->
  <h3>现在是：{{getFmtTime()}}</h3>
  <!-- 过滤器实现 -->
  <h3>现在是：{{time | timeFormater}}</h3>
  <!-- 过滤器实现（传参） -->
  <h3>现在是：{{time | timeFormater('YYYY_MM_DD') | mySlice}}</h3>
  <h3 :x="msg | mySlice">尚硅谷</h3>
</div>
```

``` js
//全局过滤器
Vue.filter('mySlice',function(value){
  return value.slice(0,4)
})

new Vue({
  el:'#root',
  data:{
    time:1621561377603, //时间戳
    msg:'你好，尚硅谷'
  },
  computed: {
    fmtTime(){
      return dayjs(this.time).format('YYYY年MM月DD日 HH:mm:ss')
    }
  },
  methods: {
    getFmtTime(){
      return dayjs(this.time).format('YYYY年MM月DD日 HH:mm:ss')
    }
  },
  //局部过滤器
  filters:{
    timeFormater(value,str='YYYY年MM月DD日 HH:mm:ss'){
      // console.log('@',value)
      return dayjs(value).format(str)
    }
  }
})

new Vue({
  el:'#root2',
  data:{
    msg:'hello,atguigu!'
  }
})
```

## 内置指令

- v-bind	: 单向绑定解析表达式, 可简写为 :xxx
- v-model	: 双向数据绑定
- v-for  	: 遍历数组/对象/字符串
- v-on   	: 绑定事件监听, 可简写为@
- v-if 	 	: 条件渲染（动态控制节点是否存存在）
- v-else 	: 条件渲染（动态控制节点是否存存在）
- v-show 	: 条件渲染 (动态控制节点是否展示)
- v-text  : 向其所在的节点中渲染文本内容，`<div v-text="name"></div>`
- v-html  : 向指定节点中渲染包含html结构的内容
- v-cloak : Vue实例创建完毕并接管容器后，会删掉v-cloak属性
- v-once  : 所在节点在初次动态渲染后，就视为静态内容了，可以用于优化性能
- v-pre   : 跳过其所在节点的编译过程

严重注意：v-html有安全性问题！

在网站上动态渲染任意HTML是非常危险的，容易导致XSS攻击。

一定要在可信的内容上使用v-html，永不要用在用户提交的内容上！

使用css配合v-cloak可以解决网速慢时页面展示出{{xxx}}的问题：

``` css
<style>
  [v-cloak]{
    display:none;
  }
</style>
```

## 自定义指令

``` js
//定义全局指令
Vue.directive('fbind',{
  //指令与元素成功绑定时（一上来）
  bind(element,binding){
    element.value = binding.value
  },
  //指令所在元素被插入页面时
  inserted(element,binding){
    element.focus()
  },
  //指令所在的模板被重新解析时
  update(element,binding){
    element.value = binding.value
  }
})

new Vue({
  el:'#root',
  data:{
    name:'尚硅谷',
    n:1
  },
  directives:{
    //big函数何时会被调用？1.指令与元素成功绑定时（一上来）。2.指令所在的模板被重新解析时。
    //指令名如果是多个单词，要使用kebab-case命名方式，不要用camelCase命名
    'big-number'(element,binding){
      // console.log('big')
      element.innerText = binding.value * 10
    },
    big(element,binding){
      console.log('big',this) //注意此处的this是window
      // console.log('big')
      element.innerText = binding.value * 10
    },
    fbind:{
      //指令与元素成功绑定时（一上来）
      bind(element,binding){
        element.value = binding.value
      },
      //指令所在元素被插入页面时
      inserted(element,binding){
        element.focus()
      },
      //指令所在的模板被重新解析时
      update(element,binding){
        element.value = binding.value
      }
    }
  }
})
```

## 生命周期

- beforeCreate
- created
- beforeMount
- mounted: 发送ajax请求、启动定时器、绑定自定义事件、订阅消息等【初始化操作】。
- beforeUpdate
- updated
- beforeDestroy: 清除定时器、解绑自定义事件、取消订阅消息等【收尾工作】。
- destroyed

## 非单文件组件

``` js
//创建student组件
const student = Vue.extend({
  template:`
    <div>
      <h2>学生姓名：{{studentName}}</h2>
      <h2>学生年龄：{{age}}</h2>
    </div>
  `,
  data(){
    return {
      studentName:'张三',
      age:18
    }
  }
})

//全局注册组件
Vue.component('student',student)

new Vue({
  el:'#root',
  data:{
    msg:'你好啊！'
  },
  //局部注册组件
  components:{
    student
  }
})
```

使用自定义组件：

``` html
<student></student>
```

关于组件名：

一个单词组成：

- 第一种写法(首字母小写)：school
- 第二种写法(首字母大写)：School

多个单词组成：

- 第一种写法(kebab-case命名)：my-school
- 第二种写法(CamelCase命名)：MySchool (需要Vue脚手架支持)

一个简写方式：

``` js
const school = Vue.extend(options)
//可简写为：
const school = options
```

## 单文件组件

``` vue
<template>
  <div>
    <h2>学生姓名：{{name}}</h2>
    <h2>学生年龄：{{age}}</h2>
  </div>
</template>

<script>
  export default {
    name:'Student',
    data(){
      return {
        name:'张三',
        age:18
      }
    }
  }
</script>

<style>
</style>
```

## 日期处理三方库

moment.js

day.js 轻量

## npm 安装指定版本软件包

```
npm view webpack versions
npm i webpack@5
```

## Vue脚手架

安装脚手架：

```
npm install -g @vue/cli
```

创建项目：

```
vue create xxx
```

启动项目：

```
npm run serve
```

配置npm淘宝镜像：

```
npm config set registry https://registry.npm.taobao.org
```

输出脚手架配置：

```
vue inspect > output.js
```

vue.js与vue.runtime.xxx.js的区别：

1. vue.js是完整版的Vue，包含：核心功能+模板解析器。
2. vue.runtime.xxx.js是运行版的Vue，只包含：核心功能；没有模板解析器。

因为vue.runtime.xxx.js没有模板解析器，所以不能使用template配置项，需要使用render函数接收到的createElement函数去指定具体内容。

``` js
//引入Vue
import Vue from 'vue'
//引入App组件，它是所有组件的父组件
import App from './App.vue'
//关闭vue的生产提示
Vue.config.productionTip = false

new Vue({
  el:'#app',
  //render函数完成了这个功能：将App组件放入容器中
  render: h => h(App),
})
```

## ref属性

``` vue
<template>
  <div>
    <h1 v-text="msg" ref="title"></h1>
    <button ref="btn" @click="showDOM">点我输出上方的DOM元素</button>
    <School ref="sch"/>
  </div>
</template>

<script>
  //引入School组件
  import School from './components/School'

  export default {
    name:'App',
    components:{School},
    data() {
      return {
        msg:'欢迎学习Vue！'
      }
    },
    methods: {
      showDOM(){
        console.log(this.$refs.title) //真实DOM元素
        console.log(this.$refs.btn) //真实DOM元素
        console.log(this.$refs.sch) //School组件的实例对象（vc）
      }
    },
  }
</script>
```

## props配置

通过 props，父组件可以向子组件传递数据。

子组件：

``` vue
<template>
  <div>
    <h1>{{msg}}</h1>
    <h2>学生姓名：{{name}}</h2>
    <h2>学生性别：{{sex}}</h2>
    <h2>学生年龄：{{myAge+1}}</h2>
    <button @click="updateAge">尝试修改收到的年龄</button>
  </div>
</template>

<script>
  export default {
    name:'Student',
    data() {
      console.log(this)
      return {
        msg:'我是一个尚硅谷的学生',
        myAge:this.age
      }
    },
    methods: {
      updateAge(){
        this.myAge++
      }
    },
    // 方式1：简单声明接收
    // props:['name','age','sex']

    // 方式2：接收的同时对数据进行类型限制
    /* props:{
      name:String,
      age:Number,
      sex:String
    } */

    // 方式3：接收的同时对数据：进行类型限制+默认值的指定+必要性的限制
    props:{
      name:{
        type:String, //name的类型是字符串
        required:true, //name是必要的
      },
      age:{
        type:Number,
        default:99 //默认值
      },
      sex:{
        type:String,
        required:true
      }
    }
  }
</script>
```

父组件：

``` html
<div>
  <Student name="李四" sex="女" :age="18"/>
</div>
```

## mixin混入

编写需要混入的配置：

``` js
// mixin.js
export const hunhe = {
  methods: {
    showName(){
      alert(this.name)
    }
  },
  mounted() {
    console.log('你好啊！')
  },
}
export const hunhe2 = {
  data() {
    return {
      x:100,
      y:200
    }
  },
}
```

全局混入，所有组件都能使用：

``` js
//引入Vue
import Vue from 'vue'
//引入App
import App from './App.vue'
import {hunhe,hunhe2} from './mixin'
//关闭Vue的生产提示
Vue.config.productionTip = false

Vue.mixin(hunhe)
Vue.mixin(hunhe2)


//创建vm
new Vue({
  el:'#app',
  render: h => h(App)
})
```

局部混入：

``` js
import {hunhe,hunhe2} from '../mixin'

export default {
  name:'Student',
  data() {
    return {
      name:'张三',
      sex:'男'
    }
  },
  mixins:[hunhe,hunhe2]
}
```

## 插件

编写自定义插件：

``` js
export default {
  install(Vue,x,y,z){
    console.log(x,y,z)
    //全局过滤器
    Vue.filter('mySlice',function(value){
      return value.slice(0,4)
    })

    //定义全局指令
    Vue.directive('fbind',{
      //指令与元素成功绑定时（一上来）
      bind(element,binding){
        element.value = binding.value
      },
      //指令所在元素被插入页面时
      inserted(element,binding){
        element.focus()
      },
      //指令所在的模板被重新解析时
      update(element,binding){
        element.value = binding.value
      }
    })

    //定义混入
    Vue.mixin({
      data() {
        return {
          x:100,
          y:200
        }
      },
    })

    //给Vue原型上添加一个方法（vm和vc就都能用了）
    Vue.prototype.hello = ()=>{alert('你好啊')}
  }
}
```

应用插件：

``` js
//引入Vue
import Vue from 'vue'
//引入App
import App from './App.vue'
//引入插件
import plugins from './plugins'
//关闭Vue的生产提示
Vue.config.productionTip = false

//应用（使用）插件
Vue.use(plugins,1,2,3)
//创建vm
new Vue({
  el:'#app',
  render: h => h(App)
})
```

## scoped样式

避免组件之间同名class样式冲突：

``` vue
<style scoped>
  .title{
    color: red;
  }
</style>
```

## 浏览器本地存储

localStorage：

``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>localStorage</title>
  </head>
  <body>
    <h2>localStorage</h2>
    <button onclick="saveData()">点我保存一个数据</button>
    <button onclick="readData()">点我读取一个数据</button>
    <button onclick="deleteData()">点我删除一个数据</button>
    <button onclick="deleteAllData()">点我清空一个数据</button>

    <script type="text/javascript" >
      let p = {name:'张三',age:18}

      function saveData(){
        localStorage.setItem('msg','hello!!!')
        localStorage.setItem('msg2',666)
        localStorage.setItem('person',JSON.stringify(p))
      }
      function readData(){
        console.log(localStorage.getItem('msg'))
        console.log(localStorage.getItem('msg2'))

        const result = localStorage.getItem('person')
        console.log(JSON.parse(result))

        // console.log(localStorage.getItem('msg3'))
      }
      function deleteData(){
        localStorage.removeItem('msg2')
      }
      function deleteAllData(){
        localStorage.clear()
      }
    </script>
  </body>
</html>
```

sessionStorage，页面关闭后数据丢失：

``` js
let p = {name:'张三',age:18}

function saveData(){
  sessionStorage.setItem('msg','hello!!!')
  sessionStorage.setItem('msg2',666)
  sessionStorage.setItem('person',JSON.stringify(p))
}
function readData(){
  console.log(sessionStorage.getItem('msg'))
  console.log(sessionStorage.getItem('msg2'))

  const result = sessionStorage.getItem('person')
  console.log(JSON.parse(result))

  // console.log(sessionStorage.getItem('msg3'))
}
function deleteData(){
  sessionStorage.removeItem('msg2')
}
function deleteAllData(){
  sessionStorage.clear()
}
```

## 组件自定义事件

子组件向父组件传递数据。

### 通过props实现

父组件通过props向子组件传递一个函数，子组件通过调用该函数，利用函数参数向父组件传递数据。

### $emit

子组件触发事件：

``` vue
<script>
  export default {
    name:'Student',
    data() {
      return {
        name:'张三',
        sex:'男',
        number:0
      }
    },
    methods: {
      add(){
        console.log('add回调被调用了')
        this.number++
      },
      sendStudentlName(){
        //触发Student组件实例身上的atguigu事件
        this.$emit('atguigu',this.name,666,888,900)
        // this.$emit('demo')
        // this.$emit('click')
      },
      unbind(){
        this.$off('atguigu') //解绑一个自定义事件
        // this.$off(['atguigu','demo']) //解绑多个自定义事件
        // this.$off() //解绑所有的自定义事件
      },
      death(){
        this.$destroy() //销毁了当前Student组件的实例，销毁后所有Student实例的自定义事件全都不奏效。
      }
    },
  }
</script>
```

父组件绑定子组件的事件：

方式一：

``` vue
<template>
  <div class="app">
    <Student @atguigu="getStudentName" @demo="m1"/>
  </div>
</template>
```

方式二：

``` vue
<template>
  <div class="app">
    <Student ref="student" @click.native="show"/>
  </div>
</template>

<script>
  import Student from './components/Student'

  export default {
    name:'App',
    components:{Student},
    data() {
      return {
        msg:'你好啊！',
        studentName:''
      }
    },
    methods: {
      getStudentName(name,...params){
        console.log('App收到了学生名：',name,params)
        this.studentName = name
      },
      show(){
        alert(123)
      }
    },
    mounted() {
      this.$refs.student.$on('atguigu',this.getStudentName) //绑定自定义事件
      // this.$refs.student.$once('atguigu',this.getStudentName) //绑定自定义事件（一次性）
    },
  }
</script>
```

## 全局事件总线

用于组件与组件之间传递数据。

安装全局事件总线：

``` js
// main.js
//引入Vue
import Vue from 'vue'
//引入App
import App from './App.vue'
//关闭Vue的生产提示
Vue.config.productionTip = false

//创建vm
new Vue({
  el:'#app',
  render: h => h(App),
  beforeCreate() {
    Vue.prototype.$bus = this //安装全局事件总线
  },
})
```

发送数据的一方触发事件：

``` js
this.$bus.$emit('hello',this.name)
```

接收数据的一方绑定事件：

``` js
this.$bus.$on('hello',(data)=>{
  console.log('我是School组件，收到了数据',data)
})
```

解绑事件：

``` js
this.$bus.$off('hello')
```

## 消息订阅与发布

安装 pubsub-js ：

```
npm i pubsub-js
```

导入：

``` js
import pubsub from 'pubsub-js'
```

发布：

``` js
pubsub.publish('hello', 666)
```

订阅和取消订阅：

``` js
import pubsub from 'pubsub-js'
export default {
  name:'School',
  data() {
    return {
      name:'尚硅谷',
      address:'北京',
    }
  },
  mounted() {
    this.pubId = pubsub.subscribe('hello',(msgName,data)=>{
    })
  },
  beforeDestroy() {
    pubsub.unsubscribe(this.pubId)
  },
}
```

## nextTick

下一次DOM更新结束后调用。

``` js
this.$nextTick(function(){
  this.$refs.inputTitle.focus()
})
```

## 过渡与动画

``` vue
<template>
  <div>
    <button @click="isShow = !isShow">显示/隐藏</button>
    <transition name="hello" appear>
      <h1 v-show="isShow">你好啊！</h1>
    </transition>
  </div>
</template>

<script>
  export default {
    name:'Test',
    data() {
      return {
        isShow:true
      }
    },
  }
</script>

<style scoped>
  h1{
    background-color: orange;
  }

  .hello-enter-active{
    animation: atguigu 0.5s linear;
  }

  .hello-leave-active{
    animation: atguigu 0.5s linear reverse;
  }

  @keyframes atguigu {
    from{
      transform: translateX(-100%);
    }
    to{
      transform: translateX(0px);
    }
  }
</style>
```

transition中如果不使用name，则样式类名默认为v-enter-active。

方法二，使用过渡：

``` vue
<style scoped>
  h1{
    background-color: orange;
  }
  /* 进入的起点、离开的终点 */
  .hello-enter,.hello-leave-to{
    transform: translateX(-100%);
  }
  .hello-enter-active,.hello-leave-active{
    transition: 0.5s linear;
  }
  /* 进入的终点、离开的起点 */
  .hello-enter-to,.hello-leave{
    transform: translateX(0);
  }
</style>
```

包含多个元素时使用transition-group：

``` html
<transition-group name="hello" appear>
  <h1 v-show="!isShow" key="1">你好啊！</h1>
  <h1 v-show="isShow" key="2">尚硅谷！</h1>
</transition-group>
```

方法三，使用animate.css：

```
npm install animate.css
```

``` vue
<template>
  <div>
    <button @click="isShow = !isShow">显示/隐藏</button>
    <transition-group 
      appear
      name="animate__animated animate__bounce" 
      enter-active-class="animate__swing"
      leave-active-class="animate__backOutUp"
    >
      <h1 v-show="!isShow" key="1">你好啊！</h1>
      <h1 v-show="isShow" key="2">尚硅谷！</h1>
    </transition-group>
  </div>
</template>

<script>
  import 'animate.css'
  export default {
    name:'Test',
    data() {
      return {
        isShow:true
      }
    },
  }
</script>

<style scoped>
  h1{
    background-color: orange;
  }
</style>
```

## 配置代理服务器

修改vue.config.js：

``` js
module.exports = {
  pages: {
    index: {
      //入口
      entry: 'src/main.js',
    },
  },
  lintOnSave:false, //关闭语法检查
  //开启代理服务器（方式一）
  /* devServer: {
    proxy: 'http://localhost:5000'
  }, */
  //开启代理服务器（方式二）
  devServer: {
    proxy: {
      '/atguigu': {
        target: 'http://localhost:5000',
        pathRewrite:{'^/atguigu':''},
        // ws: true, //用于支持websocket
        // changeOrigin: true //用于控制请求头中的host值
      },
      '/demo': {
        target: 'http://localhost:5001',
        pathRewrite:{'^/demo':''},
        // ws: true, //用于支持websocket
        // changeOrigin: true //用于控制请求头中的host值
      }
    }
  }
}
```

  [1]: https://www.bilibili.com/video/BV1Zy4y1K7SH