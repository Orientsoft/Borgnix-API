Borgnix API草稿 V0.3
===================

Borgnix系统中，API分为用户管理、设备管理、设备通信三类。用户和设备都有各自的UUID和Token。在用户管理和设备管理API中，需要涉及用户认证的部分，使用用户UUID/Token。在设备通信API中，不涉及用户认证，直接使用设备UUID/Token。

用户管理
------------
除了用户注册和登陆之外，其它操作都用用户的UUID/Token认证。

- **BorgUserRegister(username, password)**
Return: {uuid: userUUID, token: userToken}
说明：用户名密码注册，返回用户UUID和Token

- **BorgUserResetToken(uuid)**
Return: {token: userToken}
说明：用户在登陆之后才可以重设Token

- **BorgUserLogin(username, password)**
Return: {uuid: userUUID, token: userToken}
说明：用户名密码登录，返回用户UUID和Token

- **BorgUserLogout(uuid, token)**
Return: {}

- **BorgUserDelete([uuid], adminUUID, adminToken)**
Return: {del: [uuid]}
说明：该操作一般用户无权执行，需要用管理员UUID/Token验证。返回删除成功用户的UUID

设备管理
------------
设备管理API，是与用户绑定的操作，需要验证用户（通过UUID/Token）。

- **BorgDevRegister(userUuid, userToken)**
Return: {uuid: devUUID, token: devToken}
说明：该API用于设备端申请UUID和Token 

- **BorgDevClaim([uuid], userUuid, userToken)**
Return: {updated: [uuid, token]}
说明：可以一次性声明多个设备，需要用户验证。返回声明成功的设备UUID/Token

- **BorgDevList(userUuid, userToken)**
Return: {devices: [uuid, token, online]}
说明：查询用户的所有设备，返回设备UUID/Token/在线状态

- **BorgDevResetToken(uuid, userUuid, userToken)**
Return: {uuid: devUUID, token: devToken}
说明：要授权用户才可以重设授权设备的Token

- **BorgDevDelete([uuid], userUuid, userToken)**
Return: {del: [uuid]}
说明返回删除成功的设备UUID

设备通信
------------

设备通信API与用户无关，直接使用设备的认证信息。

- **BorgDevConnect(host, port, uuid, token, type, msgCB)**
Return: {status: ‘online’}
说明：在连接的时候，指定设备类型和收到数据的Callback。Connect API内部会自动管理Socket，当已经有Socket连接的时候自动重用。

- **BorgDevDisconnect(uuid, token)**
Return: {}

- **BorgDevSend(payload)**
Return: {}
说明：用户只需给出payload，API会自动添加通信协议层数据项。

- **BorgDevReceive()**
Return: dataJson