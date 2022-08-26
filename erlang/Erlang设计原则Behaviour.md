---
abbrlink: 2854299904
alias: 2017/11/20/Erlang设计原则Behaviour/index.html
categories:
- erlang
date: '2017-11-20T16:29:01'
description: ''
tags:
- Erlang
- OTP
- Behaviour
- gen_server
- supervisor
title: Erlang设计原则Behaviour
---








标准 Erlang/OTP 行为有

| Behaviour  | 功能               |
| ---------- | ---------------- |
| gen_server | 用于实现 C/S 结构中的服务端 |
| gen_fsm    | 用于实现有限状态机        |
| gen_event  | 用于实现事件处理功能       |
| supervisor | 用于实现监督树中的督程      |
| gen_statem | 新版本中的有限状态机实现     |

平时使用最多的是`gen_server`和`supervisor`

# gen_server

erlang gen_server的使用：以银行账户服务为例

<!--more-->

`bank_account.erl`如下：

```erlang
%%%-------------------------------------------------------------------
%%% @author Flowsnow
%%% @copyright (C) 2017, <COMPANY>
%%% @doc
%%% http://www.kongqingquan.com/archives/403
%%% @end
%%% Created : 13. 十一月 2017 17:19
%%%-------------------------------------------------------------------
-module(bank_account).
-author("Flowsnow").

%% 指定behaviour
-behaviour(gen_server).

%% 回调接口
-export([init/1,
  handle_call/3,
  handle_cast/2,
  handle_info/2,
  code_change/3,
  terminate/2
]).

%% API
-export([start_link/3,
  withdraw/2,
  deposit/2,
  print/1,
  stop/1]).

-record(account,{
  id :: integer(),      %% ID
  name :: string(),     %% 帐号名
  money :: integer()    %% 帐号余额
}).

-define(PRINT(Msg),io:format(Msg ++ "\n")).
-define(PRINT(Format,Msg),io:format(Format ++ "\n",Msg)).


init([Id, Name, Money]) ->
  ?PRINT("bank account init,ID:~w,Name:~p,Money:~w",[Id,Name,Money]),
  State = #account{id = Id, name = Name, money = Money},
  {ok,State}.

%% asynchronous operation
%% handle_cast接收gen_server:cast消息
%% 打印帐号信息
handle_cast(print, State = #account{id = Id, name = Name, money = Money}) ->
  ?PRINT("account info, id: ~w, name: ~p, money: ~w", [Id, Name, Money]),
  {noreply, State};
%% 返回stop，进程将停止，调用terminate
handle_cast(stop, State) ->
  ?PRINT("handle cast stop"),
  {stop, normal, State}.

%% synchronous operation
%% handle_call接收gen_server:call消息
%% 取款
handle_call({withdraw, Num}, _From, State = #account{name = Name,money = Money}) when Num > 0, Num =< Money ->
  NewMoney = Money - Num,
  NewState = State#account{money = NewMoney},
  ?PRINT("~p withdraw:~w,NewMoney:~w",[Name, Num, NewMoney]),
  {reply, true, NewState};
%% 存款
handle_call({deposit, Num}, _From, State = #account{name = Name,money = Money}) ->
  NewMoney = Money + Num,
  NewState = State#account{money = NewMoney},
  ?PRINT("~p withdraw:~w,NewMoney:~w",[Name, Num, NewMoney]),
  {reply, true, NewState}.

%% handle_info，处理直接发给进程的消息
handle_info(Info, State) ->
  ?PRINT("handle_info receive msg:~p",[Info]),
  {noreply,State}.

code_change(_OldVsn, State, _Extra) ->
  {ok,State}.

%% 进程停止时，回调terminate
terminate(Reason, #account{id = ID,name = Name,money = Money}) ->
  ?PRINT("process stop,Reason:~p",[Reason]),
  ?PRINT("account Info,ID:~w,Name:~p,Money:~w",[ID,Name,Money]),
  ok.

%% 开启帐号进程，将回调init/1函数，返回{ok,Pid}
start_link(Id, Name, Money) ->
  gen_server:start_link({local, ?MODULE}, ?MODULE, [Id, Name, Money], []).

withdraw(Pid,Num) ->
  gen_server:call(Pid,{withdraw,Num}).

deposit(Pid,Num) ->
  gen_server:call(Pid,{deposit,Num}).

print(Pid) ->
  gen_server:cast(Pid,print).

stop(Pid) ->
  gen_server:cast(Pid,stop).
```

# supervisor

**SupFlags参数{Type, Times, Sec}**

- Type: 重启策略
  - one_for_one: 一个子进程终止，只重启该进程，在init的时候会启动参数内的子进程
  - simple_one_for_one: 同one_for_one，但是在init的时候不会启动子进程，需要动态调用启动
  - one_for_all: 一个子进程终止，将重启所有子进程
  - rest_for_one: 一个子进程终止，将按顺序重启这个子进程和之后顺序的子进程
- Times: 次数(监控频率)
- Sec: 秒数(监控频率)，如果在Sec秒内重启次数超过Times，则终止所有进程，并终止监控树，将由父进程决定它的命运

**ChildSpec参数{Id, StartFunc, Restart, Shutdown, Type, Modules}**

- Id 子进程ID标识符
- StartFunc = {M, F, A}: 子程序启动入口
- Restart: 重启方案
  - `permanent`: 如果app终止了，整个系统都会停止工作（application:stop/1除外）。
  - `transient`: 如果app以normal的原因终止，没有影响。任何其它终止原因都谁导致整个系统关闭。
  - `temporary`: app可以以任何原因终止。只产生报告，没有其它任何影响。
- Shutdown: 终止策略
  - `brutal_kill`: 无条件终止
  - 超时值(毫秒): 终止时，如果超时，则强制终止
  - `infinity`: 如果子进程是监控树，设置为无限大，等待其终止为止
- Type:
  - `worker`: 普通子进程
  - `supervisor`: 子进程是监控树
- Modules:
  - `dynamic`: 当子进程是gen_event
  - `[Module]`: 当子进程是监控树、gen_server或者gen_fsm，表示回调模块名称

使用宏定义解决重复的ChildSpec指定：

```erlang
-define(CHILD(I, Type), {I, {I, start_link, []}, permanent, 5000, Type, [I]}).
```

**cdn控制服务器的sup模块示例**如下：

```erlang
-module(control_server_sup).

-behaviour(supervisor).
-include("ctrl_internal.hrl").

%% API
-export([start_link/0, start_child/0]).

%% Supervisor callbacks
-export([init/1]).

%% Helper macro for declaring children of supervisor
-define(CHILD(I, Type), {I, {I, start_link, []}, permanent, 5000, Type, [I]}).

%% ===================================================================
%% API functions
%% ===================================================================

start_link() ->
  supervisor:start_link({local, ?MODULE}, ?MODULE, []).

start_child() ->
  supervisor:start_child(?MODULE, ?CHILD(ctrl_concurrency_mgr, worker)),
  supervisor:start_child(?MODULE, ?CHILD(ctrl_hls_writer_mgr, worker)),
  supervisor:start_child(?MODULE, ?CHILD(ctrl_record_writer_mgr, worker)),
  supervisor:start_child(?MODULE, ?CHILD(ctrl_live_mgr, worker)),
  supervisor:start_child(?MODULE, ?CHILD(ctrl_live_analyser_mgr, worker)).

%% ===================================================================
%% Supervisor callbacks
%% ===================================================================

init([]) ->
  timer:apply_after(1000, control_server_sup, start_child, []),
  {ok, ErlOdbcConnectStr} = application:get_env(erl_odbc_connect_str),
  {ok, _} = qk_odbc_pool:start_odbc_pool(?ERL_ODBC_POOL_ID, ErlOdbcConnectStr, 1),
  Children = [
    ?CHILD(ctrl_cluster, worker),
    ?CHILD(ctrl_plan, worker),
    ?CHILD(ctrl_amqp_agent, worker)
  ],
  {ok, {{one_for_one, 10, 10}, Children}}.


```

start_child动态添加子进程和start_link添加的监控树的区别在于：监控树退出并重启后，动态添加的子进程会丢失。

---

参考：

1. [OTP Design Principles: Gen_Server Behaviour](http://erlang.group.iteye.com/group/wiki/1451-otp-design-principles-gen_server-behaviour)
2. [Erlang-Supervisor Behaviour](https://www.erlang.org/doc/design_principles/sup_princ.html)
3. [OTP Design Principles: Supervisor Behaviour](http://erlang.group.iteye.com/group/wiki/1454-otp-design-principles-supervisor-behaviour)
4. [erlang supervisor(监控树)的重启策略](https://www.cnblogs.com/rond/p/6234765.html)
5. [OTP设计原则——第三部分](http://www.0x01f.com/post/OTP_Design_Principle_3/)

