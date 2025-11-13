```
应用层              传输层        说明
─────────────────────────────────────────────────
HTTP协议      →     TCP        网页请求（80端口）
                    
HTTPS协议     →     TCP+TLS    加密的HTTP（443端口）
                    
WebSocket     →     TCP        实时双向通信
(ws://)                        
                    
WebSocket     →     TCP+TLS    加密的WebSocket
(wss://)                       

─────────────────────────────────────────────────
DNS查询       →     UDP        域名解析（53端口）
视频直播      →     UDP        直播流（RTP协议）
语音通话      →     UDP        VoIP（SIP协议）
```

