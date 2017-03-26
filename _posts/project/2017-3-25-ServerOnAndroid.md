---
layout: post
title: ServerOnAndroid         
category: project
description: 在Android平台运行的服务器，并基于http协议实现大规模数据上传下载       
---


声明：本博客欢迎转发，但请保留原作者信息!      
作者：高伟毅    
博客：[https://gwyve.github.io/](https://gwyve.github.io/)    
微博：[http://weibo.com/u/3225437531/](http://weibo.com/u/3225437531/)    

## 引言      

这个项目是我在2015年底就打算写的项目，期初就是就是因为个人一个十分奇葩的想法：把服务器部署到Android平台上。后来，我确实有了真正的需求，逼迫我必须写这样的项目。

## 项目来源    

1. 奇葩的想法：在Android平台上运行一个服务器
2. 完成Android与PC的数据传输。

补充：因为本人对于电池不可拆卸的手机十分排斥，至今仍然在使用中兴 红牛V5max（一款可以电池可拆卸的手机）。用了不到两年的时候usb插口坏了，虽然电池可以用座充充电，但是，数据传输十分不方便，尤其，作为一个迷恋用手机看海贼的海贼迷，传输几十集动漫十分不方便，不得已，就开始设想一个这样的应用。

ps：后来发现，把服务器部署在PC上，做一个批量下载就可以了，囧～～


## 代码托管

[https://github.com/gwyve/DataWire.git](https://github.com/gwyve/DataWire.git)

## 项目需求

随着越来越多的Android手机的使用，越来越多的数据需要从pc传到Android手机上，或者从Android手机上传到PC上。经调查，很多用户在pc与Android互传数据都会选择微信、手机qq、百度云等app或者直接使用数据线链接，而这些app都有着各种各样的问题，比如，微信不能上传10mb以上的文件，手机qq上传之后的文件难以找到具体路径；在使用数据线时，如果数据线有一点震动都会造成传输停止。所以，能够通过无线进行pc与Android进行数据传输的app就迫切需要。

现在手机尺寸越来越大，很多人都趋向于将Android手机当做一个视频播放器使用；而有的视频使用在线观看体验就比较差。比如作者喜欢看《海贼王》，而《海贼王》从最开始的各大视频平台都有，到后来只有搜狐视频才有，到目前之后爱奇艺才有，甚至一段时间只有一部分可以看。所以，很多海贼迷都趋向与把视频先下载下来然后用手机进行观看，一次性往手机上存上很多集，这就需要这个app能够满足用户的批量上传。同样，韩剧迷也有相同需求。
综上所述，具体需求：
1. 通过局域网wifi实现文件的下载
2. 通过局域网wifi实现文件的上传
3. 不必在PC上安装其他任何app
4. 实现文件的批量上传
5. 通过PC浏览器对Android的文件夹进行查看
6. 文件上传速度争取在5mb/s以上
7. 通过PC浏览器新建文件夹、删除文件(待完成)
8. 对Android的存储空间提醒（待完成）

## 项目设计

### 架构设计：

因为需求中写到不必在PC端安装任何其他软件，所以就是利用浏览器。这就需要在Android端部署一个server。因为Android所分配给app的内存空间有限，所以在具体问题上有区别部署在电脑端的server。
 
![design](/images/project/2017-3-25/design.png) 

### 协议分析设计

当收到HTTP报文之后，首先获得报文的前8k（这个数值是可以修改的，目前仍然是使用这个大小），将这8k存放在内存中，然后进行解析。如果是GET请求则根据Url返回Url所指定的目录下的文件列表；如果是POST怎认为是文件上传。

区别与传统server最大的设计在于，文件上传的过程中需要把数据量进行分割，而不是传统服务器把http报文全部存在内存，然后再对报文进行分析，再将文件进行存入文件系统。在上传文件时，浏览器端采用POST向Android server发送请求。Server根据content-length的长度，将HTTP报文body部分先分析出文件名，以及boundary，boundary之后即为文件内容。将文件内容按照第一次fileBufferSize大小读入内存，然后转存到文件系统，随后开始从socket读第二次fileBufferSize到内存，然后转存到文件系统，依次类推，直到读完一个HTTP报文。

## 主要难题以及解决方案

### 1. Android手机为每个app所分配的内存空间有限

每个厂商对自己手机的app分配内存空间大小都有自己的选择，但是，通常不会超过200Mb，这就要求我们在上传超大文件的时候，如果采用传统server先将报文全部读入内存然后解析方法，会造成outOfMemory的错误。我们所想到的解决方案有两个，第一个为使用ftp协议；第二个就是在项目设计中提到的解决方案。介于不需要用户在PC上安装其他软件的需求，以及目前网络上能找到ftp协议的内容较少，我们采用后者。

### 2.不同浏览器内核对HTTP报文的发送有区别

我们在测试过程中发现，firefox对post请求是一次性的全部发送，而chrome则选择先将head发送再将body发送或者一次性的全部发送，chrome选择那种发送方式是随机的，并没有规律。这就要求我们在接受报文的时候多做几次判断。

### 3.报文的中断

fileBufferSize是我们在想我们需要fileBufferSize大小的报文，server就能够接收到fileBufferSize大小的报文而设定的。但是，实际情况是，很多情况每次读socket报文读出的长度都不是一个固定值，这是一个随机值。所以，我们需要针对每次读socket数据大小做一个判断。当然，我们希望每次读socket的数据大小越接近fileBufferSize应该是速度越快的。

### 4.文件批量上传

针对文件批量上传，起初有两个解决方案，第一个是采用浏览器每个POST请求都带有批量的文件数据（即，报文的body里是按照boundary+文件属性+文件数据+文件属性+文件数据+…+boundary），然后根据每个fileBufferSize大小的数据进行分析看是不是一个文件的结束和文件的开始。第二个是在PC浏览器采用异步的方式，具体使用Ajax进行实现，即使用浏览器前端形成一个文件上传队列，每个请求只发送一个文件，这样每次报文只用分析一个body和第一个fileBufferSize大小的数据。需求中提到的动漫或者韩剧，这类文件都是视频，一般都是几百兆的文件，所以，尽量不要分析那么多次的报文，最好就是直接转存。为了提高速度，我们采用第二种解决方案。

### 5.提高文件的上传速度

如果，文件上传速度过慢，这样就没有满足用户的任何需求。我们在解决这样的问题，提出两个方面的解决措施：硬件和软件。硬件方面：目前wifi主要是802.11b/g/n，他被操作在2.4 GHz 的频率中，在其中这个2.4 GHz 频谱被划分为14个交叠的、错列的20 MHz 无线载波信道，它们的中心频率分别为5 MHz；随着硬件的革新，新开始普及使用的802.11a/n ，他被操作在有更多信道的 5.0GHz 频谱中，802.11n 也使用信道焊接技术联合两个 20MHz 载波信道为一个40 MHz 信道来增加吞吐量。经过测试，5GHz的频率确实比2.4GHz快，因为条件有限，导致快的因素是因为干扰问题还是硬件问题无法得知。但是，有一点目前可以确定，如果让wifi路由器使用其他wifi路由器使用最少的频道，可以有效地减少干扰，提高传输速度。软件方面：可以看出，我们之前的设计都是在考虑上传速度，根据之前的设计，我们所能修改的参数主要是fileBufferSize大小，而这个值是在考虑app内存前提下设定的，即不超过app内存，越接近每次读出socket数据大小越好。

### 6.耦合，内聚

因为整个app相当于在Android设备上部署一台小型服务器，如果处理不好耦合，那将是在Android框架的Activitiy出现各种server的代码，这样在代码的维护上产生了很大问题。所以，我们将几乎所有的server部分代码都写在com.gwyve包内，在Android的mainActivitiy中使用单例模式生成server来进行server部分的功能。因为Android自身其他进程不能阻塞界面进程的机制，server部分与Actcivity部分使用Handler进行通信，以此达到server部分控制Activity部分的显示。
 
## 代码实现

项目主要目录截图：

![code](/images/project/2017-3-25/code.png) 

说明：SplashActivity是app开机时的欢迎界面，MainActivity是app启动server和停止server的界面。MiniServer是server部分的主要内容，包括对报文的解析，上传、显示、下载文件的内容；HtmlUtil为server中用于显示html网页的内容工具，以及Ajax异步上传文件的代码；Util为工具类，主要为获得Android局域网内ip地址和app的端口。

### 1.单例模式生成server

Path:com.ve.administrator.datawire.MainActivity

```java
if (miniServerThread==null){
    miniServer = MiniServer.singletonMiniServer(Environment.getExternalStorageDirectory(), ip, textViewHandler);
    miniServerThread = new Thread(miniServer);
    miniServerThread.setDaemon(true);
    miniServerThread.start();
}else{
    if(!miniServerThread.isAlive()){
        miniServerThread.start();
    }
    textView.setText("Please put \""+miniServer.ip+":"+miniServer.port+"\" as path address in the browser!");
}
```

### 2.获得HTTP报文的前8k内容

Path：com.gwyve.MiniServer

```java
final int bufsize = 8192;
InputStream is = client.getInputStream();
if(is == null)
    return;
byte[] buf = new byte[bufsize];
int splitbyte = 0;
int hrlen = 0;
int read = is.read(buf, 0, bufsize);
{
    while (read > 0)
    {
        hrlen += read;
        splitbyte = findHeaderEnd(buf, hrlen);

        if (splitbyte > 0)
            break;
        read = is.read(buf, hrlen, bufsize - hrlen);
    }
}
decodeHeader(buf, parms,header);
```

### 3.解析HTTP报文head

Path：com.gwyve.MiniServer

```java
private void decodeHeader(byte[] buf,Properties parms,Properties header){
    String bufStr = new String(buf);
    String[] bufStrArr = bufStr.split("\r\n");
    if(bufStrArr[0].indexOf(" ") != -1 &&bufStrArr[0].indexOf("HTTP") != -1){
        parms.put("method", bufStrArr[0].substring(0, bufStrArr[0].indexOf(" ")));
        parms.put("uri",bufStrArr[0].subSequence(bufStrArr[0].indexOf(" ")+1, bufStrArr[0].indexOf("HTTP")-1));
    }
    for(String str : bufStrArr){
        if(str.indexOf("boundary")!=-1 && str.indexOf("=") != -1){
            parms.put("boundary", str.substring(str.indexOf("=")+1,str.length()));
        }else if(str.indexOf("Host")!=-1 && str.indexOf(" ") != -1){
            parms.put("host", str.substring(str.indexOf(" ")+1, str.length()));
        }else if(str.indexOf("Content-Length")!=-1 && str.indexOf(" ") != -1){
            parms.put("content-length", str.substring(str.indexOf(" ")+1,str.length()));
        }
        if(str != null && str.trim().length() > 0){
            int p = str.indexOf( ':' );
            if ( p >= 0 ){
                header.put( str.substring(0,p).trim().toLowerCase(), str.substring(p+1).trim());
            }
        }

    }
}
```

### 4.处理文件上传相关代码

Path：com.gwyve.MiniServer

```java
private String uploadFile(byte[] beginBuf,int hrlen,int splitbyte,String dirPath,InputStream is, Properties parms){
        try {
            int fileBeginbyte = 0;
            int rlen = 0 ;
            int boundaryLength = parms.getProperty("boundary").getBytes().length;
            long size = 0x7FFFFFFFFFFFFFFFl;
            if(parms.getProperty("content-length")!=null)
            {
                try { size = Integer.parseInt(parms.getProperty("content-length")); }
                catch (NumberFormatException ex) {}
            }

            fileBeginbyte = findFileBegin(beginBuf, splitbyte, hrlen);
            byte[]  fileBuf = new byte[fileBufferSize];
            if(fileBeginbyte>0){
                findBodyInfo(beginBuf, parms);
                File outFile = new File(dirPath +"/"+ parms.getProperty("filename"));
                if(outFile.exists())
                    outFile.delete();
                outFile.createNewFile();
                OutputStream os = new FileOutputStream(outFile);
                if(hrlen >= splitbyte+size-(boundaryLength+8))
                {
                    os.write(beginBuf, fileBeginbyte, splitbyte+(int)size-(boundaryLength+8 )-fileBeginbyte);
                    os.close();
                }else{
                    os.write(beginBuf, fileBeginbyte, hrlen - fileBeginbyte);

                    size -= (hrlen-splitbyte) +(boundaryLength+8) ;
                    rlen = is.read(fileBuf, 0, fileBufferSize);
                    while(size-rlen>=0 && rlen>0){
                        if(rlen == fileBufferSize){
                            os.write(fileBuf);
                        }else{
                            os.write(fileBuf, 0, rlen);
                        }
                        size -= rlen;
                        rlen = is.read(fileBuf, 0, fileBufferSize);
                    }
                    os.write(fileBuf, 0, (int)size);
                    os.close();
                }
            }else{
                fileBeginbyte = 0;
                while(rlen >= 0 && size > 0){
                    rlen = is.read(fileBuf, 0, fileBufferSize);
                    fileBeginbyte = findHeaderEnd(fileBuf, rlen);
                    if(fileBeginbyte>0){
                        break;
                    }else{
                        size -= rlen;
                    }
                }
                size-=fileBeginbyte+(boundaryLength+8);
                findBodyInfo(fileBuf, parms);
                File outFile = new File(dirPath +"/"+ parms.getProperty("filename"));
                if(outFile.exists())
                    outFile.delete();
                outFile.createNewFile();
                OutputStream os = new FileOutputStream(outFile);

                if(rlen < fileBufferSize){
                    os.write(fileBuf, fileBeginbyte, (int)size);
                    os.close();
                }else{
                    for(int i = fileBeginbyte; i <fileBufferSize;i++)
                        os.write(fileBuf[i]);

                    size += fileBeginbyte - fileBufferSize;
                    rlen = is.read(fileBuf,0,fileBufferSize);
                    while((size-rlen)>0 && rlen>0){
                        if(rlen == fileBufferSize){
                            os.write(fileBuf);
                        }else{
                            os.write(fileBuf, 0, rlen);
                        }
                        size -= rlen;
                        rlen = is.read(fileBuf,0,fileBufferSize);
                    }
                    os.write(fileBuf,0,(int)size);
                    os.close();
                }

            }
        }catch (Exception e) {
            System.out.println("error"+e.getMessage());
        }
        return null;
    }
    
```

### 5.分析HTTP POST请求关于文件开端和boundary等信息

Path：com.gwyve.MiniServer

```java
private int findFileBegin(byte[] buf, int splitbyte, int hrlen) {
    // TODO Auto-generated method stub
    while (splitbyte + 3 < hrlen)
    {
        if (buf[splitbyte] == '\r' && buf[splitbyte + 1] == '\n' && buf[splitbyte + 2] == '\r' && buf[splitbyte + 3] == '\n')
            return splitbyte + 4;
        splitbyte++;
    }
    return 0;
}
private void findBodyInfo(final byte[] buf,Properties parms){
    String string = new String(buf);
    String[] strArr = string.split("\r\n");
    for(String str : strArr){
        Log.e("111",str);
        if(str.indexOf("filename")!=-1){
            parms.put("filename", str.substring(str.indexOf("filename")+10, str.length()-1));
        }
    }
}
```

### 6.批量上传前端Ajax代码

Path：com.gwyve.HtmlUtil

```java
String js ="<script type=\"text/javascript\">" +
        "\tfunction uploadfiles(){\n" +
        "\t\tvar files = document.getElementById(\"files\").files;\n" +
        "\t\tvar url = window.location.href; \n" +
        "\t\tdocument.getElementById(\"uploading\").style.display=\"block\";\n" +
        "\t\tdocument.getElementById(\"uploaded\").style.display=\"block\";\n" +
        "\t\tvar i=0;\n" +
        "\t\tuploadfile(files,url,i);\n" +
        "\t}\n" +
        "\n" +
        "function uploadfile(files,url,i){\n" +
        "\t\tif(i<files.length){\n" +
        "\t\t\tvar file = files[i];\n" +
        "\t\t\tdocument.getElementById(\"uploading\").innerHTML = \"<p>\"+file.name+\" is uploading...\";\n" +
        "\t\t\tvar form = new FormData();\n" +
        "\t\t\tform.append(\"file\",file);\n" +
        "\t\t\tvar xhr = new XMLHttpRequest();\n" +
        "\t\t\txhr.open(\"post\",url,true);\n" +
        "\t\t\txhr.send(form);\n" +
        "\t\t\txhr.onreadystatechange = function(){\n" +
        "\t\t\t\tif(xhr.readyState == 4){\n" +
        "\t\t\t\t\tif(xhr.status == 200){\n" +
        "\t\t\t\t\t\tdocument.getElementById(\"uploading\").innerHTML = \"<p>\"+file.name+\" has been uploaded.\"+\"</p>\";\n" +
        "\t\t\t\t\t\tdocument.getElementById(\"uploaded\").innerHTML += \"<br>\"+file.name;\n" +
        "\t\t\t\t\t\txhr = null;\n" +
        "\t\t\t\t\t\tuploadfile(files,url,i+1);\n" +
        "\t\t\t\t\t}else{\n" +
        "\t\t\t\t\t\txhr = null;\n" +
        "\t\t\t\t\t\tuploadfile(files,url,i);\n" +
        "\t\t\t\t\t}\n" +
        "\t\t\t\t}\n" +
        "\t\t\t}\n" +
        "\t\t}else{\n" +
        "\t\t\tdocument.getElementById(\"uploading\").innerHTML=\"<p>all \"+files.length+\" files have been uploaded.</p>\";\n" +
        "\t\t\talert(\"All \"+files.length+\" files has been uploaded.\");\n" +
        "\t\t}\n" +
        "\t}" +
        "</script>";
```

## 界面演示

### app欢迎界面

![hello](/images/project/2017-3-25/hello.png) 

### app主界面

![main_view](/images/project/2017-3-25/main_view.png) 

### PC主界面

![PC_main_view](/images/project/2017-3-25/PC_main_view.png) 

![PC_main_view2](/images/project/2017-3-25/PC_main_view2.png) 

### 文件上传界面

![upload](/images/project/2017-3-25/upload.png) 

### 文件上传完成界面

![uploaded](/images/project/2017-3-25/uploaded.png) 

## 总结

整个项目的一开始的项目需求都是以本人的需求为出发点的，始终秉承着自己发现问题，自己解决问题的出发点，虽然满足了个人需求，属于个人的定制，但是，缺乏大众的认可度。主要就是并不知道其他用户是否有这样的需求，当然，在日后的工作生活中，仍然要寻找自己的需求，同时也要侧重一下公众需求。
从这个项目中，我学到了很多东西，尤其对课堂上所讲的HTTP协议有了更为深入的理解，除此之外，还有以下几个方面的获得：                       
- 了解了各种浏览器内核对HTTP报文发送的流程                       
- 了解了路由器的相关硬件知识，例如频率和频道                     
- 理解掌握了字节流和字符流的区别                       
- 深入实现了socket编程                                 
- 颠覆了曾经对server的理解，获得了新的认识                         
- 对Android对app内存的分配有了深入的理解                           

综上，虽然整个项目仍然在某些方面有些不足，但是整体功能是实现了的，而且获得了如上许多知识，同时当整个项目完成的时候，那种成就感也是无以言表的。

## 项目展望

随着研二回所日期的到来，整个项目也面临着一个中止。这个项目除了最后两点需求没有满足，其他的需求全部满足，这样，整个项目就是成功的。
我们不难发现，这就是校园里的课程设计，与社会上的软件产品还是有较大的区别的。如果这个项目当做一个产品来做，在以下几个方面还需要加强：                
1. 文件上传进度提醒                           
2. Android存储空间提醒                          
3. 文件以及文件夹的新建和删除                            
4. 界面的美化                        
5. 欢迎页的引导使用           
               
之前觉得这是这个项目的结束，写完上面这几个发现作为产品还有好多东西要做，我们不能一下写完美的软件，但是，我们可以选择趋于完美。


## 感谢

这里特别感谢[nanohttpd](https://github.com/NanoHttpd/nanohttpd)，部分代码借鉴nanohttpd的思想。在这里需要指出，nanohttpd没有处理一个大文件上传的情况，我在这里解决了大文件上传oom的情况，争取有朝一日能够为nanohttpd贡献自己的力量。