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

## Easy-ES

对 ES 作 CRUD 的类 Mybatis 框架，[简介 | Easy-Es](https://www.easy-es.cn/pages/7ead0d/)  

## PDF

导出带有水印的 PDF 文件，使用睿印第三方依赖实现。

参考：

[睿印帮助文档 · 语雀](https://www.yuque.com/liuer_doc/rayin)

[Git 地址](https://gitee.com/Rayin/rayin)

[SpringBoot实现pdf添加水印: SpringBoot Pdf生成](https://gitee.com/ninesuntec/pdf-add-watermark)

[历经一个月总结使用java实现pdf文件的电子签字+盖章+防伪二维码+水印+PDF文件加密的全套解决方案: 历经一个月总结使用java实现pdf文件的电子签字+盖章+防伪二维码+水印+PDF文件加密的全套解决方案](https://gitee.com/wanghengjie563135/pdf)

**1、依赖**

```xml
<dependency>
    <groupId>ink.rayin</groupId>
    <artifactId>rayin-htmladapter-openhtmltopdf</artifactId>
    <version>1.1.2</version>
</dependency>
```

**2、HTML 模板**

> 注意：
> 
> 1、声明添加 AlibabaPuHuiTi-Light 字体，
> 
> 2、html 模板中的标签切勿留有空格，否则会被识别成有效字符

```html
<!DOCTYPE html>
<html lang="zh-CN" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8"/>
    <style>
        body {
          font-family: AlibabaPuHuiTi-Light,HanaMinB;
          line-height: 1.2;
        }
        table {
              border-collapse:collapse;
              -fs-table-paginate:paginate;
              page-break-inside:auto;
              table-layout: fixed;
              width: 100%;
              word-wrap: break-word;
              word-break: break-all;
        }
          tr {
            page-break-inside:avoid;
            page-break-after:auto;
        }
          td{
              border: 1px solid #000000;
              font-size: 15px;
          }
          th {
              border: 1px solid #000000;
          }

          .cth {
              text-align: center;
              vertical-align: middle!important;
          }
          .ctd {
              padding-top: 10px;
              padding-bottom: 10px;
              text-align: center;
              vertical-align: middle!important;
          }

        .lcten {
            text-align: left;
            padding-top: 10px;
            padding-bottom: 10px;
            padding-left: 5px;
            padding-right: 5px;
            vertical-align: middle!important;
        }

          .titleDiv {
            text-align: center;
            margin-bottom: 25px;
            font-size: 25px;
          }
          .exportDiv {
            text-align: right;
            margin-bottom: 15px;
          }

          .hobbyDiv {
              margin-top: 30px;
          }
          .hobbySpan {
              font-weight: normal;
          }
          .remarkDiv {
            margin-top: 45px;
            text-align: left;
            font-size: 24px;
            font-weight: bold;
          }
          .signatureDiv {
            text-align: right;
            margin-top: 30px;
          }
          .signatureSpan {
            margin-right: 90px;
            font-size: 14px;
          }
        .page_break { page-break-after:always;}

        @page {
          size: A4 ;
          margin: 1cm;
          margin-bottom: 1cm;
        }
    </style>
</head>
<body>
<div class="titleDiv">
    <span data-th-text="${title}"></span>-人员名单
</div>
<div class="exportDiv">
    日期：<span data-th-text="${exportDate}"></span>
</div>
<div style="width: 100%;">
    <table>
        <thead style="background: lightgrey">
        <tr style="height: 17px;">
            <td style="width: 14%" class="cth">姓名</td>
            <td style="width: 8%" class="cth">年龄</td>
            <td style="width: 12%" class="cth">性别</td>
            <td style="width: 16%" class="cth">简介</td>
            <td style="width: 16%" class="cth">出生日期</td>
        </tr>
        </thead>
        <tr data-th-each="user:${users}" >
            <td class="ctd"><span data-th-text="${user.name}"></span></td>
            <td class="ctd"><span data-th-text="${user.age}"></span></td>
            <td class="ctd"><span data-th-text="${user.sex}"></span></td>
            <td class="lcten"><span data-th-text="${user.remark}"></span></td>
            <td class="ctd"><span data-th-text="${user.birthdate}"></span></td>
        </tr>
    </table>
</div>
<div class="hobbyDiv" style="display: table">
    <div style="display: table-row">
        <div style="display: table-cell; width: 14%"><span style="font-weight: bold">兴趣爱好：</span></div>
        <div style="display: table-cell;letter-spacing: 2px; line-height: 1.4"><span class="hobbySpan" data-th-text="${hobbys}"></span></div>
    </div>
</div>
<div class="remarkDiv"><span>备注：手写</span></div>
<div class="signatureDiv">
    <div class="signatureSpan"><span>签名：</span></div>
</div>
</body>
</html>
```

**3、代码**

> 这意：
> 
> 1、准备字体文件，并且在加载时建议使用绝对路径，防止加载不到字体。
> 
> 2、如果导出的 PDF 文件中的中文变成了 #，要么是没有加载字体文件，要么就是字体文件不支持中文。

```java
import com.alibaba.fastjson2.JSON;
import ink.rayin.htmladapter.base.PdfGenerator;
import ink.rayin.htmladapter.openhtmltopdf.service.PdfBoxGenerator;
import ink.rayin.tools.utils.FileUtil;
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.PDPageContentStream;
import org.apache.pdfbox.pdmodel.font.PDFont;
import org.apache.pdfbox.pdmodel.font.PDType0Font;
import org.apache.pdfbox.pdmodel.graphics.state.PDExtendedGraphicsState;
import org.apache.pdfbox.util.Matrix;

import java.io.*;
import java.nio.charset.StandardCharsets;

public class PdfUtil {
    static PdfGenerator pdfGenerator;

    static {
        try {
            // 字体目录，不要使用相对路径
            String fontPath = "/water/fonts";
            pdfGenerator = new PdfBoxGenerator();
            pdfGenerator.init(fontPath);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static <T> byte[] getBytes(T entity, String htmlLocation) {
        // 向 html 模板绑定数据，并将 html 转为 pdf 字节流
        String html = FileUtil.readToString(new File(htmlLocation), StandardCharsets.UTF_8);
        ByteArrayOutputStream byteArrayOutputStream = pdfGenerator.generatePdfSteamByHtmlStrAndData(
                html, JSON.parseObject(JSON.toJSONString(entity))
        );
        return byteArrayOutputStream.toByteArray();
    }

    public static ByteArrayOutputStream getOutStream(byte[] sourceStream, String waterText) throws IOException {
        // 加载 pdf 字节流
        PDDocument document = PDDocument.load(sourceStream);
        // 获取字体
        PDFont font = getFont(document);
        // 遍历 pdf 所有页
        for (int i = 0; i < document.getNumberOfPages(); i++) {
            PDPage page = document.getPage(i);
            PDPageContentStream contentStream = new PDPageContentStream(
                    document, page, PDPageContentStream.AppendMode.APPEND, true, true
            );
            contentStream.setFont(font, 64);
            contentStream.setNonStrokingColor(232 , 232 , 232 );
            // 添加文字
            contentStream.beginText();
            // 设置文字的旋转角度、x 和 y 的偏移量（可以按照弧度设置，可以将旋转的角度转为弧度再设置）
            Matrix matrix = new Matrix();
            Matrix rotate = Matrix.getRotateInstance(0.8, 160, 250);
            matrix.concatenate(rotate);
            contentStream.setTextMatrix(matrix);
            // 设置透明度（阿尔法通道）
            PDExtendedGraphicsState pdExtendedGraphicsState = new PDExtendedGraphicsState();
            pdExtendedGraphicsState.setNonStrokingAlphaConstant(0.51f);
            pdExtendedGraphicsState.setAlphaSourceFlag(true);
            contentStream.setGraphicsStateParameters(pdExtendedGraphicsState);
            contentStream.showText(waterText);
            contentStream.endText();
            contentStream.close();
        }
        ByteArrayOutputStream result = new ByteArrayOutputStream();
        document.save(result);
        document.close();
        return result;
    }

    private static PDFont getFont(PDDocument document) throws IOException {
        String path = "/fonts/AlibabaPuHuiTi-Light.ttf";
        InputStream font = new FileInputStream(new File(path));
        PDType0Font result = PDType0Font.load(document, font, false);
        return result;
    }

}
```

**4、实体**

```json
{
    "exportDate": "2024-07-05",
    "hobbys": "唱、跳、rap、篮球、music",
    "title": "小黑子",
    "users": [
        {
            "age": 20,
            "birthdate": "2025-10-20",
            "name": "张 0",
            "remark": "这是一行文字这是一行文字这是一行文字这是一行文字",
            "sex": "男"
        },
        {
            "age": 21,
            "birthdate": "2025-10-21",
            "name": "张 1",
            "remark": "这是一行文字这是一行文字这是一行文字这是一行文字",
            "sex": "男"
        },
        {
            "age": 22,
            "birthdate": "2025-10-22",
            "name": "张 2",
            "remark": "这是一行文字这是一行文字这是一行文字这是一行文字",
            "sex": "男"
        }
    ]
}
```

**5、使用**

```java
public class Main {
    public static void main(String[] args) throws IOException {
        PdfExport entity = new PdfExport();
        entity.setUsers(getUsers());
        byte[] pdfSourceDatas = PdfUtil.getBytes(entity, "/template/template.html");
        ByteArrayOutputStream outStream = PdfUtil.getOutStream(pdfSourceDatas, "中文123pdf");
        OutputStream out = new FileOutputStream(new File("/file/6666.pdf"));
        out.write(outStream.toByteArray());
    }
    private static List<User> getUsers() {
        List<User> users = new ArrayList<>();
        for (int i = 0; i < 10; ++i) {
            User user = new User();
            user.setName("张 " + i);
            user.setAge( 20 + i);
            user.setSex("男");
            user.setRemark("这是一行文字这是一行文字这是一行文字这是一行文字");
            user.setBirthdate("2025-10-2" + i);
            users.add(user);
        }
        return users;
    }
}
```

**6、效果**

![](https://assets.shiwokuaile.top/pic/202408/6a6d9eb4cd5931761c750e197f216dcd5fe4dd.png)
