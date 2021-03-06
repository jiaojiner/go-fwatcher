# 程序介绍
go版的文件监控程序,只支持linux系统,提供<a href="http://man7.org/linux/man-pages/man7/inotify.7.html">inotify</a>的底层的原始事件监控,使用时可以获取完全自定义事件,事件发生的时候调用你的自定义命令实现业务操作.支持递归监控,只需要设置要监控的顶级目录,里面的子目录会自动加入监控,而且支持监控目录中动态生成的子目录.  
# 文件说明
fwatcher.go是源文件  
bin/fwatcher-linux-amd64 是预编译的Linux64位程序  
bin/fwatcher-linux-386   是预编译的Linux32位程序  
# 源代码使用
依赖类库:  
github.com/tywkeene/go-fsevents  
golang.org/x/sys/unix  
1.首先下载源代码  
mkdir go-fwatcher  
cd go-fwatcher  
git clone https://github.com/snail007/go-fwatcher.git .  
2.安装依赖包  
go get ./...  
3.然后就可以正常修改源代码开发了     
由于墙的原因如果有些包下载失败，可以到这里手动下载解压到$GOPATH/src下面对应的地方，  
golang包下载助手：http://golangtc.com/download/package  

# 编译
1、编译64位程序   
GOOS=linux GOARCH=amd64 go build fwatcher.go   
2、编译32位程序   
GOOS=linux GOARCH=386 go build fwatcher.go   
提示：    
编译编译32位也就是386程序的时候，需要注视掉下面这一行：    
"IN_ONESHOT":     unix.IN_ONESHOT,     //只监测一次     

# 参数说明 
<pre>
  -dir 字符串  
    	 监控目录设置,这里填写一个绝对路径即可,程序会自动监控里面的子目录 (default "/tmp")  
  -cmd 字符串  
       设置事件发生的时候执行的命令   
       默认是: "echo %f %t"  
    	 当监控的事件发生时,就执行调用这个参数设置的命令字符串,命令字符串里面可以
	 使用%f代表发送变化的文件,%t代表发生的事件的flag,有可能是多个事件flag,
	 意思是这几个事件同时发生,是用逗号分割的,命令可以通过判断%t知道发生的事件
	 是否是自己需要处理的事件.  
  -events 字符串  
    	 设置想要监听的事件flag;  
         默认是:"IN_ALL_EVENTS,IN_ISDIR,IN_CLOSE,IN_MOVE,IN_EXCL_UNLINK"  
         全部可用的事件flag如下:  
      //基础flag  
      "IN_ACCESS":        unix.IN_ACCESS,        //文件被访问  
      "IN_ATTRIB":        unix.IN_ATTRIB,        //权限,时间戳,UID,GID,其他属性等等,
      						 //link链接的数量 (since Linux 2.6.25)   
      "IN_CLOSE_NOWRITE": unix.IN_CLOSE_NOWRITE, //以非write方式打开文件并关闭  
      "IN_CLOSE_WRITE":   unix.IN_CLOSE_WRITE,   //以write方式打开文件并关闭  
      "IN_CREATE":        unix.IN_CREATE,        //文件或目录被创建  
      "IN_DELETE":        unix.IN_DELETE,        //文件或目录被删除  
      "IN_DELETE_SELF":   unix.IN_DELETE_SELF,   //监控的根目录或文件本身被删除  
      "IN_MODIFY":        unix.IN_MODIFY,        //文件内容被修改  
      "IN_MOVED_FROM":    unix.IN_MOVED_FROM,    //文件移出被监测的目录  
      "IN_MOVED_TO":      unix.IN_MOVED_TO,      //文件移入被监测的目录  
      "IN_MOVE_SELF":     unix.IN_MOVE_SELF,     //监测的根目录或文件本身移动  
      "IN_OPEN":          unix.IN_OPEN,          //文件被打开  
      "IN_CLOEXEC":       unix.IN_CLOEXEC,  
      //集合flag  
      "IN_ALL_EVENTS": unix.IN_ALL_EVENTS, //	以上所有flag的集合"  
      "IN_CLOSE":      unix.IN_CLOSE,      //IN_CLOSE_WRITE | IN_CLOSE_NOWRITE  
      "IN_MOVE":       unix.IN_MOVE,       //IN_MOVED_FROM | IN_MOVED_TO  
      //不常用的flag  
      "IN_DONT_FOLLOW": unix.IN_DONT_FOLLOW, //不follow符号链接 (since 2.6.15)  
      "IN_EXCL_UNLINK": unix.IN_EXCL_UNLINK, //当文件从监测目中unlink后，
      					     //则不再报告该文件的相关event，
					     //比如监控/tmp使用 (since 2.6.36)  
      "IN_MASK_ADD":    unix.IN_MASK_ADD,    //追加MASK到被监测的pathname    
      "IN_ONESHOT":     unix.IN_ONESHOT,     //只监测一次  
      "IN_ONLYDIR":     unix.IN_ONLYDIR,     //只监测目录  
      //仅由read返回  
      "IN_IGNORED":    unix.IN_IGNORED,    //inotify_rm_watch，文件被删除或者文件系统被umount  
      "IN_ISDIR":      unix.IN_ISDIR,      //发生事件的是一个目录  
      "IN_Q_OVERFLOW": unix.IN_Q_OVERFLOW, //Event队列溢出  
      "IN_UNMOUNT":    unix.IN_UNMOUNT,    //文件系统unmount
      </pre>
# 提示
如果你设置了参数-events包含了集合类型flag，IN_CLOSE、IN_MOVE。  
那么当发生IN_CLOSE事件，-cmd参数里面的%t就是:IN_CLOSE_WRITE,IN_CLOSE_NOWRITE，而不是IN_CLOSE。  
那么当发生IN_MOVE事件，-cmd参数里面的%t就是:IN_MOVED_FROM,IN_MOVED_TO，而不是IN_MOVE。  
集合flag的作用是，帮助快速的设置我们要监听的事件flags，事件发生返回的flag也就是-cmd参数里面的%t只会是非集合flag的组合。  




