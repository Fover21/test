# 业务逻辑整体描述

>
>
>- 登录、注册、注销用户
>- 展示客户信息 - 有一个路由分发
>  - 有公户和私户
>  - 私户转公户，公户转私户
>  - 增加和编辑客户
>- 用户（销售）
>- 用户（销售）登录后直接跳转该用户下的客户有哪些
>  - 可以添加和编辑客户
>  - 可以点击按钮查看当前客户的跟进记录
>- 跟进记录表展示
>  - 编辑和添加跟进记录表
>  - 在用户的自己账户展示页面有一个查看这个用户的跟进记录的按钮
>- 展示表名记录 - 展示所有报名的客户
>  - 用户（销售的哪些用户里面）有一栏是记录客户的报名状态
>    - 如果没有就有一个按钮是用来报名的
>    - 报名成功后这个按钮可变为查看报名记录的添加
>- 用户（班主任）
>- 展示班级信息
>  - 添加和编辑班级信息
>- 点击展示班级信息的班级名称展示课程信息
>  - 添加和编辑课程信息
>  - 这里面可以来一个操作，
>    - 就是将这个课上课的人都加载到里面
>- 点击课程信息中的课程名称，出现课程中的学生的学习记录
>  - 可以继续编辑，保存学习学习状态

# URL

>
>
>```python
>from django.conf.urls import url, include
>from django.contrib import admin
>from crm.views import consultant  # 在app中创建了views文件夹里面存对应用户的视图py文件(用户)
>urlpatterns = [
>    url(r'^admin/', admin.site.urls),
>    # 登录
>    url(r'^login/$', consultant.login, name='login'),
>    # 注销
>    url(r'^logout/$', consultant.logout, name='logout'),
>    # 注册
>    url(r'^register/', consultant.register),
>    # 展示客户信息
>    url(r'^crm/', include('crm.urls', namespace='crm')),
>]
>```
>
>

### 登录

>
>
>用到的是auth这个app，重写了他的models，重写完models后需要在settings中配置
>
>```
>AUTH_USER_MODEL = 'crm.UserProfile'  # app名.重写的类名
>```
>
>登录：
>
>>```python
>># 登录
>>def login(request):
>>    if request.method == 'POST':
>>        username = request.POST.get('username')
>>        password = request.POST.get('password')
>>        obj = auth.authenticate(username=username, password=password)
>>        if obj:
>>            auth.login(request, obj)
>>            return redirect(reverse('crm:my_customer'))
>>    return render(request, 'login.html')
>>```
>>
>>>登录的时候需要注意的就是将前端的账号密码拿到就，与auth的user表中的密码账号对比
>>>
>>>但是我们重写了这个表，与UserProfile表中的账号密码比较。
>>>
>>>这时我们用到了auth.authenticate(username=,password=)这个就是Django帮助我们判断用户是否
>>>
>>>存在，如果存在就登录，auth.login(request, obj)是用来记录session的。

### 注册

>
>
>注册：
>
>```python
># 注册
>def register(request):
>    form_obj = forms.Register()
>    if request.method == 'POST':
>        form_obj = forms.Register(request.POST)
>        if form_obj.is_valid():
>            # 方式一
>            print('form_obj.cleaned_data', form_obj.cleaned_data)
>            # form_obj.cleaned_data.pop('re_password')
>            # models.UserProfile.objects.create_user(**form_obj.cleaned_data)
>            # 方式二
>            obj = form_obj.save()
>            # 将原生的密码改为加密的
>            obj.set_password(obj.password)
>            obj.save()
>            return redirect(reverse('login'))
>    return render(request, 'register.html', {'form_obj': form_obj})
>```

### 注销

>
>
>注销
>
>```python
>#  注销
>def logout(request):
>    auth.logout(request)
>    return redirect(reverse('login'))
>```

### 展示客户信息

### 有私户和公户之分

### 有分页

### 有查询

>
>
>