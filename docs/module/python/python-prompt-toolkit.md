# python-prompt-toolkit

如何学习和理解prompt_toolkit

- First, learn how to print text. This is important, because it covers how to use “formatted text”, which is something you’ll use whenever you want to use colors anywhere.
- Secondly, go through the asking for input section. This is useful for almost any use case, even for full screen applications. It covers autocompletions, syntax highlighting, key bindings, and so on.

- Then, learn about Dialogs, which is easy and fun.

- Finally, learn about full screen applications and read through the advanced topics.

## 打印(和使用)格式化文本

### 打印纯文本

```python
#from __future__ import unicode_literals
from prompt_toolkit import print_formatted_text

print_formatted_text('Hello world')
```

### 格式化文本

There are several ways to display colors:

- By creating an HTML object.
- By creating an ANSI object that contains ANSI  escape sequences.
- By creating a list of (style, text) tuples.
- By creating a list of (pygments.Token, text) tuples, and wrapping it in PygmentsTokens.

#### HTML

```python
from __future__ import unicode_literals, print_function
from prompt_toolkit import print_formatted_text, HTML

print_formatted_text(HTML('<b>This is bold</b>'))
print_formatted_text(HTML('<i>This is italic</i>'))
print_formatted_text(HTML('<u>This is underlined</u>'))
# Colors from the ANSI palette.
print_formatted_text(HTML('<ansired>This is red</ansired>'))
print_formatted_text(HTML('<ansigreen>This is green</ansigreen>'))

# Named colors (256 color palette, or true color, depending on the output).
print_formatted_text(HTML('<skyblue>This is sky blue</skyblue>'))
print_formatted_text(HTML('<seagreen>This is sea green</seagreen>'))
print_formatted_text(HTML('<violet>This is violet</violet>'))

print_formatted_text(HTML('<aaa fg="ansiwhite" bg="ansigreen">White on green</aaa>'))

from prompt_toolkit.styles import Style

style = Style.from_dict({
    'aaa': '#ff0066',
    'bbb': '#44ff00 italic',
})

print_formatted_text(HTML('<aaa>Hello</aaa> <bbb>world</bbb>!'), style=style)
```

#### ANSI

```python
from prompt_toolkit import print_formatted_text, ANSI

print_formatted_text(ANSI('\x1b[31mhello \x1b[32mworld'))
```

#### (style, text) tuples

```python
from __future__ import unicode_literals, print_function
from prompt_toolkit import print_formatted_text
from prompt_toolkit.formatted_text import FormattedText

text = FormattedText([
    ('#ff0066', 'Hello'),
    ('', ' '),
    ('#44ff00 italic', 'World'),
])
print_formatted_text(text)

from prompt_toolkit.styles import Style

# The text.
text = FormattedText([
    ('class:aaa', 'Hello'),
    ('', ' '),
    ('class:bbb', 'World'),
])
# The style sheet.
style = Style.from_dict({
    'aaa': '#ff0066',
    'bbb': '#44ff00 italic',
})
print_formatted_text(text, style=style)
```

#### Pygments (Token, text) tuples

[Pygments](https://python-prompt-toolkit.readthedocs.io/en/stable/pages/printing_text.html#pygments-token-text-tuples)

#### to_formatted_text

```python
from prompt_toolkit.formatted_text import to_formatted_text, HTML
from prompt_toolkit import print_formatted_text

html = HTML('<aaa>Hello</aaa> <bbb>world</bbb>!')
text = to_formatted_text(html, style='class:my_html bg:#00ff00 italic')

print_formatted_text(text)
```

## 如何输入 (prompts)

```python
from prompt_toolkit import prompt

text = prompt('Give me some input: ')
print('You said: %s' % text)
```

### PromptSession 对象

prompt是PromptSession实例的prompt方法

```python
from prompt_toolkit import PromptSession

# Create prompt object.
session = PromptSession()

# Do multiple input calls.
text1 = session.prompt()
text2 = session.prompt()
```

### 语法高亮

使用了pygments这个库，除了html还有python，shell等等

```python
from pygments.lexers.html import HtmlLexer
from prompt_toolkit.shortcuts import prompt
from prompt_toolkit.lexers import PygmentsLexer

text = prompt('Enter HTML: ', lexer=PygmentsLexer(HtmlLexer))
print('You said: %s' % text)

```

### 颜色

跟语法高亮接近，但还是有区别

```python
from pygments.lexers.html import HtmlLexer
from prompt_toolkit.shortcuts import prompt
from prompt_toolkit.styles import Style
from prompt_toolkit.lexers import PygmentsLexer

our_style = Style.from_dict({
    'pygments.comment':   '#888888 bold',
    'pygments.keyword':   '#ff88ff bold',
})

text = prompt('Enter HTML: ', lexer=PygmentsLexer(HtmlLexer),
              style=our_style)
```

#### 使用Pygments style

All Pygments style classes can be used as well, when they are wrapped through style_from_pygments_cls()

```python
from prompt_toolkit.shortcuts import prompt
from prompt_toolkit.styles import style_from_pygments_cls, merge_styles
from prompt_toolkit.lexers import PygmentsLexer

from pygments.styles.tango import TangoStyle
from pygments.lexers.html import HtmlLexer

our_style = merge_styles([
    style_from_pygments_cls(TangoStyle),
    Style.from_dict({
        'pygments.comment': '#888888 bold',
        'pygments.keyword': '#ff88ff bold',
    })
])

text = prompt('Enter HTML: ', lexer=PygmentsLexer(HtmlLexer),
              style=our_style)
```

#### 着色提示

```python
from prompt_toolkit.shortcuts import prompt
from prompt_toolkit.styles import Style

style = Style.from_dict({
    # User input (default text).
    '':          '#ff0066',

    # Prompt.
    'username': '#884444',
    'at':       '#00aa00',
    'colon':    '#0000aa',
    'pound':    '#00aa00',
    'host':     '#00ffff bg:#444400',
    'path':     'ansicyan underline',
})

message = [
    ('class:username', 'john'),
    ('class:at',       '@'),
    ('class:host',     'localhost'),
    ('class:colon',    ':'),
    ('class:path',     '/user/john'),
    ('class:pound',    '# '),
]
# If you want to have 24bit true color, this is possible by adding the true_color=True option to the prompt() function.
text = prompt(message, style=style)
```

### 自动补全

```python
from prompt_toolkit import prompt
from prompt_toolkit.completion import WordCompleter

html_completer = WordCompleter(['<html>', '<body>', '<head>', '<title>', 'pengweidang'])
text = prompt('Enter HTML: ', completer=html_completer)
print('You said: %s' % text)
```

#### 个性化补全

```python
from prompt_toolkit.completion import Completer, Completion

class MyCustomCompleter(Completer):
    def get_completions(self, document, complete_event):
        # Display this completion, black on yellow.
        yield Completion('completion1', start_position=0,
                         style='bg:ansiyellow fg:ansiblack')

        # Underline completion.
        yield Completion('completion2', start_position=0,
                         style='underline')

        # Specify class name, which will be looked up in the style sheet.
        yield Completion('completion3', start_position=0,
                         style='class:special-completion')
```

```python
from prompt_toolkit.completion import Completer, Completion
from prompt_toolkit.formatted_text import HTML

class MyCustomCompleter(Completer):
    def get_completions(self, document, complete_event):
        yield Completion(
            'completion1', start_position=0,
            display=HTML('<b>completion</b><ansired>1</ansired>'),
            style='bg:ansiyellow')
```

#### 模糊补全

Prompt_toolkit附带了FuzzyCompleter和FuzzyWordCompleter类。

#### Complete while typing

自动补全可以在键入时或用户按下Tab键时自动生成，注意会与‘enable_history_search’冲突。

```python
ext = prompt('Enter HTML: ', completer=my_completer,
              complete_while_typing=True)
```

#### 异步补全

如果生成补全会花费很多时间，最好在后台线程中完成。

```python
text = prompt('> ', completer=MyCustomCompleter(), complete_in_thread=True)
```

### 输入验证

```python
from prompt_toolkit.validation import Validator, ValidationError
from prompt_toolkit import prompt

class NumberValidator(Validator):
    def validate(self, document):
        text = document.text

        if text and not text.isdigit():
            i = 0

            # Get index of fist non numeric character.
            # We want to move the cursor here.
            for i, c in enumerate(text):
                if not c.isdigit():
                    break

            raise ValidationError(message='This input contains non-numeric characters',
                                  cursor_position=i)

number = int(prompt('Give a number: ', validator=NumberValidator()))
print('You said: %i' % number)
```

实时验证： 在prompt函数中，validate_while_typing=True

通过回调函数验证

```python
from prompt_toolkit.validation import Validator
from prompt_toolkit import prompt

def is_number(text):
    return text.isdigit()

validator = Validator.from_callable(
    is_number,
    error_message='This input contains non-numeric characters',
    move_cursor_to_end=True)

number = int(prompt('Give a number: ', validator=validator))
print('You said: %i' % number)
```

### 历史记录

如果需要保存到文件

```python
from prompt_toolkit import PromptSession
from prompt_toolkit.history import FileHistory

session = PromptSession(history=FileHistory('~/.myhistory'))

while True:
    session.prompt()
```

### 自动提示

从历史记录中自动提示

```python
from prompt_toolkit import PromptSession
from prompt_toolkit.history import InMemoryHistory
from prompt_toolkit.auto_suggest import AutoSuggestFromHistory

session = PromptSession()

while True:
    text = session.prompt('> ', auto_suggest=AutoSuggestFromHistory())
    print('You said: %s' % text)
```

### 添加底部工具栏

```python
from prompt_toolkit import prompt
from prompt_toolkit.styles import Style

def bottom_toolbar():
    return [('class:bottom-toolbar', ' This is a toolbar. ')]

style = Style.from_dict({
    'bottom-toolbar': '#ffffff bg:#333333',
})

text = prompt('> ', bottom_toolbar=bottom_toolbar, style=style)
print('You said: %s' % text)
```

默认的类名是bottom-toolbar，它也将用于填充工具栏的背景。

### 添加右侧prompt

```python
from prompt_toolkit import prompt
from prompt_toolkit.styles import Style

example_style = Style.from_dict({
    'rprompt': 'bg:#ff0066 #ffffff',
})

def get_rprompt():
    return '<rprompt>'

answer = prompt('> ', rprompt=get_rprompt, style=example_style)
```

### Vi 输入模式

Vi模式

```python
from prompt_toolkit import prompt

prompt('> ', vi_mode=True)
```

### 自定义键绑定

```python
from prompt_toolkit import prompt
from prompt_toolkit.application import run_in_terminal
from prompt_toolkit.key_binding import KeyBindings

bindings = KeyBindings()

@bindings.add('c-t')
def _(event):
    " Say 'hello' when `c-t` is pressed. "
    def print_hello():
        print('hello world')
    run_in_terminal(print_hello)

@bindings.add('c-x')
def _(event):
    " Exit when `c-x` is pressed. "
    event.app.exit()

text = prompt('> ', key_bindings=bindings)
print('You said: %s' % text)
```

#### 根据条件启用键绑定

```python
import datetime
from prompt_toolkit import prompt
from prompt_toolkit.filters import Condition
from prompt_toolkit.key_binding import KeyBindings

bindings = KeyBindings()

@Condition
def is_active():
    " Only activate key binding on the second half of each minute. "
    return datetime.datetime.now().second > 30

@bindings.add('c-t', filter=is_active)
def _(event):
    # ...
    pass

prompt('> ', key_bindings=bindings)
```

#### 在Emacs和Vi模式中动态切换

```python
from prompt_toolkit import prompt
from prompt_toolkit.application.current import get_app
from prompt_toolkit.filters import Condition
from prompt_toolkit.key_binding import KeyBindings

def run():
    # Create a set of key bindings.
    bindings = KeyBindings()

    # Add an additional key binding for toggling this flag.
    @bindings.add('f4')
    def _(event):
        " Toggle between Emacs and Vi mode. "
        app = event.app

        if app.editing_mode == EditingMode.VI:
            app.editing_mode = EditingMode.EMACS
        else:
            app.editing_mode = EditingMode.VI

    # Add a toolbar at the bottom to display the current input mode.
    def bottom_toolbar():
        " Display the current input mode. "
        text = 'Vi' if get_app().editing_mode == EditingMode.VI else 'Emacs'
        return [
            ('class:toolbar', ' [F4] %s ' % text)
        ]

    prompt('> ', key_bindings=bindings, bottom_toolbar=bottom_toolbar)

run()
```

#### 使用control-space补全

[link](https://python-prompt-toolkit.readthedocs.io/en/stable/pages/asking_for_input.html#using-control-space-for-completion)

### 其他选项

#### 多行输入

Meta+Enter 或者 Escape，Enter

```python
from prompt_toolkit import prompt

prompt('> ', multiline=True)

def prompt_continuation(width, line_number, is_soft_wrap):
    return '.' * width
    # Or: return [('', '.' * width)]

prompt('multiline input> ', multiline=True,
       prompt_continuation=prompt_continuation)
```

#### 默认值

```prompt('What is your name: ', default='%s' % getpass.getuser())```

#### 鼠标支持

光标位置和滚动

```prompt('What is your name: ', mouse_support=True)```

#### 换行

默认换行

```prompt('What is your name: ', wrap_lines=False)```

#### 密码输入

```python
from prompt_toolkit import prompt

prompt('Enter password: ', is_password=True)
```

### 异步

[link](https://python-prompt-toolkit.readthedocs.io/en/stable/pages/asking_for_input.html#prompt-in-an-asyncio-application)

## Dialogs

### Message box

```python
from prompt_toolkit.shortcuts import message_dialog

message_dialog(
    title='Example dialog window',
    text='Do you want to continue?\nPress ENTER to quit.')
```

### Input box

```python
from prompt_toolkit.shortcuts import input_dialog

text = input_dialog(
    title='Input dialog example',
    text='Please type your password:',
    password=True)
```

### Yes/No dialog

```python
from prompt_toolkit.shortcuts import yes_no_dialog

result = yes_no_dialog(
    title='Yes/No dialog example',
    text='Do you want to confirm?')
```

### Button dialog

```python
from prompt_toolkit.shortcuts import button_dialog

result = button_dialog(
    title='Button dialog example',
    text='Do you want to confirm?',
    buttons=[
        ('Yes', True),
        ('No', False),
        ('Maybe...', None)
    ],
)
```

### dialog 样式

```python
from prompt_toolkit.formatted_text import HTML
from prompt_toolkit.shortcuts import message_dialog
from prompt_toolkit.styles import Style

example_style = Style.from_dict({
    'dialog':             'bg:#88ff88',
    'dialog frame-label': 'bg:#ffffff #000000',
    'dialog.body':        'bg:#000000 #00ff00',
    'dialog shadow':      'bg:#00aa00',
})

message_dialog(
    title=HTML('<style bg="blue" fg="white">Styled</style> '
               '<style fg="ansired">dialog</style> window'),
    text='Do you want to continue?\nPress ENTER to quit.',
    style=example_style)
```

## 进度条

### 简单进度条

```python
from prompt_toolkit.shortcuts import ProgressBar
import time


with ProgressBar() as pb:
    for i in pb(range(800)):
        time.sleep(.01)
```

### 多个并行任务

```python
from prompt_toolkit.shortcuts import ProgressBar
import time
import threading


with ProgressBar() as pb:
    # Two parallel tasks.
    def task_1():
        for i in pb(range(100)):
            time.sleep(.05)

    def task_2():
        for i in pb(range(150)):
            time.sleep(.08)

    # Start threads.
    t1 = threading.Thread(target=task_1)
    t2 = threading.Thread(target=task_2)
    t1.daemon = True
    t2.daemon = True
    t1.start()
    t2.start()

    # Wait for the threads to finish. We use a timeout for the join() call,
    # because on Windows, join cannot be interrupted by Control-C or any other
    # signal.
    for t in [t1, t2]:
        while t.is_alive():
            t.join(timeout=.5)
```

### 增加标题和标签

```python
from prompt_toolkit.shortcuts import ProgressBar
from prompt_toolkit.formatted_text import HTML
import time

title = HTML('Downloading <style bg="yellow" fg="black">4 files...</style>')
label = HTML('<ansired>some file</ansired>: ')

with ProgressBar(title=title) as pb:
    for i in pb(range(800), label=label):
        time.sleep(.01)
```

### 格式化

默认style

```python
from prompt_toolkit.shortcuts.progress_bar.formatters import *

default_formatting = [
    Label(),
    Text(' '),
    Percentage(),
    Text(' '),
    Bar(),
    Text(' '),
    Progress(),
    Text(' '),
    Text('eta [', style='class:time-left'),
    TimeLeft(),
    Text(']', style='class:time-left'),
    Text(' '),
]
```

```python
from prompt_toolkit.shortcuts import ProgressBar
from prompt_toolkit.styles import Style
from prompt_toolkit.shortcuts.progress_bar import formatters
import time

style = Style.from_dict({
    'label': 'bg:#ffff00 #000000',
    'percentage': 'bg:#ffff00 #000000',
    'current': '#448844',
    'bar': '',
})

custom_formatters = [
    formatters.Label(),
    formatters.Text(': [', style='class:percentage'),
    formatters.Percentage(),
    formatters.Text(']', style='class:percentage'),
    formatters.Text(' '),
    formatters.Bar(sym_a='#', sym_b='#', sym_c='.'),
    formatters.Text('  '),
]

with ProgressBar(style=style, formatters=custom_formatters) as pb:
    for i in pb(range(1600), label='Installing'):
        time.sleep(.01)
```

### 增加键绑定和工具栏

```python
from prompt_toolkit import HTML
from prompt_toolkit.key_binding import KeyBindings
from prompt_toolkit.patch_stdout import patch_stdout
from prompt_toolkit.shortcuts import ProgressBar

import time

bottom_toolbar = HTML(' <b>[f]</b> Print "f" <b>[x]</b> Abort.')

# Create custom key bindings first.
kb = KeyBindings()
cancel = [False]

@kb.add('f')
def _(event):
    print('You pressed `f`.')

@kb.add('x')
def _(event):
    " Send Abort (control-c) signal. "
    cancel[0] = True
    os.kill(os.getpid(), signal.SIGINT)

# Use `patch_stdout`, to make sure that prints go above the
# application.
with patch_stdout():
    with ProgressBar(key_bindings=kb, bottom_toolbar=bottom_toolbar) as pb:
        for i in pb(range(800)):
            time.sleep(.01)

            # Stop when the cancel flag has been set.
            if cancel[0]:
                break
```

### 构建全屏应用

[link](https://python-prompt-toolkit.readthedocs.io/en/stable/pages/full_screen_apps.html)