{% extends "./multiplayer-guide.tmpl" %}

{% set platformName = "C\#` `" %}
{% set userId = "UserId" %}
{% set autoJoinLobby = "AutoJoinLobby" %}
{% set lobby_room_url = "https://leancloud.github.io/Play-SDK-CSharp/html/classLeanCloud_1_1Play_1_1LobbyRoom.htm" %}
{% set joinRoom = "JoinRoom" %}
{% set joinRandomRoom = "JoinRandomRoom" %}
{% set joinOrCreateRoom = "JoinOrCreateRoom" %}
{% set setCustomProperties = "SetCustomProperties" %}
{% set playerTtl = "PlayerTtl" %}
{% set api_url = "https://leancloud.github.io/Play-SDK-CSharp/html/" %}
{% set gameVersion = "这个参数" %}


{% block import %}
请阅读 [安装](multiplayer-quick-start-csharp.html#安装)，获取 dll 库文件。
{% endblock %}


{% block initialization %}
首先实时对战 SDK 在内部实会例化了一个 `Play` 类型的实例对象，我们只需要通过 `Play.Instance` 使用即可。后文中的 `play` 都是指这个对象。

```cs
private Play play = Play.Instance;
```

接着我们需要实例化一个对象作为初始化实时对战的参数。

```cs
// 初始化
play.Init(YOUR_APP_ID, YOUR_APP_KEY, YOUR_APP_REGION);
```

注意：由于 Unity 要求必须在「主线程」调用 API，所以，为了减少开发者在使用上的麻烦，SDK 将保证会在「主线程」回调 `EVENT`。但是，需要开发者在 MonoBehavior 的 Update() 中调用 play.HandleMessage()，参考代码：

```cs
// Update is called once per frame
void Update () {
    play.HandleMessage();
}
```

（建议可以做一个 DontDestroy 的 GameObject 挂载脚本调用）
{% endblock %}


{% block set_userid %}
### 设置 `{{userId}}`

我们需要设置一个 `{{userId}}` 作为客户端的唯一标识连接至服务器。

```cs
play.UserId = "leancloud";
```

需要注意，这个 `userId` 有如下限制：
- 只允许英文、数字与下划线。
- 长度不能超过 32 字符
- 一个应用内全局唯一
{% endblock %}


{% block connection %}
```cs
play.Connect();
```
{% endblock %}


{% block connection_with_gameversion %}
如果需要通过版本号隔离玩家，可以通过以下接口连接。

```cs
play.Connect("0.0.1");
```

{{gameVersion}} 表示客户端的版本号，如果允许多个版本的游戏共存，则可以根据这个版本号路由到不同的游戏服务器。
{% endblock %}


{% block connection_event %}
在连接成功后，会收到 `CONNECTED`（连接成功）事件；否则，将会收到 `CONNECT_FAILED`（连接失败）事件，回调参数 `error` 将包含失败原因。

```cs
// 注册连接成功事件
play.On(Event.CONNECTED, (evtData) => {
    // TODO 可以做跳转场景等操作

});

// 注册连接失败事件
play.On(Event.CONNECT_FAILED, (error) => {
    // TODO 可以根据 error 提示用户失败原因

});
```

### 相关事件

| 事件   | 参数     | 描述                                       |
| ------------------------------------ | ------------------ | ---------------------------------------- |
| CONNECTED | 无 | 连接成功                                  |
| CONNECT_FAILED    | error  | 连接失败                                 |

{% endblock %}



{% block join_lobby %}
```cs
play.JoinLobby();
```

```cs
// 注册成功加入大厅事件
play.On(Event.LOBBY_JOINED, (evtData) => {
    // 加入大厅成功
});
```
{% endblock %}



{% block room_list %}
```cs
play.On(Event.LOBBY_ROOM_LIST_UPDATED, (evtData) => {
    var roomList = play.LobbyRoomList; // 返回 List<LobbyRoom>
    // TODO 可以做房间列表展示的逻辑
});
```
{% endblock %}

{% block lobby_event %}
### 相关事件

| 事件   | 参数     | 描述                                       |
| ------------------------------------ | ------------------ | ---------------------------------------- |
| LOBBY_JOINED    | 无 | 加入大厅                         |
| LOBBY_LEFT   | 无  | 离开大厅 |
| LOBBY_ROOM_LIST_UPDATED | 无 | 大厅房间列表更新                                  |
{% endblock %}



{% block create_default_room %}
```cs
play.CreateRoom();
```
{% endblock %}



{% block create_custom_room %}
```cs
// 房间的自定义属性
var props = new Dictionary<string, object>();
props.Add("title", "room title");
props.Add("level", 2);
var options = new RoomOptions()
{
    Visible = false,
    EmptyRoomTtl = 10000,
    PlayerTtl = 600,
    MaxPlayerCount = 2,
    CustomRoomProperties = props,
    CustoRoomPropertyKeysForLobby = new List<string>() { "level" },
    Flag = MasterSetMaster | MasterUpdateRoomProperties,
};
var expectedUserIds = new List<string>() { "cr3_2" };
play.CreateRoom(roomName: roomName, roomOptions: options, expectedUserIds: expectedUserIds);
```
{% endblock %}



{% block room_options %}
- `Opened`：房间是否开放。如果设置为 false，则不允许其他玩家加入。
- `Visible`：房间是否可见。如果设置为 false，则不会出现在大厅的房间列表中，但是其他玩家可以通过指定「房间名称」加入房间。
- `EmptyRoomTtl`：当房间中没有玩家时，房间保留的时间（单位：秒）。默认为 0，即 房间中没有玩家时，立即销毁房间数据。最大值为 300，即 5 分钟。
- `PlayerTtl`：当玩家掉线时，保留玩家在房间内的数据的时间（单位：秒）。默认为 0，即 玩家掉线后，立即销毁玩家数据。最大值为 300，即 5 分钟。
- `MaxPlayerCount`：房间允许的最大玩家数量。
- `CustomRoomProperties`：房间的自定义属性。
- `CustomRoomPropertyKeysForLobby`：房间的自定义属性 `CustomRoomProperties` 中「键」的数组，包含在 `CustomRoomPropertyKeysForLobby` 中的属性将会出现在大厅的房间属性中（`play.LobbyRoomList`），而全部属性要在加入房间后的 `room.CustomProperties` 中查看。这些属性将会在匹配房间时用到。
- `Flag`：创建房间标志位，详情请看下文中 [MasterClient 掉线不转移](#MasterClient 掉线不转移), [指定其他成员为 MasterClient](#指定其他成员为 MasterClient)，[只允许 MasterClient 修改房间属性](#只允许 MasterClient 修改房间属性)。
{% endblock %}



{% block create_room_api %}
更多关于 `CreateRoom`，请参考 [API 文档](https://leancloud.github.io/Play-SDK-CSharp/html/classLeanCloud_1_1Play_1_1Play.htm#a11e463efda7b325c4a55e3f7553d38f3)。
{% endblock %}



{% block create_room_events %}

当玩家请求创建房间后，将有可能接收到 `ROOM_CREATED`（房间创建成功）或 `ROOM_CREATE_FAILED`（房间创建失败）事件。

```cs
// 注册创建房间成功事件
play.On(Event.ROOM_CREATED, (evtData) => {
    // 房间创建成功

});

// 注册创建房间失败事件
play.On(Event.ROOM_CREATE_FAILED, (error) => {
    // TODO 可以根据 error 提示用户创建失败

});
```

注意：当房主创建好房间后会自动加入房间，所以也会收到 `ROOM_JOINED`（加入房间成功）事件。
{% endblock %}



{% block join_room %}
```cs
// 玩家在加入 'LiLeiRoom' 房间
play.JoinRoom("LiLeiRoom");
```
{% endblock %}



{% block join_room_with_ids %}
```cs
var expectedUserIds = new List<string>() { "LiLei", "Jim" };
// 玩家在加入 "game" 房间，并为 hello 和 world 玩家占位
play.JoinRoom("game", expectedUserIds);
```
{% endblock %}



{% block join_room_api %}
更多关于 `JoinRoom`，请参考 [API 文档](https://leancloud.github.io/Play-SDK-CSharp/html/classLeanCloud_1_1Play_1_1Play.htm#a894b6a1118c6ea3aed71df3068b54997)。
{% endblock %}



{% block join_room_event %}

当玩家请求加入房间后，将有可能接收到 `ROOM_JOINED`（加入房间成功）或 `ROOM_JOIN_FAILED`（加入房间失败）事件。

```cs
// 注册加入房间成功事件
play.On(Event.ROOM_JOINED, (evtData) => {
    // TODO 可以做跳转场景之类的操作

});

// 注册加入房间失败事件
play.On(Event.ROOM_JOIN_FAILED, (evtData) => {
    // TODO 可以提示用户加入失败，请重新加入

});
```
{% endblock %}



{% block join_random_room %}
```cs
play.JoinRandomRoom();
```
{% endblock %}



{% block join_random_room_with_condition %}

```cs
// 设置匹配属性
var matchProps = new Dictionary<string, object>();
matchProps.Add("level", 2);
play.JoinRandomRoom(matchProps);
```

与[加入指定房间](#加入指定房间)一样，我们也有可能接收到 `ROOM_JOINED`（加入房间成功）或 `ROOM_JOIN_FAILED`（加入房间失败）事件。

{% endblock %}



{% block join_or_create_room %}
```cs
// 例如，有 4 个玩家同时加入一个房间名称为 「room1」 的房间，如果不存在，则创建并加入
play.JoinOrCreateRoom("room1");
```
{% endblock %}



{% block join_or_create_room_api %}
更多关于 `JoinOrCreateRoom`，请参考 [API 文档](https://leancloud.github.io/Play-SDK-CSharp/html/classLeanCloud_1_1Play_1_1Play.htm#a3b98b509a36d489bd970c1052df67f38)。
{% endblock %}

{% block join_or_create_room_events %}

当调用这个接口后，只有**第一个**调用这个接口的玩家的请求会执行「创建房间」逻辑，而其他玩家的请求将会执行「加入房间」逻辑。因此不同的场景下事件回调如下：
* 如果创建房间成功，触发 `ROOM_CREATED` （创建房间成功）及 `ROOM_JOINED` （加入房间成功）事件。
* 如果创建房间失败，触发 `ROOM_CREATE_FAILED` （创建房间失败）事件。
* 如果加入房间成功，触发 `ROOM_JOINED` （加入房间成功）事件。
* 如果加入房间失败，触发 `ROOM_JOIN_FAILED` （加入房间失败）事件。

{% endblock %}



{% block player_room_joined %}
```cs
// 注册新玩家加入事件
play.On(Event.PLAYER_ROOM_JOINED, (evtData) => {
    var newPlayer = evtData["newPlayer"] as Player;
    // TODO 新玩家加入逻辑

});
```

可以通过 `play.Room.PlayerList` 获取房间内的所有玩家。
{% endblock %}

{% block player_is_local %}
```cs
var players = client.room.playerList;
var player = palyers[0];
var isLocal = player.isLocal;
```
{% endblock %}


{% block leave_room %}
```cs
play.LeaveRoom();
```

当玩家离开房间成功后，客户端会接收到 `ROOM_LEFT`（离开房间）事件。

```cs
// 注册离开房间事件
play.On(Event.ROOM_LEFT, (evtData) => {
    // TODO 可以执行跳转场景等逻辑

});
```

{% endblock %}


{% block player_room_left_event %}
```cs
// 注册有玩家离开房间事件
play.On(Event.PLAYER_ROOM_LEFT, (evtData) => {
    var leftPlayer = evtData["leftPlayer"] as Player;
    // TODO 可以执行玩家离开的销毁工作

});
```
{% endblock %}

{% block room_events %}
### 相关事件

| 事件   | 参数     | 描述                                       |
| ------------------------------------ | ------------------ | ---------------------------------------- |
| ROOM_CREATED    | 无  | 创建房间                                 |
| ROOM_CREATE_FAILED   | error   | 创建房间失败                           |
| ROOM_JOINED    | 无 | 加入房间                         |
| ROOM_JOIN_FAILED   | error  | 加入房间失败 |
| PLAYER_ROOM_JOINED    | { newPlayer } | 新玩家加入房间                         |
| PLAYER_ROOM_LEFT   | { leftPlayer }  | 玩家离开房间 |
| MASTER_SWITCHED    | { player }  | Master 更换                                 |
| ROOM_LEFT   | 无   | 离开房间                           |

{% endblock %}


{% block open_room %}
```cs
// 设置房间关闭
play.SetRoomOpened(false);
```
{% endblock %}



{% block visible_room %}
```cs
// 设置房间不可见
play.SetRoomVisible(false);
```
{% endblock %}



{% block set_master %}
```cs
// 通过玩家的 Id 指定 MasterClient
play.SetMaster(newMasterId);
```
{% endblock %}

{% block master_switched_event %}
```cs
// 注册主机切换事件
play.On(Event.MASTER_SWITCHED, (evtData) => {
    var newMaster = evtData["newMaster"] as Player;
    // TODO 可以做主机切换的展示

    // 可以根据判断当前客户端是否是 Master，来确定是否执行逻辑处理。
    if (play.Player.IsMaster) {

    }
});
```
{% endblock %}

{% block master_fixed %}
```cs
var options = new RoomOptions()
{
   Flag = FixedMaster
};
play.CreateRoom(roomOptions: options);
```
{% endblock %}

{% block custom_props %}
为了满足开发者不同的游戏需求，实时对战 SDK 允许开发者设置「自定义属性」。
{% endblock %}



{% block set_custom_props %}
房间除了固有的属性外，还包括一个 `Dictionary<string, object>` 类型的自定义属性，比如战斗的回合数、所有棋牌等。

```cs
// 设置想要修改的自定义属性
var props = new Dictionary<string, object>();
props.Add("gold", 1000);
// 设置 gold 属性为 1000
play.Room.SetCustomProperties(props);
```
{% endblock %}



{% block custom_props_event %}
```cs
// 注册房间属性变化事件
play.On(Event.ROOM_CUSTOM_PROPERTIES_CHANGED, (changedProps) => {
    var props = play.Room.CustomProperties;
    long gold = (long)props["gold"];
    // TODO 可以做属性变化的界面展示

});
```

注意：`changedProps` 参数只表示当前修改的参数，不是「全部属性」。如需获得全部属性，请通过 `play.Room.CustomProperties` 获得。
{% endblock %}


{% block master_update_room_properties %}

```cs
var options = new RoomOptions()
{
   Flag = MasterUpdateRoomProperties
};
play.CreateRoom(roomOptions: options);
```

{% endblock %}


{% block set_player_custom_props %}
```cs
// 扑克牌对象
var poker = new Dictionary<string, object>();
poker.Add("flower", 1);
poker.Add("num", 13);
var props = new Dictionary<string, object>();
props.Add("nickname", "Li Lei");
props.Add("gold", 1000);
props.Add("poker", poker);
// 请求设置玩家属性
play.Player.SetCustomProperties(props);
```
{% endblock %}



{% block player_custom_props_event %}
```cs
// 注册玩家自定义属性变化事件
play.On(Event.PLAYER_CUSTOM_PROPERTIES_CHANGED, (evtData) => {
    // 结构事件参数
    var player = evtData["player"] as Player;
    // 得到玩家所有自定义属性
    var props = player.CustomProperties;
    var title = props["title"] as string;
    var gold = (long)props["gold"];
    // TODO 可以做属性变化的界面展示

});
```
{% endblock %}



{% block set_player_custom_props_with_cas %}
```cs
var props = new Dictionary<string, object>();
// X 分别表示当前的客户端
props.Add("tulong", X);
var expectedValues = new Dictionary<string, object>();
// 屠龙刀当前拥有者
expectedValues.Add("tulong", null);
play.Room.SetCustomProperties(props, expectedValues);
```

这样，在「第一个玩家」获得屠龙刀之后，`tulong` 对应的值为「第一个玩家」，后续的请求 `expectedValues = { tulong: null }` 将会失败。
{% endblock %}



{% block send_event %}
```cs
var options = new SendEventOptions() {
    // 设置事件的接收组为 Master
    ReceiverGroup = ReceiverGroup.MasterClient
    // 也可以指定接收者 actorId
    // targetActorIds = new List<int>() { 1 },
};
// 设置技能 Id
var eventData = new Dictionary<string, object>();
eventData.Add("skillId", 123);
// 发送 eventId 为 skill 的事件
play.SendEvent("skill", eventData, options);
```

其中 `options` 是指事件发送参数，包括「接收组」和「接收者 ID 数组」。
- 接收组（ReceiverGroup）是接收事件的目标的枚举值，包括 Others（房间内除自己之外的所有人）、All（房间内的所有人）、MasterClient（主机）。
- 接收者 ID 数组是指接收事件的目标的具体值，即玩家的 `actorId` 数组。`actorId` 可以通过 `player.actorId` 获得。
{% endblock %}



{% block send_event_event %}
```cs
// 注册自定义事件
play.On(Event.CUSTOM_EVENT, (evtData) => {
    // 获取事件参数
    var eventId = evtData["eventId"] as string;
    var eventData = evtData["eventData"] as Dictionary<string, object>;
    if (eventId == "skill") {
        // 如果是 skill 事件
        var skillId = eventData["skillId"] as string;
        // TODO 处理释放技能的表现

    }
});
```
{% endblock %}

{% block disconnect %}
## 断开连接

当游戏过程中，由于网络原因可能会断开连接，此时 SDK 会向客户端派发 `DISCONNECTED`（断开连接）事件，开发者可以根据需要注册并处理。

```cs
// 注册断开连接事件
play.On(Event.DISCONNECTED, (evtData) => {
    // TODO 如果需要，可以选择重连

});
```

开发者也可以通过下面的接口自行断开连接。

```cs
play.Disconnect();
```
{% endblock %}




{% block player_activity_changed %}

```cs
// 注册玩家掉线 / 上线事件
play.On(Event.PLAYER_ACTIVITY_CHANGED, (evtData) => {
    var player = evtData["player"] as Player;
    // 获得用户是否「活跃」状态
      // player.IsActive;
      // TODO 根据玩家的在线状态可以做显示和逻辑处理

});
```
{% endblock %}


{% block set_player_ttl %}

```cs
var options = new RoomOptions() {
    // 将 PlayerTtl 设置为 300 秒
    PlayerTtl = 300
}
play.CreateRoom(roomOptions: options);
```

{% endblock %}

{% block reconnect %}
```cs
play.On(Event.DISCONNECTED, (evtData) => {
    // 重连
    play.Reconnect();
});
```
{% endblock %}



{% block rejoin_room %}
```cs
play.On(Event.DISCONNECTED, (evtData) => {
    // 重连
    play.Reconnect();
});

play.On(Event.CONNECTED, (evtData) => {
    // TODO 根据是否有缓存的之前的房间名，回到房间。

    if (roomName) {
        play.RejoinRoom(roomName);
    }
});
```

如果房间存在并且玩家的 `ttl` 没到期，则其他玩家将会收到 `PLAYER_ACTIVITY_CHANGED` 事件。

```cs
// 注册玩家在线状态变化事件
play.On(Event.PLAYER_ACTIVITY_CHANGED, (evtData) => {
    var player = data["player"] as Player;
    // 获得用户是否「活跃」状态
      // player.IsActive;
    // TODO 根据玩家的在线状态可以做显示和逻辑处理

});
```
{% endblock %}



{% block reconnect_and_rejoin %}
```cs
play.On(Event.DISCONNECTED, (evtData) => {
    // 重连并回到房间
    play.ReconnectAndRejoin();
});

// 在加入房间后，更新数据和界面
play.On(Event.ROOM_JOINED, (evtData) => {
    // TODO 根据房间和玩家最新数据进行界面重构

});
```

这个接口相当于 `Reconnect()` 和 `RejoinRoom()` 的合并。通过这个接口，可以直接重新连接并回到「之前的房间」。
{% endblock %}



{% block error_event %}
```cs
play.On(Event.Error, (err) => {
    var code = (int)err["code"];
    var detail = err["detail"] as string;
    // 联系 LeanCloud

});
```
{% endblock %}
