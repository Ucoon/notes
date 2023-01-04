---
title: openVidu平台初体验——本地环境搭建（Windows）
---

# 简介

>OpenVidu是一个平台，可以帮助您在Web网页或移动应用程序中添加视频通话。它提供了完整的技术栈，非常易于集成到您的应用程序中。我们的主要目标是帮助开发人员可以快速且少量代码入侵的将实时通信添加到他们的应用中。

# openvidu-hello-world

## 步骤

1. Docker在Windows上的安装
2. 根据文档clone项目代码
3. 开启本地服务`http-server`
4. 开启openVidu服务

## 示例代码解析

1. 初始化`OpenVidu`实例，并初始化会话`session`

   ```javascript
   OV = new OpenVidu();
   session = OV.initSession();
   ```

2. 订阅会话`session`的事件

   ```javascript
   session.on("streamCreated", function (event) {
   	session.subscribe(event.stream,"subscriber");
   });
   ```

3. 从OpenVidu Server获取token(参考值：wss://localhost:4443?sessionId=SessionA&token=tok_ZQpY0XV2rhDEhEl8)

   **我们需要向 OpenVidu Server 请求token才能连接到我们的会话。正常情况下这个过程应该完全发生在我们的服务器端，而不是我们的客户端**

4. 使用token连接会话`session`

   ```javascript
   session.connect(token)
   ```

5. 离开会话`session`

   ```javascript
   session.disconnect();
   ```

# openvidu-android 

![原理图](https://docs.openvidu.io/en/2.22.0/img/tutorials/openvidu-android.png)

## 步骤

开启openVidu服务，**要设置当前IP**，其余步骤与hello-world一致：

```shell
docker run -p 4443:4443 --rm -e OPENVIDU_SECRET=MY_SECRET -e DOMAIN_OR_PUBLIC_IP=YOUR_OPENVIDU_IP openvidu/openvidu-server-kms:2.22.0
```

## 示例代码解析

1. 从OpenVidu Server获取token(参考值：wss://localhost:4443?sessionId=SessionA&token=tok_ZQpY0XV2rhDEhEl8)

   **我们需要向 OpenVidu Server 请求token才能连接到我们的会话。正常情况下这个过程应该完全发生在我们的服务器端，而不是我们的客户端**

2. 使用token连接opendivu-server服务

   当成功获取到token时，初始化会话`session`，设置相机和开始webSocket连接

   ```java
       private void getTokenSuccess(String token, String sessionId) {
           // Initialize our session
           session = new Session(sessionId, token, views_container, this);
   
           // Initialize our local participant and start local camera
           String participantName = participant_name.getText().toString();
           LocalParticipant localParticipant = new LocalParticipant(participantName, session, this.getApplicationContext(), localVideoView);
           localParticipant.startCamera();
           runOnUiThread(() -> {
               // Update local participant view
               main_participant.setText(participant_name.getText().toString());
               main_participant.setPadding(20, 3, 20, 3);
           });
   
           // Initialize and connect the websocket to OpenVidu Server
           startWebSocket();
       }
   ```

   1. 初始化会话`session`：在使用WebRTC进行音视频通话时，需要构建一个`PeerConnectionFactory`，这是一个连接工厂，创建本址`LocalMediaStream流`及客户端`PeerConnection`等，都需要使用；`PeerConnectionFactory`的创建流程大致分为两步：`PeerConnectionFactory.initialize`和 `PeerConnectionFactory.Builder.createPeerConnectionFactory`

      ```java
      //Creating a new PeerConnectionFactory instance
      PeerConnectionFactory.InitializationOptions.Builder optionsBuilder = PeerConnectionFactory.InitializationOptions.builder(activity.getApplicationContext());
      optionsBuilder.setEnableInternalTracer(true);//启用内部调试
      PeerConnectionFactory.InitializationOptions opt = optionsBuilder.createInitializationOptions();
      PeerConnectionFactory.initialize(opt);
      PeerConnectionFactory.Options options = new PeerConnectionFactory.Options();
      
      // Using sofware encoder and decoder.
      final VideoEncoderFactory encoderFactory; //视频编码工厂
      final VideoDecoderFactory decoderFactory; //视频解码工厂
      encoderFactory = new SoftwareVideoEncoderFactory();
      decoderFactory = new SoftwareVideoDecoderFactory();
      
      peerConnectionFactory = PeerConnectionFactory.builder()
                      .setVideoEncoderFactory(encoderFactory)
                      .setVideoDecoderFactory(decoderFactory)
                      .setOptions(options)
                      .createPeerConnectionFactory();
      ```

   2. 开启相机：Android 为我们提供了一种非常简单的方式来使用 Camera API。此 API 包括对设备上可用的各种相机和相机功能的支持，允许您在应用程序中捕获图片和视频。最后，我们需要存储视频轨道。

      >**MediaSource**：WebRTC媒体资源数据来源，它有两个子类：AudioSource（音频资源）、VideoSource（视频资源）
      >
      >**MediaStreamTrack**：媒体资源轨，一个MediaStreamTrack对应一个MediaSource，创建媒体轨需要MediaSource，同样，它也有两个子类：AudioTrack（音频轨）对应AudioSource、VideoTrack（视频轨）对应VideoSource
      >
      >**MediaStream**：媒体流，一个媒体流可以添加多条AudioTrack和VideoTrack，一般来说我们对应分报只添加一条

      ```java
          public void startCamera() {
      
              final EglBase.Context eglBaseContext = EglBase.create().getEglBaseContext();
              PeerConnectionFactory peerConnectionFactory = this.session.getPeerConnectionFactory();
      
              //创建音频源
              AudioSource audioSource = peerConnectionFactory.createAudioSource(new MediaConstraints());
              //传入AudioSource，用于创建音频轨
              this.audioTrack = peerConnectionFactory.createAudioTrack("101", audioSource);
      
              surfaceTextureHelper = SurfaceTextureHelper.create("CaptureThread", eglBaseContext);
              // create VideoCapturer
              VideoCapturer videoCapturer = createCameraCapturer();
              VideoSource videoSource = peerConnectionFactory.createVideoSource(videoCapturer.isScreencast());
              videoCapturer.initialize(surfaceTextureHelper, context, videoSource.getCapturerObserver());
              videoCapturer.startCapture(480, 640, 30);
      
              //传入VideoSource，用于创建视频轨
              this.videoTrack = peerConnectionFactory.createVideoTrack("100", videoSource);
              //将视频数据展示在SurfaceViewRenderer上
              this.videoTrack.addSink(localVideoView);
          }
      ```

   3. 通过webSocket连接openvidu server
   
      ```java
      	@Override
          protected Void doInBackground(SessionActivity... sessionActivities) {
              try {
                  WebSocketFactory factory = new WebSocketFactory();
                  SSLContext sslContext = SSLContext.getInstance("TLS");
                  sslContext.init(null, trustManagers, new java.security.SecureRandom());
                  factory.setSSLContext(sslContext);
                  factory.setVerifyHostname(false);
                  factory.setConnectionTimeout(10*1000);
                  websocket = factory.createSocket(getWebSocketAddress(openviduUrl));
                  websocket.addListener(this);
                  websocket.connect();
              } catch (KeyManagementException | NoSuchAlgorithmException | IOException | WebSocketException e) {
                  Log.e("WebSocket error", e.getMessage());
                  Handler mainHandler = new Handler(activity.getMainLooper());
                  Runnable myRunnable = () -> {
                      Toast toast = Toast.makeText(activity, e.getMessage(), Toast.LENGTH_LONG);
                      toast.show();
                      activity.leaveSession();
                  };
                  mainHandler.post(myRunnable);
                  websocketCancelled = true;
              }
              return null;
          }
      ```
   
   4. 使用 `OpenVidu Server RPC `协议：在这一部分我们将能够调用 OpenVidu Server 方法并接收来自 OpenVidu Server 的事件。
   
      1. 在和OpenVidu Server连接成功时加入房间`joinRoom`：
   
         ```java
             @Override
             public void onConnected(WebSocket ws, Map<String, List<String>> headers) throws
                     Exception {
                 Log.i(TAG, "Connected");
                 pingMessageHandler();
                 this.joinRoom();
             }
             public void joinRoom() {
                 Map<String, String> joinRoomParams = new HashMap<>();
                 joinRoomParams.put(JsonConstants.METADATA, "{\"clientData\": \"" + this.session.getLocalParticipant().getParticipantName() + "\"}");
                 joinRoomParams.put("secret", "");
                 joinRoomParams.put("session", this.session.getId());
                 joinRoomParams.put("platform", "Android " + android.os.Build.VERSION.SDK_INT);
                 joinRoomParams.put("token", this.session.getToken());
                 joinRoomParams.put("sdkVersion", "2.22.0");
                 this.ID_JOINROOM.set(this.sendJson(JsonConstants.JOINROOM_METHOD, joinRoomParams));
             }
         ```
   
      2. 加入房间之后，OpenVidu返回会话中所有现有参与者和所有已发布流的对象：
      
         ```json
         {
             "id":0,
             "result":{
                 "id":"con_RkS15m4mzj",
                 "finalUserId":"98F0A7693C7F0C15",
                 "createdAt":1655276912447,
                 "metadata":"{\"clientData\": \"Participant83\"}",
                 "value":[],
                 "session":"SessionA",
                 "version":"2.22.0",
                 "mediaServer":"kurento",
                 "videoSimulcast":false,
                 "record":true,
                 "role":"PUBLISHER",
                 "coturnIp":"192.168.10.65",
                 "coturnPort":3478,
                 "turnUsername":"1655363310:Di5eUnpt",
                 "turnCredential":"465w0xf3xrWFQqE0kczHYh8EX3U=",
                 "sessionId":"6d8phn7e5cb6o0bn4r0lmoqc53"
             },
             "jsonrpc":"2.0"
         }
         ```
      
      3. 在webSockect收到消息回调函数处理OpenVidu Server 事件：
      
         ```java
             @Override
             public void onTextMessage(WebSocket websocket, String text) throws Exception {
                 Log.i(TAG, "Text Message " + text);
                 JSONObject json = new JSONObject(text);
                 if (json.has(JsonConstants.RESULT)) {
                     handleServerResponse(json);
                 } else {
                     handleServerEvent(json);
                 }
             }
         ```
      
      4. 加入房间`joinRoom`成功后，初始化本地的`PeerConnection`和`MediaStream`并调用`publishVideo`RPC 方法来发布我们自己的相机
      
         - 初始化本地的`PeerConnection`：
      
           `PeerConnection createPeerConnection(RTCConfiguration rtcConfig, Observer observer)`：`RTCConfiguration`表征`PeerConnection`的全局配置项，`PeerConnection.Observer`是`PeerConnection`的事件回调
      
         - `PeerConnectionObserver`是`PeerConnection`的回调接口，应用层可以必须提供回调接口的实现，以便响应`PeerConnection`的事件。这些接口大致分为如下几类：
      
           几个状态相关回调：
      
           1. **OnSignalingChange**：信令状态改变。
           2. **OnConnectionChange**：PeerConnection状态改变。
           3. 远端流或者轨道的添加或者移出：
              - **OnAddStream**：收到远端Peer的一个新stream。
              - **OnRemoveStream**：收到远端Peer移出一个stream。
              - **OnAddTrack**：当一个receiver和它的track被创建时。Plan B 和 Unified Plan语法下都会被调用，但是Unified Plan语法下更建议使用OnTrack回调，OnAddTrack只是为了兼容之前的Plan B遗留的接口，二者在同样的情况下被回调。
              - **OnTrack**：该方法在收到的信令指示一个transceiver将从远端接收媒体时被调用，实际就是在调用SetRemoteDescription时被触发。该接收track可以通过transceiver->receiver()->track()方法被访问到，其关联的streams可以通过transceiver->receiver()->streams()获取。只有在Unified Plan语法下，该回调方法才会被触发。
              - **OnRemoveTrack**：该方法在收到的信令指示某个track中将不再收到媒体数据时触发。Plan B语法下，对应的receiver将被从PeerConnection中移出，并且对应track将被设置为muted状态；Unified Plan语法下, 对应的receiver将被保留，对应的transceiver将改变direction为仅发送sendonly 或者非活动inactive状态
           4. ICE过程相关：
              - **OnRenegotiationNeeded**：需要重新协商时触发，比如重启ICE时。
              - **OnIceCandidate**：收集到一个新的ICE候选项时触发。
              - **OnIceCandidateError**：收集ICE选项时出错。
              - **OnIceCandidatesRemoved**：当候选项被移除时触发。
              - **OnStandardizedIceConnectionChange**：符合标准的ICE连接状态改变。
              - **OnIceGatheringChange**：ICE收集状态改变。
              - **OnIceConnectionReceivingChange**：ICE连接接收状态改变。
              - **OnIceSelectedCandidatePairChanged**：ICE连接所采用的候选者对改变。
           5. DataChannel相关：
              - **OnDataChannel**：当远端打开data channel通道时触发。
      
         ```java
         final LocalParticipant localParticipant = this.session.getLocalParticipant();
         final String localConnectionId = result.getString(JsonConstants.ID);
         this.mediaServer = result.getString(JsonConstants.MEDIA_SERVER);
         localParticipant.setConnectionId(localConnectionId);
         PeerConnection localPeerConnection = session.createLocalPeerConnection();
         localParticipant.setPeerConnection(localPeerConnection);
         ```
      
         - 调用`publishVideo`发布我们自己的相机
      
           ```java
           MediaConstraints sdpConstraints = new MediaConstraints();
           sdpConstraints.mandatory.add(new MediaConstraints.KeyValuePair("offerToReceiveAudio", "false"));
           sdpConstraints.mandatory.add(new MediaConstraints.KeyValuePair("offerToReceiveVideo", "false"));
           session.createOfferForPublishing(sdpConstraints);
           
           public void createOfferForPublishing(MediaConstraints constraints) {
               localParticipant.getPeerConnection().createOffer(new CustomSdpObserver("createOffer") {
                    @Override
                    public void onCreateSuccess(SessionDescription sessionDescription) {
                        super.onCreateSuccess(sessionDescription);
                        Log.i("createOffer SUCCESS", sessionDescription.toString());
                        localParticipant.getPeerConnection().setLocalDescription(new CustomSdpObserver("createOffer_setLocalDescription"), sessionDescription);
                        websocket.publishVideo(sessionDescription);
                    }
           
                    @Override
                    public void onCreateFailure(String s) {
                        Log.e("createOffer ERROR", s);
                    }
                }, constraints);
            }
           
            public void publishVideo(SessionDescription sessionDescription) {
                Map<String, String> publishVideoParams = new HashMap<>();
                publishVideoParams.put("audioActive", "true");
                publishVideoParams.put("videoActive", "true");
                publishVideoParams.put("doLoopback", "false");
                publishVideoParams.put("frameRate", "30");
                publishVideoParams.put("hasAudio", "true");
                publishVideoParams.put("hasVideo", "true");
                publishVideoParams.put("typeOfVideo", "CAMERA");
                publishVideoParams.put("videoDimensions", "{\"width\":320, \"height\":240}");
                publishVideoParams.put("sdpOffer", sessionDescription.description);
                this.ID_PUBLISHVIDEO.set(this.sendJson(JsonConstants.PUBLISHVIDEO_METHOD, publishVideoParams));
            }
           ```
      
           

参考：

[openVidu官网](https://openvidu.io/)

[Install Docker Desktop on Windows](https://docs.docker.com/desktop/windows/install/)