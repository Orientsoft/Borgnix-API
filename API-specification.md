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
client.pub('user/username/login/up', {username: username, password: password})
client.sub('user/username/login/down')
server.sub('user/+/login/up')
server.pub('user/username/login/down', {uuid: userUuid, token: userToken})
```

- **BorgUserLogout(uuid, token)**  
Return: {}  
```
client.pub('user/userUuid/logout/up')
server.sub('user/userUuid/logout/up')
```

- **BorgDevList(userUuid, userToken)**  
Return: {devices: [uuid, token, online]}  
说明：查询用户的所有设备，返回设备UUID/Token/在线状态  
```
client.pub('user/userUuid/dev_list/up', {uuid:UUID, token:TOKEN})
client.sub('user/userUuid/dev_list/down')
server.sub('user/+/dev_list/up')
server.pub( 'user/userUuid/dev_list/down'
          , [{uuid:UUID, token:TOKEN, status:STATUS}])
```


设备
------------

设备通信API与用户无关，直接使用设备的认证信息。

- **BorgDevConnect(host, port, uuid, token, msgCB)**  
Return: {status: status}  
连接成功时返回值中status为‘success’，失败时为‘failed’  
说明：在连接的时候，指定收到数据的Callback。Connect API内部会自动管理Socket，当已经有Socket连接的时候自动重用。注册成功后，设备publish使用dev/UUID/up,subcribe使用dev/UUID/down。  
```
client.pub('dev/UUID/auth/up', {uuid: UUID, token: TOKEN})
client.sub('dev/UUID/auth/down')
server.sub('dev/+/auth/up')
server.pub('dev/UUID/auth/down', {status:STATUS})
```

- **BorgDevDisconnect(uuid, token)**  
Return: {}  
```
client.pub('dev/UUID/disconnect/up')
```

- **BorgDevSend(payload)**  
Return: {}  
说明：用户只需给出payload，API会自动添加通信协议层数据项。  
```
client.pub('dev/DEV_UUID/up', payload)
```

管理
-----

<<<<<<< HEAD
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
>>>>>>> origin/master
