# Mac和Linux远程连接服务器异常修复（WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!）

```shell
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:N7d5BT+g5Yuk/U+rTKpJBPwlrbxTJNC61nqSKJOaE9Y.
Please contact your system administrator.
Add correct host key in /Users/zhengyi/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /Users/zhengyi/.ssh/known_hosts:65
ECDSA host key for 10.1.12.50 has changed and you have requested strict checking.
Host key verification failed.
```

这里直接告诉我们了去更新 key

```shell
Offending ECDSA key in /Users/zhengyi/.ssh/known_hosts:65
```

我们可以直接将这行删除,然后在重新配置免密登录就可以了