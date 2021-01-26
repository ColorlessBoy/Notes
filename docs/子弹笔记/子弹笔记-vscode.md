# 子弹笔记-Vscode

## vscode对markdown文件进行自动换行

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

## C++头文件

1. 使用`g++`查看相关头文件的位置：

    ```bash
    g++ -v -E -x c++ -
    ```

    一般会有这些头文件

    ```bash
    /usr/include/c++/9
    /usr/include/x86_64-linux-gnu/c++/9
    /usr/include/c++/9/backward
    /usr/lib/gcc/x86_64-linux-gnu/9/include
    /usr/local/include
    /usr/include/x86_64-linux-gnu
    /usr/include
    ```

2. 用`vscode`打开项目文件夹，快捷键`ctrl+shift+p`搜索`C/C++`相关的`JSON`属性文件，把上面找到的头文件路径添加到属性文件中。
   
    ```json
    {
        "configurations": [
            {
                "name": "Linux",
                "includePath": [
                    "${workspaceFolder}/**",
                    "/usr/include/c++/9",
                    "/usr/include/x86_64-linux-gnu/c++/9",
                    "/usr/include/c++/9/backward",
                    "/usr/lib/gcc/x86_64-linux-gnu/9/include",
                    "/usr/local/include",
                    "/usr/include/x86_64-linux-gnu",
                    "/usr/include"
                ],
                "defines": [],
                "cStandard": "gnu11",
                "cppStandard": "c++17",
                "intelliSenseMode": "linux-gcc-x64"
            }
        ],
        "version": 4
    }
    ```