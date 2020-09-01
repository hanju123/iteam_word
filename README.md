# 常用工具方法


#### 1.如何将一个大的集合分成若干个小的集合
```java
 /**
     * 批量分批次
     * @param list
     * @param len 大小
     * @return
     */
    private static List<List<MaterialVo>> splitList(List<MaterialVo> list, int len) {
        if (list == null || list.size() == 0 || len < 1) {
            return null;
        }
        List<List<MaterialVo>> result = new ArrayList<List<MaterialVo>>();
        int size = list.size();
        int count = (size + len - 1) / len;
        for (int i = 0; i < count; i++) {
            List<MaterialVo> subList = list.subList(i * len, ((i + 1) * len > size ? size : len * (i + 1)));
            result.add(subList);
        }
        return result;
    }


```



#### 2.如何对字符串加密加密

```java
public static void main(String[] args) {
        String encrypt = SecurityUtil.encryptDes("密码", Constants.DB_KEY.getBytes());
        System.out.println(encrypt);
    }
```

#### 3.如何获取前端得IP地址

```java
    /**
     * 获取请求ip以及
     * @param request
     * @return
     */
public static String getIpAddr( HttpServletRequest request) {
        String ipAddress = null;
        //ipAddress = request.getRemoteAddr();
        ipAddress = request.getHeader("x-forwarded-for");
        if(ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("Proxy-Client-IP");
        }
        if(ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("WL-Proxy-Client-IP");
        }
        if(ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getRemoteAddr();
            if(ipAddress.equals("127.0.0.1")){
                //根据网卡取本机配置的IP
                InetAddress inet=null;
                try {
                    inet = InetAddress.getLocalHost();
                } catch (UnknownHostException e) {
                    e.printStackTrace();
                }
                ipAddress= inet.getHostAddress();
            }

        }
        //对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
        if(ipAddress!=null && ipAddress.length()>15){ //"***.***.***.***".length() = 15
            if(ipAddress.indexOf(",")>0){
                ipAddress = ipAddress.substring(0,ipAddress.indexOf(","));
            }
        }
        return ipAddress;
    }


```



























