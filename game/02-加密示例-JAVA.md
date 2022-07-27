<h2>加密示例-JAVA</h2>

#### 描述:

> 商户在对接TikDP平台时需要针对接口进行sign的安全校验,sign的规则如下
> 所有参数字段按照键值排序后经过n次MD5加密
>
> * sign=n次MD5("key1=value1&key2=value2")
> * 加密的次数n直接拼接在sign最后

<h4>示例代码：</h4>

```java

import cn.hutool.core.map.MapUtil;
import org.apache.commons.lang3.math.NumberUtils;
import org.springframework.util.DigestUtils;

import java.math.BigInteger;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.*;

/**
 * MD5加密示例
 */
public class Md5Utils {

    /**
     * 测试
     * @param args 
     */
    public static void main(String[] args) {

        Map<String, Object> requestParameter = new HashMap<>();
        requestParameter.put("userId", "100000950006");
        requestParameter.put("serverId", "100000950007");
        requestParameter.put("roleId", "100000950008");

        System.out.println(generateSign(requestParameter));

    }

    /**
     * java自带jar工具 java.security.MessageDigest 实现
     *
     * @param text 要加密的字符串
     * @return 16进制
     */
    public static String javaStrToMD5(String text) {

        // 进制：比如 16、32
        int rad = 16;
        // 否转换为大写
        boolean isUpp = false;

        MessageDigest messageDigest = null;
        try {
            //通过MessageDigest类来的静态方法getInstance获取MessageDigest对象
            messageDigest = MessageDigest.getInstance("MD5");
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
        // 4、获取明文字符串对应的字节数组
        byte[] input = text.getBytes();
        // 5、执行加密
        messageDigest.update(input);
        //这里也可以直接使用byte[] output = messageDigest.digest(input)方法来进行加密，就省略了上面的update方法了
        byte[] output = messageDigest.digest();
        // 6、创建BigInteger对象
        // signum为1表示正数、-1表示负数、0表示0。不写默认表示正数
        int signum = 1;
        BigInteger bigInteger = new BigInteger(signum, output);
        // 7、按照16进制(或32进制)将bigInteger转为字符串
        return isUpp ? bigInteger.toString(rad).toUpperCase() : bigInteger.toString(rad);
    }

    /**
     * spring自带的工具 org.springframework.util.DigestUtils 实现
     *
     * @param text 要加密的字符串
     * @return 16进制
     */
    public static String springStrToMD5(String text) {
        return DigestUtils.md5DigestAsHex(text.getBytes());
    }

    /**
     * 生成签名:所有字段按照键排序,经过n次md5加密,加密次数直接拼接在最后
     *
     * @param parameter
     * @return
     */
    public static String generateSign(Map<String, Object> parameter) {

        // 所有字段按照键排序
        TreeMap<String, Object> requestParameter = new TreeMap<>(String::compareTo);
        Optional.ofNullable(parameter).orElse(MapUtil.empty()).forEach((k, v) -> requestParameter.put(k, v));

        StringBuffer signParameter = new StringBuffer();
        requestParameter.forEach((k, v) -> signParameter.append(k).append("=").append(v).append("&"));

        if (signParameter.length() > NumberUtils.INTEGER_ZERO) {
            signParameter.deleteCharAt(signParameter.length() - 1);
        }

        // 经过n次md5加密,加密次数直接拼接在最后
        String sign = signParameter.toString();
        int n = new Random().nextInt(3) + 2;
        for (int i = 0; i < n; i++) {
            sign = javaStrToMD5(sign);
        }

        return String.format("%s%s", sign, n);
    }

}
```



