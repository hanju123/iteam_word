# 常用工具方法


## 1.如何将一个大的集合分成若干个小的集合
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



## 2.如何对字符串加密加密

```java
public static void main(String[] args) {
        String encrypt = SecurityUtil.encryptDes("密码", Constants.DB_KEY.getBytes());
        System.out.println(encrypt);
    }
```

## 3.如何获取前端得IP地址

```java
 
    /**
     * ip常量
     */
    private static final  String IP="127.0.0.1";

    /**
     * 获取请求ip
     * @param request
     * @return
     */
    public static String getIpAddr( HttpServletRequest request) {
        String ipAddress = null;
        ipAddress = request.getHeader("x-forwarded-for");
        boolean flag2= ipAddress.length() == 0;
        boolean flag3= "unknown".equalsIgnoreCase(ipAddress);
        if(ipAddress == null || flag2 || flag3) {
            ipAddress = request.getHeader("Proxy-Client-IP");
        }
        if(ipAddress == null || flag2 || flag3) {
            ipAddress = request.getHeader("WL-Proxy-Client-IP");
        }
        if(ipAddress == null || flag2 || flag3) {
            ipAddress = request.getRemoteAddr();
            if(IP.equals(ipAddress)){
                try {
                    InetAddress inet = InetAddress.getLocalHost();
                    ipAddress= inet.getHostAddress();
                } catch (UnknownHostException e) {
                    LogUtil.sysEventError(e.getMessage());
                }

            }

        }
        //对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
        //"***.***.***.***".length() = 15
        if(ipAddress!=null && ipAddress.length()>15){
            if(ipAddress.indexOf(",")>=1){
                ipAddress = ipAddress.substring(0,ipAddress.indexOf(","));
            }
        }
        return ipAddress;
    }


```

## 4.如何去掉数据库的空格与参数比对

```java
select count(1) from LP_MST_WLMXB where isnull(rtrim(ltrim(name)),'')=isnull(rtrim(ltrim('60内开扇D(塑料型材)15cm-样空格测 试')),'')
```

## 5.mapper文件里处理if标签中的对比字符串的问题

```java
		<if test='"已过期" ==limit and !"" ==limit'>
            and getdate() &gt; a.time_to
        </if>
        <if test='"未过期" ==limit and !"" ==limit'>
            and getdate() &lt; a.time_to
        </if>
        
//报错的
/*Caused by: org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: java.lang.NumberFormatException: For input string: "未过期"
### Cause: java.lang.NumberFormatException: For input string: "未过期"
*/
//换一种写法还是解决不了
            
      <if test='limit == "已过期"  and !limit == ""'>
            and getdate() &gt; a.time_to
        </if>
        <if test='limit == "未过期"  and !limit == ""'>
            and getdate() &lt; a.time_to
        </if> 
        
            
//换一种思路，改成数字比较          
         <if test='limit == "1" '>
            and getdate() &gt; a.time_to
        </if>
        <if test='limit == "0"'>
            and getdate() &lt; a.time_to
        </if>
            
//原因分析
//1.中文可能存在乱码问题，所以比较会出现问题
            
            
            
            
```

## 6.批量插入的sql语句

```java
 update LP_RAL_MER_PRICE_TIMES
        <trim prefix="set" suffixOverrides=",">
            <trim prefix="MODIFY_UID =case" suffix="end,">
                <foreach collection="onList" item="item" index="index">
                    <if test="item.modifyUid !=null">
                        when guid=#{item.guid} then #{item.modifyUid}
                    </if>
                </foreach>
            </trim>
            <trim prefix="MODIFY_TIME =case" suffix="end,">
                <foreach collection="onList" item="item" index="index">
                    <if test="item.modifyTime !=null">
                        when guid=#{item.guid} then #{item.modifyTime}
                    </if>
                </foreach>
            </trim>
            <trim prefix="time_from =case" suffix="end,">
                <foreach collection="onList" item="item" index="index">
                    <if test="item.timeFrom !=null">
                        when guid=#{item.guid} then #{item.timeFrom}
                    </if>
                </foreach>
            </trim>
            <trim prefix="time_to =case" suffix="end,">
                <foreach collection="onList" item="item" index="index">
                    <if test="item.timeTo !=null">
                        when guid=#{item.guid} then #{item.timeTo}
                    </if>
                </foreach>
            </trim>
            <trim prefix="price1 =case" suffix="end,">
                <foreach collection="onList" item="item" index="index">
                    <if test="item.price1 !=null">
                        when guid=#{item.guid} then #{item.price1}
                    </if>
                </foreach>
            </trim>
        </trim>
        where guid in
        <foreach collection="onList" index="index" item="item" separator="," open="(" close=")">
            #{item.guid}
        </foreach>
        
            
            
//SQL编译之后
update LP_RAL_AREA_MER_PRICE set MODIFY_UID =case when guid=? then ? end, MODIFY_TIME =case when guid=? then ? end, time_from =case when guid=? then ? end, time_to =case when guid=? then ? end, price =case when guid=? then ? end where guid in ( ? ) 
```

## 7.hashset去重

```java
List<String> beforeList = new ArrayList<String>();
         
        beforeList.add("sun");
        beforeList.add("star");
        beforeList.add("moon");
        beforeList.add("earth");
        beforeList.add("sun");
        beforeList.add("earth");
         
        Set<String> middleHashSet = new HashSet<String>(beforeList);
         
        List<String> afterHashSetList = new ArrayList<String>(middleHashSet);
         
        System.out.println(beforeList);
        System.out.println(afterHashSetList);


//利用LinkedHashSet (去重后顺序一致)

List<String> beforeList = new ArrayList<String>();
         
        beforeList.add("sun");
        beforeList.add("star");
        beforeList.add("moon");
        beforeList.add("earth");
        beforeList.add("sun");
        beforeList.add("earth");
         
        Set<String> middleLinkedHashSet = new LinkedHashSet<String>(beforeList);
         
        List<String> afterHashSetList = new ArrayList<String>(middleLinkedHashSet);
         
        System.out.println(beforeList);
        System.out.println(afterHashSetList);

```

## 8.校验时间段重复

```java
       //参数
        for (int i = 0; i < inPutAreaMerPriceVos1.size(); i++) {
            InPutAreaMerPriceVo inPutAreaMerPriceVo = inPutAreaMerPriceVos1.get(i);
            String voItemNum = inPutAreaMerPriceVo.getItemNum();
            long timeFrom1 = stringToDate(inPutAreaMerPriceVo.getTimeFrom()).getTime();
            long timeTo1 = stringToDate(inPutAreaMerPriceVo.getTimeTo()).getTime();

            //数据库
            for (int j = 0; j < verifyTimes.size(); j++) {
                VerifyTimesDto verifyTimesDto = verifyTimes.get(j);
                String dtoItemNum = verifyTimesDto.getItemNum();
                long timeFrom2 = verifyTimesDto.getTimeFrom().getTime();
                long timeTo2 = verifyTimesDto.getTimeTo().getTime();
                boolean timeFalse=((timeFrom2 >= timeFrom1 && timeFrom2 <= timeTo1)||(timeFrom1>= timeFrom2&& timeFrom1 <= timeTo2));
                if (voItemNum.equals(dtoItemNum)&&timeFalse){
                    String same1 = inPutAreaMerPriceVo.getSame();
                    if (DataUtil.isNotEmpty(same1)){
                        inPutAreaMerPriceVo.setSame(same1+",timeFrom,timeTo");
                        inPutAreaMerPriceVos2.add(inPutAreaMerPriceVo);
                        break;
                    }else {
                        inPutAreaMerPriceVo.setSame("timeFrom,timeTo");
                        inPutAreaMerPriceVos2.add(inPutAreaMerPriceVo);
                        break;
                    }

                }
            }
        }
```







## 9.40W大数据批量导出受dubbo数据传输限制的问题

计算6W数据可达8M

处理方案又三种：

**1.在service层生成excel文件上传到fastDFS,前端通过fastDFS下载即可。**

优点：速度快，方便简单调用即可

缺点：依赖文件系统，文件系统不可用该功能不可用

**2.通过controller层遍历的去调用service查询数据库，在controller进行拼接，返回给前端**

优点：不依赖其他系统

缺点：多次遍历查询数据库，速度变慢，网关响应容易超时

**3.通过service层一次查询并生成文件，在通过controller层遍历获取service层的生成文件，完成搬运工作**

优点：不依赖其他系统，只查询一次，速度快

缺点:文件搬运容易丢失数据，操做复杂，占内存

最终选择第三种方案

**第一种方案**：在service层生成excel文件上传到fastDFS,前端通过fastDFS下载即可。建议使用，得需要fastDFS系统的相关资源

```java
//controller



/**
     * 导出财务区域物料价格段-数据导出
     *
     * @param areaMaterialPriceVo
     * @return 导出财务区域物料价格段-数据导出
     * @see AreaMaterialPriceVo
     * @see AreaMaterialPriceDto
     */
    @PostMapping("/outAreaMaterialPrice")
    @ApiOperation(value = "导出财务区域物料价格段-数据导出--方案一")
    public ResponseEntity<byte[]> outAreaMaterialPrice(@RequestBody(required = true) AreaMaterialPriceVo areaMaterialPriceVo, HttpServletResponse response) throws Exception {
        Map<String, String> map = areaMaterialPriceService.outAreaMaterialPrice(areaMaterialPriceVo);
        //获取路径
        String filePath = map.get("filePath");
        String xlsxName = map.get("xlsxName");
        String name = map.get("name");
        String viewPath = map.get("viewPath");
        CheckFileResult checkFileResult = new CheckFileResult();
        checkFileResult.setGroupName("OM");
        checkFileResult.setName(name);
        checkFileResult.setViewPath(map.get("viewPath"));
        ResponseEntity<byte[]> responseEntity = fastdfsFileUtil.downLocalFile(checkFileResult);

//        checkFileResult.setUid("fsynjc");
//        checkFileResult.setFileMd5(map.get("md5"));
//        ApiResult apiResult = fastdfsFileUtil.doDelete(checkFileResult);
        return responseEntity;
    }


//service


    /**
     * 导出财务区域物料价格段-数据导出
     * @param areaMaterialPriceVo
     * @return 导出财务区域物料价格段-数据导出
     * @see AreaMaterialPriceVo
     * @see AreaMaterialPriceDto
     */
    Map<String, String> outAreaMaterialPrice(AreaMaterialPriceVo areaMaterialPriceVo) throws Exception;




//service.impl

    /**
     * 导出财务区域物料价格段-数据导出
     * @param areaMaterialPriceVo
     * @return 导出财务区域物料价格段-数据导出
     * @see vo.AreaMaterialPriceVo
     * @see OutAreaPriceDto
     */
    @Override
    public Map<String, String> outAreaMaterialPrice(AreaMaterialPriceVo areaMaterialPriceVo) throws Exception {
        List<OutAreaPriceDto> outAreaPriceDtos = areaMaterialPriceMapper.outAreaMaterialPrice(areaMaterialPriceVo);
        HashMap<String, String> map = new HashMap<>();
        //Date date = new Date();
       //SimpleDateFormat format = new SimpleDateFormat("yyyyMMddHHmmss");
       //文件名
        String name = "财务区域价格管理";
        String xlsxName = name +".xlsx";
        String path = System.getProperty("user.dir")+"\\temporary";
        createDir(path);
        //文件全路径
        String filePath = path+"\\"+xlsxName;
        SXSSFWorkbook sxssfWorkbook = null;
        //返回参数
        map.put("filePath",filePath);
        map.put("xlsxName",xlsxName);
        map.put("name",name);
        BufferedOutputStream outputStream = null;
        //创建文件并写入
        XSSFWorkbook workbook =  null;
        BufferedOutputStream outputStream1 = null;
        try {
            File fileXlsxPath = new File(filePath);
            outputStream1 = new BufferedOutputStream(new FileOutputStream(fileXlsxPath));
            workbook = new XSSFWorkbook();
            workbook.createSheet("财务区域价格管理");
            workbook.write(outputStream1);
            //保留100条数据在内存中，其它的数据都会写到磁盘里
            sxssfWorkbook = new SXSSFWorkbook(workbook,100);
            //获取第一个Sheet页
            SXSSFSheet sheet = sxssfWorkbook.getSheetAt(0);
            SXSSFRow row = sheet.createRow(0);
            row.createCell(0).setCellValue("物料编号");
            row.createCell(1).setCellValue("物料名称");
            row.createCell(2).setCellValue("仓库类别");
            row.createCell(3).setCellValue("物料类别");
            row.createCell(4).setCellValue("财务区域");
            row.createCell(5).setCellValue("起始时间");
            row.createCell(6).setCellValue("结束时间");
            row.createCell(7).setCellValue("价格");
            for (int k = 1; k < outAreaPriceDtos.size(); k++) {
                SXSSFRow row1 = sheet.createRow(k);
                OutAreaPriceDto outAreaPriceDto = outAreaPriceDtos.get(k);
                for (int f = 0; f < 8; f++) {
                    SXSSFCell cell = row1.createCell(f);
                    if (f ==0){
                        cell.setCellValue(outAreaPriceDto.getItemNum());
                    }else if (f==1){
                        cell.setCellValue(outAreaPriceDto.getName());
                    }else if (f==2){
                        cell.setCellValue(outAreaPriceDto.getCommodities());
                    }else if (f==3){
                        cell.setCellValue(outAreaPriceDto.getProperties());
                    }else if (f==4){
                        cell.setCellValue(outAreaPriceDto.getDqname());
                    }else if (f==5){
                        cell.setCellValue(outAreaPriceDto.getTimeFrom());
                    }else if (f==6){
                        cell.setCellValue(outAreaPriceDto.getTimeTo());
                    }else if (f==7){
                        cell.setCellValue(outAreaPriceDto.getPrice());
                    }
                }
            }
            outputStream = new BufferedOutputStream(new FileOutputStream(filePath));
            sxssfWorkbook.write(outputStream);
            outputStream.flush();

        } catch (IOException e) {
            LogUtil.sysEventError(e.getMessage());
        }finally {
            if(workbook!=null) {
                // 释放资源
                workbook.close();
            }
            if(sxssfWorkbook!=null) {
                // 释放资源
                sxssfWorkbook.dispose();
            }

            if(outputStream!=null) {
                try {
                    outputStream.close();
                } catch (IOException e) {
                    LogUtil.sysEventError(e.getMessage());
                }
            }
        }
        //上传fastDFS
        String viewPath = fastdfsFileUtil.uploadFile("OM", filePath, "fsynjc", "9999033");
        map.put("viewPath",viewPath);
        TimingDeleteExcel.delFolder(path);
        return map;
    }



 /**
     * 创建临时目录
     * @param destDirName 创建临时目录
     * @return 目录创建成功返回true，否则返回false
     */
    public  boolean createDir(String destDirName) {
        File dir = new File(destDirName);
        if(dir.exists()) {
            return false;
        }
        if(!destDirName.endsWith(File.separator)){
            destDirName = destDirName + File.separator;
        }
        // 创建单个目录
        if(dir.mkdirs()) {
            return true;
        } else {
            return false;
        }
    }



//定时删除工具类

/**
 * 定时删除临时文件
 * @author hanju
 * @since 2020/9/15
 */
@Component
public class TimingDeleteExcel {

    @Autowired
    FastdfsFileUtil fastdfsFileUtil;



    /**
     * 定期删除文件系统文件，每天凌晨2点检测一次
     */
    @Scheduled(cron = "0 0 2 * * ?")
    public void deleteExcel(){
        CheckFileResult cFileResult=new CheckFileResult();
        cFileResult.setGroupName("OM");
        cFileResult.setFileMd5(md5);
        cFileResult.setUid("fsynjc");
        try {
            fastdfsFileUtil.doDelete(cFileResult);
        } catch (Exception e) {
            LogUtil.sysEventError(e.getMessage());
        }
    }


    /***
     * 删除文件夹
     *
     * @param folderPath 文件夹完整绝对路径
     */
    public  static void delFolder(String folderPath) {
        try {
            // 删除完里面所有内容
            delAllFile(folderPath);
            String filePath = folderPath;
            filePath = filePath.toString();
            java.io.File myFilePath = new java.io.File(filePath);
            // 删除空文件夹
            myFilePath.delete();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    /***
     * 删除指定文件夹下所有文件
     * @param path 文件夹完整绝对路径
     * @return
     */
    public static  boolean delAllFile(String path) {
        boolean flag = false;
        File file = new File(path);
        if (!file.exists()) {
            return flag;
        }
        if (!file.isDirectory()) {
            return flag;
        }
        String[] tempList = file.list();
        File temp = null;
        for (int i = 0; i < tempList.length; i++) {
            if (path.endsWith(File.separator)) {
                temp = new File(path + tempList[i]);
            } else {
                temp = new File(path + File.separator + tempList[i]);
            }
            if (temp.isFile()) {
                temp.delete();
            }
            if (temp.isDirectory()) {
                // 先删除文件夹里面的文件
                delAllFile(path + "/" + tempList[i]);
                // 再删除空文件夹
                delFolder(path + "/" + tempList[i]);
                flag = true;
            }
        }
        return flag;
    }

}






```





**第二种方案**：通过controller层遍历的去调用service查询数据库，在controller进行拼接，返回给前端,不建议使用用时较长40W超过2分钟

```java
 


  /**
     * 导出财务区域物料价格段-数据导出
     * @param areaMaterialPriceVo
     * @return 导出财务区域物料价格段-数据导出
     * @see AreaMaterialPriceVo
     * @see OutAreaPriceDto
     */
    @PostMapping("/outAreaMaterialPrice")
    public ResponseEntity<byte[]> outAreaMaterialPrice(AreaMaterialPriceVo areaMaterialPriceVo) {
        List<OutAreaPriceDto> outAreaPriceDtos = getResourse(areaMaterialPriceVo);
        Date date = new Date();
        SimpleDateFormat format = new SimpleDateFormat("yyyyMMddHHmmss");
        //文件名
        String name = "财务区域价格管理"+format.format(date);
        String xlsxName = name +".xlsx";
        String path = System.getProperty("user.dir")+"\\temporary";
        createDir(path);
        //文件全路径
        String filePath = path+"\\"+xlsxName;
        //生成
        doExcel(outAreaPriceDtos, filePath);
        byte[] content =null;
        ByteArrayOutputStream baos =null;
        InputStream is =null;
        try {
            baos = new ByteArrayOutputStream();
            byte[] b = new byte[1024];
            int i = 0;
            is = new FileInputStream(filePath);
            while((i = is.read(b)) != -1){
                baos.write(b, 0, i);
            }
            content = baos.toByteArray();

        } catch (IOException e) {
            LogUtil.sysEventError(e.getMessage());
        }finally {
            if (is!=null){
                try {
                    is.close();
                } catch (IOException e) {
                    LogUtil.sysEventError(e.getMessage());
                }
            }
            if (baos!=null){
                try {
                    baos.close();
                } catch (IOException e) {
                    LogUtil.sysEventError(e.getMessage());
                }
            }
        }

        HttpHeaders headers = new HttpHeaders();
        try {
            headers.setContentDispositionFormData("attachment", new String(xlsxName.getBytes("UTF-8"), "ISO-8859-1"));
        } catch (UnsupportedEncodingException e) {
            LogUtil.sysEventError(e.getMessage());
        }
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
         ResponseEntity<byte[]> responseEntity = new ResponseEntity(content, headers, HttpStatus.OK);
        //删除临时文件
        delFolder(path);       
        return responseEntity;
    }



   
    /**
     * 批量导出获取数据源
     * @param areaMaterialPriceVo
     * @return 查询数据
     */

    public List<OutAreaPriceDto> getResourse(AreaMaterialPriceVo areaMaterialPriceVo) {
      //获取数据库总数
        Integer count = areaMaterialPriceService.getAreaMaterialPriceCount(areaMaterialPriceVo);
        List<OutAreaPriceDto> outAreaPriceDtos = new ArrayList<>();
        //分页查询，每次60000条
        int i = count % 60000;
        int t = count / 60000;
        if (t > 0) {
            for (int j = 1; j <= t; j++) {
                areaMaterialPriceVo.setStartIndex((j - 1) * 60000 + 1);
                areaMaterialPriceVo.setEndIndex(j * 60000);
                List<OutAreaPriceDto> outAreaPriceDtos1 = areaMaterialPriceService.outAreaMaterialPrice(areaMaterialPriceVo);
                outAreaPriceDtos.addAll(outAreaPriceDtos1);
            }
            areaMaterialPriceVo.setStartIndex(t * 60000 + 1);
            areaMaterialPriceVo.setEndIndex(t * 60000 + i);
            List<OutAreaPriceDto> outAreaPriceDtos1 = areaMaterialPriceService.outAreaMaterialPrice(areaMaterialPriceVo);
            outAreaPriceDtos.addAll(outAreaPriceDtos1);
        } else {
            areaMaterialPriceVo.setStartIndex(1);
            areaMaterialPriceVo.setEndIndex(i);
            List<OutAreaPriceDto> outAreaPriceDtos1 = areaMaterialPriceService.outAreaMaterialPrice(areaMaterialPriceVo);
            outAreaPriceDtos.addAll(outAreaPriceDtos1);
        }

        return outAreaPriceDtos;
    }



/**
     * 生成excel临时文件
     * @param outAreaPriceDtos
     * @param filePath 文件路径
     */
    private  void doExcel(List<OutAreaPriceDto> outAreaPriceDtos,String filePath){
        SXSSFWorkbook sxssfWorkbook = null;
        BufferedOutputStream outputStream = null;
        //创建文件并写入
        XSSFWorkbook workbook =  null;
        BufferedOutputStream outputStream1 = null;
        try {
            File fileXlsxPath = new File(filePath);
            outputStream1 = new BufferedOutputStream(new FileOutputStream(fileXlsxPath));
            workbook = new XSSFWorkbook();
            workbook.createSheet("财务区域价格管理");
            workbook.write(outputStream1);
            //保留100条数据在内存中，其它的数据都会写到磁盘里
            sxssfWorkbook = new SXSSFWorkbook(workbook,100);
            //获取第一个Sheet页
            SXSSFSheet sheet = sxssfWorkbook.getSheetAt(0);
            SXSSFRow row = sheet.createRow(0);
            row.createCell(0).setCellValue("物料编号");
            row.createCell(1).setCellValue("物料名称");
            row.createCell(2).setCellValue("仓库类别");
            row.createCell(3).setCellValue("物料类别");
            row.createCell(4).setCellValue("财务区域");
            row.createCell(5).setCellValue("起始时间");
            row.createCell(6).setCellValue("结束时间");
            row.createCell(7).setCellValue("价格");
            for (int k = 1; k < outAreaPriceDtos.size(); k++) {
                SXSSFRow row1 = sheet.createRow(k);
                OutAreaPriceDto outAreaPriceDto = outAreaPriceDtos.get(k);
                for (int f = 0; f < 8; f++) {
                    SXSSFCell cell = row1.createCell(f);
                    if (f ==0){
                        cell.setCellValue(outAreaPriceDto.getItemNum());
                    }else if (f==1){
                        cell.setCellValue(outAreaPriceDto.getName());
                    }else if (f==2){
                        cell.setCellValue(outAreaPriceDto.getCommodities());
                    }else if (f==3){
                        cell.setCellValue(outAreaPriceDto.getProperties());
                    }else if (f==4){
                        cell.setCellValue(outAreaPriceDto.getDqname());
                    }else if (f==5){
                        cell.setCellValue(outAreaPriceDto.getTimeFrom());
                    }else if (f==6){
                        cell.setCellValue(outAreaPriceDto.getTimeTo());
                    }else if (f==7){
                        cell.setCellValue(outAreaPriceDto.getPrice());
                    }
                }
            }
            outputStream = new BufferedOutputStream(new FileOutputStream(filePath));
            sxssfWorkbook.write(outputStream);
            outputStream.flush();

        } catch (IOException e) {
            LogUtil.sysEventError(e.getMessage());
        }finally {
            if(workbook!=null) {
                // 释放资源
                try {
                    workbook.close();
                } catch (IOException e) {
                    LogUtil.sysEventError(e.getMessage());
                }
            }
            if(sxssfWorkbook!=null) {
                // 释放资源
                sxssfWorkbook.dispose();
                try {
                    sxssfWorkbook.close();
                } catch (IOException e) {
                    LogUtil.sysEventError(e.getMessage());
                }
            }
            if(outputStream!=null) {
                try {
                    outputStream.close();
                } catch (IOException e) {
                    LogUtil.sysEventError(e.getMessage());
                }
            }
        }
    }



    /***
     * 删除文件夹
     * @param folderPath 文件夹完整绝对路径
     */
    public  static void delFolder(String folderPath) {
        try {
            // 删除完里面所有内容
            delAllFile(folderPath);
            String filePath = folderPath;
            filePath = filePath.toString();
            java.io.File myFilePath = new java.io.File(filePath);
            // 删除空文件夹
            if (!myFilePath.delete()){
            }
        } catch (Exception e) {
            LogUtil.sysEventError(e.getMessage());
        }
    }


    /***
     * 删除指定文件夹下所有文件
     * @param path 文件夹完整绝对路径
     * @return
     */
    public static  boolean delAllFile(String path) {
        boolean flag = false;
        File file = new File(path);
        if (!file.exists()) {
            return flag;
        }
        if (!file.isDirectory()) {
            return flag;
        }
        String[] tempList = file.list();
        File temp = null;
        for (int i = 0; i < tempList.length; i++) {
            if (path.endsWith(File.separator)) {
                temp = new File(path + tempList[i]);
            } else {
                temp = new File(path + File.separator + tempList[i]);
            }
            if (temp.isFile()) {
                if (!temp.delete()){
                }
            }
            if (temp.isDirectory()) {
                // 先删除文件夹里面的文件
                delAllFile(path + "/" + tempList[i]);
                // 再删除空文件夹
                delFolder(path + "/" + tempList[i]);
                flag = true;
            }
        }
        return flag;
    }


    /**
     * 创建临时目录
     * @param destDirName 创建临时目录
     * @return 目录创建成功返回true，否则返回false
     */
    public  boolean createDir(String destDirName) {
        File dir = new File(destDirName);
        if(dir.exists()) {
            return false;
        }
        if(!destDirName.endsWith(File.separator)){
            destDirName = destDirName + File.separator;
        }
        // 创建单个目录
        if(dir.mkdirs()) {
            return true;
        } else {
            return false;
        }
    }






//servcie



     List<OutAreaPriceDto>  outAreaMaterialPrice(AreaMaterialPriceVo areaMaterialPriceVo);


//servcie.impl

   /**
     * 批量导出并生成文件
     *
     * @param areaMaterialPriceVo
     * @return  map
     */
    @Override
    public List<OutAreaPriceDto> outAreaMaterialPrice(AreaMaterialPriceVo areaMaterialPriceVo) {
        List<OutAreaPriceDto> outAreaPriceDtos = areaMaterialPriceMapper.outAreaMaterialPrice(areaMaterialPriceVo);
   return outAreaPriceDtos;
        }





```





**第三种方案**：通过service层一次查询并生成文件，在通过controller层遍历获取service层的生成文件，完成搬运工作。推荐使用用时最短40W数据仅用20来秒

```java

//controller代码



  /**
     * 导出财务区域物料价格段-数据导出
     * @param areaMaterialPriceVo
     * @return 导出财务区域物料价格段-数据导出
     * @see AreaMaterialPriceVo
     */
    @PostMapping("/outAreaMaterialPrice")
    @ApiOperation(value = "导出财务区域物料价格段-数据导出")
    public ResponseEntity<byte[]> outAreaMaterialPrice(@RequestBody(required = true) AreaMaterialPriceVo areaMaterialPriceVo) throws Exception {
        Map<String, String> map = areaMaterialPriceService.doResourse(areaMaterialPriceVo);
        String xlsxName = map.get("xlsxName");
        byte[] content = null;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
            //最大70M
            for (int i = 0; i < 30; i++) {
                ReadeFile readeFile = areaMaterialPriceService.getResourse(map, i);
                if (DataUtil.isNotEmpty(readeFile)){
                    baos.write(readeFile.getBytes(),0,readeFile.getI());
                }
            }
            content = baos.toByteArray();
            baos.close();
        HttpHeaders headers = new HttpHeaders();
        headers.setContentDispositionFormData("attachment", new String(xlsxName.getBytes("UTF-8"), "ISO-8859-1"));
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
        return new ResponseEntity(content, headers, HttpStatus.OK);
    }





//service代码


 /**
     * 生成文件
     * @param areaMaterialPriceVo
     * @return map
     */
    Map<String, String> doResourse(AreaMaterialPriceVo areaMaterialPriceVo);

    /**
     * 获取文件流
     * @param map
     * @param index
     * @return ReadeFile
     */
    ReadeFile getResourse(Map<String, String> map, Integer index);




//service.impl代码


 /**
     * 获取导出获取数据源
     * @param map 地址
     * @see  ReadeFile
     */
    @Override
    public ReadeFile getResourse(Map<String, String> map,Integer index) {
        String filePath = map.get("filePath");
        InputStream is = null;
        try {
            byte[] b = new byte[1048576];
            int t=0;
            int i = 0;
            is = new FileInputStream(filePath);
            while ((i = is.read(b)) != -1) {
                if (index==t){
                    ReadeFile readeFile=new ReadeFile();
                    readeFile.setBytes(b);
                    readeFile.setI(i);
                    return readeFile;
                }
                t++;
            }
        } catch (IOException e) {
            LogUtil.sysEventError(e.getMessage());
        } finally {
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    LogUtil.sysEventError(e.getMessage());
                }
            }
        }
        //删除文件
        if (index==29){
            File file=new File(filePath);
            if (!file.delete()){
            }
        }
      return null;
    }


    /**
     * 批量导出并生成文件
     *
     * @param areaMaterialPriceVo
     * @return  map
     */
    @Override
    public Map<String, String> doResourse(AreaMaterialPriceVo areaMaterialPriceVo) {
        List<OutAreaPriceDto> outAreaPriceDtos = areaMaterialPriceMapper.outAreaMaterialPrice(areaMaterialPriceVo);
        Map<String, String> map = new HashMap<>();
        Date date = new Date();
        SimpleDateFormat format = new SimpleDateFormat("yyyyMMddHHmmss");
        //文件名
        String name = "财务区域价格管理" + format.format(date);
        String xlsxName = name + ".xlsx";
        String path = System.getProperty("user.dir");
        //文件全路径
        String filePath = path + "\\" + xlsxName;
        //生成
        doExcel(outAreaPriceDtos, filePath);
        map.put("filePath",filePath);
        map.put("xlsxName",xlsxName);
        return map;
    }


    /**
     * 生成excel临时文件
     * @param outAreaPriceDtos
     * @param filePath  文件路径
     */
    private void doExcel(List<OutAreaPriceDto> outAreaPriceDtos, String filePath) {
        SXSSFWorkbook sxssfWorkbook = null;
        BufferedOutputStream outputStream = null;
        //创建文件并写入
        XSSFWorkbook workbook = null;
        BufferedOutputStream outputStream1 = null;
        try {
            File fileXlsxPath = new File(filePath);
            outputStream1 = new BufferedOutputStream(new FileOutputStream(fileXlsxPath));
            workbook = new XSSFWorkbook();
            workbook.createSheet("财务区域价格管理");
            workbook.write(outputStream1);
            //保留100条数据在内存中，其它的数据都会写到磁盘里
            sxssfWorkbook = new SXSSFWorkbook(workbook, 100);
            //获取第一个Sheet页
            SXSSFSheet sheet = sxssfWorkbook.getSheetAt(0);
            SXSSFRow row = sheet.createRow(0);
            row.createCell(0).setCellValue("物料编号");
            row.createCell(1).setCellValue("物料名称");
            row.createCell(2).setCellValue("仓库类别");
            row.createCell(3).setCellValue("物料类别");
            row.createCell(4).setCellValue("财务区域");
            row.createCell(5).setCellValue("起始时间");
            row.createCell(6).setCellValue("结束时间");
            row.createCell(7).setCellValue("价格");
            for (int k = 1; k < outAreaPriceDtos.size(); k++) {
                SXSSFRow row1 = sheet.createRow(k);
                OutAreaPriceDto outAreaPriceDto = outAreaPriceDtos.get(k);
                for (int f = 0; f < 8; f++) {
                    SXSSFCell cell = row1.createCell(f);
                    if (f == 0) {
                        cell.setCellValue(outAreaPriceDto.getItemNum());
                    } else if (f == 1) {
                        cell.setCellValue(outAreaPriceDto.getName());
                    } else if (f == 2) {
                        cell.setCellValue(outAreaPriceDto.getCommodities());
                    } else if (f == 3) {
                        cell.setCellValue(outAreaPriceDto.getProperties());
                    } else if (f == 4) {
                        cell.setCellValue(outAreaPriceDto.getDqname());
                    } else if (f == 5) {
                        cell.setCellValue(outAreaPriceDto.getTimeFrom());
                    } else if (f == 6) {
                        cell.setCellValue(outAreaPriceDto.getTimeTo());
                    } else if (f == 7) {
                        cell.setCellValue(outAreaPriceDto.getPrice());
                    }
                }
            }
            outputStream = new BufferedOutputStream(new FileOutputStream(filePath));
            sxssfWorkbook.write(outputStream);
            outputStream.flush();

        } catch (IOException e) {
            LogUtil.sysEventError(e.getMessage());
        } finally {
            if (workbook != null) {
                // 释放资源
                try {
                    workbook.close();
                } catch (IOException e) {
                    LogUtil.sysEventError(e.getMessage());
                }
            }
            if (sxssfWorkbook != null) {
                // 释放资源
                sxssfWorkbook.dispose();
                try {
                    sxssfWorkbook.close();
                } catch (IOException e) {
                    LogUtil.sysEventError(e.getMessage());
                }
            }
            if (outputStream != null) {
                try {
                    outputStream.close();
                } catch (IOException e) {
                    LogUtil.sysEventError(e.getMessage());
                }
            }
        }
    }

//实体类

//1

 /**
 * 获取财务区域物料价格段管理 参数
 * @author hanju
 * @since 2020/9/3
 */
public class AreaMaterialPriceVo extends BaseModel<AreaMaterialPriceVo> {
    private static final long serialVersionUID = 1L;
    /**
     * 物料编号
     */
    private String itemNum;
    /**
     * 物料描述
     */
    private String name;
    /**
     * 仓库类别
     */
    private String commodities;
    /**
     * 财务区域
     */
    private String financialArea;
    /**
     * 物料类别
     */
    private String properties;

    /**
     * 时间
     */
    private String time;
    /**
     * 起始
     */
    private  Integer startIndex;
    /**
     * 结束
     */
    private  Integer endIndex;
    /**
     * 1-已过期    0未过期
     */
    private  String limit;
    //getter 
    //...
    // setter    
	//...

}

//2

/**
 * 读取文件
 * @author hanju
 * @since 2020/9/18
 */
public class ReadeFile extends BaseModel<ReadeFile> {
    private static final long serialVersionUID = 1L;
    /**
     * 字节数组
     */
    private byte[] bytes;
    /**
     * reade中的i
     */
    private  Integer i;

 //getter 
    //...
    // setter    
	//...

}




//mapper



<select id="outAreaMaterialPrice" resultType="com.lesso.model.materialManage.dto.OutAreaPriceDto">
        select * from (select
        ROW_NUMBER() OVER (order by a.item_num) AS PSROWID,
        a.item_num,
        a.item_name as name,
        b.commodities,
        b.properties,
        c.dqname,
        a.time_from,
        a.time_to,
        a.price
        from LP_RAL_AREA_MER_PRICE a
        INNER JOIN LP_MST_DQ c on c.dqno = a.financial_area
        left join LP_MST_WLMXB b on a.foreign_key=b.GUID
        where 1=1
        <include refid="BaseConditionSql"></include>) as a1
        <if test="startIndex != null and endIndex != null and startIndex != 0 and endIndex != 0">
            where a1.PSROWID between #{startIndex} and #{endIndex}
        </if>
    </select>






```

## 10.压缩文件

```java
/**
 *
 * @param srcfile 文件名数组
 * @param zipfile 压缩后文件地址
 */
public static void zipFiles(java.io.File[] srcfile, java.io.File zipfile) {
    byte[] buf = new byte[1048576];
    try {
        ZipOutputStream out = new ZipOutputStream(new FileOutputStream(
                zipfile));
        for (int i = 0; i < srcfile.length; i++) {
            FileInputStream in = new FileInputStream(srcfile[i]);
            out.putNextEntry(new ZipEntry(srcfile[i].getName()));
            int len;
            while ((len = in.read(buf)) > 0) {
                out.write(buf, 0, len);
            }
            out.closeEntry();
            in.close();
        }
        out.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```



#### 11.总结了30条了可以实际优化数据库的经验

**1.**对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。

**2.**应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：select id from t where num is null可以在num上设置默认值0，确保表中num列没有null值，然后这样查询：select id from t where num=0

**3.**应尽量避免在 where 子句中使用!=或<>操作符，否则引擎将放弃使用索引而进行全表扫描。

**4.**应尽量避免在 where 子句中使用or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：select id from t where num=10 or num=20可以这样查询：select id from t where num=10 union all select id from t where num=20

**5.**in 和 not in 也要慎用，否则会导致全表扫描，如：select id from t where num in(1,2,3) 对于连续的数值，能用 between 就不要用 in 了：select id from t where num between 1 and 3

**6.**下面的查询也将导致全表扫描：select id from t where name like '李%'若要提高效率，可以考虑全文检索。

**7.**如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然 而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：select id from t where num=@num可以改为强制查询使用索引：select id from t with(index(索引名)) where num=@num
**8.**应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：select id from t where num/2=100应改为:select id from t where num=100*2

**9.**应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：select id from t where substring(name,1,3)='abc' ，name以abc开头的id

应改为:

select id from t where name like 'abc%'

**10.**不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。

**11.**在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。

**12.**不要写一些没有意义的查询，如需要生成一个空表结构：select col1,col2 into #t from t where 1=0

这类代码不会返回任何结果集，但是会消耗系统资源的，应改成这样：

create table #t(...)

**13.**很多时候用 exists 代替 in 是一个好的选择：select num from a where num in(select num from b)

用下面的语句替换：

select num from a where exists(select 1 from b where num=a.num)

**14.**并不是所有索引对查询都有效，SQL是根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引，如一表中有字段sex，male、female几乎各一半，那么即使在sex上建了索引也对查询效率起不了作用。

索引并不是越多越好，索引固然可 以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有 必要。

应尽可能的避免更新 clustered 索引数据列，因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。

**17.**尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。

**18.**尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。

**19.**任何地方都不要使用 select * from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段。

**20.**尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）。

**21.**避免频繁创建和删除临时表，以减少系统表资源的消耗。

**22.**临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用表中的某个数据集时。但是，对于一次性事件，最好使用导出表。

**23.**在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table，避免造成大量 log ，以提高速度；如果数据量不大，为了缓和系统表的资源，应先create table，然后insert。

**24.**如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样可以避免系统表的较长时间锁定。

**25.**尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1万行，那么就应该考虑改写。

**26.**使用基于游标的方法或临时表方法之前，应先寻找基于集的解决方案来解决问题，基于集的方法通常更有效。

与临时表一样，游标并不是不可使 用。对小型数据集使用 FAST_FORWARD 游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括“合计”的例程通常要比使用游标执行的速度快。如果开发时 间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好。
**28.**在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。无需在执行存储过程和触发器的每个语句后向客户端发送DONE_IN_PROC 消息。

**29.**尽量避免大事务操作，提高系统并发能力。

**30.**尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。



#### 12.在MySQL中重复的插入数据怎么办？

```java


/*
最常见的方式就是为字段设置主键或唯一索引，当插入重复数据时，抛出错误，程序终止，但这会给后续处理带来麻烦，因此需要对插入语句做特殊处理，尽量避开或忽略异常，下面我简单介绍一下，感兴趣的朋友可以尝试一下：
这里为了方便演示，我新建了一个user测试表，主要有id，username，sex，address这4个字段，其中主键为id（自增），同时对username字段设置了唯一索引：
*/

//01 insert ignore into
/*
即插入数据时，如果数据存在，则忽略此次插入，前提条件是插入的数据字段设置了主键或唯一索引，测试SQL语句如下，当插入本条数据时，MySQL数据库会首先检索已有数据（也就是idx_username索引），如果存在，则忽略本次插入，如果不存在，则正常插入数据：
*/
insert ignore into user (username,sex,address) values("Jack","male","New York");

//02 on duplicate key update
/*
即插入数据时，如果数据存在，则执行更新操作，前提条件同上，也是插入的数据字段设置了主键或唯一索引，测试SQL语句如下，当插入本条记录时，MySQL数据库会首先检索已有数据（idx_username索引），如果存在，则执行update更新操作，如果不存在，则直接插入：
*/
insert  into user(username,sex,address) on duplicate key update user (username,sex,address) values("Jack","male","New York") on duplicate key update sex='male',address='New York';


//03 replace into
/*
即插入数据时，如果数据存在，则删除再插入，前提条件同上，插入的数据字段需要设置主键或唯一索引，测试SQL语句如下，当插入本条记录时，MySQL数据库会首先检索已有数据（idx_username索引），如果存在，则先删除旧数据，然后再插入，如果不存在，则直接插入：
*/
replace into  user (username,sex,address) values("Jack","male","New York");



//04 insert if not exists
/*
即insert into … select … where not exist ... ，这种方式适合于插入的数据字段没有设置主键或唯一索引，当插入一条数据时，首先判断MySQL数据库中是否存在这条数据，如果不存在，则正常插入，如果存在，则忽略：
*/
insert  into user(username,sex,address) select "Jack","male","New York" from user where not exists(select username from user where username="Jack");

/*
目前，就分享这4种MySQL处理重复数据的方式吧，前3种方式适合字段设置了主键或唯一索引，最后一种方式则没有此限制，只要你熟悉一下使用过程，很快就能掌握的，网上也有相关资料和教程，介绍的非常详细，感兴趣的话，可以搜一下。
*/
```



















