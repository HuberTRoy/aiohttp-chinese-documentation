# 贡献须知
## 贡献者说明
首先需要clone GitHub上的仓库: 打开链接，并位于右上角的点击“Fork”按钮。
:) 我想应该所有人都会使用git和github吧~。

之后要做的步骤很清晰:
1. clone这个GitHub仓库。
2. 进行修改。
3. 确保所有代码测试通过。
4. 在CHANGES文件夹中添加一个说明文件（用于更新Changelog）。
5. 提交到自己的aiohttp仓库。
6. 发起一个关于你的修改的PR。

### 注意
    如果你的PR有很长历史记录或者很多提交，请在创建PR之前重新从主仓库创建一次，确保它没有这些记录。


## 运行aiohttp测试组件的准备
我们建议你使用pyhton虚拟环境来运行我们的测试。
关于创建虚拟环境有以下几种方法:

如果你喜欢使用virtualenv，可以这样用:
```
$ cd aiohttp
$ virtualenv --python=`which python3` venv
$ . venv/bin/activate
```
或者使用python 标准库中的venv:
```
$ cd aiohttp
$ python3 -m venv venv
$ . venv/bin/activate
```
还可以使用virtualenvwarapper:
```
$ cd aiohttp
$ mkvirtualenv --python=`which python3` aiohttp
```
还有其他可用的工具比如`pyvenv`，不过我们只是要让你知道: 需要创建一个Python3虚拟环境并使用它。

弄完之后我们要安装开发所需的包:
```
$ pip install -r requirements/dev.txt
```
### 注意
    如果你计划在测试组件中使用pdb或ipdb，执行:
    ```
      $ py.test tests -s
    ```
    使用命令禁止输出捕获。
恭喜，到这一步我们就准备好我们的测试组件了。

##  运行aiohttp测试组件
全部准备完之后，是时候运行了:
```
$ make test
```
第一次使用命令运行时会运行`flask8`工具（抱歉，我们不接受关于pep8和pyflakes相关错误的PR）。
`flake8`成功运行后，测试也会随之运行。
请稍微注意下输出的语句。
任何额外的文本信息（打印的条款等）都会被删除。

## 测是覆盖
我们正尽可能地有一个好的测试覆盖率，请不要把它变得更糟。
运行下这个命令:
```
$ make cov
```
来加载测试组件并收集覆盖信息。一旦命令执行完成，会在最后一行显示如下输出:`open file:///.../aiohttp/coverage/index.html`
请看一下这个链接并确保你的修改已被覆盖到。
aiohttp使用`codecov.io`来存储覆盖结果。 你可以去这个页面查看: https://codecov.io/gh/aio-libs/aiohttp 其他详细信息。
强烈推荐浏览器扩展`https://docs.codecov.io/docs/browser-extension`，仅在GitHub Pull Request上的"已修改文件"标签上就能做分析。

## 文档
我们鼓励完善文档。
发起一个关于修改文档的PR时请先运行下这个命令:
```
$ make doc
```
完成后会输出一个索引页面: `file:///.../aiohttp/docs/_build/html/index.html`
请看一下，确保样式没问题。

## 拼写检测
我们使用`pyenchant`和`sphinxcontrib-spelling`检测文档中的拼写:
```
$ make doc-spelling
```
不幸的是，在MacOS X上会有些问题。
在Linux上运行拼写检测前要先安装下:
```
$ sudo apt-get install enchant
$ pip install sphinxcontrib-spelling
```

## 更新 修改日志
CHANGES.rst文件使用`towncrier`工具管理，所有重要的修改都要有一个条目。
要为新文件增加一个条目，首要需要创建一个关于你想怎么做的issue。一个PR本身的功能也和这个一样，但有一个正经关于此修改的issue更好（举个例子，万一因为代码质量问题驳回了PR呢...）。

一旦你有了一个issue或PR，你会得到一个号码并在CHANGES/目录内以此issue的号码和其扩展内容如 `.removal, .feature, .bugfix, .doc`创建一个文件。比如你的issue或PR号码是1234，是关于修复bug的，这样就会创建一个CHANGES/1234.bugfixs的文件。PR们可以创建多个类别的说明文件（比如，你添加了一个新功能，并且要删除或不在赞成使用某个旧功能，那就创建CHANGES/NNNN.feature和CHANGES/NNNN.removal）。同样地，如果一个PR涉及到多个issues/PR你可以为它们每个都创建相同的内容，Towncrier会删除重复的部分。

文件的内容使用`reStruredText`格式化内容，格式化后的内容会作为新文件条目来使用。你不需要为issue或PR添加关联，towncrier会自动为所有涉及到的issues添加关联。

## 最后
做完这些之后，请在GitHub上发起PR，谢谢~。





