# 原生实现

## 执行宿主机的命令

Java 实现（Linux + Window）

```java
import cn.hutool.core.util.StrUtil;
import lombok.extern.slf4j.Slf4j;

import java.io.*;
import java.net.InetAddress;
import java.net.NetworkInterface;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;

/**
 * @ClassName DeviceUtil
 * @Description 来源：https://www.cnblogs.com/kdy11/p/8243649.html
 */
@Slf4j
public class DeviceUtil {

    /**
     * 获取当前操作系统名称. return 操作系统名称 例如:windows xp,linux 等.
     */
    public static String getOSName() {
        return System.getProperty("os.name").toLowerCase();
    }

    ///////////////////////////////////////////// uuid \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\

    /**
     * 获取主板 UUID （windos）
     * @return
     */
    private static String getMainboardUUIDFromWindow() {
         try {
             String[] comand = {"cmd", "/c", "wmic csproduct list full | findstr UUID"};
             Process process = Runtime.getRuntime().exec(comand);
             process.getOutputStream().close();
             InputStream inputStream = process.getInputStream();
             BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
             String line;
             StringBuilder builder = new StringBuilder();
             while ((line =  reader.readLine()) != null) {
                 builder.append(line);
             }
             inputStream.close();
             process.getOutputStream().close();
             return builder.delete(0, 5).toString();
         } catch (Exception e) {
             log.error("【DeviceUtil.getMainboardUUIDFromWindow】get mainboard id fail", e);
        }
         return "";
    }

    /**
     * 获取主板 UUID （Linux）
     * @return
     */
    private static String getMainBoardUUIDFromLinux() {
        try {
            String[] shell = {"/bin/bash", "-c", "dmidecode -s system-uuid"};
            Process process = Runtime.getRuntime().exec(shell);
            InputStream inputStream = process.getInputStream();
            BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
            String line;
            StringBuilder builder = new StringBuilder();
            while ((line =  reader.readLine()) != null) {
                builder.append(line);
            }
            inputStream.close();
            process.getOutputStream().close();
            return builder.toString();
        } catch (Exception e) {
            log.error("【DeviceUtil.getMainBoardUUIDFromLinux】get mainborad id fail", e);
        }
        return "";
    }

    /**
     * 获取主板 UUID
     * @return
     */
    public static String getMainBoardUUID() {
        String os = getOSName();
        if (StrUtil.isNotBlank(os)) {
            if (os.contains("windows")) {
                return getMainboardUUIDFromWindow();
            } else if (os.contains("linux")) {
                return getMainBoardUUIDFromLinux();
            }
        }
        return "";
    }

    ///////////////////////////////////////////// MainBoard Serial Number \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\



    /**
     * 获取 Windows 主板序列号
     * @return
     */
    private static String getMainBoardSerialNumberFromWindows() {
        StringBuilder result = new StringBuilder();
        try {
            File file = File.createTempFile("realhowto", ".vbs");
            file.deleteOnExit();
            FileWriter fw = new java.io.FileWriter(file);
            // 通过 vbs 获取
            String vbs = "Set objWMIService = GetObject(\"winmgmts:\\\\.\\root\\cimv2\")\n"
                    + "Set colItems = objWMIService.ExecQuery _ \n"
                    + "   (\"Select * from Win32_BaseBoard\") \n"
                    + "For Each objItem in colItems \n"
                    + "    Wscript.Echo objItem.SerialNumber \n"
                    + "    exit for  ' do the first cpu only! \n" + "Next \n";
            fw.write(vbs);
            fw.close();
            Process p = Runtime.getRuntime().exec("cscript //NoLogo " + file.getPath());
            BufferedReader input = new BufferedReader(new InputStreamReader(p.getInputStream()));
            String line;
            while ((line = input.readLine()) != null) {
                result.append(line);
            }
            input.close();
        } catch (Exception e) {
            log.error("【DeviceUtil.getMainBoardSerialNumberFromWindows】get mainboard serial number fail", e);
        }
        return result.toString().trim();
    }

    /**
     * 获取 Linux 主板序列号
     * @return
     */
    private static String getMainBoardSerialNumberFromlinux() {
        StringBuilder result = new StringBuilder();
        String maniBord_cmd = "dmidecode | grep 'Serial Number' | awk '{print $3}' | tail -1";
        try {
            // 管道
            Process p = Runtime.getRuntime().exec(new String[] { "sh", "-c", maniBord_cmd });
            BufferedReader br = new BufferedReader(new InputStreamReader(p.getInputStream()));
            String line;
            while ((line = br.readLine()) != null) {
                result.append(line);
                break;
            }
            br.close();
        } catch (IOException e) {
            log.error("【DeviceUtil.getMainBoardSerialNumberFromlinux】get mainboard serial number fail", e);
        }
        return result.toString();
    }

    /**
     * 获取主板序列号
     * @return
     */
    public static String getMainBoardSerialNumber() {
        String os = getOSName();
        if (StrUtil.isNotBlank(os)) {
            if (os.contains("windows")) {
                return getMainBoardSerialNumberFromWindows();
            } else if (os.contains("linux")) {
                return getMainBoardSerialNumberFromlinux();
            }
        }
        return "";
    }


    ///////////////////////////////////////////// Mac \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\

    /**
     * 获取 unix 网卡的 mac 地址. 非 windows 的系统默认调用本方法获取. 如果有特殊系统请继续扩充新的取 mac 地址方法.
     *
     * @return mac地址
     */
    private static String getMacFromLinux() {
        String mac = "";
        BufferedReader bufferedReader = null;
        try {
            // linux下的命令，一般取 eth0 作为本地主网卡
            Process process =Runtime.getRuntime().exec("ifconfig eth0");
            // 显示信息中包含有 mac 地址信息
            bufferedReader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String line = null;
            int index = -1;
            while ((line = bufferedReader.readLine()) != null) {
                // 寻找标示字符串 [hwaddr]
                index = line.toLowerCase().indexOf("hwaddr");
                if (index >= 0) {// 找到了
                    // 取出 mac 地址并去除2边空格
                    mac = line.substring(index + "hwaddr".length() + 1).trim();
                    break;
                }
            }
        } catch (IOException e) {
            log.error("【DeviceUtil.getMacFromLinux】get mac fail", e);
        } finally {
            try {
                if (bufferedReader != null) {
                    bufferedReader.close();
                }
            } catch (IOException e1) {
                log.error("【DeviceUtil.getMacFromLinux】bufferedReader close fail", e1);
            }
        }
        return mac;
    }

    /**
     * 获取 widnows 网卡的 mac 地址.
     *
     * @return mac地址
     */
    private static String getMacFromWindows() {
        List<String> macList = new ArrayList<>();
        try {
            Enumeration<NetworkInterface> netInterfaces = (Enumeration<NetworkInterface>) NetworkInterface.getNetworkInterfaces();
            while (netInterfaces.hasMoreElements()) {
                NetworkInterface ni  = netInterfaces.nextElement();
                // ----------特定情况，可以考虑用ni.getName判断
                // 遍历所有ip
                Enumeration<InetAddress> ips = ni.getInetAddresses();
                while (ips.hasMoreElements()) {
                    InetAddress ip = ips.nextElement();
                    if (!ip.isLoopbackAddress() // 非127.0.0.1
                            && ip.getHostAddress().matches("(\\d{1,3}\\.){3}\\d{1,3}")) {
                        macList.add(getMacFromBytes(
                                ni.getHardwareAddress()
                                ));
                    }
                }
            }
        } catch (Exception e) {
            log.error("【DeviceUtil.getMacFromWindows】get mac fail", e);
        }
        if (macList.size() > 0) {
            return macList.get(0);
        } else {
            return "";
        }

    }

    /**
     *
     * @param bytes
     * @return
     */
    private static String getMacFromBytes(byte[] bytes) {
        StringBuffer mac = new StringBuffer();
        byte currentByte;
        boolean first = false;
        for (byte b : bytes) {
            if (first) {
                mac.append("-");
            }
            currentByte = (byte) ((b & 240) >> 4);
            mac.append(Integer.toHexString(currentByte));
            currentByte = (byte) (b & 15);
            mac.append(Integer.toHexString(currentByte));
            first = true;
        }
        return mac.toString().toUpperCase();
    }

    /**
     * 获取 Mac 地址
     * @return
     */
    @Deprecated
    public static String getMac() {
        String os = getOSName();
        if (StrUtil.isNotBlank(os)) {
            if (os.contains("windows")) {
                return getMacFromWindows();
            } else if (os.contains("linux")) {
                return getMacFromLinux();
            }
        }
        return "";
    }


    ///////////////////////////////////////////// CPU ID \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\
    /**
     * 获取 Windows CPU ID
     *
     * @return
     */
    private static String getCpuIdFromWindows() {
        StringBuilder result = new StringBuilder();
        try {
            File file = File.createTempFile("tmp", ".vbs");
            file.deleteOnExit();
            FileWriter fw = new java.io.FileWriter(file);
            String vbs = "Set objWMIService = GetObject(\"winmgmts:\\\\.\\root\\cimv2\")\n"
                    + "Set colItems = objWMIService.ExecQuery _ \n"
                    + "   (\"Select * from Win32_Processor\") \n"
                    + "For Each objItem in colItems \n"
                    + "    Wscript.Echo objItem.ProcessorId \n"
                    + "    exit for  ' do the first cpu only! \n" + "Next \n";
            fw.write(vbs);
            fw.close();
            Process p = Runtime.getRuntime().exec("cscript //NoLogo " + file.getPath());
            BufferedReader input = new BufferedReader(new InputStreamReader(p.getInputStream()));
            String line;
            while ((line = input.readLine()) != null) {
                result.append(line);
            }
            input.close();
            file.delete();
        } catch (Exception e) {
            log.error("【DeviceUtil.getCpuIdFromWindows】get cpu id fail", e);
        }
        return result.toString().trim();
    }

    /**
     * 获取 Linux CPU ID
     * @return
     */
    private static String getCpuIdFromLinux() {
        String result = "";
        String CPU_ID_CMD = "dmidecode";
        try {
            Process p = Runtime.getRuntime().exec(new String[]{ "sh", "-c", CPU_ID_CMD });// 管道
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(p.getInputStream()));
            String line = null;
            int index = -1;
            while ((line = bufferedReader.readLine()) != null) {
                // 寻找标示字符串[hwaddr]
                index = line.toLowerCase().indexOf("uuid");
                if (index >= 0) {// 找到了
                    // 取出mac地址并去除2边空格
                    result = line.substring(index + "uuid".length() + 1).trim();
                    break;
                }
            }

        } catch (IOException e) {
            log.error("【DeviceUtil.getCpuIdFromLinux】get cpu id fail", e);
        }
        return result.trim();
    }

    /**
     * 获取 CPU ID
     * @return
     * @throws InterruptedException
     */
    public static String getCpuId()  {
        String os = getOSName();
        if (StrUtil.isNotBlank(os)) {
            if (os.contains("windows")) {
                return getCpuIdFromWindows();
            } else if (os.contains("linux")) {
                return getCpuIdFromLinux();
            }
        }
        return "";
    }

    /**
     * 获取 mac 地址
     * @return
     */
    public static String getMacAddress() {
        try {
            Enumeration<NetworkInterface> allNetInterfaces = NetworkInterface.getNetworkInterfaces();
            byte[] mac = null;
            while (allNetInterfaces.hasMoreElements()) {
                NetworkInterface netInterface = (NetworkInterface) allNetInterfaces.nextElement();
                if (netInterface.isLoopback() || netInterface.isVirtual() || netInterface.isPointToPoint() || !netInterface.isUp()) {
                    continue;
                } else {
                    mac = netInterface.getHardwareAddress();
                    if (mac != null) {
                        StringBuilder sb = new StringBuilder();
                        for (int i = 0; i < mac.length; i++) {
                            sb.append(String.format("%02X%s", mac[i], (i < mac.length - 1) ? "-" : ""));
                        }
                        if (sb.length() > 0) {
                            return sb.toString();
                        }
                    }
                }
            }
        } catch (Exception e) {
            log.error("【DeviceUtil.getMacAddress】get mac fail", e);
        }
        return "";
    }

}
```

## RSA 加解密

生成公钥和私钥、加密与解密。

```java
import cn.hutool.core.util.StrUtil;
import com.google.gson.Gson;
import com.him.security.core.verify.AuthorizedRecord;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.codec.binary.Base64;

import javax.crypto.Cipher;
import java.io.ByteArrayOutputStream;
import java.nio.charset.StandardCharsets;
import java.security.*;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.HashMap;
import java.util.Map;

/**
 * 来源：
 * https://www.cnblogs.com/Likfees/p/16444989.html#rsa%E5%B7%A5%E5%85%B7%E7%B1%BB2
 * https://www.cnblogs.com/zouzhijun/p/16920818.html
 */
@Slf4j
public class SecurityUtil {

    /**
     * RSA最大加密明文大小 2048/8-11
     */
    private static final int MAX_ENCRYPT_BLOCK = 117;

    /**
     * RSA最大解密密文大小 2048/8
     */
    private static final int MAX_DECRYPT_BLOCK = 128;

    /**
     * 定义加密方式
     */
    private static final String KEY_RSA = "RSA";

    /**
     * RSA 私钥加密
     * @param secreteText
     * @param privateKey
     * @return
     */
    public static String encryptByPrivateWithRsa(String secreteText, String privateKey) {
        return encryptByPrivate(secreteText, privateKey, KEY_RSA);
    }

    /**
     * RSA 公钥解密
     * @param secreteText
     * @param publicKey
     * @return
     */
    public static String decryptByPublicWithRsa(String secreteText, String publicKey) {
        return decryptByPublic(secreteText, publicKey, KEY_RSA);
    }

    /**
     * RSA 公钥解密并转为 AuthorizedRecord 对象
     * @param secreteText
     * @param publicKey
     * @return
     */
    public static AuthorizedRecord decryptToAuthorizedRecord(String secreteText, String publicKey) {
        String text = decryptByPublic(secreteText, publicKey, KEY_RSA);
        if (StrUtil.isNotBlank(text)) {
            try {
                Gson gson = new Gson();
                AuthorizedRecord result = gson.fromJson(text, AuthorizedRecord.class);
                return result;
            } catch (Exception e) {
                log.error("【SecurityUtil.decryptToAuthorizedRecord】gson format fail, text={}", text, e);
            }
        }
        return new AuthorizedRecord();
    }

    /**
     * 公钥加密  如果大于245则分段加密
     * @param encryptingStr
     * @param publicKeyStr
     * @param algorithm
     * @return
     */
    private static String encryptByPublic(String encryptingStr, String publicKeyStr, String algorithm) {
        try {
            // 将公钥由字符串转为UTF-8格式的字节数组
            byte[] publicKeyBytes = Base64.decodeBase64(publicKeyStr);
            // 获得公钥
            X509EncodedKeySpec keySpec = new X509EncodedKeySpec(publicKeyBytes);
            // 取得待加密数据
            byte[] data = encryptingStr.getBytes(StandardCharsets.UTF_8);
            KeyFactory factory;
            factory = KeyFactory.getInstance(algorithm);
            PublicKey publicKey = factory.generatePublic(keySpec);
            // 对数据加密
            Cipher cipher = Cipher.getInstance(factory.getAlgorithm());
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);
            int inputLen = data.length;
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            int offSet = 0;
            byte[] cache;
            int i = 0;
            // 对数据分段加密
            while (inputLen - offSet > 0) {
                if (inputLen - offSet > MAX_ENCRYPT_BLOCK) {
                    cache = cipher.doFinal(data, offSet, MAX_ENCRYPT_BLOCK);
                } else {
                    cache = cipher.doFinal(data, offSet, inputLen - offSet);
                }
                out.write(cache, 0, cache.length);
                i++;
                offSet = i * MAX_ENCRYPT_BLOCK;
            }
            byte[] encryptedData = out.toByteArray();
            out.close();
            // 返回加密后由Base64编码的加密信息
            return new String(Base64.encodeBase64(encryptedData));
        } catch (Exception e) {
            e.printStackTrace();
        }

        return "";
    }

    /**
     * 私钥解密 如果大于256则分段解密
     * @param encryptedStr
     * @param privateKeyStr
     * @param algorithm
     * @return
     */
    private static String decryptByPrivate(String encryptedStr, String privateKeyStr, String algorithm) {
        try {
            // 对私钥解密
            byte[] privateKeyBytes = Base64.decodeBase64(privateKeyStr);
            // 获得私钥
            PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(privateKeyBytes);
            // 获得待解密数据
            byte[] data = Base64.decodeBase64(encryptedStr);
            KeyFactory factory = KeyFactory.getInstance(algorithm);
            PrivateKey privateKey = factory.generatePrivate(keySpec);
            // 对数据解密
            Cipher cipher = Cipher.getInstance(factory.getAlgorithm());
            cipher.init(Cipher.DECRYPT_MODE, privateKey);
            int inputLen = data.length;
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            int offSet = 0;
            byte[] cache;
            int i = 0;
            // 对数据分段解密
            while (inputLen - offSet > 0) {
                if (inputLen - offSet > MAX_DECRYPT_BLOCK) {
                    cache = cipher.doFinal(data, offSet, MAX_DECRYPT_BLOCK);
                } else {
                    cache = cipher.doFinal(data, offSet, inputLen - offSet);
                }
                out.write(cache, 0, cache.length);
                i++;
                offSet = i * MAX_DECRYPT_BLOCK;
            }
            byte[] decryptedData = out.toByteArray();
            out.close();
            // 返回UTF-8编码的解密信息
            return new String(decryptedData, StandardCharsets.UTF_8);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return "";
    }


    /**
     * 私钥加密  如果大于245则分段加密
     * @param encryptingStr
     * @param privateKeyStr
     * @param algorithm
     * @return
     */
    private static String encryptByPrivate(String encryptingStr, String privateKeyStr, String algorithm) {
        try {
            byte[] privateKeyBytes = Base64.decodeBase64(privateKeyStr);
            // 获得私钥
            PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(privateKeyBytes);
            // 取得待加密数据
            byte[] data = encryptingStr.getBytes(StandardCharsets.UTF_8);
            KeyFactory factory = KeyFactory.getInstance(algorithm);
            PrivateKey privateKey = factory.generatePrivate(keySpec);
            // 对数据加密
            Cipher cipher = Cipher.getInstance(factory.getAlgorithm());
            cipher.init(Cipher.ENCRYPT_MODE, privateKey);
            int inputLen = data.length;
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            int offSet = 0;
            byte[] cache;
            int i = 0;
            // 对数据分段加密
            while (inputLen - offSet > 0) {
                if (inputLen - offSet > MAX_ENCRYPT_BLOCK) {
                    cache = cipher.doFinal(data, offSet, MAX_ENCRYPT_BLOCK);
                } else {
                    cache = cipher.doFinal(data, offSet, inputLen - offSet);
                }
                out.write(cache, 0, cache.length);
                i++;
                offSet = i * MAX_ENCRYPT_BLOCK;
            }
            byte[] encryptedData = out.toByteArray();
            out.close();
            // 返回加密后由Base64编码的加密信息
            return new String(Base64.encodeBase64(encryptedData));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "";
    }


    /**
     * 公钥解密 如果大于256则分段解密
     * @param encryptedStr
     * @param publicKeyStr
     * @param algorithm
     * @return
     */
    private static String decryptByPublic(String encryptedStr, String publicKeyStr, String algorithm) {
        try {
            // 对公钥解密
            byte[] publicKeyBytes = Base64.decodeBase64(publicKeyStr);
            // 取得公钥
            X509EncodedKeySpec keySpec = new X509EncodedKeySpec(publicKeyBytes);
            // 取得待加密数据
            byte[] data = Base64.decodeBase64(encryptedStr);
            KeyFactory factory = KeyFactory.getInstance(algorithm);
            PublicKey publicKey = factory.generatePublic(keySpec);
            // 对数据解密
            Cipher cipher = Cipher.getInstance(factory.getAlgorithm());
            cipher.init(Cipher.DECRYPT_MODE, publicKey);
            int inputLen = data.length;
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            int offSet = 0;
            byte[] cache;
            int i = 0;
            // 对数据分段解密
            while (inputLen - offSet > 0) {
                if (inputLen - offSet > MAX_DECRYPT_BLOCK) {
                    cache = cipher.doFinal(data, offSet, MAX_DECRYPT_BLOCK);
                } else {
                    cache = cipher.doFinal(data, offSet, inputLen - offSet);
                }
                out.write(cache, 0, cache.length);
                i++;
                offSet = i * MAX_DECRYPT_BLOCK;
            }
            byte[] decryptedData = out.toByteArray();
            out.close();
            // 返回UTF-8编码的解密信息
            return new String(decryptedData, StandardCharsets.UTF_8);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "";
    }

    /**
     * base64 编码
     * @param key
     * @return
     */
    public static String encryptBASE64(byte[] key) {
        return Base64.encodeBase64String(key);
    }

    /**
     * 随机生成密钥对
     * @param algorithm 非对称算法名称（RSA）
     * @param keySize 512/1024/2048
     * @throws Exception
     * @return 密钥对（key → 公钥，value → 私钥）
     */
    public static Map<String, String> genKeyPair(String algorithm, int keySize) throws Exception {
        Map<String, String> keyMap = new HashMap<>();
        // KeyPairGenerator类用于生成公钥和私钥对，基于RSA算法生成对象
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance(algorithm);
        // 初始化密钥对生成器
        keyPairGen.initialize(keySize, new SecureRandom());
        // 生成一个密钥对
        KeyPair keyPair = keyPairGen.generateKeyPair();
        // 得到私钥
        PrivateKey privateKey = keyPair.getPrivate();
        // 得到公钥
        PublicKey publicKey = keyPair.getPublic();
        String publicKeyString = encryptBASE64(publicKey.getEncoded());
        // 得到私钥字符串
        String privateKeyString = encryptBASE64(privateKey.getEncoded());
        // 去掉换行符
        keyMap.put(
                StrUtil.removeAllLineBreaks(publicKeyString), StrUtil.removeAllLineBreaks(privateKeyString)
        );
        return keyMap;
    }
}
```

**注意**

> 实现过程，可能会出现：“RSA 加密过程出现数据不可以超过 117 的异常”。
> 
> 参考：https://blog.51cto.com/u_1472521/3714682?articleABtest=0
> 
> 设置分段加密，并且长度不超过 117，解密也如此，但长度可以设置 128

## IO 操作

> 编码：将字符转变成字节的过程。
> 
> 解码：将字节转变成字符的过程。
> 
> 在 UTF-8 中，换行符号 '\n' 是占据两个字节（Byte）
> 
> '\r'：回车符号
> 
> 在 UTF-8 中，一个中文字符占据三个字节
> 
> 在 GBK 中，一个中文字符占据两个字节

**读取末行**

```java
import cn.hutool.core.io.FileUtil;
import cn.hutool.core.util.StrUtil;
import lombok.extern.slf4j.Slf4j;

import java.io.*;
import java.nio.charset.StandardCharsets;

/**
 * @Description
 * 来源：https://www.cnblogs.com/yejg1212/archive/2013/05/15/3079307.html
 */
@Slf4j
public class FileUtil {

    /**
     *
     * @param filePath
     * @param charset
     * @return
     */
    public static String readTailFirst(String filePath, String charset) {
        File touch = FileUtil.touch(new File(filePath));
        return readTailFirst(touch, charset);
    }

    /**
     * 读取最后部位空的一行
     * @param file    file path
     * @param charset character
     */
    public static String readTailFirst(File file, String charset) {
        try (RandomAccessFile rf = new RandomAccessFile(file, "r")) {
            long fileLength = rf.length();
            // 返回此文件中的当前偏移量
            long start = rf.getFilePointer();
            long readIndex = start + fileLength -1;
            while (readIndex > start) {
                // 设置偏移量为文件末尾
                rf.seek(readIndex);
                int c = rf.read();
                if (c == '\n' || c == '\r') {
                    String line = rf.readLine();
                    if (StrUtil.isNotBlank(line)) {
                        return new String(line.getBytes(StandardCharsets.ISO_8859_1), charset);
                    }
                    readIndex--;
                }
                readIndex --;
                if (readIndex == 0) { // 当文件指针退至文件开始处，输出第一行
                    rf.seek(readIndex);
                    String line = rf.readLine();
                    if (StrUtil.isNotBlank(line)) {
                        return new String(line.getBytes(StandardCharsets.ISO_8859_1), charset);
                    }
                }
            }
        } catch (Exception e) {
            log.error("【FileUtil.readTailFirst】file={}, charset={}", file, charset, e);
        }
        return "";
    }

    /**
     * 追加写入文件
     * @param file
     * @param content
     * @param charset
     */
    public static void writeAppend(File file, String content, String charset) {
        try (
                BufferedWriter writer = new BufferedWriter(
                        new OutputStreamWriter(
                                new FileOutputStream(file, true), charset
                        )
                )
        ){
            writer.write(content);
            writer.write('\n');
        } catch (Exception e) {
            log.error("【FileUtil.writeAppend】file={}, content={}, charset={}", file, content, charset, e);
        }
    }
}
```

**遇到的问题**

> 问题：RandomAccessFile 类读取中文字符乱码
> 
> 参考：https://blog.csdn.net/qq_51214556/article/details/123431290
> 
> 分析：
> 
> 在 IO 的源码中，能够找到 String 默认的字符集会从 JVM 的设定中拿到，如果没有就设置为 UTF-8，所以通过 BufferReader 和 FileInpuStream 读取文件时会默认使用 UTF-8 来进行编码和解码。
> 
> 至于随机文件读取中文乱码，在于读取字节时，限制了大小范围（0~255），也就是只能读一个字节，也就造成编码时长度不一致的问题，因此需要用单个字节大小的字符集 “IOS-8859-1” 来编码，然后再用文件原有的编码方式来解码。
> 
> 解决：
> 
> 先用 “IOS-8859-1” 编码，然后再用 UTF-8 解码（虽然文件的编码格式是 UTF-8，但是用 UTF-8 编码再解码旧出现了乱码）。
> 
> 其他：
> 
> BufferReader 和 FileInpuStream 在不指定编码和解码字符，字符是能够保真输出。