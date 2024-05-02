# qqwryMGeoip_mmdb
（qqwry ip CN库）与（MaxMind GeoLite）合并

  合并了(qqwry-纯真 IP库CN段)和(MaxMind GeoLite2.mmdb)的IP数据库。也就是：提取了qqwry的CN段覆盖MaxMind的IP库。最终格式为MaxMind的*.mmdb  
因为感觉qqwry的海外IP不准，而MaxMind的中国IP不准。才这么搞。(再之前因为折腾NexusPHP PT站，新版本网站用的是mmdb识别IP地址。)

源ip库文件：  
qqwary.dat，2024年4月24日版。MD5: c3ac7c7606bdfd83f31907f060b7b607  
GeoLite2-City.mmdb 2024年5月01日版 MD5: b4e66cdb59dcb1297ffea4e8582e3540  
GeoLite2-Country.mmdb 2024年5月01日版 MD5: b72b6db6d4b10ddec23daccd06145f90  

制作步骤：本咸鱼不会代码，全用工具搞的。  
大纲，转换步骤：qqwry.dat>qqwry.mmdb>qqwry.json>qqwry-CN.json>qqwry-CN.json+GeoLite2.mmdb=Geo+qqwryCN.mmdb

1. qqwry转mmdb，工具qqwry2mmdb，https://github.com/leolovenet/qqwry2mmdb 。运行这个我用的Linux环境
   由于MaxMind官方软件更新，新版本不支持（MaxMind::DB::Writer::Tree）指令。需要安装旧版软件。
   需要按照qqwry2mmdb的安装再执行以下命令，安装旧版Writer。
```cpanm http://www.cpan.org/authors/id/M/MA/MAXMIND/MaxMind-DB-Writer-0.300003.tar.gz```

   做成了Docker:
   ```
   docker run -itd -p 11451:11451 --name q2m-ok2 gduc233/qqwry2mmdb
   ``` 
   ssh-Port:11451 ID/Pass:root/114514。脚本在目录/home/qqwry2mmdb*下
   转换命令perl qqwry2mmdb.pl qqwry.dat
得到qqwry.mmdb
  
3. qqwry转json，我用的win环境。工具mmdbinspect  https://github.com/maxmind/mmdbinspect 。运行这个我用的Win10环境
  ```mmdbinspect.exe -db qqwry.mmdb 0.0.0.0/0 >> qqwry.json```

4. qqwry.json合并入GeoLite2.mmdb，工具mmedit https://github.com/iglov/mmdb-edito  运行这个我用的Win10环境

  首先需要检查一下qqwry.json是否符合工具识别的json格式，qqwry.json应该需要修改一些：
  运行下行命令，json不符合的地方，mmedit会给出提示。参照mmedit主页的mmdb-editor/testdata/dataset.json，的标准格式，进行修改qqwry.json。
  
  ```mmdb-editor-windows-amd64.exe -d qqwry.json -i GeoLite2.mmdb -o merge.mmdb```
  
  4. 提取中国IP，我用的UltraEditor和EmEditor。
  
     4.1 UltraEditor通配符转换替换，制作成每个json段为一行。记得标记回车符，例如\n替换为RRR233，这样做为了之后恢复文件成json。
     
     4.2 再用EmEditor使用通配符，查找+提取为新文件。通配符:
     
          ```.*北京.*|.*天津.*|.*河北.*|.*山西.*|.*内蒙古.*|.*辽宁.*|.*吉林.*|.*黑龙江.*|.*上海.*|.*江苏.*|.*浙江.*|.*安徽.*|.*福建.*|.*江西.*|.*山东.*|.*河南.*|.*湖北.*|.*湖南.*|.*广东.*|.*广西.*|.*海南.*|.*重庆.*|.*四川.*|.*贵州.*|.*云南.*|.*西藏.*|.*陕西.*|.*甘肃.*|.*青海.*|.*宁夏.*|.*新疆.*|.*中国.*```
     
     4.3 恢复为json。查找替换标记的回车符RRR233为\n。得到qqwryCN.json

  6. 进行最后的合并，使用工具mmedit。
     

     5.1 以GeoIp的城市版，做基底，就用GeoLite2-City.mmdb。这样国外ip会精确到城市+国内的qqwry是城市ip段。
     
        ```mmdb-editor-windows-amd64.exe -d qqwryCN.json -i GeoLite2-City.mmdb -o mergeCity.mmdb```
   
     5.2 以GeoIp的国家版，做基底，就用GeoLite2-Country.mmdb。这样国外ip会精确到国家+国内的qqwry是城市ip段。
     
        ```mmdb-editor-windows-amd64.exe -d qqwryCN.json -i GeoLite2-Country.mmdb -o mergeCountry.mmdb```

完成。
-    
-----附录--------leolovenet/qqwry2mmdb的环境安装方式----------------
```
apt update
apt install git curl wget unzip gcc make perl libmaxmind-db-writer-perl 
cpan App::cpanminus
cpanm --notest --force IP::QQWry::Decoded
cpanm --notest --force http://www.cpan.org/authors/id/M/MA/MAXMIND/MaxMind-DB-Writer-0.300003.tar.gz

wget https://github.com/leolovenet/qqwry2mmdb/archive/refs/heads/master.zip
unzip master.zip
cd qqwry2mmdb-master
```
--  
---  
----  
-----  
             
-----------以下是PT Nexus的mmdb识别问题，与主题ip库合并无关-----------
折腾PT站 xiaomlove/nexusphp时，ip识别用的是mmdb。然而加载自定义*.mmdb时会出现一些问题，日志显示  
```production.ERROR /nexusphp/include/functions.php:5887 {closure} The city method cannot be used to open a userqqwry database#0 /nexusphp/vendor/geoip2/geoip2/src/Database/Reader.php(231): GeoIp2\Database\Reader->getRecord()```
原因是，默认加载mmdb文件为MaxMind的city类型，也就是上面官方的GeoLite2-City.mmdb。php有功能会检测mmdb的info段的datebase值必须为'GeoIP2-City。其他的'GeoIP2-Country'或自定义都不行。  
我是改了文件 /nexusphp/vendor/geoip2/geoip2/src/Database/Reader.php 。删除了一段。
```
            $method = lcfirst((new \ReflectionClass($class))->getShortName());

            throw new \BadMethodCallException(
                "The $method method cannot be used to open a {$this->dbType} database"
            );
```
之后就能读取到mmdb了。查看生效时要换个ip，因为有缓存。  



      


