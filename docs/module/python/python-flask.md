# flask

flask

## flask_login

Flask-Login提供Flask的用户会话管理。它处理登录，注销和记住用户长时间会话的常见任务.

1. 在session中存储活跃用户ID, 让你轻松登录登出
2. 限制登录用户或者访客的视图
3. 处理棘手的‘remember me’功能
4. 保护用户sessis不被窃取
5. 与其他插件集成，比如Flask-Principal 或者其他认证扩展

### Installation

```shell
pip install flask-login
```

### Configuring your Application

```python
login_manager = LoginManager()
login_manager.init_app(app)
# Flask-Login uses sessions for authentication. This means you must set the secret key on your application
# app.secret_key = b'_5#y2L"F4Q8z\n\xec]/' # random key
```

```python
class LoginManager(object):
    # ...
    def init_app(self, app, add_context_processor=True):
            '''
            Configures an application. This registers an `after_request` call, and
            attaches this `LoginManager` to it as `app.login_manager`.
            :param app: The :class:`flask.Flask` object to configure.
            :type app: :class:`flask.Flask`
            :param add_context_processor: Whether to add a context processor to
                the app that adds a `current_user` variable to the template.
                Defaults to ``True``.
            :type add_context_processor: bool
            '''
            app.login_manager = self
            app.after_request(self._update_remember_cookie)

            if add_context_processor:
                app.context_processor(_user_context_processor)
```

### How it Works

You will need to provide a user_loader callback. This callback is used to reload the user object from the user ID stored in the session. It should take the unicode ID of a user, and return the corresponding user object.For example:

```python
@login_manager.user_loader
def load_user(user_id):
    return User.get(user_id)
```

### Your User Class¶

The class that you use to represent users needs to implement these properties and methods:

```text
is_authenticated
    This property should return True if the user is authenticated, i.e. they have provided valid credentials. (Only authenticated users will fulfill the criteria of login_required.)
is_active
    This property should return True if this is an active user - in addition to being authenticated, they also have activated their account, not been suspended, or any condition your application has for rejecting an account. Inactive accounts may not log in (without being forced of course).
is_anonymous
    This property should return True if this is an anonymous user. (Actual users should return False instead.)
get_id()
    This method must return a unicode that uniquely identifies this user, and can be used to load the user from the user_loader callback. Note that this must be a unicode - if the ID is natively an int or some other type, you will need to convert it to unicode.
```

To make implementing a user class easier, you can inherit from **UserMixin**, which provides default implementations for all of these properties and methods. (It’s not required, though.)

例子

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    # Here we use a class of some kind to represent and validate our
    # client-side form data. For example, WTForms is a library that will
    # handle this for us, and we use a custom LoginForm to validate.
    form = LoginForm()
    if form.validate_on_submit():
        # Login and validate the user.
        # user should be an instance of your `User` class
        login_user(user)

        flask.flash('Logged in successfully.')

        next = flask.request.args.get('next')
        # is_safe_url should check if the url is safe for redirects.
        # See http://flask.pocoo.org/snippets/62/ for an example.
        if not is_safe_url(next):
            return flask.abort(400)

        return flask.redirect(next or flask.url_for('index'))
    return flask.render_template('login.html', form=form)
```

### Customizing the Login Process

```python
# 登录视图的名称
login_manager.login_view = "users.login"

# 闪烁的默认消息是“请登录以访问此页面. To customize the message, set LoginManager.login_message:
login_manager.login_message = u"Bonvolu ensaluti por uzi tiun paĝon."

# 自定义消息类别, set LoginManager.login_message_category:
login_manager.login_message_category = "info"
```

### session protect

```python
login_manager.session_protection = "strong"
login_manager.session_protection = None
```

### other

[flask-login文档](https://flask-login.readthedocs.io/en/latest/)