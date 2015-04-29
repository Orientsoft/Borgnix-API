Borgnix API草稿 V0.3
===================

Borgnix系统中，API分为用户管理、设备管理、设备通信三类。用户和设备都有各自的UUID和Token。在用户管理和设备管理API中，需要涉及用户认证的部分，使用用户UUID/Token。在设备通信API中，不涉及用户认证，直接使用设备UUID/Token。

用户
------------
除了用户注册和登陆之外，其它操作都用用户的UUID/Token认证。

- **BorgUserLogin(username, password)**  
Return: {uuid: userUUID, token: userToken}  
说明：用户名密码登录，返回用户UUID和Token  
```
client.pub('user/login/up', {username: username, password: password})
client.sub('user/username/login/down')
server.sub('user/login/up')
server.pub('user/username/login/down', {uuid: userUuid, token: userToken})
```

- **BorgUserLogout(uuid, token)**  
Return: {}  


- **BorgDevList(userUuid, userToken)**  
Return: {devices: [uuid, token, online]}  
说明：查询用户的所有设备，返回设备UUID/Token/在线状态  
```
client.pub('user/dev_list/up', {uuid:UUID, token:TOKEN})
client.sub('user/userUuid/dev_list/down')
server.sub('user/dev_list/up')
server.pub( 'user/userUuid/dev_list/down'
          , [{uuid:UUID, token:TOKEN, status:STATUS}])
```


设备
------------


- **Connect**

  1. 使用设备uuid和token做验证，用户指定clientID（用于表示设备类型，可以重复，与认证无关），连接到borgnix MQTT broker；
  2. 连接成功后，subscribe ***uuid/i***，确定收到消息后的回调函数。

- **Disconnect**

  1. 断开与borgnix MQTT broker 的连接。

- **Send**

  1. 将payload根据通信层协议封装；
  2. 将封装后的包用***uuid/o*** publish到borgnix MQTT broker。

- **Receive**

  1. 收到***uuid/i***的消息时，根据通信层协议对消息进行解码，得到payload；
  2. 将payload传给相应的回调函数。

**规定**
- **通信层协议**
```json
{
      src: <sender_uuid>,
      tgt: <receiver_uuid>,
      time: <timestamp>,
      action: ['cmd'/'data'...],
      type: <sensor_type>,
      sign: <msg signature>,
      ver: '1.0',
      payload: { content }
}
```
**src**：发送者的UUID
**dst**：接收者的UUID（与云服务器通信其实不需要该字段，但保留该字段，用于以后支持透传功能）
**time**：发送时间戳，有两个作用，一是确定消息时间，二是配合加密算法抵抗重播攻击
**action**：该信息的意义
**type**：发送者的类型（sensor、回路、Gate、Server...实际取值要具体说明是什么Sensor，比如temperature、light等）
**sign**：对该条消息的签名（利用公钥算法，防止消息伪造，Demo中可以不实现，保留字段）
**ver**：协议的版本号（在协议升级的时候，服务器可以识别老的设备，也可以实现新老设备混用）
**payload**：业务数据

- **MQTT topic**
  sub: ***uuid/i***
  pub: ***uuid/o***

管理
-----


- **BorgUserRegister(username, password)**  
  Return: {uuid: userUUID, token: userToken}  
  说明：用户名密码注册，返回用户UUID和Token
  **Note**：在注册失败时返回值的确定？


- **BorgUserResetToken(uuid)**  
Return: {token: userToken}  
说明：用户在登陆之后才可以重设Token  


- **BorgUserDelete([uuid], adminUUID, adminToken)**  
Return: {del: [uuid]}  
说明：该操作一般用户无权执行，需要用管理员UUID/Token验证。返回删除成功用户的UUID  

- **BorgDevRegister(devType, devDesc)**  
Return: {uuid: devUUID, token: devToken}  
说明：该API用于设备端申请UUID和Token，注册后需要与用户进行关联   
```
client.pub('user/userUuid/dev_reg/up', {devType:devType, devToken:devToken})
client.sub('user/userUuid/dev_reg/down')
server.sub('user/+/dev_reg/up')
server.pub('user/userUuid/dev_reg/down')
```

- **BorgDevResetToken(uuid, userUuid, userToken)**  
Return: {uuid: devUUID, token: devToken}  
说明：要授权用户才可以重设授权设备的Token  

- **BorgDevDelete([uuid], userUuid, userToken)**  
Return: {del: [uuid]}  
说明返回删除成功的设备UUID  
```
client.pub('user/userUuid/dev_delete/up', [uuid])
client.sub('user/userUuid/dev_delete/down')
server.sub('user/+/dev_delete/up')
server.pub('user/userUuid/dev_delete/down', {status:STATUS, devices:[uuid]})
```

- **BorgDevClaim([uuid], userUuid, userToken)**  
Return: {updated: [uuid, token]}  
说明：将多个设备与用户进行关联，返回关联成功的设备uuid  
Note：设备token在注册获取还是在关联获取？
```
client.pub('user/userUuid/claim/up'
    , { uuid:userUuid
      , token:userToken
      , devices:[DEV_UUID]
      })
```
=======
Retired:
- **BorgDevReceive()**  
Return: dataJson  
