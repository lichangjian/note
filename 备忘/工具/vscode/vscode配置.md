# 配置 C 语言开发环境
官方配置文档： https://code.visualstudio.com/docs/languages/cpp

VS Code 只是一个编辑器，并没有提供编译器，所以 Win 环境下还需要下载 MinGW。
下载C/C++插件，有提供代码补全，Debug等工具。

在 .vscode 文件夹中，添加3个文件，文件可以由编辑器自动生成，也可以去官方文档里拷贝。
c_cpp_properties.json  配置 include 目录，宏，以及项目配置。

task.json 配置编译任务，生成最终build文件。如果要调试需要加入编译参数 -g

launch.json Debug配置，配置调试器和调试文件等。"preLaunchTask": "build" 可以在Debug之前先执行build任务 。

# 内置参数
* ${workspaceRoot} the path of the folder opened in VS Code(VSCode中打开文件夹的路径)
* ${workspaceRootFolderName} the name of the folder opened in VS Code without any solidus (/)(VSCode中打开文件夹的路径, 但不包含"/")
* ${file} the current opened file(当前打开的文件)
* ${relativeFile} the current opened file relative to workspaceRoot(当前打开的文件,相对于workspaceRoot)
* ${fileBasename} the current opened file's basename(当前打开文件的文件名, 不含扩展名)
* ${fileDirname} the current opened file's dirname(当前打开文件的目录名)
* ${fileExtname} the current opened file's extension(当前打开文件的扩展名)
* ${cwd} the task runner's current working directory on startup()
