package com.kapphk.pms.util;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.Iterator;
import java.util.List;
import java.util.regex.Pattern;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.apache.commons.io.FileUtils;
import org.apache.tools.zip.ZipEntry;
import org.apache.tools.zip.ZipOutputStream;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.multipart.MultipartHttpServletRequest;

/**
 * 文件读取	
 * @author 胡子
 * since 2018年6月11日
 */
public class FileRead {
	
	  private static final int  BUFFER_SIZE = 50 * 1024;
	  
	  /**
	   * 编码格式
	   */
	  public static final String ENCODE = "gb2312";
	  
	  /**
	   * 正则表达式
	   */
	  static Pattern pattern = Pattern.compile("[0-9A-Za-z_\\.\u4e00-\u9fa5]+$");
//	  /**
//	   * 测试
//	   * @throws Exception
//	   */
//	  @Test
//	  public void a() throws Exception{   
//	      //这是需要获取的文件夹路径  
//	      String path1 = "F:/MYL_I02D43.myl";   
//	      String path2 = "F:/20180808";
//	      Result rs = filtrateFile(path1, path2, false);
//	      JSONObject json  = rs.getData();			 	  		//转成json
//		  String zipPath = (String)json.get("zipPath");     	//json转成string
//	      System.out.println("zip文件路径：" + zipPath);
//	      
//	      String type = "2";
//	      if("1".equals(type)) {
//				rs.put("zipPath", zipPath);
////				rs.put("versionCode", hs.getVersionCode());
//		  }
//		  else if("2".equals(type)){
//				File file = new File(zipPath);
//				boolean a = file.delete();
//				if (a == true) {
//					System.out.println("删除成功啦~");
//				}
//		  }
//	      

    
    /**
	 * 作用：读取app传至后台的.myl文件内容和解压文件夹下所有文件名，
	 * 		将多出部分文件和末尾标“x”的文件筛选出后复制并压缩成zip包。
	 * 
     * @param request 
	 * @param request
	 * @param mylFile 		    .myl文件路径
	 * @param unzipPath  		文件夹路径
	 * @param KeepDirStructure  是否保留原来的目录结构,
	 * 							true:保留目录结构; 
	 * 							false:所有文件跑到压缩包根目录下(注意：不保留目录结构可能会出现同名文件,会压缩失败)
	 * @throws Exception
	 */
	@SuppressWarnings("rawtypes")
	public static Result filtrateFile(HttpServletRequest request, String unzipPath, boolean KeepDirStructure) throws Exception {
//	public static Result filtrateFile(String mylFile, String unzipPath, boolean KeepDirStructure) throws Exception {
		
		Result rs = new Result();
		HttpSession session = request.getSession();
		List<String> fileNameAs = new ArrayList<String>();
		List<String> fileNameBs = new ArrayList<String>();
		
		
		//在request中获取app传过来的.myl文件
		MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest)request;
		Iterator iter = multiRequest.getFileNames();
		while (iter.hasNext()) {
			MultipartFile mFile = multiRequest.getFile((String) iter.next());
			if(mFile != null){
				String mylFile = mFile.getOriginalFilename();
		
			//******************************** 读写app传至后台的.myl文件内容 **********************************/	
				
				
				//判断文件是否为空
				if(!mylFile.isEmpty()){
					
					String path = FileUploadUtils.class.getClassLoader().getResource("../../").getPath() ;  
					String path1 = "/upload/myl/";				//文件的上级目录
					path = path + path1;
					
					File file1 = new File(path);
					if(!file1.exists()){						//不存在就创建目录
						file1.mkdirs() ;
					}
					
					String fileStr = file1 + "/" + mylFile;		//文件路径
					session.setAttribute("fileStr", fileStr);	//将数据存储到session中
					InputStream inputStream = mFile.getInputStream();//文件流
					
					
					String content = read(inputStream,ENCODE);	//读取文件时指定编码格式为gb2312(因为源文件本身编码格式为gb2312)
					write(fileStr, content);					//写入文件内容
					
					//BufferedReader按行读取文件  
			        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(new FileInputStream(fileStr)));  
			              
			        String str = null; 	
			        while((str = bufferedReader.readLine()) != null)  
			        {  
						fileNameAs.add(str);
//				        System.out.println("myl文件内容："+str);
			        } 
			        bufferedReader.close();
				}
				
			//*********************************** 读取 解压目录 下的所有文件名  **********************************/	
				
		        File file = new File(unzipPath);    		// 获得指定文件对象
		        if(file.exists()) {
		        	 File[] array = file.listFiles();    	// 获得该文件夹内的所有文件   
				        for(int i=0;i<array.length;i++)
				        {   	
				           if(array[i].isFile()){			//如果是文件
				            	fileNameBs.add(array[i].getName()); 
				           }
				        } 
				}else{
					rs.setStatus(Contents.NOT_EXIST);
					return rs;
				}
		        
		        
		    //****************************** 筛选出多出的文件和后缀名末尾标x的文件 ******************************/
		        
		        //筛选出解压文件中多出的文件
		        List<String> list1 = new ArrayList<String>(fileNameBs);	//构建fileNameBs的副本
		        list1.removeAll(fileNameAs);
		        System.out.println("解压文件中比.myl文件多出"+ list1.size() +"个文件：" + list1);
		        
		        String lastChar;
		        List<String> list2 = new ArrayList<String>();
		        
		        //遍历 解压目录 下的所有文件名，筛选出文件名最后一个字母为“x”的文件
		        for (String fileName : fileNameBs) {
		        	lastChar = fileName.substring(fileName.length()-1);
		        	if (lastChar.equals("x")) {
		        		list2.add(fileName);
					}
				}
		        
		        System.out.println("检测到有"+ list2.size() +"个后缀末尾字母为“x”的元素：" + list2);
		         
		        //合并两个集合的元素
		        list1.addAll(list2);
		        System.out.println("共筛选出"+ list1.size() +"个多出文件：" + list1);
		        removeDuplicate(list1);
		        
		        
		        //****************************** 复制筛选出的文件  *******************************************/
		        
		        
		        //拼接文件路径
		        String rootPath = FileUploadUtils.class.getClassLoader().getResource("../../").getPath();//绝对路径
		        String filePath;										//解压文件所在路径
		        String fileSrc = "upload/equipmentCopy/" + DateUtils.getLocalDate("yyyyMMddHHmmss");
				String copyPath = rootPath + fileSrc;					//目的地路径
				
				//通过JVM读取java.io.tmpdir属性取得临时文件夹
		        File targetDir = new File(System.getProperty("java.io.tmpdir"));
		        
		        for (String fileName : list1) {							//遍历每个文件名
		        	
		        	filePath = unzipPath + "/" + fileName;
		        	
		        	lastChar = fileName.substring(fileName.length()-1);	//如果文件名末尾为“x”，则去掉的“x”
		        	if (lastChar.equals("x")) {	
		        		fileName = fileName.toString().substring(0, fileName.length()-1);
		        		System.out.println("去除“x”后元素：" + fileName);
		        	}
		        	
		        	File file1 = new File(filePath);					//源文件
		        	File file2 = new File(copyPath  + "/" + fileName);  //备份文件
		        	
		            try
		            {
		            	copyFile(file1, file2);							 // 在同一个文件夹复制文件
		                FileUtils.copyFileToDirectory(file1, targetDir); // 根据指定的文件夹复制
		                
		            } catch (IOException e)
		            {
		                e.printStackTrace();
		            }
		        }
		        
		        //****************************** 将筛选出的文件压缩成zip包   *********************************************/
		        
		        long start = System.currentTimeMillis();
				
				File sourceFile = null;
				FileOutputStream fos = new FileOutputStream(new File(copyPath + ".zip"));//创建压缩文件
				ZipOutputStream zos = new ZipOutputStream(fos);
				
				try {
					sourceFile = new File(copyPath);
					compress(sourceFile,zos,sourceFile.getName(),KeepDirStructure);
					long end = System.currentTimeMillis();
					System.out.println("压缩完成，耗时：" + (end - start) +" ms");
					
					removeDir(sourceFile);//压缩成功后删除备份文件（文件夹）
					
				} catch (Exception e) {
					throw new RuntimeException("zip error from ZipUtils",e);
				}finally{
					if(zos != null){
						try {
							zos.close();
							fos.flush();
							fos.close();
							rs.put("zipPath", fileSrc + ".zip");//将zip文件所在路径返回
							
						} catch (IOException e) {
							e.printStackTrace();
						}
					}
				}
			}
		}
		return rs;
	}
                
	
	
	/**
	 * 复制文件
	 * @param source	目标文件
	 * @param dest		目的地
	 * @throws IOException
	 */
	private static void copyFile(File source, File dest) throws IOException {
	    FileUtils.copyFile(source, dest);
	}
	
	
	/**
	 * 递归压缩方法
	 * @param sourceFile 		源文件
	 * @param zos		 		zip输出流
	 * @param name				压缩后的名称
	 * @param KeepDirStructure  是否保留原来的目录结构,true:保留目录结构; 
	 * 							false:所有文件跑到压缩包根目录下(注意：不保留目录结构可能会出现同名文件,会压缩失败)
	 * @throws Exception
	 */
	private static void compress(File sourceFile, ZipOutputStream zos, String name, boolean KeepDirStructure) throws Exception{
		byte[] buf = new byte[BUFFER_SIZE];
		
		if(sourceFile.isFile()){
			
			// 向zip输出流中添加一个zip实体，构造器中name为zip实体的文件的名字
			zos.putNextEntry(new ZipEntry(name));
			// copy文件到zip输出流中
			int len;
			FileInputStream in = new FileInputStream(sourceFile);
			while ((len = in.read(buf)) != -1){
				zos.write(buf, 0, len);
			}
			zos.setEncoding(ENCODE);
			zos.closeEntry();
			in.close();
			
		} else {
			File[] listFiles = sourceFile.listFiles();
			if(listFiles == null || listFiles.length == 0){
				// 需要保留原来的文件结构时,需要对空文件夹进行处理
				if(KeepDirStructure){
					// 空文件夹的处理
					zos.putNextEntry(new ZipEntry(name + "/"));
					// 没有文件，不需要文件的copy
					zos.closeEntry();
				}
				
			}else {
				for (File file : listFiles) {
					// 判断是否需要保留原来的文件结构
					if (KeepDirStructure) {
						// 注意：file.getName()前面需要带上父文件夹的名字加一斜杠,
						// 不然最后压缩包中就不能保留原来的文件结构,即：所有文件都跑到压缩包根目录下了
						compress(file, zos, name + "/" + file.getName(),KeepDirStructure);
					} else {
						compress(file, zos, file.getName(),KeepDirStructure);
					}
					
				}
			}
		}
	}
	
	
    /**
	 * 删除一个带内容的目录 
	 * 原理：必须要从最里面往外删 ,需要深度遍历 
	 * @param dir 目录
	 */ 
	public static void removeDir(File dir) {  
        File[] files=dir.listFiles();  
          
        for(File file:files) {  
            if(file.isDirectory()) {  
            	removeDir(file);
            }  
            else {  
               file.delete();  
            }  
              
        }  
        dir.delete();
    }  
	
	
	/**
	 * list中元素去重
	 * @param list
	 */
	@SuppressWarnings({ "rawtypes", "unchecked" })
	public static void removeDuplicate(List list) {
		
		HashSet h = new HashSet(list);
		list.clear();
		list.addAll(h);
		System.out.println("集合去重复后：" + list);
		
		}
	
	
	/**
	 * 按指定编码读取文件内容
	 * @author  		huzi
	 * @date    		2018年6月29日	
	 * @param path		需读取文件的路径
	 * @param encoding	要转成的编码格式
	 * @return
	 * @throws IOException
	 */
	public static String read(InputStream inputStream, String encoding) throws IOException {
		String content = "";
	    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, encoding));
		String line = null;
		while ((line = reader.readLine()) != null) {
			content += line + "\n";
		}
		reader.close();
		return content;
	}
	

	
	/**
	 * 写入文件内容
	 * @author  		huzi
	 * @date    		2018年6月29日	
	 * @param path		目的地文件路径
	 * @param content	需写入的内容
	 * @return
	 * @throws IOException
	 */
	public static void write(String path, String content)throws IOException {
		File file = new File(path);
		file.delete();
		file.createNewFile();
		BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(file)));
		writer.write(content);
		writer.close();
	}


}
