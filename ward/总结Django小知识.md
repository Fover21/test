#  0.ORM操作

>1、必会的13条
>
>- 返回对象列表的
>  - all
>  - filter
>  - exclude
>  - order_by
>  - reverse
>  - distinct
>- 特殊的对象列表
>  - values
>  - values_list
>- 返回对象的
>  - get
>  - first
>  - last
>- 返回布尔值
>  - exist
>- 返回数字的
>  - count
>
>2、单表中双下滑线
>
>- - __gt
>  - __lt
>  - __in
>  - __range
>- - __countains
>  - __icountains
>- - __startswith
>  - __istartswith
>  - __endswith
>  - __iendswith
>- - __isnull

# 1.一对多（外键的查询）

>正向查询
>
>​        按照对象查询，这个时候只要屡清楚外键设置在哪个方向，按照对象一步一步的
>
>往过屡就ok。
>
>
>
>反向查询
>
>​         反向查询的时候通过表名（小写的类名）_set，拿到这个管理对象，对这个对象
>
>可以进行表的一些操作。
>
>
>
>按字段查询
>
>​        按字段查询的时候就没有必要屡主外键在哪方面，只需要看着要查的结果和条件
>
>然后按双下划线方法去查找就ok



>例子：
>
>```python
># 查找书名是“1”的书的出版社出版的其他书籍的名字和价格
>ret = models.Publisher.objects.filter(book__title='1').first().books.exclude(title='1').values('title','price')
>print(ret)
>
>ret = models.Book.objects.filter(publisher=models.Publisher.objects.filter(book__title='1').first()).exclude(
>    title='1')
>print(ret)
>
>
># 查找书名是“1”的书的作者们的姓名以及出版的所有书籍名称和价钱
>
>ret = models.Author.objects.values('book__title', 'book__price').filter(book__title='1').distinct()
>
>for i in ret:
>    print(i)
>
># ret = models.Book.objects.get(title='1').author.values('name', 'book__title', 'book__price').distinct()
># print(ret)
>```
>
>查出的数据肯定是一条一条的，如果是一对多条，会一行一行的列出来吗，然后 再筛选



# 2.多对多

>正向查询
>
>​        Obj.多对多关系     -》   管理对象
>
>​            add()
>
>​            remove()
>
>​            clear()
>
>​             set()   []
>
>​            create()
>
>反向查询
>
>​        Obj.表名（小写的类名）_set     -》   管理对象

# 3.聚合     aggregate

>聚合是一个终止语句      aggregate

# 4.分组    annotate

>分组的目的是为了进行聚合操作
>
>分组的结果放到了字段中（values中）
>
>正着来
>
>​         如果不指定分组对象，那么按照表名（表名id）分
>
>反着来
>
>​        这样是指定values(字段)（也就是分组名称）来分组
>
>拿到的是对象

# 5.F查询

>from django.db.model import F
>
>两个字段之间的比较
>
>​        子段__gt=F("字段")
>
>​        字段=F(“字段”) * 2
>
>​        字段=Concat(F("title"), Value("("), Value("第一版"), Value(")"))

# 6.Q查询

>from django.db.model import Q
>
>两个条件之间的或关系
>
>​        Q(筛选条件) | Q(筛选条件)
>
>两者条件之间的与关系
>
>​        Q(筛选条件) & Q(筛选条件)
>
>条件取反
>
>​        ~Q(筛选条件)

# 7.事务

>把一些列的操作（步骤）当作一个事务
>
>​        全部的步骤都成功才成功
>
>经典例子：银行转账
>
>
>
>代码实现：
>
>import os
>
>if __name__ == '__main__':
>​    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "BMS.settings")
>​    import django
>​    django.setup()
>
>```python
>import datetime
>from app01 import models
>
>try:
>    from django.db import transaction  # 事务
>    with transaction.atomic():  # 里面是执行的所有步骤 
>        new_publisher = models.Publisher.objects.create(name="火星出版社")
>        models.Book.objects.create(title="橘子物语", publish_date=datetime.date.today(), publisher_id=10)  # 指定一个不存在的出版社id
>except Exception as e:
>    print(str(e))
>```





# Cookie

>
>
>- 是一个保存在浏览器本地的一组组键值对
>- 为什么用cookie？http协议是无状态的，每次请求都是无关联的，没有办法保存状态
>- Network中，请求头中（随着request传到服务器）
>- 禁用之后登陆不了，登陆是配合cookie做的
>- 特性
>  - 服务器让浏览器进行保存的cookie
>  - 浏览器有权利是否进行保存
>  - 再次访问服务器的时候会携带相应的cookie
>- Django使用
>  - 获取
>    - request.COOKIES
>  - 设置
>    - ret = HttpRequest("XXX")
>    - ret.set_cookie(key, max-age=5)
>  - 删除
>    - ret = HttpResponse("XX")
>    - ret.delete_cookie(key)
>- 用途
>  - 登录
>  - 投票
>  - 记录浏览习惯





# Session

>
>
>- 保存着服务器上的键值对
>- 为什么要用session？
>  - cookie保存在浏览器上，不安全、
>  - cookie的长度受限制   4096字节
>- Django中操作session
>  - 设置session
>    - request.session[key] = value
>    - request.session.setdefault(key, valuue)
>  - 获取session
>    - request.session[key]
>    - request.session.get(key)
>  - 删除session
>    - del request.session[key] 删除某一个
>    - request.session.delete() 删除该用户的所有数据，不删除cookie
>    - request.session.flush() 删除该用户的所有数据，删除cookie
>  - 设置超时时间
>    - request.session.set_expiry(value)
>  - 清除所有过期的session
>    - request.session.clear_expired()



# SESSION的配置

>
>
>- from django.conf import global_settings
>
>- SEESSION配置
>
>- 458行
>
>  - SESSION_COOKIE_NAME             名字
>
>  - SESSION_COOKIE_AGE                 过期时间
>
>  - SESSION_SAVE_EVERY_REQUEST = False  每次保存一次session
>
>  - SESSION_EXPIRE_AT_BROWSER_CLOSE = False    关闭浏览器就清除cookie
>
>  - SESSION_ENGINE = '' 设置引擎
>
>    - Django中默认支持session，其内部提供了5种类型的session供开发者使用
>
>      ```python
>      1. 数据库Session
>      SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）
>      
>      2. 缓存Session
>      SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
>      SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置
>      
>      3. 文件Session
>      SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
>      SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir() 
>      
>      4. 缓存+数据库
>      SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎
>      
>      5. 加密Cookie Session
>      SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎
>      
>      ```
>
>    - ```python
>      其他公用设置项：
>      SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
>      SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
>      SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
>      SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
>      SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
>      SESSION_COOKIE_AGE = 1209600                             # Session的cookie失效日期（2周）（默认）
>      SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
>      SESSION_SAVE_EVERY_REQUEST = False                       # 是否每次请求都保存Session，默认修改之后才保存（默认）
>      ```





# 中间件

>
>
>发请求：a标签，location，form表单，地址栏
>
>钩子：在某个位置预留了位置，没有挂钩子函数的话继续执行，如果写了钩子函数就执行完后继续执行。
>
>- 中间件所有的动作都会走，会影响全局，谨慎使用
>
>- settings中的MIDDLEWARE
>
>- Django中的中间件？是什么？
>
>  - 是一个python类用来在全局范围内处理请求和响应的一个钩子。
>
>- 自定义中间件
>
>  - 五个方法
>
>    - 参数，执行时间，执行顺序
>
>    - 1、process_request(self, request)
>
>      - 执行时间：在路由匹配之前，在视图函数执行之前,
>
>      - 参数：
>
>        - request：视图函数用到中用到的request
>
>      - 执行顺序：按照注册顺序执行
>
>      - 返回值：
>
>        - None：正常流程
>
>        - HttpResponse对象：当前中间件后面的中间件的process_request方法
>
>          process_Response方法和视图的Http_Response对象不执行，执行当前
>
>          中间件的process_response方法以及之前的process_reques方法
>
>    - 2、process_response(self, request, response)
>
>      - 执行时间：在视图函数执行之后
>
>      - 参数：
>
>        - request：视图函数用到中用到的request
>
>        - response：视图函数中返回的response对象，如果中间件返回response对象，就不走
>
>          视图函数了
>
>      - 执行顺序：按照注册顺序倒序执行
>
>      - 返回值：
>
>        - 不能为None，否则会报错
>        - 必须是response对象
>
>    - 3、process_view(self, request, view_func, view_args, view_kwargs)
>
>      - 执行时间：在视图函数之前，在process_reques方法之后，以及路由匹配之后
>      - 参数：
>        - request：视图函数用到中用到的request
>        - view_func：要执行的视图函数
>        - view_args：视图函数的位置参数
>        - view_kwargs：视图函数的关键字参数
>      - 返回值：
>        - None：正常流程
>        - HttpResponse对象：他之后的中间件process_view方法、视图都不执行，执行所有中
>        - 间件的process_response方法
>      - 执行属性：按照注册顺序执行
>
>    - 4、process_exception(self, request, exception)
>
>      - 执行时间：
>        - 触发条件：有异常才执行
>        - 在视图函数之后，在process_response之前
>      - 参数：
>        - request：视图函数用到中用到的request
>        - exception：错误信息对象
>      - 返回值：HttpResponse对象：
>        - None：正常执行
>        - HttpResponse对象：
>          - 注册顺序之前的所有中间件的process_exception方法不走了
>          - 执行所有中间件的的process_response
>      - 执行顺序：按照注册顺序倒序执行
>
>    - 5、process_template_response(self, request, response)
>
>      - 执行时间：
>        - 触发条件：返回的response对象要有一个render方法
>        - 在视图函数之后，在process_response方法之前
>      - 参数“
>        - request：视图函数用到中用到的request
>        - response：视图函数中返回的response对象
>      - 返回值：
>        - 不能为None，否则会报错
>        - 必须是response对象
>      - 执行顺序：按照注册顺序倒序执行（不会截断，能够重写）
>
>- CSRF中间件
>
>  - CSRF跨站请求伪造
>
>- 在form表单中加{%csrf_token%}干了两件事
>
>  - 隐藏一个input标签
>  - 生成一个cookie
>
>- 两个装饰器
>
>  - from django.views.decorators.csrf import csrf_exempt, csrf_protect
>    - csrf_exempt：给单个视图排除校验
>    -  csrf_protect：给单个视图必须校验
>
>- 源码分析
>
>  - process_request
>
>    - 从请求的cookie中获取csrftoken的值   ——》csrf_token   ——》
>
>      request.META['CSRF_COOKIE']
>
>  - preocess_view
>
>    - 如果视图函数加上了csrf_exempt的装饰器  不做校验
>
>    - 如果请求方式是'GET', 'HEAD', 'OPTIONS', 'TRACE' 也不做校验
>
>    - 其他的请求方式做校验
>
>      - request.META.get('CSRF_COOKIE') —— 》 csrf_token 
>
>
>
>        request_csrf_token = "" 
>
>        从request.POST中获取csrfmiddlewaretoken对应的值
>
>        request_csrf_token = request.POST.get('csrfmiddlewaretoken', '')
>
>         从请求头中获取X-csrftoken 的值 
>
>        request_csrf_token = request.META.get(settings.CSRF_HEADER_NAME, '')
>
>
>
>        request_csrf_token 和 csrf_token 做对比  
>
>        如果校验成功 正常走
>
>        如果校验不成功 拒绝 
>
>- 浏览器向服务器发送请求的形式
>
>  - 地址栏输入地址   回车  GET
>  - form表单   点击  submit
>  - a标签
>  - ajax

# json

>
>
>- json是一种数据结构，跨平台
>- python中的json数据结构转换
>  - 数据类型
>    - 字符串、数字、bool、列表、字典、None
>  - 序列化
>    - dumps（python数据类型）
>    - dump（python的数据类型， f）
>  - 反序列化
>    - loads
>    - load
>- js中json数据的转换
>  - 数据类型
>    - 字符串、数字、布尔、数组、对象、null
>  - 序列化
>    - stringify
>  - 反序列化
>    - parse
>- django中json
>  - JsonResponse
>    - JsonResponse({})
>    - JsonResponse([], safe=False)

# ajax

>- 是一个与服务器进行交互的技术。（js技术）
>
>- 特点：
>  - 异步
>  - 不刷新页面     数据量小
>
>- 浏览器向服务器发送请求
>
>  - 地址栏输入地址  回车
>  - form表单
>  - a标签
>  - ajax
>
>- ajax
>
>  - 参数
>
>    ```js
>    $('#b1').click(function () {
>    			$.ajax({
>    				url: '/calc/',             # url地址
>    				type: 'post',			   # 请求方式 
>    				data: {                    # 发送的数据  
>    					i1: $('[name="i1"]').val(),
>    					i2: $('[name="i2"]').val()
>    					hobby:JSON.stringify(['抽烟','喝酒','烫头'])   # 传个列表  进行序列化
>    					
>    				},
>    				success: function (res) {         # 回调函数   成功时调用  res 返回的内容 响应的响应体
>    					console.log(res);
>    					$('[name="i3"]').val(res)
>     
>    				}, error: function (res) {        # 回调函数  失败时调用
>    
>    
>                    console.log('这是错误的')
>                    console.log(res)
>                }
>    				
>    			})
>    		});
>    ```
>
>- ajax发post请求 通过CSRF验证的方法
>
>  - 页面中使用{% csrf_token %}
>
>    ```js
>    $('#b1').click(function () {
>    				$.ajax({
>    					url: '/csrf_test/',
>    					type: 'post',
>    					data: {
>    						csrfmiddlewaretoken: $('[name="csrfmiddlewaretoken"]').val(),  # **********
>    						name: 'ward',
>    						age: '18',
>    					},
>    					success: function (res) {
>    						console.log(res);
>    					}
>    				})
>    			});
>    ```
>
>  - 设置请求头
>
>  - ```js
>    $('#b1').click(function () {
>    				$.ajax({
>    					url: '/csrf_test/',
>    					type: 'post',
>    					headers: {"X-CSRFToken": $('[name="csrfmiddlewaretoken"]').val()},
>    					data: {
>    						name: 'ward',
>    						age: '18',
>    					},
>    					success: function (res) {
>    						console.log(res);
>    					}
>    				})
>    			});
>    ```
>
>  - 全局设置（自己写的js代码）
>
>    ```js
>    function getCookie(name) {
>        var cookieValue = null;
>        if (document.cookie && document.cookie !== '') {
>            var cookies = document.cookie.split(';');
>            for (var i = 0; i < cookies.length; i++) {
>                var cookie = jQuery.trim(cookies[i]);
>                // Does this cookie string begin with the name we want?
>                if (cookie.substring(0, name.length + 1) === (name + '=')) {
>                    cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
>                    break;
>                }
>            }
>        }
>        return cookieValue;
>    }
>    var csrftoken = getCookie('csrftoken');
>    
>    function csrfSafeMethod(method) {
>      // these HTTP methods do not require CSRF protection
>      return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
>    }
>    
>    $.ajaxSetup({
>      beforeSend: function (xhr, settings) {
>        if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
>          xhr.setRequestHeader("X-CSRFToken", csrftoken);
>        }
>      }
>    });
>    ```
>
>
>
>    ```html
>    # 从cookie中取值 必要有cookie
>    			有cookie的方式
>    				1. 使用{% csrf_token %}
>    				2. 不使用{% csrf_token %}
>    					from django.views.decorators.csrf import ensure_csrf_cookie
>    					将ensure_csrf_cookie加在视图上  保证返回的响应有cookie
>    ```
>
>

# auth模块

>
>
>- 创建超级用户
>
>  - python3 manager.py createsuperuser
>
>- 认证  校验  用户的用户名与密码
>
>  ```python
>  obj = auth.authenticate(request, username=username, password=password)
>  ```
>
>  认证成功：对象
>
>  认真失败：None
>
>- 保存登录状态  记录到session
>
>  ```python
>  login(request, user)
>  ```
>
>- 注销  删除session
>
>  ```python
>  logout(request)
>  ```
>
>- 判断登录状态
>
>  ```python
>  requset.user.is_authenticated()
>  ```
>
>- 创建用户
>
>  ```pyt
>  from django.cotrib.auth.models import User
>  ```
>
>  - 密码是明文的
>
>    ```python
>    User.objects.create(username=username,password=password)
>    ```
>
>  - 密码是密文的普通用户
>
>    ```python
>    User.objects.create_user(**form_obj.cleaned_data)
>    ```
>
>  - 创建超级用户
>
>    ```python
>    User.objects.create_superuser(email='',**form_obj.cleaned_data)
>    ```
>
>- 密码相关
>
>  - 校验密码
>
>    ```python
>    request.user.check_password('root1234')
>    ```
>
>  - 设置密码
>
>    ```python
>    request.user.set_password('admin1234')
>    request.user.save()
>    ```
>
>  - 扩展默认的auth_user表
>
>  这内置的认证系统这么好用，但是auth_user表字段都是固定的那几个，我在项目中没法拿来直接使用啊！
>
>  比如，我想要加一个存储用户手机号的字段，怎么办？
>
>  可能会想到新建另外一张表然后通过一对一和内置的auth_user表关联，这样虽然能满足要求但是有没有更好的实现方式呢？
>
>  答案是当然有了。
>
>  我们可以通过继承内置的 AbstractUser 类，来定义一个自己的Model类。
>
>  这样既能根据项目需求灵活的设计用户表，又能使用Django强大的认证系统了。
>
>  ```
>  from django.contrib.auth.models import AbstractUser
>  class UserInfo(AbstractUser):
>      """
>      用户信息表
>      """
>      nid = models.AutoField(primary_key=True)
>      phone = models.CharField(max_length=11, null=True, unique=True)
>      
>      def __str__(self):
>          return self.username
>  ```
>
>  注意：
>
>  按上面的方式扩展了内置的auth_user表之后，一定要在settings.py中告诉Django，我现在使用我新定义的UserInfo表来做用户认证。写法如下：
>
>  ```
>  # 引用Django自带的User表，继承使用时需要设置
>  AUTH_USER_MODEL = "app名.UserInfo"
>  ```
>
>  再次注意：
>
>  一旦我们指定了新的认证系统所使用的表，我们就需要重新在数据库中创建该表，而不能继续使用原来默认的auth_user表了。

# Form组件

>
>
>- form
>
>  - 完成的事情
>    - 有input标签，让用户可以填数据
>    - 校验form表单提交数据
>    - 提示错误信息
>
>- Django的form
>
>  - 定义
>
>    ```python
>    from django import forms
>    
>    # 定义form
>    class RegForm(forms.Form):
>        user = forms.CharField(label='用户名')
>        pwd = forms.CharField(label='密码')
>    ```
>
>  - 使用
>
>    - 视图中
>
>      ```python
>      def register2(request):
>          form_obj = RegForm()
>          return render(request, 'register2.html', {'form_obj': form_obj})
>      ```
>
>    - 模板中
>
>      ```html
>          <form action="" method="post">
>              {% csrf_token %}
>              {{ form_obj.as_p }}
>              <p>
>                  <input type="submit" value="注册">
>              </p>
>          </form>
>      ```
>
>      ```html
>          <form action="" method="post">
>              <p>
>                  {{ form_obj.user.lable }}
>                  {{ form_obj.user }}
>              </p>
>              <p>
>                  {{ form_obj.pwd.lable }}
>                  {{ form_obj.pwd }}
>              </p>
>              <p>
>                  <input type="submit" value="注册">
>              </p>
>      
>          </form>
>      ```
>
>      总结：
>
>      ​	form_obj.as_p   ——>   自动生成多个p标签    包含lable input框
>
>      ​	form_obj.user/form_obj.pwd  ——>  自动生成某个字段的input框
>
>      ​        form_obj.user.errors  ——>	某个字段的所有错误信息
>
>      ​	form_obj.user.errors.0 ——>     摸个字段的错误信息的第一个
>
>      ​	form_obj.use.id_for_lable  
>
>  - 字段和参数
>
>    - 参数
>
>      label='用户名',        # 标签的名字
>      min_length=6,		   # 校验的规则 最小长度 
>      initial='alexdsb',	   # 初始值
>      error_messages={	   # 自定义错误提示 	
>      ​		'min_length': '你的长度太短了，还不到6',
>      ​		'required': '不能为空'
>
>       }
>      widget=widgets.PasswordInput()   # 插件  指定字段的类型 
>
>  - 校验
>
>    - 每个字段有默认的校验方法
>
>      min_length=6
>
>      max_length=6
>
>      required=False
>
>      disabled   是否不可修改
>
>    - 自定义校验规则
>
>      validators = [ 校验器1，校验器2 ]
>          1. from django.core.validators import RegexValidator
>
>                RegexValidator(r'^1[3-9]\d{9}$', '手机号不正经')
>
>                   2. 自定义函数
>                        from django.core.exceptions import ValidationError
>
>                        def check_name(value):
>                        ​	if 'sb' in value:
>                           		raise ValidationError('不符合社会主义核心价值观')
>
>                        validators = [  check_name,  ].      # 校验函数
>
>    - 钩子函数
>
>      - 局部钩子
>
>        ```python
>        def clean_phone(self):
>        	value = self.cleaned_data.get('phone')
>        	if re.match(r'^1[3-9]\d{9}$',value):
>        		return value
>        	raise ValidationError('手机号不正经')
>        ```
>
>      - 全局钩子
>
>        ```python
>        def clean(self):
>            pwd = self.cleaned_data.get('pwd')
>            re_pwd = self.cleaned_data.get('re_pwd')
>        
>            if pwd == re_pwd:
>                return self.cleaned_data
>            self.add_error('re_pwd','两次密码不一致')
>            raise ValidationError('两次密码不一致')
>        ```
>







