# 1、ES6常用语法

>1.1  ES5只有全局作用域和函数作用域
>
>​	ES6包括全局作用域、函数作用域和块级作用域
>
>​	let username = 'ward';
>
>​	const定义常量
>
>1.2  模板字符串
>
>​	1.2.1 反引号  ``
>
>​	1.2.2 存储变量	${}
>
>1.3  函数的扩展
>
>​	1.3.1 ES6中有默认值参数
>
>​	1.3.2 箭头函数
>
>​			this指向函数（函数定义时就指向了）
>
>​			let obj = {};
>
>​			function foo(){}
>
>​			var id =10;
>
>​			foo.call({id:20})  //改变定义的作用域
>
>​	1.3.3 普通函数定义时，指向普通函数
>
>1.4 类的扩展
>
>​	// 定义一个类
>
>​	function Foo(){
>
>​		this.id = 1;
>
>​	}
>
>​	// 给类一个方法
>
>​	Foo.prototype.showInfo = function(){}
>
>​	let foo = new Foo();
>
>​	foo.id = 1;
>
>```
>class Person{
>    constructor (username, age, mind){
>        this.username = username;
>        this.age = age;
>        this.mind = mind;
>    }
>    showInfo(){
>        console.log(this.username, this.age. this.mind)
>    }
>}
>let  xxx = new Person('xxx', 18, '000);
>xxx.showInfo()
>```
>
>- class关键字定义一个类
>  - 必须有constructor方法（构造方法，如果没有，constructor{}
>  - 必须使用new来实例化
>- 类的继承
>  - 必须在子类的constructor方法中写super方法
>
>```
>
>class Peiqi extends Person{
>    constructor (username, age){
>        super()
>        this.username = username;
>        this.age = age;
>    }    
>}
>
>let peiqi = new Peiqi('peiqi', 73)
>peiqi.shoeInfo()  # 继承了父类
>```
>
>1.5 模块化编程
>
>1.6 单体函数

# 2、Vue.js的指令系统

>{{  }}
>
>v-text
>
>v-html
>
>v-for
>
>v-if
>
>v-show
>
>v-bind
>
>v-on
>
>v-model
>
>自定义指令
>
>获取DOM元素  $refs.myref   <div ref="myRef”></div>
>
>计算属性
>
>侦听属性
>
>修饰符

# 3、组件系统

>3.1 全局组件
>
>3.2 局部组件
>
>3.3 父子组件之间的数据传递
>
>3.4 子父组件之间的数据传递
>
>3.5 插槽
>
>3.6 具名插槽
>
>3.7 混入

# 4、声明周期

>### vue生命周期之beforeCreate
>
>> ```html
>> 实例创建之前除标签外，所有的vue需要的数据，事件都不存在
>> ```
>
>### vue生命周期之created
>
>> ```html
>> 实例创建之后，data和事件已经被解析到，el还没有找到
>> ```
>
>### vue生命周期之beforeMount
>
>> ```html
>> 开始找标签，数据还没有被渲染，事件也没有被监听
>> ```
>
>### vue生命周期之mounted
>
>> ```html
>> 开始渲染数据，开始监听事件
>> ```
>
>### vue生命周期之beforeUpdata
>
>> ```html
>> 数据已经被修改在虚拟DOM，但没有被渲染到页面上
>> ```
>
>### vue生命周期之updata
>
>> ```html
>> 开始使用Diff算法，将虚拟DOM中的修改应用大页面上，此时真实DOM中数据被修改
>> ```
>
>### vue生命周期之beforeDestory
>
>> ```html
>> 所有的数据都存在
>> ```
>
>### vue生命周期之destory
>
>> ```html
>> 所有的数据都有(虚拟DOM中找)
>> ```
>
>### \<keep-alive>\</keep-alive>vue提供的用来缓存被消除的标签
>
>> ```html
>> 用activated和deactivated取代了beforeUpdate和destory的执行
>> ```

# 5、Vue路由系统

>5.1	实现原理
>
>5.2	安装使用
>
>5.3	路由命名
>
>5.4	路由参数的实现
>
>5.5	路由参数的实现原理
>
>5.6	子路由
>
>5.7	子路由的append
>
>5.8	路由的钩子函数
>
>5.9	去掉#号

