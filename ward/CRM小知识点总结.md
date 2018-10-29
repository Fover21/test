# 1、更改系统auth表时的配置

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
>>>>    
>>>>    ```