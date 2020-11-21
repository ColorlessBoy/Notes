# vscode对markdown文件进行自动换行

vscode设置完自动换行后，markdown文件没办法自动换行。这个问题搞了我很久，中文搜索不出结果，最后还是从google搜索出来解决办法。

对markdown文件需要专门的配置：

```json
{
    // ...
    "[markdown]": {
        "editor.wordWrap": "wordWrapColumn",
        "editor.wordWrapColumn": 80,
    }
    // ...
}
```