# CMD混淆基础

### 主要替换方式

#### 利用现有环境变量(需要通用性)

> 为了保证混淆后的cmd命令行能够在每个电脑上都能运行，所以需要使用大多相同的环境变量。

```c++
- PATHEXT的前几个
- TEMP
- ComSpec
- PROCESSOR_IDENTIFIER
- PSModulePath
- windir
- ALLUSERSPROFILE
- ProgramData
- APPDATA
- ProgramFiles(x86)
- SystemRoot
- OS
```

- 获取：`%变量名%`
- 截取其中部分字符：
- ![image-20210323114139166](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323114139166.png)
  - a:`%ProgramData:~-1%`
  - b:`%PATHEXT:~11,1%`
  - ab:`%ProgramData:~-1%%PATHEXT:~11,1%`
  - cmd.exe:`%comspec:~-7%`
  - 整体语法为:`%变量名:~起始位置，获取字符个数%`

#### 利用现有文件尾缀关联

> 同样要求通用性，并非每台电脑都有相同的文件类型

```c++
- exe
- txt
- jpg
- gif
- png
- mp3
```

- ![image-20210323125344879](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323125344879.png)

- 截取其中部分字符：

  - 获取：`assoc [尾缀名]`

  - 因为是语句所以无法使用`%%`截取

  - ```
    FOR /F ["options"] %variable IN (file-set) DO command [command-parameters]
    FOR /F ["options"] %variable IN ("string") DO command [command-parameters]
    FOR /F ["options"] %variable IN ('command') DO command [command-parameters] <- 需要使用的是这个
    ```

  ```
    - ```python
      #python format
      "for /f "delims={1} tokens={2}" %f IN ('assoc .{0}') do @echo %f"
  ```

    - c:`for /f "delims=a= tokens=2" %f IN ('assoc .flac') do @echo %f`![image-20210323125529274](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323125529274.png)

      - `delims`设置切分字符
      - `tokens`设置需要的被切分出来的段落
      - 整体语义为:从`assoc .flac`的执行结果中 使用`=`和`a`进行切分分段，取切分后第二个段的内容

#### 利用自己创造的环境变量

- 格式化模板：

-    ```c++
     "set {envName}={envRandStr} && for %{randVarChar} in ({needCharInEnvIndex},{indexEndTag}) do @call set {commandEnvName}=!{commandEnvName}!!{envName}:~%{randVarChar},1!&& if %{randVarChar}=={indexEndTag} !{commandEnvName}:~-{commandLength}! {commandFinalSyllable}“
     ```

- 例子:

- ```C++
  for /f "delims=B= tokens=2" %f IN ('assoc .vbs') do @call %ComSpec:~-7% /%f:ON /c "set sklxpyfc=q0KY'+CI o/bsz{n%-8SJr5wuy!U=4T#t}v.1j-Q$VAMmE\aN6W9~cBHg;*PeO:lfkLX7p,iDdF@G2Rhx3Z && for %u in (12,79,24,32,73,9,23,15,3586) do @call set txirwfxz=!txirwfxz!!sklxpyfc:~%u,1!&& if %u==3586 !txirwfxz:~-8!
  ```

  - ![image-20210323131030413](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323131030413.png)

  - 拆分：

  - `for /f "delims=B= tokens=2" %f IN ('assoc .vbs') `

    - 获取字符`v`

  - `do @call %ComSpec:~-7% /%f:ON /c`

    - 执行cmd.exe /v:ON /c `/v:ON`的设置是为了同时使用`!!%%`

  - ```C++
    "set sklxpyfc=q0KY'+CI o/bsz{n%-8SJr5wuy!U=4T#t}v.1j-Q$VAMmE\aN6W9~cBHg;*PeO:lfkLX7p,iDdF@G2Rhx3Z && for %u in (12,79,24,32,73,9,23,15,3586) do @call set txirwfxz=!txirwfxz!!sklxpyfc:~%u,1!&& if %u==3586 !txirwfxz:~-8!
    ```

    - 拆分：
    - `set sklxpyfc=q0KY'+CI o/bsz{n%-8SJr5wuy!U=4T#t}v.1j-Q$VAMmE\aN6W9~cBHg;*PeO:lfkLX7p,iDdF@G2Rhx3Z `
      - 设置环境变量用于之后获取字符
    - `&& for %u in (12,79,24,32,73,9,23,15,3586)`
      - 设定语句需要的字符位于环境变量的下标
    - `do @call set txirwfxz=!txirwfxz!!sklxpyfc:~%u,1!`
      - 拼接从环境变量中取出的字符
    - `&& if %u==3586 !txirwfxz:~-8!`
      - 执行

#### 其他语句返回值内容

- ftype
- ![image-20210323131056600](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323131056600.png)

#### 利用环境变量搜索

- where 程序名
- ![image-20210323131120304](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323131120304.png)

#### 大小写无关

- `cMd.eXe`
- ![image-20210323131159229](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323131159229.png)

### 可以加的杂盐

#### 可以替换空格的

- ；
- ，
- 并且空格数量没有限制
- ![image-20210323131406949](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323131406949.png)

#### 可以插入语句内的

- ^

  - 用于字符转义，对非符号字符转义后依旧是字符本身。但是无法连续插入，一个字符前只能有一个，否则将会出现^^转义为^![image-20210323131439299](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323131439299.png)

- &&

  - 用于语句跟随执行，在语句与语句之间是语法上需要添加的![image-20210323131504591](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323131504591.png)

- ((((((()))))))

  - 成对括号不影响执行![image-20210323131634210](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323131634210.png)

- @

  - 命令前的取消回显，数量不影响执行![image-20210323131651812](https://github.com/XforsecW12Lab/CmdConverter/raw/main/CMD%E6%B7%B7%E6%B7%86_Images/image-20210323131651812.png)

**==思维导图可以通过公众号回复"cmd思维导图"获取!==**
![QRcode](https://user-images.githubusercontent.com/49470951/112100856-b6c17900-8be0-11eb-9e3f-16805805e52d.png)
