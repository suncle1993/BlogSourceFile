---
categories:
  - 工程实践
date: '2018-03-24T19:46:25'
description: ''
tags:
  - BFS
  - 剪枝
  - 搜索
title: BFS+剪枝查找目标转推流节点
---




需求：在各个国家都有可能部署转推流节点，因此需要高效快捷的查找到离推理地点最近的一个目标转推流节点。

现状：国内以省为单位，国外以国家为单位，虽然当前节点较少，但是为了保障业务拓展之后的效果，因此需要及时进行图优化。

分析：建立中国地图和世界地图，根据ip地址在ip数据库中查找，得到ip所属的国家名称，国家代码，省份名称，省份代码。用国家代码在世界地图中查找最近的国家节点，用省份代码在中国地图中查找最近的省份节点。

搜索：搜索方式为广度优先搜索BFS，用于寻找最近的目标点。

<!--more-->

BFS+剪枝实现的中国地图和世界地图中查找目标转推流节点的代码如下：

```erlang
%%%-------------------------------------------------------------------
%%% @author ChenLiang
%%% @copyright (C) 2018, <COMPANY>
%%% @doc
%%%
%%% @end
%%% Created : 2018/03/20 14:59
%%%-------------------------------------------------------------------
-module(detonate_deliver_site_graph).
-author("ChenLiang").

-behaviour(gen_server).
-include("detonate_server.hrl").

%% API
-export([start_link/0, lookup_deliver_node/2]).

%% gen_server callbacks
-export([init/1,
  handle_call/3,
  handle_cast/2,
  handle_info/2,
  terminate/2,
  code_change/3]).

-define(SERVER, ?MODULE).

-record(state, {
  china_graph = digraph:new() :: digraph:graph(),
  china_tid = ets:new(undefined, []) :: ets:tid(),
  world_graph = digraph:new() :: digraph:graph(),
  world_tid = ets:new(undefined, []) :: ets:tid()
  }
).

%%%===================================================================
%%% API
%%%===================================================================

%%--------------------------------------------------------------------
%% @doc
%% Starts the server
%%
%% @end
%%--------------------------------------------------------------------
-spec(start_link() ->
  {ok, Pid :: pid()} | ignore | {error, Reason :: term()}).
start_link() ->
  gen_server:start_link({local, ?SERVER}, ?MODULE, [], []).

lookup_deliver_node(lookup_province, Province) ->
  lager:debug("lookup_province: ~p ~n", [Province]),
  gen_server:call(?MODULE, {lookup_province, Province});
lookup_deliver_node(lookup_country, CountryCode) ->
  lager:debug("lookup_country: ~p ~n", [CountryCode]),
  gen_server:call(?MODULE, {lookup_country, CountryCode}).

%%%===================================================================
%%% gen_server callbacks
%%%===================================================================

%%--------------------------------------------------------------------
%% @private
%% @doc
%% Initializes the server
%%
%% @spec init(Args) -> {ok, State} |
%%                     {ok, State, Timeout} |
%%                     ignore |
%%                     {stop, Reason}
%% @end
%%--------------------------------------------------------------------
-spec(init(Args :: term()) ->
  {ok, State :: #state{}} | {ok, State :: #state{}, timeout() | hibernate} |
  {stop, Reason :: term()} | ignore).
init([]) ->
  gen_server:cast(?MODULE, init_china),
  gen_server:cast(?MODULE, init_world),
  {ok, #state{}}.

%%--------------------------------------------------------------------
%% @private
%% @doc
%% Handling call messages
%%
%% @end
%%--------------------------------------------------------------------
-spec(handle_call(Request :: term(), From :: {pid(), Tag :: term()},
    State :: #state{}) ->
  {reply, Reply :: term(), NewState :: #state{}} |
  {reply, Reply :: term(), NewState :: #state{}, timeout() | hibernate} |
  {noreply, NewState :: #state{}} |
  {noreply, NewState :: #state{}, timeout() | hibernate} |
  {stop, Reason :: term(), Reply :: term(), NewState :: #state{}} |
  {stop, Reason :: term(), NewState :: #state{}}).
handle_call({lookup_province, Province}, _From, State=#state{china_graph = ChinaGraph, china_tid = ChinaTid}) ->
  Reply = lookup_deliver_node(lookup_province, ChinaGraph, ChinaTid, Province),
  {reply, Reply, State};
handle_call({lookup_country, CountryCode}, _From, State=#state{world_graph = WorldGraph, world_tid = WorldTid}) ->
  Reply = lookup_deliver_node(lookup_country, WorldGraph, WorldTid, CountryCode),
  {reply, Reply, State};
handle_call(_Request, _From, State) ->
  {reply, ok, State}.

%%--------------------------------------------------------------------
%% @private
%% @doc
%% Handling cast messages
%%
%% @end
%%--------------------------------------------------------------------
-spec(handle_cast(Request :: term(), State :: #state{}) ->
  {noreply, NewState :: #state{}} |
  {noreply, NewState :: #state{}, timeout() | hibernate} |
  {stop, Reason :: term(), NewState :: #state{}}).
handle_cast(init_china, State=#state{china_graph = ChinaGraph, china_tid = ChinaTid}) ->
  ChinaVertexFileName = filename:join(code:lib_dir(detonate_server, priv), "china_province_code.csv"),
  ChinaEdgeFileName = filename:join(code:lib_dir(detonate_server, priv), "china_province_border.csv"),
  initialize_graph(china, ChinaGraph, ChinaTid, ChinaVertexFileName, ChinaEdgeFileName),
  {noreply, State};
handle_cast(init_world, State=#state{world_graph = WorldGraph, world_tid = WorldTid}) ->
  WorldVertexFileName = filename:join(code:lib_dir(detonate_server, priv), "world_country_code.csv"),
  WorldEdgeFileName = filename:join(code:lib_dir(detonate_server, priv), "world_country_border.csv"),
  initialize_graph(world, WorldGraph, WorldTid, WorldVertexFileName, WorldEdgeFileName),
  {noreply, State};
handle_cast(_Request, State) ->
  {noreply, State}.

%%--------------------------------------------------------------------
%% @private
%% @doc
%% Handling all non call/cast messages
%%
%% @spec handle_info(Info, State) -> {noreply, State} |
%%                                   {noreply, State, Timeout} |
%%                                   {stop, Reason, State}
%% @end
%%--------------------------------------------------------------------
-spec(handle_info(Info :: timeout() | term(), State :: #state{}) ->
  {noreply, NewState :: #state{}} |
  {noreply, NewState :: #state{}, timeout() | hibernate} |
  {stop, Reason :: term(), NewState :: #state{}}).
handle_info(_Info, State) ->
  {noreply, State}.

%%--------------------------------------------------------------------
%% @private
%% @doc
%% This function is called by a gen_server when it is about to
%% terminate. It should be the opposite of Module:init/1 and do any
%% necessary cleaning up. When it returns, the gen_server terminates
%% with Reason. The return value is ignored.
%%
%% @spec terminate(Reason, State) -> void()
%% @end
%%--------------------------------------------------------------------
-spec(terminate(Reason :: (normal | shutdown | {shutdown, term()} | term()),
    State :: #state{}) -> term()).
terminate(_Reason, _State) ->
  ok.

%%--------------------------------------------------------------------
%% @private
%% @doc
%% Convert process state when code is changed
%%
%% @spec code_change(OldVsn, State, Extra) -> {ok, NewState}
%% @end
%%--------------------------------------------------------------------
-spec(code_change(OldVsn :: term() | {down, term()}, State :: #state{},
    Extra :: term()) ->
  {ok, NewState :: #state{}} | {error, Reason :: term()}).
code_change(_OldVsn, State, _Extra) ->
  {ok, State}.

%%%===================================================================
%%% Internal functions
%%%===================================================================
initialize_graph(china, Graph, Tid, VertexFileName, EdgeFileName)->
  {ok, VertexIoDevice} = file:open(VertexFileName, [read, binary]),
  add_vertex(china, Graph, Tid, VertexIoDevice),
  file:close(VertexIoDevice),
  {ok, EdgeIoDevice} = file:open(EdgeFileName, [read, binary]),
  add_edge(china, Graph, Tid, EdgeIoDevice),
  file:close(EdgeIoDevice);
initialize_graph(world, Graph, Tid, VertexFileName, EdgeFileName)->
  {ok, VertexIoDevice} = file:open(VertexFileName, [read, binary]),
  add_vertex(world, Graph, Tid, VertexIoDevice),
  file:close(VertexIoDevice),
  {ok, EdgeIoDevice} = file:open(EdgeFileName, [read, binary]),
  add_edge(world, Graph, Tid, EdgeIoDevice),
  file:close(EdgeIoDevice).

add_vertex(china, Graph, Tid, VertexIoDevice) ->
  case file:read_line(VertexIoDevice) of
    {ok, Line} ->
      case binary:split(Line, [<<",">>, <<"\n">>], [global]) of
        [Province, DeliverSite | _] ->
          ets:insert(Tid, {Province, DeliverSite}),
          digraph:add_vertex(Graph, DeliverSite),
          add_vertex(china, Graph, Tid, VertexIoDevice);
        _ ->
          ok
      end;
    eof ->
      ok
  end;
add_vertex(world, Graph, Tid, VertexIoDevice) ->
  case file:read_line(VertexIoDevice) of
    {ok, Line} ->
      case binary:split(Line, [<<",">>, <<"\n">>], [global]) of
        [_Continent,<<>>] ->
          add_vertex(world, Graph, Tid, VertexIoDevice);
        [CountryCode, CountryChineseName, CountryEnglishName | _] ->
          ets:insert(Tid, {CountryCode, CountryChineseName, CountryEnglishName}),
          digraph:add_vertex(Graph, CountryCode),
          add_vertex(world, Graph, Tid, VertexIoDevice);
        _ ->
          ok
      end;
    eof ->
      ok
  end.

add_edge(china, Graph, Tid, EdgeIoDevice) ->
  case file:read_line(EdgeIoDevice) of
    {ok, Line} ->
      case binary:split(Line, [<<",">>, <<"\n">>], [global]) of
        [Province1, Province2 | _] ->
          [{_, DeliverSite1}] = ets:lookup(Tid, Province1),
          [{_, DeliverSite2}] = ets:lookup(Tid, Province2),
          digraph:add_edge(Graph, DeliverSite1, DeliverSite2),
          digraph:add_edge(Graph, DeliverSite2, DeliverSite1),
          add_edge(china, Graph, Tid, EdgeIoDevice);
        _ ->
          ok
      end;
    eof ->
      ok
  end;
add_edge(world, Graph, Tid, EdgeIoDevice) ->
  case file:read_line(EdgeIoDevice) of
    {ok, Line} ->
      case binary:split(Line, [<<",">>, <<"\n">>], [global]) of
        [_Continent,<<>>] ->
          add_edge(world, Graph, Tid, EdgeIoDevice);
        [_CountryCode1, CountryCode2 | _] when byte_size(CountryCode2) =/= 2 ->
          add_edge(world, Graph, Tid, EdgeIoDevice);
        [CountryCode1, CountryCode2 | _] ->
          digraph:add_edge(Graph, CountryCode1, CountryCode2),
          digraph:add_edge(Graph, CountryCode2, CountryCode1),
          add_edge(world, Graph, Tid, EdgeIoDevice);
        _ ->
          ok
      end;
    eof ->
      ok
  end.

lookup_deliver_node(lookup_province, ChinaGraph, Tid, Province) ->
  [{_, DeliverSite}] = ets:lookup(Tid, Province),
  case bfs(lookup_province, ChinaGraph, DeliverSite) of
    not_found ->
      {ok, ?HANGZHOU};
    {ok, RecentDeliverSite} ->
      {ok, RecentDeliverSite}
  end;
lookup_deliver_node(lookup_country, WorldGraph, _Tid, CountryCode) ->
  case bfs(lookup_country, WorldGraph, CountryCode) of
    not_found ->
      {ok, ?TOKYO};
    {ok, RecentDeliverSite} ->
      {ok, RecentDeliverSite}
  end.

bfs(Graph, Queue, NodeList, VisitedList) ->
  case queue:is_empty(Queue) of
    true ->
      not_found;
    _ ->
      {{value, Vertex} ,NewQueue} = queue:out(Queue),
      NewVisitedList = lists:append(VisitedList, [Vertex]),
      case find(Vertex, NodeList) of
        not_found ->
          VertexList = find_vertex_list(Graph, Vertex),
          NotVisitedVertexList = filter(VertexList, NewVisitedList),
          NewQueue1 = in_queue(NotVisitedVertexList, NewQueue),
          bfs(Graph, NewQueue1, NodeList, NewVisitedList);
        {ok, DeliverSite} ->
          {ok, DeliverSite}
      end
  end.

bfs(lookup_province, Graph, Vertex) ->
  Queue = queue:new(),
  {ok, SupportedNodeList} = application:get_env(china_deliver_sites),
  NewQueue = in_queue(Vertex, Queue),
  bfs(Graph, NewQueue, SupportedNodeList, []);
bfs(lookup_country, Graph, Vertex) ->
  Queue = queue:new(),
  {ok, SupportedNodeList} = application:get_env(world_deliver_sites),
  NewQueue = in_queue(Vertex, Queue),
  bfs(Graph, NewQueue, SupportedNodeList, []).

find(_Element, []) ->
  not_found;
find(Element, [{CountryCode, DeliverSite}|Tail]) ->
  case string:equal(Element, CountryCode) of
    true ->
      {ok, DeliverSite};
    _ ->
      find(Element, Tail)
  end;
find(Element, [DeliverSite|Tail]) ->
  case string:equal(Element, DeliverSite) of
    true ->
      {ok, DeliverSite};
    _ ->
      find(Element, Tail)
  end.

is_visited(Element, List) ->
  case find(Element, List) of
    not_found ->
      false;
    _ ->
      true
  end.

filter(VertexList, VisitedList) ->
  filter(VertexList, VisitedList, []).

filter([Head|Tail], VisitedList, ResultList) ->
  case is_visited(Head, VisitedList) of
    false ->
      filter(Tail, VisitedList, ResultList ++ [Head]);
    _ ->
      filter(Tail, VisitedList, ResultList)
  end;
filter([], _VisitedList, ResultList) ->
  ResultList.

find_vertex_list(Graph, Vertex) ->
  Edges = digraph:edges(Graph, Vertex),
  find_vertex_list(Graph, Vertex, Edges, []).

find_vertex_list(Graph, Vertex, [Head|Tail], VertexList) ->
  {_, V1, V2, _} = digraph:edge(Graph, Head),
  case string:equal(V1, Vertex) of
    true ->
      find_vertex_list(Graph, Vertex, Tail, VertexList ++ [V2]);
    _ ->
      find_vertex_list(Graph, Vertex, Tail, VertexList)
  end;
find_vertex_list(_Graph, _Vertex, [], VertexList) ->
  VertexList.

in_queue([], Queue) ->
  Queue;
in_queue([Head|Tail], Queue) ->
  NewQueue = queue:in(Head, Queue),
  in_queue(Tail, NewQueue);
in_queue(Item, Queue) ->
  queue:in(Item, Queue).

```

代码到此为止，关于ip数据库的查询部分可以参见`ipip.net`。

---

附录：

- [个人整理的世界地图的各个国家（中英文，简写）](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/deliver_site_graph/world_country_code.csv)
- [个人整理的世界地图的各个国家的粗略相邻关系](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/deliver_site_graph/world_country_border.csv)
- [个人整理的中国地图的各个省份（中英文）](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/deliver_site_graph/china_province_code.csv)
- [个人整理的中国地图的各个省份的相邻关系](https://flowsnow.oss-cn-shanghai.aliyuncs.com/history/deliver_site_graph/china_province_border.csv)