#  1、更改系统auth表时的配置

>
>
>- settings中 AUTH_USER_MODEL = 'crm.UserProfile'  # app名.auth重改名 

# 2、配置全局的日期

>
>
>- settings中
>
>  ```python
>  USE_L10N = False
>  
>  DATETIME_FORMAT = 'Y-m-d H:i:s'
>  DATE_FORMAT = 'Y-m-d'
>  ```

# 3、如果models的字段有choice属性和自定义方法

>
>
>- 字段有choice属性
>
>我们拿他对应的数据可以用
>
>```python
>get_status_display # py文件中加括号，模板中不加
>```
>
>- 多对多的时候
>
>  多对多的时候可以在models中定义方法，模板中调用这个方法
>
>- 显示状态
>
>  from django.utils.safestring import mark_safe  # 字符串不进行转义
>  from django.utils.html import format_html  # 字符串不进行转义

# 4、向页面展示内容，如果不设置\__str__内置方法会返回对象

>
>
>设置\__str__方法就是为了我们返回的不是一个对象，而是一个具体输出的值，我们可以通过\__str__来实现

# 5、直接向页面发送html代码

>分页
>from django.utils.safestring import mark_safe  # 字符串不进行转义
>from django.utils.html import format_html  # 字符串不进行转义
>
>```python
>from django.utils.safestring import mark_safe
>```
>
>例子：
>
>```python
>    def show_status(self):
>        color_dict = {
>            'signed': "btn btn-primary",
>            'unregistered': "btn btn-success",
>            'studying': 'btn btn-info',
>            'paid_in_full': "btn btn-danger",
>        }
>        return mark_safe('<span style="color: black" class="{}">{}</span>'.
>                         format(color_dict[self.status], self.get_status_display()))
>```
>
>```python
>from django.utils.html import format_html
>```
>
>例子：
>
>```python
>color_dict = {
>            'signed': "green",
>            'unregistered': "grey",
>            'studying': 'red',
>            'paid_in_full': "#821e1e",
>        }
>        return format_html(
>            '<span style="background-color: {};color: white;padding: 3px">{}</span>'.format(color_dict[self.status],
>                                                                                            self.get_status_display()))
>```
>
>

# 6、模板的内容显示

内容显示
​			普通字段 {{ customer.qq }}
​			choices属性的  {{ customer.get_class_type_display }}
​			多对多   定义方法  返回字符串
​			显示状态 定义方法  返回HTML代码段 mark_safe ，format_html

​		from django.utils.safestring import mark_safe  # 字符串不进行转义
​		from django.utils.html import format_html  # 字符串不进行转义				

# 7、分页组件

```python
"""
分页器
"""

from django.utils.safestring import mark_safe


class Pagination:

    # request 为request请求， all_count为所有数据的个数， per_num为一页展示多少数据， max_show分多少页
    def __init__(self, request, all_count, per_num=10, max_show=11):
        # 基本的URL
        self.base_url = request.path_info
        # 当前页码
        try:
            self.current_page = int(request.GET.get('page', 1))
            if self.current_page <= 0:
                self.current_page = 1
        except Exception as e:
            self.current_page = 1
            print(e)
        # 最多显示的页码数
        self.max_show = max_show
        half_show = max_show // 2

        # 每页显示的数据条数
        self.per_num = per_num
        # 总数据量
        self.all_count = all_count

        # 总页码数
        self.total_num, more = divmod(all_count, per_num)
        if more:
            self.total_num += 1

        # 总页码数小于最大显示数：显示总页码数
        if self.total_num <= max_show:
            self.page_start = 1
            self.page_end = self.total_num
        else:
            # 总页码数大于最大显示数：最多显示11个
            if self.current_page <= half_show:
                self.page_start = 1
                self.page_end = max_show
            elif self.current_page + half_show >= self.total_num:
                self.page_end = self.total_num
                self.page_start = self.total_num - max_show + 1
            else:
                self.page_start = self.current_page - half_show
                self.page_end = self.current_page + half_show

    @property
    def start(self):
        return (self.current_page - 1) * self.per_num

    @property
    def end(self):
        return self.current_page * self.per_num

    @property
    def show_li(self):
        # 存放li标签的列表
        html_list = []

        first_li = '<li><a href="{}?page=1">首页</a></li>'.format(self.base_url)
        html_list.append(first_li)

        if self.current_page == 1:
            prev_li = '<li class="disabled"><a><<</a></li>'
        else:
            prev_li = '<li><a href="{1}?page={0}"><<</a></li>'.format(self.current_page - 1, self.base_url)
        html_list.append(prev_li)

        for num in range(self.page_start, self.page_end + 1):
            if self.current_page == num:
                li_html = '<li class="active"><a href="{1}?page={0}">{0}</a></li>'.format(num, self.base_url)
            else:
                li_html = '<li><a href="{1}?page={0}">{0}</a></li>'.format(num, self.base_url)
            html_list.append(li_html)

        if self.current_page == self.total_num:
            next_li = '<li class="disabled"><a>>></a></li>'
        else:
            next_li = '<li><a href="{1}?page={0}">>></a></li>'.format(self.current_page + 1, self.base_url)

        html_list.append(next_li)

        last_li = '<li><a href="{1}?page={0}">尾页</a></li>'.format(self.total_num, self.base_url)
        html_list.append(last_li)

        return mark_safe(''.join(html_list))

```

# 7.5、升级版

```python
"""
分页器
"""

from django.utils.safestring import mark_safe
from django.http import QueryDict


class Pagination:

    # request 为request请求， all_count为所有数据的个数， query_params为查询的时候将查询结果与页数进行拼接， per_num为一页展示多少数据， max_show分多少页
    def __init__(self, request, all_count, query_params=QueryDict(), per_num=10, max_show=11):
        # 基本的URL
        self.base_url = request.path_info
        # 查询条件
        self.query_params = query_params
        self.query_params._mutable = True
        # 当前页码
        try:
            self.current_page = int(request.GET.get('page', 1))
            if self.current_page <= 0:
                self.current_page = 1
        except Exception as e:
            self.current_page = 1
            print(e)
        # 最多显示的页码数
        self.max_show = max_show
        half_show = max_show // 2

        # 每页显示的数据条数
        self.per_num = per_num
        # 总数据量
        self.all_count = all_count

        # 总页码数
        self.total_num, more = divmod(all_count, per_num)
        if more:
            self.total_num += 1

        # 总页码数小于最大显示数：显示总页码数
        if self.total_num <= max_show:
            self.page_start = 1
            self.page_end = self.total_num
        else:
            # 总页码数大于最大显示数：最多显示11个
            if self.current_page <= half_show:
                self.page_start = 1
                self.page_end = max_show
            elif self.current_page + half_show >= self.total_num:
                self.page_end = self.total_num
                self.page_start = self.total_num - max_show + 1
            else:
                self.page_start = self.current_page - half_show
                self.page_end = self.current_page + half_show

    @property
    def start(self):
        return (self.current_page - 1) * self.per_num

    @property
    def end(self):
        return self.current_page * self.per_num

    @property
    def show_li(self):
        # 存放li标签的列表
        html_list = []

        self.query_params['page'] = 1
        # query=a&page=1

        first_li = '<li><a href="{}?{}">首页</a></li>'.format(self.base_url, self.query_params.urlencode())
        html_list.append(first_li)

        if self.current_page == 1:
            prev_li = '<li class="disabled"><a><<</a></li>'
        else:
            self.query_params['page'] = self.current_page - 1
            prev_li = '<li><a href="{0}?{1}"><<</a></li>'.format(self.base_url, self.query_params.urlencode())
        html_list.append(prev_li)

        for num in range(self.page_start, self.page_end + 1):
            self.query_params['page'] = num
            if self.current_page == num:
                li_html = '<li class="active"><a href="{0}?{1}">{2}</a></li>'.format(self.base_url,
                                                                                     self.query_params.urlencode(), num)
            else:
                li_html = '<li><a href="{0}?{1}">{2}</a></li>'.format(self.base_url,
                                                                       self.query_params.urlencode(), num)
            html_list.append(li_html)

        if self.current_page == self.total_num:
            next_li = '<li class="disabled"><a>>></a></li>'
        else:
            self.query_params['page'] = self.current_page + 1
            next_li = '<li><a href="{0}?{1}">>></a></li>'.format(self.base_url, self.query_params.urlencode())

        html_list.append(next_li)

        self.query_params['page'] = self.total_num
        last_li = '<li><a href="{}?{}">尾页</a></li>'.format(self.base_url, self.query_params.urlencode())
        html_list.append(last_li)

        return mark_safe(''.join(html_list))
```



# 8、static静态文件相关

>
>
>使用static静态文件相关 需要重新导入 {% load static %}

# 9、blank=True校验的时候可以不填

# 10、null=True数据库允许为空

# 11、编辑用户的时候可以用instance参数

>
>
>```python
>form_obj = CustomerForm(instance=obj)
>form_obj带着原有的数据，根据数据生成input的值
>
>form_obj = CustomerForm(request.POST,instance=obj)
>将提交的数据和要修改的实例交给form对象
>form_obj.save()  对要修改的实例进行修改
>```

# 12、公户变私户

>
>​	
>
>```python
>CBV
>	self.request
>	
>	反射
>	orm操作：		models.Customer.objects.filter(id__in=ids).update(consultant=self.request.user)
>self.request.user.customers.add(*models.Customer.objects.filter(id__in=ids))
>```

# 13、私户变公户

>
>
>```python
>orm操作：
>			models.Customer.objects.filter(id__in=ids).update(consultant=None)
>			self.request.user.customers.remove(*models.Customer.objects.filter(id__in=ids))
>```

# 14、模糊查询

>
>
>```python
># 方式一
>all_customer = models.Customer.objects.filter(Q(qq__contains=query) | Q(name__contains=query),consultant__isnull=True)
>
>all_customer = models.Customer.objects.filter(Q(('qq__contains',query)) | Q(('name__contains',query)), consultant__isnull=True)
>		
># 方式二
>q = self.get_search_content(['qq', 'name', 'birthday'])  # 列表中的值为数据库的字段
>
>def get_search_contion(self,query_list):
>
>    query = self.request.GET.get('query', '')
>    q = Q()
>    q.connector = 'OR'
>    for i in query_list:
>        q.children.append(Q(('{}__contains'.format(i), query)))
>
>    return q
>```

# 15、保留搜索条件

>
>
>```python
>from django.http import QueryDict
>print('query',request.GET)  #  <QueryDict: {'query': ['ward']}>
>print(request.GET.urlencode()) # query=ward
>request.GET.copy()  深拷贝
>```

# 16、add和set的区别

>
>
>add是在原来的基础上更改,set为覆盖原来的
>
>一对多的管理对象:add只能是管理对象
>
>多对多的管理对象:add可以是管理对象可以是id
>
>>
>>
>>当字段的属性null=True的时候一对多的管理对象才有remove和clear方法

# 17、添加和编辑后跳转回原页面

>
>
>```python
>qd =QueryDict()
>qd._mutable = True
>qd['next'] = request.get_full_path() 
>qd.urlencode()
>		
>```

# 18、展示客户的跟进记录、增加和编辑

>
>
>```python
>obj = models.ConsultRecord.objects.filter(id=edit_id).first() or models.ConsultRecord(consultant=request.user)
>form_obj = ConsultRecordForm(instance=obj)
>
># 限制客户是当前销售的私户
>self.fields['customer'].widget.choices = customer_choice
># 限制跟进人是当前的用户（销售）
>self.fields['consultant'].widget.choices = [(self.instance.consultant_id, self.instance.consultant), ]
>```

# 19、报名记录

>
>
>```python
># 限制当前的客户只能是传的id对应的客户
>self.fields['customer'].widget.choices = [(self.instance.customer_id, self.instance.customer), ]
># 限制当前可报名的班级是当前客户的意向班级
>self.fields['enrolment_class'].widget.choices = [(i.id, i) for i in self.instance.customer.class_list.all()]
>		
>    # 在客户列表中显示不同a标签
>    def enroll_link(self):
>        if not self.enrollment_set.exists():
>            return mark_safe('<a href="{}">添加报名表</a>'.format(reverse('add_enrollment',args=(self.id,))))
>        else:
>            return mark_safe('<a href="{}">添加</a> | <a href="{}">查看</a>'.format(reverse('add_enrollment',
>```
>
>

# 20

>
>
>1. CRM
>   客户管理系统
>
>2. 干什么用的？
>   管理客户以及与客户相关人员的对客户的管理
>
>3. 功能：
>
>   1. 登录 注册 注销  
>      auth 
>      ​	from django.contrib.auth.models import AbstractBaseUser，AbstractUser
>      ​	只想扩展  继承AbstractUser
>      ​	改写      继承AbstractBaseUser, PermissionsMixin
>      ​	
>
>      ```
>      settings AUTH_USER_MODEL = 'app名字.表名'
>      ```
>
>   2. 客户相关
>
>      - 客户列表
>      - 新增客户
>      - 编辑客户
>      - 模糊搜索
>        Q对象  Q(('条件',内容))
>      - 分页
>        QueryDict  request.GET  urlencode()   ——》 query=ward&page=1
>      - 批量操作
>        - 公户变私户
>        - 私户变公户
>          反射
>          ORM操作  
>      - 保留原路径
>
>   3. 跟进记录
>
>      - 展示跟进记录
>        - 展示销售所有的跟进记录
>        - 展示某个客户的所有跟进记录
>      - 新增跟进记录
>      - 编辑跟进记录
>        - 限制客户是当前销售的私户
>        - 跟进人是当前的用户
>          obj = models.ConsultRecord(consultant=request.user)   —— 在内存中
>          obj.save()
>
>   4. 报名记录
>
>      - 客户列表中展示报名相关的地址
>        - 没有报名记录  显示添加报名记录
>        - 有报名记录    显示添加、查看报名记录
>          - 客户model   enroll_link
>      - 展示报名记录
>        - 展示所有记录
>        - 展示某个客户的记录
>      - 新增报名记录
>        - 报名成功后修改客户状态
>      - 编辑报名记录
>
>   5. 缴费记录
>
>      - 展示缴费记录
>      - 添加、编辑缴费记录
>
>1. 今日内容
>
>   1. 公户变私户的问题解决
>
>      - 多个销售同时申请一个客户
>        谁先申请就是谁的
>        mysql数据中加行级锁
>        开始事务：begin；
>        加锁  for update ：  select * from student where id=1 for update;
>        结束事务：commit；
>
>   2. 班主任的功能
>
>      - 班级的管理
>
>        - 班级的展示
>        - 添加班级
>        - 编辑班级
>
>      - 课程的管理
>
>        - 
>
>      - 学习记录的管理
>
>        - 批量添加
>          student_list = []
>
>          for student in all_students:
>
>          ```
>          student_list.append(models.StudyRecord(course_record=course_obj, student=student))
>          ```
>
>          models.StudyRecord.objects.bulk_create(student_list)
>
>        - 展示编辑学习记录
>
>          FormSet = modelformset_factory(models.StudyRecord, StudyRecordForm, extra=0)
>          queryset = models.StudyRecord.objects.filter(course_record_id=course_id)
>          form_set = FormSet(queryset=queryset)
>
>          {% for form in form_set %}
>
>          ```
>          {{ form.instance.student.name }}    form.instance ——》 学习记录 
>          {{ form.attendance }}          ————》对应的input 
>          {{ form.score }}
>          {{ form.homework_note }}
>          ```
>
>        ```
>        提交编辑
>        {{ form_set.management_form }}
>        {{ form.id }}
>        {{ form.student }}
>        
>        form_set = FormSet(request.POST)
>        if form_set.is_valid():
>        	form_set.save()
>        ```
>
>        ​	

# 权限管理   RBAC

>
>
>1. 权限管理
>
>​	1. 为什么要有权限？
>
>​		
>
>​	2. 开发一套权限的组件。为什么要开发组件？
>
>​		
>
>​	3. 权限是什么？
>
>​		web 开发中  URL 约等于 权限
>
>​		
>
>​	4. 表结构的设计
>
>​		
>
>​		权限表 
>
>​			ID     URL  
>
>​			1      /customer/list/
>
>​			2      /customer/add/
>
>​			
>
>​			
>
>​		用户表
>
>​			ID    name    pwd  
>
>​			1     ward    123
>
>​			
>
>​			
>
>​		用户和权限的关系表（多对多）
>
>​			ID  user_id   permission_id 
>
>​			1   1		  1
>
>​			1   1		  2
>
>​			
>
>​	5. 写代码
>
>​		1. 查询出用户的权限写入session
>
>​		2. 读取权限信息，判断是否有权限

>
>
>>>### 最初版的权限管理梳理流程
>>>
>>>>表结构
>>>>
>>>>```python
>>>>from django.db import models
>>>>
>>>>
>>>>class Permission(models.Model):
>>>>    """
>>>>    权限表
>>>>    """
>>>>    title = models.CharField(max_length=32, verbose_name='标题')
>>>>    url = models.CharField(max_length=32, verbose_name='权限')
>>>>    
>>>>    class Meta:
>>>>        verbose_name_plural = '权限表'
>>>>        verbose_name = '权限表'
>>>>    
>>>>    def __str__(self):
>>>>        return self.title
>>>>
>>>>
>>>>class Role(models.Model):
>>>>    name = models.CharField(max_length=32, verbose_name='角色名称')
>>>>    permissions = models.ManyToManyField(to='Permission', verbose_name='角色所拥有的权限', blank=True)
>>>>    
>>>>    def __str__(self):
>>>>        return self.name
>>>>
>>>>
>>>>class User(models.Model):
>>>>    """
>>>>    用户表
>>>>    """
>>>>    name = models.CharField(max_length=32, verbose_name='用户名')
>>>>    password = models.CharField(max_length=32, verbose_name='密码')
>>>>    roles = models.ManyToManyField(to='Role', verbose_name='用户所拥有的角色', blank=True)
>>>>    
>>>>    def __str__(self):
>>>>        return self.name
>>>>
>>>>```
>>>>
>>>>- settings文件配置
>>>>
>>>>  - ```python
>>>>    #  ###### 权限相关的配置 ######
>>>>    PERMISSION_SESSION_KEY = 'permissions'
>>>>    WHITE_URL_LIST = [
>>>>        r'^/login/$',
>>>>        r'^/logout/$',
>>>>        r'^/reg/$',
>>>>        r'^/admin/.*',
>>>>    ]
>>>>    ```
>>>>
>>>>- 其实权限就是用户能够访问那些url，不能访问那些url，我们所做的就是将每个不同身份的人
>>>>
>>>>  分配不同的url
>>>>
>>>>- 在最初用户登录的时候就查询出用户的权限。并将此次权限存入到session中
>>>>
>>>>  - 为什么要存入session中啊，为了不重复读取数据库，存到session中
>>>>
>>>>    我们可以配置session然后将session存到缓存中（非关系型数据库中）
>>>>
>>>>    这样读取的速度回很快
>>>>
>>>>- 登录成功后如何查看当前用户的权限并将其写入到session中
>>>>
>>>>  - ```python
>>>>    from django.shortcuts import render, HttpResponse, redirect, reverse
>>>>    from rbac import models
>>>>    from django.conf import settings
>>>>    
>>>>    ...
>>>>    
>>>>    user = models.User.objects.filter(name=username, password=pwd).first()
>>>>    # 登录成功
>>>>            # 将权限信息写入到session
>>>>            
>>>>            # 1. 查当前登录用户拥有的权限
>>>>            permission_list = user.roles.filter(permissions__url__isnull=False).values_list(
>>>>                                                                                       'permissions__url').distinct()
>>>>            # for i in permission_list:
>>>>            #     print(i)
>>>>            
>>>>            # 2. 将权限信息写入到session  # 这里的键值我们做了全局配置
>>>>            request.session[settings.PERMISSION_SESSION_KEY] = list(permission_list)
>>>>            # 得到的permission_list是一个QuerySet的元组对象，因为session的存储是有数据类型限制所以转换为列表(列表中套元组)
>>>>    ```
>>>>
>>>>- 然后，该用户能够访问那些，不能访问那些，这时，我们可以将这个逻辑写在中间件这里
>>>>
>>>>  - ```python
>>>>    from django.utils.deprecation import MiddlewareMixin
>>>>    from django.conf import settings
>>>>    from django.shortcuts import HttpResponse
>>>>    import re
>>>>    
>>>>    
>>>>    class PermissionMiddleware(MiddlewareMixin):
>>>>        # 每一个请求来，都会走这个钩子函数
>>>>        def process_request(self, request):
>>>>            # 对权限进行校验
>>>>            # 1. 当前访问的URL
>>>>            current_url = request.path_info
>>>>    
>>>>            # 白名单的判断我们这里将白名单设置在了settings中，往settings中加就ok
>>>>            for i in settings.WHITE_URL_LIST:
>>>>                if re.match(i, current_url):
>>>>                    return
>>>>    
>>>>            # 2. 获取当前用户的所有权限信息
>>>>            permission_list = request.session.get(settings.PERMISSION_SESSION_KEY)
>>>>            # 3. 权限的校验
>>>>            print(current_url)  # Django的session做了转换将元组转换成为一个列表
>>>>            for item in permission_list:
>>>>                url = item[0]
>>>>                if re.match("^{}$".format(url), current_url):
>>>>                    return
>>>>            else:
>>>>                return HttpResponse('没有权限')
>>>>    ```
>>
>>### 升级版
>>
>>>
>>>
>>>>
>>
>># 动态生成一级菜单
>>
>>>
>>>
>>>表结构的设计
>>>
>>>>```python
>>>>from django.db import models
>>>>
>>>>
>>>>class Permission(models.Model):
>>>>    """
>>>>    权限表
>>>>    """
>>>>    title = models.CharField(max_length=32, verbose_name='标题')
>>>>    url = models.CharField(max_length=32, verbose_name='权限')
>>>>	# 用来判断哪些url是菜单，哪些不是菜单
>>>>    is_menu = models.BooleanField(default=False, verbose_name='是否是菜单')
>>>>    # 记录该菜单对应的图标信息(这里是属性样式类)
>>>>    icon = models.CharField(max_length=32, verbose_name='图标', null=True, blank=True)
>>>>
>>>>    class Meta:
>>>>        verbose_name_plural = '权限表'
>>>>        verbose_name = '权限表'
>>>>    
>>>>    def __str__(self):
>>>>        return self.title
>>>>
>>>>
>>>>class Role(models.Model):
>>>>    name = models.CharField(max_length=32, verbose_name='角色名称')
>>>>    permissions = models.ManyToManyField(to='Permission', verbose_name='角色所拥有的权限', blank=True)
>>>>    
>>>>    def __str__(self):
>>>>        return self.name
>>>>
>>>>
>>>>class User(models.Model):
>>>>    """
>>>>    用户表
>>>>    """
>>>>    name = models.CharField(max_length=32, verbose_name='用户名')
>>>>    password = models.CharField(max_length=32, verbose_name='密码')
>>>>    roles = models.ManyToManyField(to='Role', verbose_name='用户所拥有的角色', blank=True)
>>>>    
>>>>    def __str__(self):
>>>>        return self.name
>>>>
>>>>```
>>>>
>>>>
>>>
>>>注册层成功之后：
>>>
>>>```python
>>>user = models.User.objects.filter(name=username, password=pwd).first()
>>>```
>>>
>>>```python
>>># 将权限信息写入到session中
>>>init_permission(request, user)
>>>```
>>>
>>>```python
>>>def init_permission(request, user):
>>>    # 1. 查当前登录用户拥有的权限
>>>    permission_query = user.roles.filter(permissions__url__isnull=False).values(
>>>        'permissions__url',
>>>        'permissions__is_menu',
>>>        'permissions__icon',
>>>        'permissions__title'
>>>    ).distinct()
>>>    print('permission_query', permission_query)
>>>    # 存放权限信息
>>>    permission_list = []
>>>    # 存放菜单信息
>>>    menu_list = []
>>>    for item in permission_query:
>>>        permission_list.append({'url': item['permissions__url']})
>>>        if item.get('permissions__is_menu'):  # 如若菜单这个字段为True
>>>            # 将这个菜单的信息先存入一个字典，然后存入session
>>>            menu_list.append({
>>>                'url': item['permissions__url'],  # 权限信息
>>>                'icon': item['permissions__icon'],  # 图标(Bootstrap的类样式)
>>>                'title': item['permissions__title'],  # 标题
>>>            })
>>>
>>>    # 2. 将权限信息写入到session
>>>    request.session[settings.PERMISSION_SESSION_KEY] = permission_list
>>>    # 将菜单的信息写入到session中
>>>    request.session[settings.MENU_SESSION_KEY] = menu_list
>>>```
>>>
>>>#### 母版中的菜单（一级菜单）
>>>
>>>>在母版中合适的位置导入这个include_tag
>>>>
>>>>```html
>>>>{% load rbac %}
>>>>{% menu request %}
>>>>```
>>>>
>>>>在templatetags下的rbac.py文件中写(自定义过滤器)
>>>>
>>>>```python
>>>>import re
>>>>from django import template
>>>>from django.conf import settings
>>>>
>>>>register = template.Library()
>>>>
>>>>
>>>>@register.inclusion_tag('rbac/menu.html')
>>>>def menu(request):
>>>>    menu_list = request.session.get(settings.MENU_SESSION_KEY)
>>>>    for item in menu_list:
>>>>        url = item.get('url')
>>>>        if re.match('^{}$'.format(url), request.path_info):
>>>>            item['class'] = 'active'
>>>>    return {"menu_list": menu_list}
>>>>```
>>>>
>>>>在templates下的rbac文件夹下创建enum.html
>>>>
>>>>```html
>>>><div class="static-menu">
>>>>
>>>>    {% for item in menu_list %}
>>>>        <a href="{{ item.url }}" class="{{ item.class }}">
>>>>            <span class="icon-wrap"><i class="fa {{ item.icon }}"></i></span>{{ item.title }}</a>
>>>>    {% endfor %}
>>>>
>>>></div>
>>>><--这个代码的样式可以放到该app文件夹下的static下的css中建立一个menu.css-->
>>>>```
>>>>
>>>>#### 因为将数据存入了session中，所以我们可以通过request.session.来获取数据
>>>>
>>>>```css
>>>>.left-menu .menu-body .static-menu {
>>>>
>>>>}
>>>>
>>>>.left-menu .menu-body .static-menu .icon-wrap {
>>>>    width: 20px;
>>>>    display: inline-block;
>>>>    text-align: center;
>>>>}
>>>>
>>>>.left-menu .menu-body .static-menu a {
>>>>    text-decoration: none;
>>>>    padding: 8px 15px;
>>>>    border-bottom: 1px solid #ccc;
>>>>    color: #333;
>>>>    display: block;
>>>>    background: #efefef;
>>>>    background: -webkit-gradient(linear, left bottom, left top, color-stop(0, #efefef), color-stop(1, #fafafa));
>>>>    background: -ms-linear-gradient(bottom, #efefef, #fafafa);
>>>>    background: -moz-linear-gradient(center bottom, #efefef 0%, #fafafa 100%);
>>>>    background: -o-linear-gradient(bottom, #efefef, #fafafa);
>>>>    filter: progid:dximagetransform.microsoft.gradient(startColorStr='#e3e3e3', EndColorStr='#ffffff');
>>>>    -ms-filter: "progid:DXImageTransform.Microsoft.gradient(startColorStr='#fafafa',EndColorStr='#efefef')";
>>>>    box-shadow: inset 0px 1px 1px white;
>>>>}
>>>>
>>>>.left-menu .menu-body .static-menu a:hover {
>>>>    color: #2F72AB;
>>>>    border-left: 2px solid #2F72AB;
>>>>}
>>>>
>>>>.left-menu .menu-body .static-menu a.active {
>>>>    color: #2F72AB;
>>>>    border-left: 2px solid #2F72AB;
>>>>}
>>>>```
>>>>
>>>>settings的配置
>>>>
>>>>```python
>>>>#  ###### 权限相关的配置 ######
>>>>PERMISSION_SESSION_KEY = 'permissions'
>>>>MENU_SESSION_KEY = 'menus'
>>>>WHITE_URL_LIST = [
>>>>    r'^/login/$',
>>>>    r'^/logout/$',
>>>>    r'^/reg/$',
>>>>    r'^/admin/.*',
>>>>]
>>>>```
>>>>
>>>>
>>>>
>>>>中间件的配置
>>>>
>>>>在middlewares目录（中间件目录中）创建rbac.py文件
>>>>
>>>>```python
>>>>from django.utils.deprecation import MiddlewareMixin
>>>>from django.conf import settings
>>>>from django.shortcuts import HttpResponse
>>>>import re
>>>>
>>>>
>>>>class PermissionMiddleware(MiddlewareMixin):
>>>>    def process_request(self, request):
>>>>        # 对权限进行校验
>>>>        # 1. 当前访问的URL
>>>>        current_url = request.path_info
>>>>
>>>>        # 白名单的判断(settings中配置好了)
>>>>        for i in settings.WHITE_URL_LIST:
>>>>            if re.match(i, current_url):
>>>>                return
>>>>
>>>>        # 2. 获取当前用户的所有权限信息
>>>>        permission_list = request.session.get(settings.PERMISSION_SESSION_KEY)
>>>>        
>>>>        # 3. 权限的校验
>>>>        for item in permission_list:
>>>>            url = item['url']
>>>>            if re.match("^{}$".format(url), current_url):
>>>>                return
>>>>        else:
>>>>            return HttpResponse('没有权限')
>>>>
>>>>```
>
># 应用rbac组件
>
>>
>>
>>1、拷贝rbac组件到新的项目中并注册APP
>>
>>2、配置权限的相关信息
>>
>>>```python
>>>#  ###### 权限相关的配置 ######
>>>PERMISSION_SESSION_KEY = 'permissions'
>>>MENU_SESSION_KEY = 'menus'
>>>WHITE_URL_LIST = [
>>>    r'^/login/$',
>>>    r'^/logout/$',
>>>    r'^/reg/$',
>>>    r'^/admin/.*',
>>>]
>>>```
>>
>>3、创建跟权限相关的表
>>
>>- 执行命令
>>  - python3 manage.py makemigrations
>>  - python3 manage.py migrate
>>
>>4、录入权限信息
>>
>>- 创建超级用户
>>- 录入所有权限信息
>>- 创建角色 给角色分权限
>>- 创建用户 给用户分角色
>>
>>5、在登录成功之后 写入权限和菜单的信息到session中
>>
>>6、配置上中间件，进行权限的校验
>>
>>7、使用动态菜单
>>
>>```html
>><!-导入静态文件-->
>><link rel="stylesheet" href="{% static 'css/menu.css' %}">
>>
>>使用inclusion_tag 
>><div class="left-menu">
>>    <div class="menu-body">
>>        {% load rbac %}
>>        {% menu request %}
>>    </div>
>></div>
>>```
>>
>>
>
>>>### 母版中的菜单（动态生成二级菜单）
>>>
>>>>
>>>>
>>>>信息管理
>>>>
>>>>​	客户列表
>>>>
>>>>财务管理
>>>>
>>>>​	缴费列表
>>>>
>>>>>>
>>>>>>
>>>>>>User name pwd
>>>>>>
>>>>>>Role name permissions(FK)  2user
>>>>>>
>>>>>>Permission title(二) url menu(FK) 2role
>>>>>>
>>>>>>Menu title(一)
>>>>
>>>>>### Models.py
>>>>>
>>>>>```python
>>>>>from django.db import models
>>>>>
>>>>>
>>>>>class Menu(models.Model):
>>>>>    """
>>>>>    一级菜单
>>>>>    """
>>>>>    title = models.CharField(max_length=32, unique=True)  # 一级菜单的名字
>>>>>    icon = models.CharField(max_length=32, verbose_name='图标', null=True, blank=True)
>>>>>
>>>>>    class Meta:
>>>>>        verbose_name_plural = '菜单表'
>>>>>        verbose_name = '菜单表'
>>>>>
>>>>>    def __str__(self):
>>>>>        return self.title
>>>>>
>>>>>
>>>>>class Permission(models.Model):
>>>>>    """
>>>>>    权限表
>>>>>    有关联Menu的二级菜单
>>>>>    没有关联Menu的不是二级菜单，是不可以做菜单的权限
>>>>>    """
>>>>>    title = models.CharField(max_length=32, verbose_name='标题')
>>>>>    url = models.CharField(max_length=32, verbose_name='权限')
>>>>>    menu = models.ForeignKey('Menu', null=True, blank=True)
>>>>>
>>>>>    class Meta:
>>>>>        verbose_name_plural = '权限表'
>>>>>        verbose_name = '权限表'
>>>>>
>>>>>    def __str__(self):
>>>>>        return self.title
>>>>>
>>>>>
>>>>>class Role(models.Model):
>>>>>    name = models.CharField(max_length=32, verbose_name='角色名称')
>>>>>    permissions = models.ManyToManyField(to='Permission', verbose_name='角色所拥有的权限', blank=True)
>>>>>
>>>>>    def __str__(self):
>>>>>        return self.name
>>>>>
>>>>>
>>>>>class User(models.Model):
>>>>>    """
>>>>>    用户表
>>>>>    """
>>>>>    name = models.CharField(max_length=32, verbose_name='用户名')
>>>>>    password = models.CharField(max_length=32, verbose_name='密码')
>>>>>    roles = models.ManyToManyField(to='Role', verbose_name='用户所拥有的角色', blank=True)
>>>>>
>>>>>    def __str__(self):
>>>>>        return self.name
>>>>>
>>>>>```
>>>>>
>>>>>### 登录
>>>>>
>>>>>>
>>>>>>
>>>>>>```python
>>>>>>from django.shortcuts import render, HttpResponse, redirect, reverse
>>>>>>from rbac import models
>>>>>>from django.conf import settings
>>>>>>import copy
>>>>>>from rbac.server.init_permission import init_permission
>>>>>>
>>>>>>
>>>>>>def login(request):
>>>>>>    if request.method == 'POST':
>>>>>>        username = request.POST.get('username')
>>>>>>        pwd = request.POST.get('pwd')
>>>>>>
>>>>>>        user = models.User.objects.filter(name=username, password=pwd).first()
>>>>>>
>>>>>>        if not user:
>>>>>>            err_msg = '用户名或密码错误'
>>>>>>            return render(request, 'login.html', {'err_msg': err_msg})
>>>>>>        # 登录成功
>>>>>>        # 将权限信息写入到session
>>>>>>        init_permission(request, user)
>>>>>>        return redirect(reverse('customer'))
>>>>>>    return render(request, 'login.html')
>>>>>>
>>>>>>```
>>>>>>
>>>>>>```python
>>>>>>def init_permission(request, user):
>>>>>>    # 1. 查当前登录用户拥有的权限
>>>>>>    permission_query = user.roles.filter(permissions__url__isnull=False).values(
>>>>>>        'permissions__url',
>>>>>>        'permissions__title',
>>>>>>        'permissions__menu_id',
>>>>>>        'permissions__menu__title',
>>>>>>        'permissions__menu__icon',
>>>>>>    ).distinct()
>>>>>>    print(permission_query)
>>>>>>    # 存放权限信息
>>>>>>    permission_list = []
>>>>>>    # 存放菜单信息
>>>>>>    menu_dict = {}
>>>>>>    for item in permission_query:
>>>>>>        permission_list.append({'url': item['permissions__url']})
>>>>>>        menu_id = item.get('permissions__menu_id')
>>>>>>        if not menu_id:
>>>>>>            continue
>>>>>>        if menu_id not in menu_dict:
>>>>>>            menu_dict[menu_id] = {
>>>>>>                'title': item['permissions__menu__title'],
>>>>>>                'icon': item['permissions__menu__icon'],
>>>>>>                'children': [
>>>>>>                    {
>>>>>>                        'title': item['permissions__title'],
>>>>>>                        'url': item['permissions__url']}
>>>>>>                ]
>>>>>>            }
>>>>>>        else:
>>>>>>            menu_dict[menu_id]['children'].append(
>>>>>>                {'title': item['permissions__title'], 'url': item['permissions__url']})
>>>>>>
>>>>>>    # 2. 将权限信息写入到session
>>>>>>    request.session[settings.PERMISSION_SESSION_KEY] = permission_list
>>>>>>    # 将菜单的信息写入到session中
>>>>>>    request.session[settings.MENU_SESSION_KEY] = menu_dict
>>>>>>
>>>>>>```
>>>>>>
>>>>>>将拿到的数据存入session
>>>>>>
>>>>>>写在一个自定义inclusion_tag
>>>>>>
>>>>>>母版
>>>>>>
>>>>>>```python
>>>>>>{% load rbac %}
>>>>>>{% menu request %}
>>>>>>```
>>>>>>
>>>>>>rbac.py
>>>>>>
>>>>>>```pytho
>>>>>>import re
>>>>>>from django import template
>>>>>>from django.conf import settings
>>>>>>
>>>>>>register = template.Library()
>>>>>>
>>>>>>
>>>>>>@register.inclusion_tag('rbac/menu.html')
>>>>>>def menu(request):
>>>>>>    menu_list = request.session.get(settings.MENU_SESSION_KEY)
>>>>>>    return {"menu_list": menu_list}
>>>>>>
>>>>>>```
>>>>>>
>>>>>>menu.html
>>>>>>
>>>>>>```html
>>>>>><div class="multi-menu">
>>>>>>    {% for item in menu_list.values %}
>>>>>>        <div class="item">
>>>>>>            <div class="title"><i class="fa {{ item.icon }}"></i> {{ item.title }}</div>
>>>>>>            <div class="body hide">
>>>>>>                {% for child in item.children %}
>>>>>>                    <a href="{{ child.url }}">{{ child.title }}</a>
>>>>>>                {% endfor %}
>>>>>>            </div>
>>>>>>        </div>
>>>>>>    {% endfor %}
>>>>>></div>
>>>>>>```
>>>>>>
>>>>>>menu.css0
>>>>>>
>>>>>>```css
>>>>>>.static-menu .icon-wrap {
>>>>>>    width: 20px;
>>>>>>    display: inline-block;
>>>>>>    text-align: center;
>>>>>>}
>>>>>>
>>>>>>.static-menu a {
>>>>>>    text-decoration: none;
>>>>>>    padding: 8px 15px;
>>>>>>    border-bottom: 1px solid #ccc;
>>>>>>    color: #333;
>>>>>>    display: block;
>>>>>>    background: #efefef;
>>>>>>    background: -webkit-gradient(linear, left bottom, left top, color-stop(0, #efefef), color-stop(1, #fafafa));
>>>>>>    background: -ms-linear-gradient(bottom, #efefef, #fafafa);
>>>>>>    background: -moz-linear-gradient(center bottom, #efefef 0%, #fafafa 100%);
>>>>>>    background: -o-linear-gradient(bottom, #efefef, #fafafa);
>>>>>>    filter: progid:dximagetransform.microsoft.gradient(startColorStr='#e3e3e3', EndColorStr='#ffffff');
>>>>>>    -ms-filter: "progid:DXImageTransform.Microsoft.gradient(startColorStr='#fafafa',EndColorStr='#efefef')";
>>>>>>    box-shadow: inset 0px 1px 1px white;
>>>>>>}
>>>>>>
>>>>>>.static-menu a:hover {
>>>>>>    color: #2F72AB;
>>>>>>    border-left: 2px solid #2F72AB;
>>>>>>}
>>>>>>
>>>>>>.static-menu a.active {
>>>>>>    color: #2F72AB;
>>>>>>    border-left: 2px solid #2F72AB;
>>>>>>}
>>>>>>
>>>>>>.multi-menu .item {
>>>>>>    background-color: white;
>>>>>>}
>>>>>>
>>>>>>.multi-menu .item > .title {
>>>>>>    padding: 10px 5px;
>>>>>>    border-bottom: 1px solid #dddddd;
>>>>>>    cursor: pointer;
>>>>>>    color: #333;
>>>>>>    display: block;
>>>>>>    background: #efefef;
>>>>>>    background: -webkit-gradient(linear, left bottom, left top, color-stop(0, #efefef), color-stop(1, #fafafa));
>>>>>>    background: -ms-linear-gradient(bottom, #efefef, #fafafa);
>>>>>>    background: -o-linear-gradient(bottom, #efefef, #fafafa);
>>>>>>    filter: progid:dximagetransform.microsoft.gradient(startColorStr='#e3e3e3', EndColorStr='#ffffff');
>>>>>>    -ms-filter: "progid:DXImageTransform.Microsoft.gradient(startColorStr='#fafafa',EndColorStr='#efefef')";
>>>>>>    box-shadow: inset 0 1px 1px white;
>>>>>>}
>>>>>>
>>>>>>.multi-menu .item > .body {
>>>>>>    border-bottom: 1px solid #dddddd;
>>>>>>}
>>>>>>
>>>>>>.multi-menu .item > .body a {
>>>>>>    display: block;
>>>>>>    padding: 5px 20px;
>>>>>>    text-decoration: none;
>>>>>>    border-left: 2px solid transparent;
>>>>>>    font-size: 13px;
>>>>>>
>>>>>>}
>>>>>>
>>>>>>.multi-menu .item > .body a:hover {
>>>>>>    border-left: 2px solid #2F72AB;
>>>>>>}
>>>>>>
>>>>>>.multi-menu .item > .body a.active {
>>>>>>    border-left: 2px solid #2F72AB;
>>>>>>}
>>>>>>
>>>>>>```
>>>>>>
>>>>>>
>>>>>>
>>>>>>menu.js
>>>>>>
>>>>>>```js
>>>>>>$('.item .title').click(function () {
>>>>>>    $(this).next().toggleClass('hide')
>>>>>>})
>>>>>>```
>>
>>### 三级菜单
>>
>>>
>>>
>>>此实现的是，点击菜单其他菜单关闭(js的操作)然后点击不是菜单的链接
>>>
>>>菜单的状态保持不变，还是展示当前状态
>>
>>>
>>>
>>>models.py
>>>
>>>```python
>>>from django.db import models
>>>
>>>
>>>class Menu(models.Model):
>>>    """
>>>    一级菜单
>>>    """
>>>    title = models.CharField(max_length=32, unique=True)  # 一级菜单的名字
>>>    icon = models.CharField(max_length=32, verbose_name='图标', null=True, blank=True)
>>>    weight = models.IntegerField(default=1)
>>>
>>>    class Meta:
>>>        verbose_name_plural = '菜单表'
>>>        verbose_name = '菜单表'
>>>
>>>    def __str__(self):
>>>        return self.title
>>>
>>>
>>>class Permission(models.Model):
>>>    """
>>>    权限表
>>>    有关联Menu的二级菜单
>>>    没有关联Menu的不是二级菜单，是不可以做菜单的权限
>>>    """
>>>    title = models.CharField(max_length=32, verbose_name='标题')
>>>    url = models.CharField(max_length=32, verbose_name='权限')
>>>    menu = models.ForeignKey('Menu', null=True, blank=True)
>>>    # 该权限关联的其他权限是否也是在当前url上展示
>>>    parent = models.ForeignKey(to='Permission', null=True, blank=True)
>>>
>>>    class Meta:
>>>        verbose_name_plural = '权限表'
>>>        verbose_name = '权限表'
>>>
>>>    def __str__(self):
>>>        return self.title
>>>
>>>
>>>class Role(models.Model):
>>>    name = models.CharField(max_length=32, verbose_name='角色名称')
>>>    permissions = models.ManyToManyField(to='Permission', verbose_name='角色所拥有的权限', blank=True)
>>>
>>>    def __str__(self):
>>>        return self.name
>>>
>>>
>>>class User(models.Model):
>>>    """
>>>    用户表
>>>    """
>>>    name = models.CharField(max_length=32, verbose_name='用户名')
>>>    password = models.CharField(max_length=32, verbose_name='密码')
>>>    roles = models.ManyToManyField(to='Role', verbose_name='用户所拥有的角色', blank=True)
>>>
>>>    def __str__(self):
>>>        return self.name
>>>
>>>```
>>>
>>>登录
>>>
>>>```python
>>>from django.shortcuts import render, HttpResponse, redirect, reverse
>>>from rbac import models
>>>from django.conf import settings
>>>import copy
>>>from rbac.server.init_permission import init_permission
>>>
>>>
>>>def login(request):
>>>    if request.method == 'POST':
>>>        username = request.POST.get('username')
>>>        pwd = request.POST.get('pwd')
>>>
>>>        user = models.User.objects.filter(name=username, password=pwd).first()
>>>
>>>        if not user:
>>>            err_msg = '用户名或密码错误'
>>>            return render(request, 'login.html', {'err_msg': err_msg})
>>>
>>>        # 登录成功
>>>        # 将权限信息写入到session
>>>        init_permission(request, user)
>>>
>>>        return redirect(reverse('customer'))
>>>
>>>    return render(request, 'login.html')
>>>```
>>>
>>>当中间件校验通过之后init_permission.py将权限写入到session
>>>
>>>```python
>>>from django.conf import settings
>>>
>>>def init_permission(request, user):
>>>    # 1. 查当前登录用户拥有的权限
>>>    permission_query = user.roles.filter(permissions__url__isnull=False).values(
>>>        'permissions__url',
>>>        'permissions__title',
>>>        'permissions__id',
>>>        'permissions__parent_id',
>>>        'permissions__menu_id',
>>>        'permissions__menu__title',
>>>        'permissions__menu__icon',
>>>        'permissions__menu__weight',  # 带单排序用的
>>>    ).distinct()
>>>    print(permission_query)
>>>    # 存放权限信息
>>>    permission_list = []
>>>    # 存放菜单信息
>>>    menu_dict = {}
>>>    for item in permission_query:
>>>        permission_list.append({'url': item['permissions__url'],
>>>                                'id': item['permissions__id'],
>>>                                'parent_id': item['permissions__parent_id'], })
>>>        menu_id = item.get('permissions__menu_id')
>>>        if not menu_id:
>>>            continue
>>>        if menu_id not in menu_dict:
>>>            menu_dict[menu_id] = {
>>>                'title': item['permissions__menu__title'],
>>>                'icon': item['permissions__menu__icon'],
>>>                'weight': item['permissions__menu__weight'],
>>>                'children': [
>>>                    {
>>>                        'title': item['permissions__title'],
>>>                        'url': item['permissions__url'],
>>>                        'id': item['permissions__id'],
>>>                        'parent_id': item['permissions__parent_id'],
>>>                    }
>>>                ]
>>>            }
>>>        else:
>>>            menu_dict[menu_id]['children'].append(
>>>                {
>>>                    'title': item['permissions__title'],
>>>                    'url': item['permissions__url'],
>>>                    'id': item['permissions__id'],
>>>                    'parent_id': item['permissions__parent_id'],
>>>                })
>>>
>>>    # 2. 将权限信息写入到session
>>>    request.session[settings.PERMISSION_SESSION_KEY] = permission_list
>>>    # 将菜单的信息写入到session中
>>>    request.session[settings.MENU_SESSION_KEY] = menu_dict
>>>
>>>```
>>>
>>>
>>>
>>>中间件
>>>
>>>```python
>>>from django.utils.deprecation import MiddlewareMixin
>>>from django.conf import settings
>>>from django.shortcuts import HttpResponse
>>>import re
>>>
>>>
>>>class PermissionMiddleware(MiddlewareMixin):
>>>    def process_request(self, request):
>>>        # 对权限进行校验
>>>        # 1. 当前访问的URL
>>>        current_url = request.path_info
>>>
>>>        # 白名单的判断
>>>        for i in settings.WHITE_URL_LIST:
>>>            if re.match(i, current_url):
>>>                return
>>>
>>>        # 2. 获取当前用户的所有权限信息
>>>        permission_list = request.session.get(settings.PERMISSION_SESSION_KEY)
>>>       
>>>    	# 3. 权限的校验
>>>        for item in permission_list:
>>>            url = item['url']
>>>            if re.match("^{}$".format(url), current_url):
>>>                parent_id = item['parent_id']
>>>                id = item['id']
>>>                if parent_id:
>>>                    # 表示当前权限是子权限，让父权限是展开
>>>                    request.current_menu_id = parent_id
>>>                else:
>>>                    # 表示当前权限是父权限，要展开的二级菜单
>>>                    request.current_menu_id = id
>>>                return
>>>        else:
>>>            return HttpResponse('没有权限')
>>>
>>>```
>>>
>>>templatetags的rbac.py的inclusion_tag
>>>
>>>```python
>>>import re
>>>from collections import OrderedDict
>>>from django import template
>>>from django.conf import settings
>>>
>>>register = template.Library()
>>>
>>>@register.inclusion_tag('rbac/menu.html')
>>>def menu(request):
>>>    menu_list = request.session.get(settings.MENU_SESSION_KEY)
>>>    order_dict = OrderedDict()  # 创建一个有序的字典，为输出的菜单属性做准备
>>>
>>>    for key in sorted(menu_list, key=lambda x: menu_list[x]['weight'], reverse=True):
>>>        order_dict[key] = menu_list[key]
>>>        item = order_dict[key]
>>>        item['class'] = 'hide'  # 一级菜单加类属性
>>>        for i in item['children']:
>>>            # 相当于事件的委托，如果是菜单就展示菜单，如果不是菜单，就委托给菜单展示页面
>>>            if i['id'] == request.current_menu_id: 
>>>                i['class'] = 'active'  #　二级菜单加类属性
>>>                item['class'] = ''　　# 如果二级菜单是展开的将隐藏属性去掉
>>>    return {"menu_list": order_dict}
>>>```
>>>
>>>menu.html
>>>
>>>```html
>>><div class="multi-menu">
>>>    <!-- 循环展示一级菜单 -->
>>>    {% for item in menu_list.values %}
>>>        <div class="item">
>>>            <div class="title"><i class="fa {{ item.icon }}"></i> {{ item.title }}</div>
>>>            <div class="body {{ item.class }}">
>>>                <!-- 循环展示二级菜单，设置该按钮为被选中 -->
>>>                {% for child in item.children %}
>>>                    <a href="{{ child.url }}" class="{{ child.class }}">{{ child.title }}</a>
>>>                {% endfor %}
>>>            </div>
>>>        </div>
>>>    {% endfor %}
>>></div>
>>>```
>
># 路径导航和权限粒度控制到按钮级别
>
>>
>>
>>表结构
>>
>>models.py
>>
>>```python
>>from django.db import models
>>
>>
>>class Menu(models.Model):
>>    """
>>    一级菜单
>>    """
>>    title = models.CharField(max_length=32, unique=True)  # 一级菜单的名字
>>    icon = models.CharField(max_length=32, verbose_name='图标', null=True, blank=True)
>>    # 用来记录菜单的展现先后顺序
>>    weight = models.IntegerField(default=1)
>>
>>    class Meta:
>>        verbose_name_plural = '菜单表'
>>        verbose_name = '菜单表'
>>
>>    def __str__(self):
>>        return self.title
>>
>>
>>class Permission(models.Model):
>>    """
>>    权限表
>>    有关联Menu的二级菜单
>>    没有关联Menu的不是二级菜单，是不可以做菜单的权限
>>    """
>>    title = models.CharField(max_length=32, verbose_name='标题')
>>    url = models.CharField(max_length=32, verbose_name='权限')
>>    menu = models.ForeignKey('Menu', null=True, blank=True)
>>    # 该权限关联的其他权限是否也是在当前url上展示
>>    parent = models.ForeignKey(to='Permission', null=True, blank=True)
>>	# 用来记录次url对应的名字 eg:  web:customer
>>    name = models.CharField(max_length=32, null=True, blank=True, unique=True)
>>
>>    class Meta:
>>        verbose_name_plural = '权限表'
>>        verbose_name = '权限表'
>>
>>    def __str__(self):
>>        return self.title
>>
>>
>>class Role(models.Model):
>>    name = models.CharField(max_length=32, verbose_name='角色名称')
>>    permissions = models.ManyToManyField(to='Permission', verbose_name='角色所拥有的权限', blank=True)
>>
>>    def __str__(self):
>>        return self.name
>>
>>
>>class User(models.Model):
>>    """
>>    用户表
>>    """
>>    name = models.CharField(max_length=32, verbose_name='用户名')
>>    password = models.CharField(max_length=32, verbose_name='密码')
>>    roles = models.ManyToManyField(to='Role', verbose_name='用户所拥有的角色', blank=True)
>>
>>    def __str__(self):
>>        return self.name
>>
>>```
>>
>>登录：
>>
>>```python
>>from django.shortcuts import render, HttpResponse, redirect, reverse
>>from rbac import models
>>from django.conf import settings
>>import copy
>>from rbac.server.init_permission import init_permission
>>
>>
>>def login(request):
>>    if request.method == 'POST':
>>        username = request.POST.get('username')
>>        pwd = request.POST.get('pwd')
>>
>>        user = models.User.objects.filter(name=username, password=pwd).first()
>>
>>        if not user:
>>            err_msg = '用户名或密码错误'
>>            return render(request, 'login.html', {'err_msg': err_msg})
>>
>>        # 登录成功
>>        # 将权限信息写入到session
>>        init_permission(request, user)
>>        return redirect(reverse('web:customer'))
>>
>>    return render(request, 'login.html')
>>
>>```
>>
>>登录成功后将权限信息写入到session中init_permission.py文件
>>
>>```python
>>from django.conf import settings
>>def init_permission(request, user):
>>    # 1. 查当前登录用户拥有的权限
>>    permission_query = user.roles.filter(permissions__url__isnull=False).values(
>>        'permissions__url',  # 当前用户的权限信息
>>        'permissions__title',  # 当前用户的权限信息的标题
>>        'permissions__id',  # 当前用户的ｉｄ
>>        'permissions__name',  # 当前用户权限对应的名字
>>        'permissions__parent_id',  # 权限的外键
>>        'permissions__parent__name',  # 该权限关联的权限的名字
>>        'permissions__menu_id',
>>        'permissions__menu__title',
>>        'permissions__menu__icon',
>>        'permissions__menu__weight',  # 带单排序用的
>>    ).distinct()
>>    print(permission_query)
>>    # 存放权限信息
>>    permission_dict = {}
>>    # 存放菜单信息
>>    menu_dict = {}
>>    for item in permission_query:  # 遍历该用户所有的权限
>>        # 存放权限信息，以字典的形式，键为权限的名字
>>        # 将每一个权限的信息已字典的信息存起来，这些信息包括
>>        # 权限、权限id、该权限的父权限的id、name、权限的标题
>>        permission_dict[item['permissions__name']] = ({
>>            'url': item['permissions__url'],
>>            'id': item['permissions__id'],
>>            'parent_id': item['permissions__parent_id'],
>>            'parent_name': item['permissions__parent__name'],
>>            'title': item['permissions__title'],
>>        })
>>        # 获取每一个权限的菜单id
>>        menu_id = item.get('permissions__menu_id')
>>        # 如果没有menu_id则没有菜单
>>        if not menu_id:
>>            continue
>>        # 如果有menu_id则有菜单
>>        if menu_id not in menu_dict: 
>>            # 如果这一个权限有菜单，那么将这个权限的菜单信息存下来
>>            # 菜单的标题、菜单的图标、菜单的权重、还有就是子菜单信息
>>            menu_dict[menu_id] = {
>>                'title': item['permissions__menu__title'],
>>                'icon': item['permissions__menu__icon'],
>>                'weight': item['permissions__menu__weight'],
>>                # 子菜单存，标题、访问权限、id、和他的父权限
>>                'children': [
>>                    {
>>                        'title': item['permissions__title'],
>>                        'url': item['permissions__url'],
>>                        'id': item['permissions__id'],
>>                        'parent_id': item['permissions__parent_id'],
>>                    }
>>                ]
>>            }
>>        else:
>>            # 如果键相同只需要将子菜单存起来就行
>>            menu_dict[menu_id]['children'].append(
>>                {
>>                    'title': item['permissions__title'],
>>                    'url': item['permissions__url'],
>>                    'id': item['permissions__id'],
>>                    'parent_id': item['permissions__parent_id'],
>>                })
>>
>>    # 2. 将权限信息写入到session
>>    request.session[settings.PERMISSION_SESSION_KEY] = permission_dict
>>    # 将菜单的信息写入到session中
>>    request.session[settings.MENU_SESSION_KEY] = menu_dict
>>
>>```
>>
>>
>>
>>```python
>>from django.utils.deprecation import MiddlewareMixin
>>from django.conf import settings
>>from django.shortcuts import HttpResponse
>>import re
>>
>>
>>class PermissionMiddleware(MiddlewareMixin):
>>    def process_request(self, request):
>>        # 对权限进行校验
>>        # 1. 当前访问的URL
>>        current_url = request.path_info
>>
>>        # 白名单的判断
>>        for i in settings.WHITE_URL_LIST:
>>            if re.match(i, current_url):
>>                return
>>
>>        # 2. 获取当前用户的所有权限信息
>>        permission_dict = request.session.get(settings.PERMISSION_SESSION_KEY)
>>		
>>        # 路劲导航
>>        request.breadcrumd_list = [
>>            {"title": '首页', 'url': '#'},
>>        ]
>>
>>        # 3. 权限的校验
>>        print(current_url)
>>		
>>        # 从session中拿出当前用户的权限信息，遍历每个权限，与当前访问的权限做比较
>>        # 在权限列表中的就可以访问，没有权限的就没有权限
>>        for item in permission_dict.values():
>>            url = item['url']  # 当前这个权限的url
>>            # 如果这个权限url与当前访问的url匹配上了
>>            if re.match("^{}$".format(url), current_url):
>>                # 拿到这个权限的父权限的id、本身的id、父权限的名字
>>                parent_id = item['parent_id']
>>                id = item['id']
>>                parent_name = item['parent_name']
>>                # 如果存在父权限
>>                if parent_id:
>>                    # 表示当前权限是子权限，让父权限是展开
>>                    request.current_menu_id = parent_id
>>                    # 添加面包屑导航
>>                    # 如果当前权限是自权限，就应该先将父权限的内容加进来
>>                    request.breadcrumd_list.extend([
>>                        {"title": permission_dict[parent_name]['title'],
>>                         'url': permission_dict[parent_name]['url']},
>>                        {"title": item['title'], 'url': item['url']},
>>                    ])
>>                else:
>>                    # 表示当前权限是父权限，要展开的二级菜单
>>                    request.current_menu_id = id
>>                    # 添加面包屑导航
>>                    # 如果当前权限是父权限直接将当前导航的内容存起来就行
>>                    request.breadcrumd_list.append({"title": item['title'], 'url': item['url']})
>>                return
>>        else:
>>            return HttpResponse('没有权限')
>>
>>```
>>
>>templatetags中的rbac.py
>>
>>```python
>>import re
>>from collections import OrderedDict
>>from django import template
>>from django.conf import settings
>>
>>register = template.Library()
>>
>>
>># 用来展示菜单
>>@register.inclusion_tag('rbac/menu.html')
>>def menu(request):
>>    menu_list = request.session.get(settings.MENU_SESSION_KEY)
>>    order_dict = OrderedDict()
>>
>>    for key in sorted(menu_list, key=lambda x: menu_list[x]['weight'], reverse=True):
>>        order_dict[key] = menu_list[key]
>>        item = order_dict[key]
>>        item['class'] = 'hide'
>>
>>        for i in item['children']:
>>
>>            if i['id'] == request.current_menu_id:
>>                i['class'] = 'active'
>>                item['class'] = ''
>>    return {"menu_list": order_dict}
>>
>>
>># 用来控制路径导航
>>@register.inclusion_tag('rbac/ssssss.html')
>>def breadcrumb(request):
>>    return {"breadcrumd_list": request.breadcrumd_list}
>>
>># 用来控制  权限力度控制到按钮级别
>>@register.filter
>>def has_permission(request, permission):
>>    if permission in request.session.get(settings.PERMISSION_SESSION_KEY):
>>        return True
>>
>>```
>>
>>menu.html
>>
>>```html
>><div class="multi-menu">
>>    {% for item in menu_list.values %}
>>        <div class="item">
>>            <div class="title"><i class="fa {{ item.icon }}"></i> {{ item.title }}</div>
>>            <div class="body {{ item.class }}">
>>                {% for child in item.children %}
>>                    <a href="{{ child.url }}" class="{{ child.class }}">{{ child.title }}</a>
>>                {% endfor %}
>>            </div>
>>        </div>
>>    {% endfor %}
>></div>
>>```
>>
>>ssssss.html
>>
>>```html
>><ol class="breadcrumb no-radius no-margin" style="border-bottom: 1px solid #ddd;">
>>
>>    <!-- 这个判断就是为了让最后一个导航不能有链接 -->
>>    {% for li in breadcrumd_list %}
>>        {% if forloop.last %}
>>        <li>{{ li.title }}</li>
>>        {% else %}
>>        <li><a href="{{ li.url }}">{{ li.title }}</a></li>
>>        {% endif %}
>>    {% endfor %}
>>
>>
>></ol>
>>```
>>
>>layout.html
>>
>>```html
>><!--用来生成菜单与inclution_tag配合-->
>>{% load rbac %}
>>{% menu request %}
>>
>><!--用来生成导航栏与inclution_tag配合-->
>>{% breadcrumb request %}
>>```
>>
>>customer_list.html用来展示
>>
>>通过权限的名字判断来按钮的展示（与路由分发的include的namespace和方向解析的name配合使用）
>>
>>```html
>>{% extends 'layout.html' %}
>>
>>{% block content %}
>>    {% load rbac %}
>>
>>    <div class="luffy-container">
>>        <div class="btn-group" style="margin: 5px 0">
>>            <!--判断当前权限是否拥有-->
>>            {% if request|has_permission:'web:customer_add' %}
>>                <a class="btn btn-default" href="{% url 'web:customer_add' %}">
>>                    <i class="fa fa-plus-square" aria-hidden="true"></i> 添加客户
>>                </a>
>>            {% endif %}
>>
>>        </div>
>>        <table class="table table-bordered table-hover">
>>            <thead>
>>            <tr>
>>                <th>ID</th>
>>                <th>客户姓名</th>
>>                <th>年龄</th>
>>                <th>邮箱</th>
>>                <th>公司</th>
>>                {% if request|has_permission:'web:customer_edit' or request|has_permission:'web:customer_del' %}
>>                    <th>选项</th>
>>                {% endif %}
>>            </tr>
>>            </thead>
>>            <tbody>
>>            {% for row in data_list %}
>>                <tr>
>>                    <td>{{ row.id }}</td>
>>                    <td>{{ row.name }}</td>
>>                    <td>{{ row.age }}</td>
>>                    <td>{{ row.email }}</td>
>>                    <td>{{ row.company }}</td>
>>                    {% if request|has_permission:'web:customer_edit' or request|has_permission:'web:customer_del' %}
>>                    <td>
>>                    {% if request|has_permission:'web:customer_edit' %}
>>                        <a style="color: #333333;" href="{% url 'web:customer_edit' row.id %}">
>>                            <i class="fa fa-edit" aria-hidden="true"></i></a>
>>                        {% endif %}
>>                    {% if request|has_permission:'web:customer_del' %}
>>                        <a style="color: #d9534f;" href="{% url 'web:customer_del' row.id  %}"><i class="fa fa-trash-o"></i></a>
>>                    {% endif %}
>>                    </td>
>>                    {% endif %}
>>                </tr>
>>            {% endfor %}
>>            </tbody>
>>        </table>
>>    </div>
>>{% endblock %}
>>```

# 权限管理的自动化配置

>
>
>```python
>from django.db import models
>
>
>class Menu(models.Model):
>    """
>    一级菜单
>    """
>    title = models.CharField(max_length=32, verbose_name='标题', unique=True)  # 一级菜单的名字
>    icon = models.CharField(max_length=32, verbose_name='图标', null=True, blank=True)
>    weight = models.IntegerField(verbose_name='权重', default=1)
>
>    class Meta:
>        verbose_name_plural = '菜单表'
>        verbose_name = '菜单表'
>
>    def __str__(self):
>        return self.title
>
>
>class Permission(models.Model):
>    """
>    权限表
>    有关联Menu的二级菜单
>    没有关联Menu的不是二级菜单，是不可以做菜单的权限
>    """
>    title = models.CharField(max_length=32, verbose_name='标题')
>    url = models.CharField(max_length=32, verbose_name='权限')
>    menu = models.ForeignKey('Menu', null=True, blank=True, verbose_name='是否是菜单')
>    # 该权限关联的其他权限是否也是在当前url上展示
>    parent = models.ForeignKey(to='Permission', null=True, blank=True)
>
>    name = models.CharField(max_length=32, null=True, blank=True, unique=True, verbose_name='权限的别名')
>
>    class Meta:
>        verbose_name_plural = '权限表'
>        verbose_name = '权限表'
>
>    def __str__(self):
>        return self.title
>
>
>class Role(models.Model):
>    name = models.CharField(max_length=32, verbose_name='角色名称')
>    permissions = models.ManyToManyField(to='Permission', verbose_name='角色所拥有的权限', blank=True)
>
>    def __str__(self):
>        return self.name
>
>
>class User(models.Model):
>    """
>    用户表
>    """
>    name = models.CharField(max_length=32, verbose_name='用户名')
>    password = models.CharField(max_length=32, verbose_name='密码')
>    roles = models.ManyToManyField(to='Role', verbose_name='用户所拥有的角色', blank=True)
>
>    def __str__(self):
>        return self.name
>
>```
>
>urls.py
>
>> ```python
>> from django.conf.urls import url
>> from rbac import views
>> 
>> urlpatterns = [
>>     # 角色
>>     url(r'^role/list/$', views.role_list, name='role_list'),
>>     url(r'^role/add/$', views.role, name='role_add'),
>>     url(r'^role/edit/(\d+)$', views.role, name='role_edit'),
>>     url(r'^role/del/(\d+)$', views.role_del, name='role_del'),
>>     # 菜单 和 权限展示在一个页面
>>     url(r'^menu/list/$', views.menu_list, name='menu_list'),
>>     # 菜单的增删改
>>     url(r'^menu/add/$', views.menu, name='menu_add'),
>>     url(r'^menu/edit/(\d+)$', views.menu, name='menu_edit'),
>>     url(r'^menu/del/(\d+)$', views.menu_del, name='menu_del'),
>>     # 权限的增删改
>>     url(r'^permission/add/$', views.permission, name='permission_add'),
>>     url(r'^permission/edit/(\d+)$', views.permission, name='permission_edit'),
>>     url(r'^permission/del/(\d+)$', views.permission_del, name='permission_del'),
>> ]
>> ```
>
>views.py
>
>> ```python
>> from django.shortcuts import render, reverse, redirect, HttpResponse
>> from rbac import models
>> from rbac import forms
>> from django.db.models import Q
>> 
>> 
>> # Create your views here.
>> 
>> 
>> # 角色列表
>> def role_list(request):
>>     all_roles = models.Role.objects.all()
>>     return render(request, 'rbac/role_list.html', {
>>         "all_roles": all_roles
>>     })
>> 
>> 
>> # 角色的添加和编辑
>> def role(request, edit_id=None):
>>     obj = models.Role.objects.filter(id=edit_id).first()
>>     title = '添加角色' if not edit_id else '编辑角色'
>>     form_obj = forms.RoleForm(instance=obj)
>>     if request.method == 'POST':
>>         form_obj = forms.RoleForm(request.POST, instance=obj)
>>         if form_obj.is_valid():
>>             form_obj.save()
>>             return redirect(reverse('rbac:role_list'))
>>     return render(request, 'rbac/form.html', {
>>         "form_obj": form_obj,
>>         "title": title
>>     })
>> 
>> 
>> # 角色的删除
>> def role_del(request, del_id):
>>     models.Role.objects.filter(id=del_id).delete()
>>     return redirect(reverse('rbac:role_list'))
>> 
>> 
>> # 菜单和权限的展示
>> # 点击每一个菜单出现对应的权限信息
>> def menu_list(request):
>>     all_menu = models.Menu.objects.all()
>>     # 拿到菜单对应的菜单id
>>     mid = request.GET.get('mid')
>>     # 如果拿到菜单id代表着有子权限
>>     if mid:
>>         # 从子权限出发 拿到 父权限对应的菜单id对应的权限  或者  菜单对应的权限（也就是二级菜单） 因为自己关联自己（从父亲和儿子两方面出发）
>>         permission_query = models.Permission.objects.filter(Q(menu_id=mid) | Q(parent__menu_id=mid))
>>     # 如果没有菜单id则输出所有的权限信息
>>     else:
>>         permission_query = models.Permission.objects.all()
>>     # 拿到查询出的权限对应的信息
>>     all_permission = permission_query.values('id', 'url', 'title', 'name', 'menu_id', 'parent_id', 'menu__title')
>>     all_permission_dict = {}
>>     for item in all_permission:
>>         menu_id = item.get('menu_id')
>>         # 找到有菜单id的权限，将其存入字典，键为权限的id
>>         if menu_id:
>>             all_permission_dict[item['id']] = item
>>             # 可以改都是引用
>>             # 得到所有有菜单的权限后，将每一个权限都设置一个children键值对，用来存储子权限信息
>>             item['children'] = []
>>     for item in all_permission:
>>         pid = item.get('parent_id')
>>         # 如果有父id代表的是子权限
>>         if pid:
>>             # 如果是子权限，就将子权限的信息存入多上一步做的处理（有菜单的父权限）children中
>>             all_permission_dict[pid]['children'].append(item)
>>     return render(request, 'rbac/menu_list.html', {
>>         "mid": mid,
>>         "all_menu": all_menu,
>>         "all_permission_dict": all_permission_dict,
>>     })
>> 
>> 
>> # 菜单的添加和编辑
>> def menu(request, edit_id=None):
>>     obj = models.Menu.objects.filter(id=edit_id).first()
>>     title = '添加菜单' if not edit_id else '编辑菜单'
>>     form_obj = forms.MenuForm(instance=obj)
>>     if request.method == 'POST':
>>         form_obj = forms.MenuForm(request.POST, instance=obj)
>>         if form_obj.is_valid():
>>             form_obj.save()
>>             return redirect(reverse('rbac:menu_list'))
>>     return render(request, 'rbac/form.html', {
>>         "form_obj": form_obj,
>>         "title": title
>>     })
>> 
>> 
>> # 菜单的删除
>> def menu_del(request, del_id):
>>     models.Menu.objects.filter(id=del_id).delete()
>>     return redirect(reverse('rbac:menu_list'))
>> 
>> 
>> # 权限的添加和编辑
>> def permission(request, edit_id=None):
>>     obj = models.Permission.objects.filter(id=edit_id).first()
>>     title = '添加权限' if not edit_id else '编辑权限'
>>     form_obj = forms.PermissionForm(instance=obj)
>>     if request.method == 'POST':
>>         form_obj = forms.PermissionForm(request.POST, instance=obj)
>>         if form_obj.is_valid():
>>             form_obj.save()
>>             return redirect(reverse('rbac:menu_list'))
>>     return render(request, 'rbac/form.html', {
>>         "form_obj": form_obj,
>>         "title": title
>>     })
>> 
>> 
>> # 权限的删除
>> def permission_del(request, del_id):
>>     models.Permission.objects.filter(id=del_id).delete()
>>     return redirect(reverse('rbac:menu_list'))
>> 
>> ```
>
>form.html
>
>>```html
>>
>>{% extends 'layout.html' %}
>>
>>{% block css %}
>>    <style>
>>        ul {
>>            list-style-type: none;
>>            padding: 0;
>>        }
>>
>>        ul li {
>>            float: left;
>>            padding: 10px;
>>            padding-left: 0;
>>            width: 80px;
>>        }
>>
>>        ul li i {
>>            font-size: 18px;
>>            margin-left: 5px;
>>            color: #6d6565;
>>        }
>>
>>    </style>
>>{% endblock %}
>>{% block content %}
>>    <div class="panel panel-default">
>>        <!-- Default panel contents -->
>>        <div class="panel-heading">
>>            {{ title }}
>>        </div>
>>
>>        <div class="panel-body">
>>
>>
>>            <div class="col-sm-8 col-sm-offset-2 ">
>>
>>                <form action="" method="post" novalidate>
>>                    {% csrf_token %}
>>                    {% for field in form_obj %}
>>
>>                        <div class="form-group row {% if field.errors %}has-error{% endif %} ">
>>
>>                            <label for="{{ field.id_for_label }}"
>>                                   class="col-sm-2 control-label"> {{ field.label }}</label>
>>                            <div class="col-sm-10">
>>                                {{ field }}
>>                                <span class="help-block">
>>                            {{ field.errors.0 }}
>>                        </span>
>>                            </div>
>>                        </div>
>>
>>                    {% endfor %}
>>
>>{#                    {{ form_obj.errors }}#}
>>                    <div class="form-group">
>>                        <div class="col-sm-offset-2 col-sm-10">
>>                            <button type="submit" class="btn btn-primary">提交</button>
>>                        </div>
>>                    </div>
>>
>>                </form>
>>
>>            </div>
>>
>>        </div>
>>
>>
>>    </div>
>>{% endblock %}
>>```
>
>role_list.html
>
>>
>>
>>```html
>>{% extends 'layout.html' %}
>>
>>{% block content %}
>>
>>
>>
>>    <div>
>>        <h1>角色管理</h1>
>>        <a href="{% url 'rbac:role_add' %}" class="btn btn-success">添加</a>
>>
>>        <table class="table table-bordered table-hover" style="margin-top: 5px">
>>            <thead>
>>            <tr>
>>                <th>序号</th>
>>                <th>名称</th>
>>                <th>操作</th>
>>            </tr>
>>            </thead>
>>            <thead>
>>
>>            {% for role in all_roles %}
>>                <tr>
>>                    <td>{{ forloop.counter }}</td>
>>                    <td>{{ role.name }}</td>
>>                    <td>
>>                        <a href="{% url 'rbac:role_edit' role.id %}"> <i class="fa fa-edit"></i> </a>
>>                        <a href="{% url 'rbac:role_del' role.id %}"> <i class="fa fa-trash-o"></i> </a>
>>                    </td>
>>                </tr>
>>
>>            {% endfor %}
>>
>>
>>            </thead>
>>
>>        </table>
>>
>>    </div>
>>
>>{% endblock %}
>>```
>
>menu_list.html
>
>```html
>{% extends 'layout.html' %}
>{% block css %}
>    <style>
>        .permission-area tr.parent {
>            background-color: #cae7fd;;
>        }
>
>        .menu-body tr .active {
>            background-color: #f1f7fd;
>            border-left: 3px solid #fdc00f;
>        }
>
>
>    </style>
>{% endblock %}
>
>{% block content %}
>    <div style="margin-top: 20px">
>        <div class="col-sm-3">
>            <div class="panel panel-default">
>                <!-- Default panel contents -->
>                <div class="panel-heading">
>                    <i class="fa fa-book"></i> 菜单管理
>                    <a href="{% url 'rbac:menu_add' %}" class="btn btn-success btn-sm pull-right "
>                       style="padding: 2px 8px;margin: -3px;">
>                        <i class="fa fa-plus"></i> 新建</a>
>                </div>
>
>                <!-- Table -->
>                <table class="table">
>                    <thead>
>                    <tr>
>                        <th>名称</th>
>                        <th>图标</th>
>                        <th>操作</th>
>                    </tr>
>                    </thead>
>                    <tbody>
>                    {% for menu in all_menu %}
>                        <tr class="{% if menu.id|safe == mid %} active {% endif %}">
>{#                                点击这个链接后，将这个菜单的id发给服务器端#}
>                            <td><a href="?mid={{ menu.id }}">{{ menu.title }}({{ menu.weight }})</a></td>
>                            <td><i class="fa {{ menu.icon }} "></i></td>
>                            <td>
>                                <a href="{% url 'rbac:menu_edit' menu.id %}"> <i class="fa fa-edit"></i> </a>
>                                <a href="{% url 'rbac:menu_del' menu.id %}"> <i class="fa fa-trash-o"></i> </a>
>                            </td>
>                        </tr>
>
>                    {% endfor %}
>
>                    </tbody>
>
>                </table>
>            </div>
>
>        </div>
>
>        <div class="col-sm-9">
>            <div class="panel panel-default">
>                <!-- Default panel contents -->
>                <div class="panel-heading"><i class="fa fa-cubes"></i> 权限管理
>                    <a href="{% url 'rbac:permission_add' %}" class="btn btn-success btn-sm pull-right "
>                       style="padding: 2px 8px;margin: -3px;">
>                        <i class="fa fa-plus"></i> 新建</a>
>                </div>
>
>                <!-- Table -->
>                <table class="table">
>                    <thead>
>                    <tr>
>                        <th>名称</th>
>                        <th>URL</th>
>                        <th>URL别名</th>
>                        <th>菜单</th>
>                        <th>所属菜单</th>
>                        <th>操作</th>
>                    </tr>
>                    </thead>
>                    <tbody class="permission-area">
>{#                           第一次循环是父权限， 第二层3循环是子权限 ， #}
>{#                           第一次循环的键是父权限的id，所有应该循环此id对应的值  #}
>                    {% for p_permission in all_permission_dict.values %}
>                        <tr class="parent" id="{{ p_permission.id }}">
>                            <td class="title"><i class="fa fa-caret-down"></i>{{ p_permission.title }}</td>
>                            <td>{{ p_permission.url }}</td>
>                            <td>{{ p_permission.name }}</td>
>                            <td>是</td>
>                            <td>{{ p_permission.menu__title }}</td>
>                            <td>
>                                <a href="{% url 'rbac:permission_edit' p_permission.id %}"> <i class="fa fa-edit"></i>
>                                </a>
>                                <a href="{% url 'rbac:permission_del' p_permission.id %}"> <i class="fa fa-trash-o"></i>
>                                </a>
>                            </td>
>                        </tr>
>                        {% for c_permission in p_permission.children %}
>                            <tr pid="{{ c_permission.parent_id }}">
>                                <td>{{ c_permission.title }}</td>
>                                <td>{{ c_permission.url }}</td>
>                                <td>{{ c_permission.name }}</td>
>                                <td></td>
>                                <td></td>
>
>                                <td>
>                                    <a href="{% url 'rbac:permission_edit' c_permission.id %}"> <i
>                                            class="fa fa-edit"></i> </a>
>                                    <a href="{% url 'rbac:permission_del' c_permission.id %}"> <i
>                                            class="fa fa-trash-o"></i> </a>
>                                </td>
>                            </tr>
>                        {% endfor %}
>
>                    {% endfor %}
>
>                    </tbody>
>
>                </table>
>            </div>
>        </div>
>    </div>
>{% endblock %}
>
>{% block js %}
>    <script>
>
>
>        $('.permission-area').on('click', '.parent .title', function () {
>            var caret = $(this).find('i');
>            var id = $(this).parent().attr('id');
>            if (caret.hasClass('fa-caret-right')) {
>                caret.removeClass('fa-caret-right').addClass('fa-caret-down');
>                $(this).parent().nextAll('tr[pid="' + id + '"]').removeClass('hide');
>            } else {
>                caret.removeClass('fa-caret-down').addClass('fa-caret-right');
>                $(this).parent().nextAll('tr[pid="' + id + '"]').addClass('hide');
>
>            }
>        })
>
>
>    </script>
>{% endblock %}
>```
>
>



# 分类管理权限

>
>
>models.py
>
>>
>>
>>```python
>>from django.db import models
>>
>>
>>class Menu(models.Model):
>>    """
>>    一级菜单
>>    """
>>    title = models.CharField(max_length=32, verbose_name='标题', unique=True)  # 一级菜单的名字
>>    icon = models.CharField(max_length=32, verbose_name='图标', null=True, blank=True)
>>    weight = models.IntegerField(verbose_name='权重', default=1)
>>
>>    class Meta:
>>        verbose_name_plural = '菜单表'
>>        verbose_name = '菜单表'
>>
>>    def __str__(self):
>>        return self.title
>>
>>
>>class Permission(models.Model):
>>    """
>>    权限表
>>    有关联Menu的二级菜单
>>    没有关联Menu的不是二级菜单，是不可以做菜单的权限
>>    """
>>    title = models.CharField(max_length=32, verbose_name='标题')
>>    url = models.CharField(max_length=32, verbose_name='权限')
>>    menu = models.ForeignKey('Menu', null=True, blank=True, verbose_name='菜单')
>>    # 该权限关联的其他权限是否也是在当前url上展示
>>    parent = models.ForeignKey(to='Permission', null=True, blank=True, verbose_name='父权限')
>>
>>    name = models.CharField(max_length=32, null=True, blank=True, unique=True, verbose_name='权限的别名')
>>
>>    class Meta:
>>        verbose_name_plural = '权限表'
>>        verbose_name = '权限表'
>>
>>    def __str__(self):
>>        return self.title
>>
>>
>>class Role(models.Model):
>>    name = models.CharField(max_length=32, verbose_name='角色名称')
>>    permissions = models.ManyToManyField(to='Permission', verbose_name='角色所拥有的权限', blank=True)
>>
>>    def __str__(self):
>>        return self.name
>>
>>
>>class User(models.Model):
>>    """
>>    用户表
>>    """
>>    name = models.CharField(max_length=32, verbose_name='用户名')
>>    password = models.CharField(max_length=32, verbose_name='密码')
>>    roles = models.ManyToManyField(to='Role', verbose_name='用户所拥有的角色', blank=True)
>>
>>    def __str__(self):
>>        return self.name
>>
>>```
>>
>>
>
>urls.py
>
>>
>>
>>```python
>># 分类管理权限
>>url(r'^distribute/permissions/$', views.distribute_permissions, name='distribute_permissions'),
>>```
>
>views.py
>
>>
>>
>>```python
>>def distribute_permissions(request):
>>    """
>>    分配权限
>>    :param request:
>>    :return:
>>    """
>>    uid = request.GET.get('uid')
>>    rid = request.GET.get('rid')
>>
>>    if request.method == 'POST' and request.POST.get('postType') == 'role':
>>        user = models.User.objects.filter(id=uid).first()
>>        if not user:
>>            return HttpResponse('用户不存在')
>>        user.roles.set(request.POST.getlist('roles'))
>>
>>    if request.method == 'POST' and request.POST.get('postType') == 'permission' and rid:
>>        role = models.Role.objects.filter(id=rid).first()
>>        if not role:
>>            return HttpResponse('角色不存在')
>>        role.permissions.set(request.POST.getlist('permissions'))
>>
>>    user_list = models.User.objects.all()
>>
>>    user_has_roles = models.User.objects.filter(id=uid).values('id', 'roles')
>>
>>    user_has_roles_dict = {item['roles']: None for item in user_has_roles}
>>
>>    role_list = models.Role.objects.all()
>>
>>    if rid:
>>        role_has_permissions = models.Role.objects.filter(id=rid, permissions__id__isnull=False).values('id',
>>                                                                                                        'permissions')
>>    elif uid and not rid:
>>        user = models.User.objects.filter(id=uid).first()
>>        if not user:
>>            return HttpResponse('用户不存在')
>>        role_has_permissions = user.roles.filter(permissions__id__isnull=False).values('id', 'permissions')
>>    else:
>>        role_has_permissions = []
>>
>>    role_has_permissions_dict = {item['permissions']: None for item in role_has_permissions}
>>
>>    all_menu_list = []
>>
>>    queryset = models.Menu.objects.values('id', 'title')
>>    menu_dict = {}
>>
>>    for item in queryset:
>>        item['children'] = []
>>        menu_dict[item['id']] = item
>>        all_menu_list.append(item)
>>
>>    other = {'id': None, 'title': '其他', 'children': []}
>>    all_menu_list.append(other)
>>    menu_dict[None] = other
>>
>>    root_permission = models.Permission.objects.filter(menu__isnull=False).values('id', 'title', 'menu_id')
>>    root_permission_dict = {}
>>
>>    for per in root_permission:
>>        per['children'] = []
>>        nid = per['id']
>>        menu_id = per['menu_id']
>>        root_permission_dict[nid] = per
>>        menu_dict[menu_id]['children'].append(per)
>>
>>    node_permission = models.Permission.objects.filter(menu__isnull=True).values('id', 'title', 'parent_id')
>>
>>    for per in node_permission:
>>        pid = per['parent_id']
>>        if not pid:
>>            menu_dict[None]['children'].append(per)
>>            continue
>>        root_permission_dict[pid]['children'].append(per)
>>
>>    return render(
>>        request,
>>        'rbac/distribute_permissions.html',
>>        {
>>            'user_list': user_list,
>>            'role_list': role_list,
>>            'user_has_roles_dict': user_has_roles_dict,
>>            'role_has_permissions_dict': role_has_permissions_dict,
>>            'all_menu_list': all_menu_list,
>>            'uid': uid,
>>            'rid': rid
>>        }
>>    )
>>
>>```
>>
>>distribute_permissions.html
>>
>>>
>>>
>>>```html
>>>{% extends 'layout.html' %}
>>>{% load rbac %}
>>>{% block css %}
>>>    <style>
>>>        .user-area ul {
>>>            padding-left: 20px;
>>>        }
>>>
>>>        .user-area li {
>>>            cursor: pointer;
>>>            padding: 2px 0;
>>>        }
>>>
>>>        .user-area li a {
>>>            display: block;
>>>        }
>>>
>>>        .user-area li.active {
>>>            font-weight: bold;
>>>            color: red;
>>>        }
>>>
>>>        .user-area li.active a {
>>>            color: red;
>>>        }
>>>
>>>        .role-area tr td a {
>>>            display: block;
>>>        }
>>>
>>>        .role-area tr.active {
>>>            background-color: #f1f7fd;
>>>            border-left: 3px solid #fdc00f;
>>>        }
>>>
>>>        .permission-area tr.root {
>>>            background-color: #f1f7fd;
>>>            cursor: pointer;
>>>        }
>>>
>>>        .permission-area tr.root td i {
>>>            margin: 3px;
>>>        }
>>>
>>>        .permission-area .node {
>>>
>>>        }
>>>
>>>        .permission-area .node input[type='checkbox'] {
>>>            margin: 0 5px;
>>>        }
>>>
>>>        .permission-area .node .parent {
>>>            padding: 5px 0;
>>>        }
>>>
>>>        .permission-area .node label {
>>>            font-weight: normal;
>>>            margin-bottom: 0;
>>>            font-size: 12px;
>>>        }
>>>
>>>        .permission-area .node .children {
>>>            padding: 0 0 0 20px;
>>>        }
>>>
>>>        .permission-area .node .children .child {
>>>            display: inline-block;
>>>            margin: 2px 5px;
>>>        }
>>>
>>>        table {
>>>            font-size: 12px;
>>>        }
>>>
>>>        .panel-body {
>>>            font-size: 12px;
>>>        }
>>>
>>>        .panel-body .form-control {
>>>            font-size: 12px;
>>>        }
>>>    </style>
>>>{% endblock %}
>>>
>>>{% block content %}
>>>    <div class="luffy-container">
>>>        <div class="col-md-3 user-area">
>>>            <div class="panel panel-default">
>>>                <!-- Default panel contents -->
>>>                <div class="panel-heading">
>>>                    <i class="fa fa-address-book-o" aria-hidden="true"></i> 用户信息
>>>                </div>
>>>
>>>                <div class="panel-body">
>>>                    <ul>
>>>                        {% for user in user_list %}
>>>
>>>                            <li class= {% if user.id|safe == uid %} "active" {% endif %}>
>>>                                <a href="?uid={{ user.id }}">{{ user.name }}</a> </li>
>>>
>>>                        {% endfor %}
>>>                    </ul>
>>>                </div>
>>>
>>>            </div>
>>>        </div>
>>>
>>>        <div class="col-md-3 role-area">
>>>            <form method="post">
>>>                {% csrf_token %}
>>>                <input type="hidden" name="postType" value="role">
>>>                <div class="panel panel-default">
>>>                    <!-- Default panel contents -->
>>>                    <div class="panel-heading">
>>>                        <i class="fa fa-book" aria-hidden="true"></i> 角色
>>>                        {% if uid %}
>>>                            <button type="submit" class="right btn btn-success btn-xs"
>>>                                    style="padding: 2px 8px;margin: -3px;">
>>>                                <i class="fa fa-save" aria-hidden="true"></i>
>>>                                保存
>>>                            </button>
>>>                        {% endif %}
>>>                    </div>
>>>                    <div class="panel-body" style="color: #d4d4d4;padding:10px  5px;">
>>>                        提示：点击用户后才能为其分配角色
>>>                    </div>
>>>                    <table class="table">
>>>                        <thead>
>>>                        <tr>
>>>                            <th>角色</th>
>>>                            <th>选择</th>
>>>                        </tr>
>>>                        </thead>
>>>                        <tbody>
>>>                        {% for role in role_list %}
>>>                            <tr {% if role.id|safe == rid %} class="active"  {% endif %}>
>>>
>>>                                <td><a href="?{% gen_role_url request role.id %}">{{ role.name }}</a></td>
>>>                                <td>
>>>                                    {% if role.id in user_has_roles_dict %}
>>>                                        <input type="checkbox" name="roles" value="{{ role.id }}" checked/>
>>>                                    {% else %}
>>>                                        <input type="checkbox" name="roles" value="{{ role.id }}"/>
>>>                                    {% endif %}
>>>                                </td>
>>>                            </tr>
>>>                        {% endfor %}
>>>
>>>                        </tbody>
>>>                    </table>
>>>
>>>                </div>
>>>            </form>
>>>        </div>
>>>
>>>        <div class="col-md-6 permission-area">
>>>            <form method="post">
>>>                {% csrf_token %}
>>>                <input type="hidden" name="postType" value="permission">
>>>                <div class="panel panel-default">
>>>                    <!-- Default panel contents -->
>>>                    <div class="panel-heading">
>>>                        <i class="fa fa-sitemap" aria-hidden="true"></i> 权限分配
>>>                        {% if rid %}
>>>                            <button class="right btn btn-success btn-xs" style="padding: 2px 8px;margin: -3px;">
>>>                                <i class="fa fa-save" aria-hidden="true"></i>
>>>                                保存
>>>                            </button>
>>>                        {% endif %}
>>>                    </div>
>>>                    <div class="panel-body" style="color: #d4d4d4;padding: 10px 5px;">
>>>                        提示：点击角色后，才能为其分配权限。
>>>                    </div>
>>>                    <table class="table">
>>>                        <tbody>
>>>                        {% for item in all_menu_list %}
>>>                            <tr class="root">
>>>                                <td><i class="fa fa-caret-down" aria-hidden="true"></i>{{ item.title }}</td>
>>>                            </tr>
>>>                            <tr class="node">
>>>                                <td>
>>>                                    {% for node in item.children %}
>>>                                        <div class="parent">
>>>                                            {% if node.id in role_has_permissions_dict %}
>>>                                                <input id="permission_{{ node.id }}" name="permissions"
>>>                                                       value="{{ node.id }}" type="checkbox" checked>
>>>                                            {% else %}
>>>                                                <input id="permission_{{ node.id }}" name="permissions"
>>>                                                       value="{{ node.id }}" type="checkbox">
>>>                                            {% endif %}
>>>
>>>                                            {% if forloop.parentloop.last %}
>>>                                                <label for="permission_{{ node.id }}">{{ node.title }}</label>
>>>                                            {% else %}
>>>                                                <label for="permission_{{ node.id }}">{{ node.title }}（菜单）</label>
>>>                                            {% endif %}
>>>                                        </div>
>>>                                        <div class="children">
>>>                                            {% for child in node.children %}
>>>                                                <div class="child">
>>>                                                    {% if child.id in role_has_permissions_dict %}
>>>                                                        <input id="permission_{{ child.id }}" name="permissions"
>>>                                                               type="checkbox" value="{{ child.id }}" checked>
>>>                                                    {% else %}
>>>                                                        <input id="permission_{{ child.id }}" name="permissions"
>>>                                                               type="checkbox" value="{{ child.id }}">
>>>                                                    {% endif %}
>>>
>>>                                                    <label for="permission_{{ child.id }}">{{ child.title }}</label>
>>>                                                </div>
>>>                                            {% endfor %}
>>>                                        </div>
>>>                                    {% endfor %}
>>>                                </td>
>>>                            </tr>
>>>                        {% endfor %}
>>>                        </tbody>
>>>                    </table>
>>>                </div>
>>>            </form>
>>>        </div>
>>>
>>>    </div>
>>>{% endblock %}
>>>{% block js %}
>>>    <script>
>>>        $(function () {
>>>            bindRootPermissionClick();
>>>        });
>>>
>>>        function bindRootPermissionClick() {
>>>            $('.permission-area').on('click', '.root', function () {
>>>                var caret = $(this).find('i');
>>>                if (caret.hasClass('fa-caret-right')) {
>>>                    caret.removeClass('fa-caret-right').addClass('fa-caret-down');
>>>                    $(this).next().removeClass('hide');
>>>                } else {
>>>                    caret.removeClass('fa-caret-down').addClass('fa-caret-right');
>>>                    $(this).next().addClass('hide');
>>>
>>>                }
>>>            })
>>>        }
>>>    </script>
>>>{% endblock %}
>>>
>>>```
>>>
>>>routes.py
>>>
>>>```python
>>>from django.conf import settings
>>>from django.utils.module_loading import import_string
>>>from django.urls import RegexURLResolver, RegexURLPattern
>>>from collections import OrderedDict
>>>
>>>
>>>def recursion_urls(pre_namespace, pre_url, urlpatterns, url_ordered_dict):
>>>    for item in urlpatterns:
>>>        if isinstance(item, RegexURLResolver):
>>>            if pre_namespace:
>>>                if item.namespace:
>>>                    namespace = "%s:%s" % (pre_namespace, item.namespace,)
>>>                else:
>>>                    namespace = pre_namespace
>>>            else:
>>>                if item.namespace:
>>>                    namespace = item.namespace
>>>                else:
>>>                    namespace = None
>>>            recursion_urls(namespace, pre_url + item.regex.pattern, item.url_patterns, url_ordered_dict)
>>>        else:
>>>
>>>            if pre_namespace:
>>>                name = "%s:%s" % (pre_namespace, item.name,)
>>>            else:
>>>                name = item.name
>>>            if not item.name:
>>>                raise Exception('URL路由中必须设置name属性')
>>>
>>>            url = pre_url + item._regex
>>>            url_ordered_dict[name] = {'name': name, 'url': url.replace('^', '').replace('$', '')}
>>>
>>>
>>>def get_all_url_dict(ignore_namespace_list=None):
>>>    """
>>>    获取路由中
>>>    :return:
>>>    """
>>>    ignore_list = ignore_namespace_list or []
>>>    url_ordered_dict = OrderedDict()
>>>    # urls.py
>>>    md = import_string(settings.ROOT_URLCONF)
>>>    urlpatterns = []
>>>
>>>    for item in md.urlpatterns:
>>>        if isinstance(item, RegexURLResolver) and item.namespace in ignore_list:
>>>            continue
>>>        urlpatterns.append(item)
>>>    recursion_urls(None, "/", urlpatterns, url_ordered_dict)
>>>    return url_ordered_dict
>>>
>>>```
>>>
>>>templatetags下的rbac.py文件
>>>
>>>```python
>>>from collections import OrderedDict
>>>from django import template
>>>from django.conf import settings
>>>
>>>register = template.Library()
>>>
>>>@register.simple_tag
>>>def gen_role_url(request, rid):
>>>    params = request.GET.copy()
>>>    params._mutable = True
>>>    params['rid'] = rid
>>>    return params.urlencode()
>>>```
>
>有一个批量操作

内置函数

网络（七层）

并发编程 - 进程 线程 协程

数据库  case --- when

Django - 中间件 WSGI协议  模块怎么整的

缓存redis  武sir

flask 等框架

IO多路复用

数据库的引擎

restful api

算法

celery分布式任务队列

从头开始查问题

一定做完全部面试题



权限的业务与每个细节

