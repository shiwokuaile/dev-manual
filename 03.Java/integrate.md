# 技术整合

## WebSocket

SpringBoot 整合 WebSocket。

**1、引入依赖**

```xml
<!--websocket-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

**2、配置**

```java
@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

**3、使用**

```java
@Slf4j
@ServerEndpoint("/wsmessage/{userid}")
@Component
public class WebSocketHandler {

    @OnOpen
    public void onOpen(@PathParam("userid") String userid,  Session session) {
        log.info("on open, sessionid={}, userid={}", session.getId(), userid);
    }

    /**
     * 客户端关闭
     * @param session session
     */
    @OnClose
    public void onClose(Session session) {
        log.info("onclose session id={}, userid={}", session.getId(), this.userid);
    }

    /**
     * 发生错误
     * @param e
     */
    @OnError
    public void onError(Throwable e) {
        e.printStackTrace();
    }

    /**
     * 收到客户端发来消息
     * @param message  消息对象
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        log.info("on message session id={}, userid={}, message={}", session.getId(), this.userid, message);
    }
}
```

**注意**

> 1、websocket 连接地址需要加上 context-path
> 
> 2、@ServerEndpoint 注解下，不能自动注入
> 
> 3、注意连接 websocket 时是否有拦截器
> 
> 4、如果请求链中有 nginx 节点，那么需要配置 nginx 支持 websocket（需要针对特定路径匹配）
> 
> 参考：https://zhuanlan.zhihu.com/p/485722514

**Nginx 配置**

```nginx
http {
    #
    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    } 
    server {
        location /api{
            proxy_pass   http://127.0.0.1;
            proxy_http_version 1.1;
            proxy_connect_timeout 5s;
            proxy_read_timeout 60s;
            proxy_send_timeout 30s;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "$connection_upgrade";
        }
    }
}
```

## NTP

NTP 是网络时间服务器，用于同步时间，是作为第三方可信任和可靠的时间标准。

> 客户端一般是不会获取本地时间作为参考时间（存在被篡改的风险），一般都是获取服务器或者是可靠的第三方 NTP 网络时间。

**引入依赖**

```xml
<dependency>
    <groupId>commons-net</groupId>
    <artifactId>commons-net</artifactId>
    <version>3.9.0</version>
</dependency>
```

**使用**

```java
/**
 * 获取 NTP 时间（时间戳：毫秒）
 * @return
 */
public long getNtpTimestamp() {
    try {
        // ntp.ntsc.ac.cn 是中科院的 NTP 服务器
        NTPUDPClient timeClient = new NTPUDPClient();
        String ntpServerHost = properties.getNtpServerHost();
        InetAddress timeServerAddress = InetAddress.getByName(ntpServerHost);
        TimeInfo timeInfo = timeClient.getTime(timeServerAddress);
        TimeStamp timeStamp = timeInfo.getMessage().getTransmitTimeStamp();
        return timeStamp.getTime();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return -1;
}
```

## 文件上传和下载

**上传和下载**

参考：

[springboot同时接收上传的文件和json_springboot json方式可以接受文件吗-CSDN博客](https://blog.csdn.net/weixin_39736281/article/details/119136126)

[springboot实现返回文件流_java_脚本之家](https://www.jb51.net/article/241277.htm)

```java
// 下载时，声明 produces 
@PostMapping(value = "/generate", produces = "application/octet-stream")
public void generate(@RequestPart("param") Param param,
                            @RequestPart("nodeId") MultipartFile nodeId,
                            HttpServletResponse response) throws Exception {
    String content = "";
    // 使用输出流，下载
    ServletOutputStream outputStream = response.getOutputStream();
    // 添加响应头（必填），这样客户端才知道这是文件流
    response.addHeader("Content-Disposition", "attachment; filename=" + URLEncoder.encode("fielname", StandardCharsets.UTF_8.name()));
    outputStream.write(content.getBytes(StandardCharsets.UTF_8));
    outputStream.close();
}
```

**注意**

> 客户端调用文件上传接口
> 
> 文件 + json 参数的形式，那么以表单 form-data 形式提交
> 
> 参数名和内容类型：content-type 



## OSHI

获取宿主机的硬件信息。

**引入依赖**

```xml
<dependency>
    <groupId>com.github.oshi</groupId>
    <artifactId>oshi-core</artifactId>
    <version>${oshi-version}</version>
</dependency>
```

**使用**

```java
import cn.hutool.core.collection.CollectionUtil;
import cn.hutool.core.util.StrUtil;
import lombok.extern.slf4j.Slf4j;
import oshi.SystemInfo;
import oshi.hardware.*;

import java.util.List;

/**
 * @Description 功能描述
 */
@Slf4j
public class OshiUtil {

    /**
     * 获取 cpu id
     * @return
     */
    public static String cpuId() {
        SystemInfo systemInfo = new SystemInfo();
        HardwareAbstractionLayer hardwareInfo = systemInfo.getHardware();
        CentralProcessor processor = hardwareInfo.getProcessor();
        CentralProcessor.ProcessorIdentifier processorIdentifier = processor.getProcessorIdentifier();
        return processorIdentifier.getProcessorID();
    }

    /**
     * 获取主板 id
     * @return
     */
    public static String mainBoardId() {
        SystemInfo systemInfo = new SystemInfo();
        HardwareAbstractionLayer hardwareInfo = systemInfo.getHardware();
        ComputerSystem computerSystem = hardwareInfo.getComputerSystem();
        return computerSystem.getHardwareUUID();
    }


    /**
     * 获取主板序列号
     * @return
     */
    public static String mainBoardSerialNumber() {
        SystemInfo systemInfo = new SystemInfo();
        HardwareAbstractionLayer hardwareInfo = systemInfo.getHardware();
        ComputerSystem computerSystem = hardwareInfo.getComputerSystem();
        Baseboard baseboard = computerSystem.getBaseboard();
        String serialNumber = baseboard.getSerialNumber();
        return serialNumber;
    }

    /**
     * 获取 mac 地址
     * @return
     */
    public static String macAddress() {
        SystemInfo systemInfo = new SystemInfo();
        HardwareAbstractionLayer hardwareInfo = systemInfo.getHardware();
        List<NetworkIF> networkIFs = hardwareInfo.getNetworkIFs();
        if (CollectionUtil.isNotEmpty(networkIFs)) {
            for (NetworkIF networkIF : networkIFs) {
                String macaddr = networkIF.getMacaddr();
                if (StrUtil.isNotBlank(macaddr) && !"unknown".equals(macaddr)) {
                    return macaddr;
                }
            }
        }
        return "";
    }
}
```



## Kaptcha

Google 的验证码框架
