# 文档参考 #
文档地址：http://www.cnblogs.com/kingthy/archive/2009/08/17/net-vtemplate.html


# <h3>3.7版本更新日志</h3> #
1、修改CurrentRenderingDocument与CurrentRenderingTag的实现方式，以便VT在低安全信用级别下运行时也能正常运行。

2、变量标签，可以简化为“{$变量标识}”模式，即可以不输入“:”，如以下的模板代码是等同的

```
用户名{$:user.name}与{$user.name}
```

注：此版本以上，建议采用此新方式编写变量标签。


3、变量属性(var与index等)可以增加“$”字符。如以下的模板代码是等同的

```
<vt:foreach from="$items" item="item" index="i">
....
</vt:foreach>
```

此版本后也可以写为如下：

```
<vt:foreach from="$items" item="$item" index="$i">
....
</vt:foreach>
```


注：此修改是为了避免某些朋友在此版本以前常犯错误而加的兼容。


4、`<vt:if>`里的compare里增加三种条件判断方式

A、 ^=  : 判断是否以某些值开始

B、 $=  : 判断是否以某些值结束

C、 `*=`  : 判断是否包含有某些值



注意：此三种条件判断方式都以“字符串”形式处理，并不区分大小区比较




5、增加`<vt:panel>`标签，用于方便实现“母版页”(MasterPage）的功能。


`<vt:panel id="">`：定义容器面板标签，用于设置一个容器位。

`<vt:panel container="容器标签的id">`：内容面板标签，此标签内的内容将在container定义的panel标签里输出。


注：此标签必须定义id或者container属性，或者同时定义。定义了id属性则表示是一个容器面板标签，定义了container属性则表示是一个内容面板标签。


模板示例可以参考WebTester项目里的“panel\_tag\_test.ashx”文件



# <h3>3.6版本更新日志</h3> #

1、添加特殊的VT属性标签，属性标签格式定义为: vt='<vt:标签名.......' 或vt="<vt:标签名........"

其中这里的“<vt:标签名”可以为任意标签，如果“vt=”后面跟随的是“单引号”则标签里的数据都不能出现“单引号”否则是看作是属性标签的结束，后跟随“双引号”同理。


例子：
```
<select name="Country">
<option value="中国" vt="<vt:if var='#.Country' value='中国'>selected='selected'</vt:if>'>中国</option>
<option value="美国" vt="<vt:if var='#.Country' value='美国'>selected='selected'</vt:if>'>中美国</option>
</select>
```

在上面例子中“vt=`<vt:if......</vt:if>`'”这一个标签就是VT属性标签。当上面的#.Country值等于“中国”时，上面的模板将最终解析为以下值：

```
<select name="Country">
<option value="中国" selected='selected'>中国</option>
<option value="美国">中美国</option>
</select>
```

2、修复CurrentRenderingDocument与CurrentRenderingTag在多线程下会出乱(错)的BUG.


# <h3>3.4版本更新日志</h3> #

1、修正单标签的子元素解析错位的BUG！

例子：
```
    用户：
    <vt:if var="user.age" value="20">
       年龄处于青年
    <vt:else />
        <vt:if var="user.name" value="张三">(特殊人物）</vt:if>
        <vt:if var="user.age" value="20" compare=">">年龄处于中年</vt:if>
    </vt:if>
```

在上面模板中
```
<vt:if var="user.age" value="20" compare=">">年龄处于中年</vt:if>
```
将导致解析错位，最终导致输出结果有误。


2、改进`<vt:if>`,`<vt:elseif>`标签的条件判断。

例子：
```
    <vt:for form="1" to="10" index="i">
      <vt:if var="i" value="5" compare="<">小于5<vt:else />大于等于5</vt:if>
    </vt:for>
```
以上模板在以前版本中将会导致全部输出“大于等于5“，需要输出正确的答案就要将if标签的判断变量表达式改为"i.value“，新版本则可以直接进行判断。


3、修改在解析模板时,如果出错抛出异常时则输出解析出错的模板文件名.

> 以前版本如果模板解析错误时，只有简单的输出错误所在的行列号与相关模板文本，但却无法得知是哪个文件的模板文本，导致调试非常不方便。新版本则一并输出解析错误时所在的文件地址（如果存在的话）


4、限制`<vt:foreachelse>`,`<vt:else>`标签在某个父节点(`<vt:foreach>`,`<vt:if>`)中只允许定义一次.

5、标签的属性值允许定义变量表达式(以$字符开头).除了id, name, var、output属性和以下特殊的标签属性.
> A. `<vt:template>`与`<vt:include>`标签的file,charset属性

> B. `<vt:foreach>`标签的item,index属性

> C. `<vt:for>`标签的index属性

例子：
  1. 、`<vt:foreach>`标签中的groupsize使用变量表达式处理分组的大小。
```
    <vt:foreach form="$users" item="usergroup" index="i" groupsize="$users.groupsize">
      <vt:foreach from="$usergroup" item="user">
            ............(模板代码).........
      </vt:foreach>
    </vt:foreach>   
```
> 以上模板中，更改users.groupsize的值即可以更改输出数据的不同。

> 2)、自定义属性中定义变量表达式，在程序中则可以通过属性获取值。
```
    <vt:function var="GetUserProfile" method="getuserprofile" type="$this" userid="$user.id" />
```

```
        /// <summary>
        /// 获取用户的资料
        /// </summary>
        /// <returns></returns>
        public object GetUserProfile()
        {
            Tag tag = this.Document.CurrentRenderingTag;

            //获取自定义属性
            var attribute = tag.Attributes["userid"];
            if (attribute == null) return null;

            //获取自定义属性的值
            int userId = Convert.ToInt32(attribute.Value.GetValue());

            //取得了用户的userid，所以后面可以通过其它方式取得对应用户的资料。
        }
```
> 3）、变量标签中的属性定义变量表达。
```
    描述说明:{$:user.description htmlencode="$user.needencode"}
```
> htmlencode的值已绑定到user.needencode的值，所以输出时是否需要编码是根据needencode的值而定义。


6、去除变量表达式的值缓存功能。
> 变量表达式的值缓存功能是为了提高处理速度，但在某些情况下(如方法内根据调用的标签属性值不同返回不同的数据）却会导致无法获取最新的正常的值（永远获取的都是第一次缓存的值），所以新版本已将此值缓存功能去除。

7、增加兼容模式(TemplateDocumentConfig.CompatibleMode)
> 3.4版本中某些标签的属性定义已定为“变量表达式”，但在新版本却已定义为“表达式”（也即可以为“常量表达式”或“变量表达式”（以$字符开头），而如果旧版本的模板不更改直接用3.4版本的引擎解析就会导致那些属性的值变成了“常量表达式”，从而导致解析错误或结果输出错误。

  1. 、`<vt:foreach>`标签的from属性,如：from="users"。如果不开启兼容模式，则新版本中必须定义为以“$”字符开头的变量表达式，如from="$users“，否则导致解析出错。

> 2)、`<vt:expression>`的args属性，如: args="i"，如果不开启兼容模式，则新版本中必须定义为以“$"字符开头的变量表达式，如args="$i“，否则将直接按“常量表达式”处理！

> 3)、`<vt:datareader>`的parameters属性，如: parameters="name"，如果不开启兼容模式，则新版本中必须定义为以“$"字符开头的变量表达式，如parameters="$name“，否则将直接按“常量表达式”处理！

# <h3>3.3版本更新日志</h3> #
此版本主要增加两个属性，用于处理模板引擎在呈现（解析）数据时的“上下文”对象，方便程序处理数据。

1、TemplateDocument增加CurrentRenderingTag实例属性，用于返回当前正在呈现数据的标签对象

例子：
```
   程序：
    Tag tag = this.Document.CurrentRenderingTag;

```
2、TemplateDocument增加CurrentRenderingDocument静态属性，用于返回当前正在呈现数据的模版文档对象
例子：
```
   程序：
   TemplateDocument document = TemplateDocument.CurrentRenderingDocument;
```

关于这两个属性的具体作用可参考示例项目中的“currentrendertag\_test.ashx”范例.

# <h3>3.2版本更新日志</h3> #

1、修正变量标签在某种情况下不能获取值的BUG。

例子：
```
   程序：
    document.SetValue("Member.Location.City","广州")
    
   模版：
   所在城市{$:Member.Location.City}

```
在上面例子中，如果Member.Location的值等于null，那么模版里则是获取不了所在城市的。

2、`<vt:function>`标签更改为不管是否定义output属性都可以不定义var属性。
> 这样更改是为方便使用此标签调用没有返回值的函数方法。

# <h3>3.1版本更新日志</h3> #

1、增加`<vt:import>`类型导入标签。

2、`<vt:datareader>`标签增加CommandType与ParameterFormat属性

3、`<vt:serverdata>`标签的Type属性增加AppSetting类型，以便可在模版里访问读取AppSettings配置参数

4、修改不可重复注册“用户自定义函数”的BUG。


# <h3>3.0版本更新日志</h3> #

1、增加“变量标识”概念（此变更导致部分标签的属性语法与2.X版本不兼容）。

2、增加`<vt:output>`数据输出标签。

3、去除VariableFunction函数原型，更改为UserDefinedFunction函数原型。

4、增强`<vt:function>`标签的功能，当未定义type属性时调用已注册的UserDefinedFunction。

5、变量标签增加RemoveHtml属性。

6、去除SetExpValue方法，将其功能实现并入SetValue方法。

7、其它细节修改……

# <h3>3.0版本与2.X版本的不兼容性</h3> #

## <h4>1。部分标签中的属性的定义语法改变：</h4> ##

受影响的标签与属性有:<br />
`<vt:for>`标签的index属性.<br />
`<vt:foreach>`标签的item,index属性<br />
`<vt:function>`标签的var属性<br />
`<vt:property>`标签的var属性<br />
`<vt:serverdata>`标签的var属性<br />
`<vt:expression>`标签的var属性<br />
`<vt:datareader>`标签的var属性<br />
`<vt:set>`标签的var属性<br />

在2.X版本中上面标签的属性只允许定义“变量名”，并且定义的“变量”归属于当前模版块，如以下VT模版：

```
<vt:template id="my">
<vt:foreach from="users" item="user" index="i">
{$:#.i}、{$:#.user.name}的年龄是{$:#.user.age}岁。
</vt:foreach>
</vt:template>
```
<br />
以上代码中。foreach标签的item与index属性只允许定义“变量名”（也即“user”与“i”）,并且定义的变量只归属当前模版块,也即“user”、“i”都属于my这个模版块。

但在3.0版本（以上）中，上面标签的属性允许定义的是“变量名”（不带“变量前缀”的变量标识）或“变量标识”。 “变量标识”是由“变量前缀”与“变量名”组成。“变量标识”的概念就是为了解决2.X版本以前语法混乱（从上面VT模版可看到item这里定义的是user，但下面定义的变量标签就采用了{$:#.user.name}）而新加入的。

如上面VT模版不变，则foreach标签的item与index属性定义的“变量”，不归属于my模版块，而是归属于“根模版块”！！上面VT模版在3.0版本（以上）就需要改为如下：
<br />
```
<vt:template id="my">
<vt:foreach from="users" item="#.user" index="#.i">
{$:#.i}、{$:#.user.name}的年龄是{$:#.user.age}岁。
</vt:foreach>
</vt:template>
```



## <h4>2。VariableFunction函数原型已去除。</h4> ##

在3.0版本（以上）中已去除VariableFunction函数原型，改用UserDefinedFunction函数原型代替。它们之间的差别是VariableFunction函数原型只接受一个object参数，而UserDefinedFunction函数原型接受的是一个object[.md](.md) 数组参数！

如下代码：

2.0版本：
```

        /// <summary>
        /// 获取年龄的说明
         /// </summary>
        /// <param name="age"></param>
        /// <returns></returns>
        private object GetAgeRemark(object age)
        {
            string remark = "未知";
            if (age != null && age != DBNull.Value && age is int)
            {
                int a = (int)age;
                if (a < 30)
                {
                    remark = "青年";
                }
                else if (a < 50)
                {
                    remark = "中年";
                }
                else
                {
                    remark = "老年";
                }
            }
            return remark;
        }

```

3.0版本：

```
        /// <summary>
        /// 获取年龄的说明
         /// </summary>
        /// <param name="age"></param>
        /// <returns></returns>
        private object GetAgeRemark(object[] ages)
        {
            string remark = "未知";
            object age = ages[0];
            if (age != null && age != DBNull.Value && age is int)
            {
                int a = (int)age;
                if (a < 30)
                {
                    remark = "青年";
                }
                else if (a < 50)
                {
                    remark = "中年";
                }
                else
                {
                    remark = "老年";
                }
            }
            return remark;
        }

```


## <h4>3。Template或VariableCollection类中的SetExpValue函数方法已去除。</h4> ##
在3.0版本（以上）中已去除Template或VariableCollection类中的SetExpValue函数方法，其功能实现已合并入SetValue方法。

如下代码：

2.0版本：<br />
```
template.setValue("course",user.course);
template.SetExpValue("user.age", 19);
template.SetExpValue("user.location.city", "广州");
```

3.0版本：<br />
```
template.setValue("course",user.course);
template.setValue("user.age", 19);
template.setValue("user.location.city", "广州");
```