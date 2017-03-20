---
layout: post
category: Linux
title: 《shell判断是否为数字的几种方法》
tagline: by Vv
tags: Linux 
---

## 目录

> 1.通过expr 计算变量与一个整数值相加，如果能正常执行则为整数，否则执行出错,$?将是非0的值
>
> 2.打印变量通过sed替换的方式，将变量中的数字替换为空，如果执行替换后变量为空，则为整数



## 内容

### 1.通过expr 计算变量与一个整数值相加，如果能正常执行则为整数，否则执行出错,$?将是非0的值
```
expr $args + 0 &>/dev/null
```
事例：
```
checkInt(){
        expr $1 + 0 &>/dev/null
        [ $? -ne 0 ] && { echo "Args must be integer!";exit 1; }
}
```

###  2.打印变量通过sed替换的方式，将变量中的数字替换为空，如果执行替换后变量为空，则为整数
```
echo $args | sed 's/[0-9]//g'
```
事例：
```
checkInt(){
        tmp=`echo $1 |sed 's/[0-9]//g'`
        [ -n "${tmp}" ]&& { echo "Args must be integer!";exit 1; }
}
```

### 3.通过正则表达式验证

```
read var
if [[ $var =~ ^[0-9]+$ ]]
then
    echo "Number."
elif [[ $var =~ ^[A-Za-z]+$ ]]
then
    echo "String."
else
    echo "mixed number and string or others "
fi  
    
```
