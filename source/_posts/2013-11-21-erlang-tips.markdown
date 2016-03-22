---
layout: post
title: "Erlang 的一些好玩 Tips"
date: 2013-11-21 11:36:18 +0800
comments: true
categories: erlang
---
可以模拟OO, 模块级别的继承.

    -extends(parent).

反编译 beam, 对加密过或者 hipe 的似乎无效.

```erlang
{ok,{_,[{abstract_code,{_,AC}}]}} = beam_lib:chunks(Beam,[abstract_code]).
io:fwrite("~s~n", [erl_prettypr:format(erl_syntax:form_list(AC))]).
```

语句块, 比较少用到.

```erlang
begin
   ...
end
```

Parameterised Modules 参数化模块, 也是模拟 OO 的.
很多 web 框架都这么用, 不过这货在 R16 被去掉了, 也是... 其实挺恶心.

    -module(xxx, [Arg1, Arg2]).

Stateful Modules, 用于替换 Parameterised Modules. 将模块和对应状态保存在 tuple 里,
看起来像是调用某一对象的方法一样.

```
X = {Mod, P1, P2, ..., Pn},
X:Func(A1, A2, ..., An).
%% 等价于
Mod:Func(A1, A2, ..., An, X).
```

所以 web 框架就转为使用这个特性了, 看起来很无害而且不反 FP.
