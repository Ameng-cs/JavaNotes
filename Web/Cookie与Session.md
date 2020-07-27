# Cookie
- ### 为什么发明Cookie
    HTTP是无状态的协议,即每次request请求是相互独立的,也就是说当前的请求并不会记录之前请求的信息,这就可能导致 你在登陆之后点击一个超链接,进去之后网页会让你再次登陆,因为HTTP无法记录你的登陆状态.所以我们需要Cookie来记录之前request请求的状态.
- ### Cookie工作的原理
    **Cookie是HTTP协议规定的,存在头信息中,以键值对的形式存在.** 首先浏览器在发送request请求之前是没有Cookie的,第一个Cookie由服务器产生,服务器会根据request请求中携带的表单信息或者是用户信息用response.addSession("key","value")来创建Cookie,并且保存在响应头中发回给浏览器,这个头信息就可以保存浏览器的状态(登录状态等),之后每次浏览器发送HTTP请求都会将Cookie发送给服务器,以表明状态.
![image](https://upload-images.jianshu.io/upload_images/13949989-dcf024be2733e725.png?imageMogr2/auto-orient/strip|imageView2/2/w/400)
- ### Cookie的属性
    - **Path:** 表示这个cookie影响到的路径，当前访问的路径不满足该匹配时，浏览器则不发送这个cookie.
        - 例如:aCookie.path = /path1/,bCookie.path=/path1/path2,cCookie.path = path1/path2/path3
        - 访问/path1/index.jsp时 ,归还aCookie
        - 访问/path1/path2/index.jsp时,归还aCookie和bcookie
        - 访问/path1/path2/path3/index.jsp时 归还aCookie和bcookie以及cCookie
        - **path的默认值**:当前访问路径的父路径
    - **maxAge:** Cookie的最大生命时长,单位为秒
        - maxAge>0:浏览器会把cookie保存在客户机的硬盘上,有效时长为maxAge值决定
        - maxAge<0: Cookie只在浏览器内存中存在,当浏览器关闭时Cookie也死亡了
        - maxAge=0:浏览器会马上删除这个cookie
    - **Domain** 用来指定生成Cookie的域名,当多个二级域名共享Cookie时才有用
- ### Cookie 的主要方法:
    - response.addCookie() 向浏览器添加Cookie
    - request.getCookies() 获取浏览器归还的Cookie
- ### Cookie的缺陷:
    - **Cookie的数量和长度的限制.** 每个domain最多只能有20条cookie，每个cookie长度不能超过4KB，否则会被截掉。
    - **安全性问题**.如果cookie被人拦截了，那人就可以取得所有的session信息。即使加密也与事无补，因为拦截者并不需要知道cookie的意义，他只要原样转发cookie就可以达到目的了
    - **有些状态不可能保存在客户端**。例如，为了防止重复提交表单，我们需要在服务器端保存一个计数器。如果我们把这个计数器保存在客户端，那么它起不到任何作用。
# Session
- ### 为什么发明Session
    同Cookie的原因大致相似,因为HTTP协议的无状态的特性,为了保存客户端的状态信息,就在**服务端**创建一个Session对象用来存储客户端的状态.
- ### 为什么有了Cookie还要Session?
    - Session是基于Cookie的,Session会为每个浏览器的一次会话生成一个SessionID,存在Cookie里面
    - Cookie的大小有限制,浏览器的Cookie无法存放大量信息,而Session是存在服务器上的,仅仅通过一个存SessionID就可以获得服务器上的大量信息
    - Cookie在HTTP请求中以明文传递,可能存在安全问题,而Session存在服务器,一般无法获取.
- ### Session原理
    - Session就是服务器为了存储用户状态信息在服务器内存开辟的一个内存空间
    - jsp文件会自动帮你生成SessionID,而Servlet只有显式的调用getSisson方法才会生成SessionID
    - 如果创建了一个新的Session,浏览器会得到一个包含了SessionID的Cookie,这个Cookie的生命值为-1,即存在浏览器内存中,关闭浏览器该Cookie就会死亡,也就是说关闭当前浏览器你就会丢失当前对话
    - 首先当你第一次获取Session的时候(没获取的话就没有),服务器会为该会话生成一个独一无二的SessionID该ID用来标识该会话状态信息在服务器存储的位置.服务器会把该SessionID以Cookie的形式放在响应头里,传给浏览器,浏览器将该ID保存.浏览器在每次request请求的时候将SessionID发送服务器,服务器通过id查找,若内存中已经存在该session的信息,则直接从服务器内存中获取信息.
- ### Session的主要方法
    - Session.setAttribute("key","value") 
    - Session.getAttribute("key")
    - Session.removeAttribute("key")
- ### Session其他方法
    - **String getId():** 获取SessionID,SessionID是不重复的十六进制字符串
    - **int getMaxInactiveInterval():** 获取Session可以的最大不活动时间(秒).默认为30分钟.Session在30内没有使用的话Tomcat会从Session池中将过期的Session移除.
    - **void invalidate():** 让Session失效.
    - **boolean isNew():** 查看Session是否为新.当客户第一次请求的时候  ,服务器会为浏览器创建Session,但这时服务器还没有响应客户端,也就是还没有把SessionID发给客户端,这时Session就是新的.也就是说request.getSession().isNew()会告诉你服务器是返回给你已存在的Session还是在创建一个新的Session
- #### URL重写
    - Session是依赖Cookie工作的.客户端请求时归还SessionID,服务器在缓存中根据ID找到相应的Session
    - 如果浏览器禁用Cookie,浏览器无法归还SessionID这样的话Session不就无法工作了么?
    - 可以利用URL中的参数来携带SessionID,也即URL重写
    - response.encodeURL(String url) 该方法会将网站所有的超链接以及表单中都添加一个特殊的请求参数,该参数就用来存放SessionID,这样服务器就可以通过请求参数来的到SessionID
    - 该方法会对URL进行智能的重写,当请求中没有归还SessionID这个Cookie,那么该方法就会重写URL,否则不会(URL必须是指向本站的URL).
## Session和Cookie的区别
- ### Cookie数据存放在客户机浏览器上,Session数据存放在服务器里
- ### Cookie不是很安全,可以通过访问本地Cookie进行Cookie欺骗,而Session是安全的
- ### 单个Cookie只能保存4K的数据,一个站点最多保存20条Cookie,而Session容量大得多 
- ### session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用cookie