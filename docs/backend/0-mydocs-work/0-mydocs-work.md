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
     * @see com.lesso.model.materialManage.vo.AreaMaterialPriceVo
     * @see com.lesso.model.materialManage.dto.AreaMaterialPriceDto
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
     * @see com.lesso.model.materialManage.vo.AreaMaterialPriceVo
     * @see com.lesso.model.materialManage.dto.AreaMaterialPriceDto
     */
    Map<String, String> outAreaMaterialPrice(AreaMaterialPriceVo areaMaterialPriceVo) throws Exception;




//service.impl

    /**
     * 导出财务区域物料价格段-数据导出
     * @param areaMaterialPriceVo
     * @return 导出财务区域物料价格段-数据导出
     * @see com.lesso.model.materialManage.vo.AreaMaterialPriceVo
     * @see com.lesso.model.materialManage.dto.OutAreaPriceDto
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
     * @see com.lesso.model.materialManage.vo.AreaMaterialPriceVo
     * @see com.lesso.model.materialManage.dto.OutAreaPriceDto
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
     * @see com.lesso.model.materialManage.vo.AreaMaterialPriceVo
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
     * @return com.lesso.model.materialManage.ReadeFile
     */
    ReadeFile getResourse(Map<String, String> map, Integer index);




//service.impl代码


 /**
     * 获取导出获取数据源
     * @param map 地址
     * @see  com.lesso.model.materialManage.ReadeFile
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

























