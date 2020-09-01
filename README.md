# 常用工具


##### 1.如何将一个大的集合分成若干个小的集合
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



##### 1.如何对字符串加密加密

```java
public static void main(String[] args) {
        String encrypt = SecurityUtil.encryptDes("密码", Constants.DB_KEY.getBytes());
        System.out.println(encrypt);
    }
```

























