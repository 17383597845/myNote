```python

import tornado.ioloop
import tornado.web
 
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.render('index.html')       
class LoginHandler(BaseHandler):
    def get(self):
        '''
        当用户访登录的时候我们就得给他写cookie了,但是这里没有写在哪里写了呢?
        在哪里呢?之前写的Handler都是继承的RequestHandler,这次继承的是BaseHandler是自己写的Handler
        继承自己的类,在类了加扩展initialize! 在这里我们可以在这里做获取用户cookie或者写cookie都可以在这里做
        '''
        '''
        我们知道LoginHandler对象就是self,我们可不可以self.set_cookie()可不可以self.get_cookie()
        '''
        # self.set_cookie()
        # self.get_cookie()
        self.render('login.html', **{'status': ''})
def login(request):
    #获取用户输入
    login_form = AccountForm.LoginForm(request.POST)
    if request.method == 'POST':
        #判断用户输入是否合法
        if login_form.is_valid():#如果用户输入是合法的
            username = request.POST.get('username')
            password = request.POST.get('password')
            if models.UserInfo.objects.get(username=username) and models.UserInfo.objects.get(username=username).password == password:
                    request.session['auth_user'] = username
                    return redirect('/index/')
            else:
                return render(request,'account/login.html',{'model': login_form,'backend_autherror':'用户名或密码错误'})
        else:
            error_msg = login_form.errors.as_data()
            return render(request,'account/login.html',{'model': login_form,'errors':error_msg})
    # 如果登录成功，写入session，跳转index
    return render(request, 'account/login.html', {'model': login_form}


```