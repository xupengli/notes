### http会话机制实现方式
***
1. 在两个页面（较少页面之间）通过$_GET或者$_POST数组之间实现数据的共享！
2. 使用cookie将用户的信息存放在客户端的计算机中，用于保存并不重要的数据
3. 通过session将用户的信息保存在服务器中

###session会话机制

1. **什么是session**  
在 web 应用开发中，Session 被称为会话。主要被用于保存某个访问者的数据。由于 HTTP 无状态的特点，服务端是不会记住客户端的，对服务端来说，每一个请求都是全新的。既然如此，那么服务端怎么知道是哪个访问者在请求它呢？又如何将不同的数据对应上正确的访问者？答案  是，给访问者一个唯一获取 Session 中数据的身份标示。 
 

 打个比方：当我们去超市购物时，被保安告之我们是不能带物品进去的，必须将物品寄放在超市的储物箱中。我们把物品交给了他，他怎么知道这些物品谁是谁的，于是他给了我们不同的钥匙。当我们要取走我们的物品时，用唯一的钥匙打开对应的箱子即可。  

 就如同上面的比方一样，可以将 Session 理解为存放我们数据的“箱子”，当然，这些“箱子”都在服务端那。服务器给访问者唯一的“钥匙”，这个“钥匙”被称作 session_id。访问者凭借自己的 session_id，就能获取到自己存在服务器端的数据。session_id 通过两种方式传给访问者（客户端）：URL 或 cookie。详情参见：传送会话ID

2. **什么是Cookie**  
Cookie 也是由于 HTTP 无状态的特点而产生的技术。也被用于保存访问者的身份标示和一些数据。每次客户端发起 HTTP 请求时，会将 Cookie 数据加到 HTTP header 中，提交给服务端。这样服务端就可以根据 Cookie 的内容知道访问者的信息了。

3. **Session 和 Cookie 有什么关系**  
Session 和 Cookie 做着相似的事情，只是 Session 是将数据保存在服务端，通过客户端提交来的 session_id 来获取对应的数据；而 Cookie 是将数据保存在客户端，每次发起请求时将数据提交给服务端的。session_id 可以通过 URL 或 cookie 来传递，由于 URL 的方式比 cookie 的方式更加不安全且使用不方便，所以一般是采用 cookie 来传递 session_id;  
  * **session_start()函数的作用究竟是什么**    
	如果是基于cookie的会话机制，在调用session_start（）之前，是不能够有任何实际的输出的，
	即使是空格或者是空行！因为session_start()函数调用的时候，其实是通过setCookie()函数
	向cookie中设置了PHPSESSID这个key，对应的value是一个随机的、唯一的32位字符串！ 
	而setCookie前面是不可以有任何实际的输出的！在服务器端，会产生一个以PHPSESSID的value值
	为名字的文件！其中保留的是session中的数据！同时，在脚本中创建$_SESSION超全局数组，并
	将session文件的数据反序列化，添加到$_SESSION数组中！这里的PHPSESSID名字是在	session_name()设置或者在php.ini文件中进行的配置！   
 
 > > session.name= PHPSESSID;
 
  * **Session 和 Cookie不同**  
   同cookie不同的是，session中的数据不仅可以存放字符串，还可以存放数组和对象！
   
4. **Session处理器的重定义**  
在 PHP 中，默认的 Session 处理器是 files，处理器可以用户自己实现。成熟的 Session 处理器还有很多：Redis、Memcached、MongoDB……
为什么不推荐使用 PHP 自带的 files 类型处理器，PHP 官方手册中给出过这样一段 Note：
> 无论是通过调用函数 session_start() 手动开启会话， 还是使用配置项 session.auto_start 自动开启会话， 对于基于文件的会话数据保存（PHP 的默认行为）而言， 在会话开始的时候都会给会话数据文件加锁， 直到 PHP 脚本执行完毕或者显式调用 session_write_close() 来保存会话数据。 在此期间，其他脚本不可以访问同一个会话数据文件。

 其含义在同一个浏览器中同时访问该网站A网页和B网页，提交给服务器的 session_id 就是相同的 (这样才能标记访问者，这是我们期望的效果)。当访问A网页时，PHP 根据提交的 session_id，在服务器保存 Session 文件的路径（默认为 /tmp，通过 php.ini 中的 session.save_path 或者函数 session_save_path() 来修改）中找到了对应的 Session 文件，并对其加锁。如果不显式调用 session_write_close()，那么直到当前 PHP 脚本执行完毕才会释放文件锁。如果在脚本中有比较耗时的操作（比如耗时30s），那么另一个持有相同 session_id 的请求由于文件被锁，所以只能被迫等待，于是就发生了请求阻塞的情况。  
 
 *官方给出的方案：*  
>对于大量使用 Ajax 或者并发请求的网站而言，这可能是一个严重的问题。 解决这个问题最简单的做法是如果修改了会话中的变量， 那么应该尽快调用 session_write_close() 来保存会话数据并释放文件锁。 还有一种选择就是使用支持并发操作的会话保存管理器来替代文件会话保存管理器。

 官方给出的前者我们并不赞同，因为那样操作比较繁琐，个人建议使用支持并发操作的会话保存管理器来替代文件会话保存管理器。例如Redis  
 
 *为什么不选择memcache*  
  * 如果用memcached存储Session，那么当memcached集群发生故障（比如内存溢出）或者维护（比如升级、增加或减少服务器）时，用户会无法登录，或者被踢掉线;
  * memcached的回收机制可能会导致用户无缘无故地掉线。Memcached使用“最近最少使用（LRU）”算法回收缓存;这意味着，如果所有Session的大小大致相同，那么它们会分成两三个slab类。所有其它大小大致相同的数据也会放入同一些slab，与Session争用存储空间。一旦slab满了，即使更大的slab中还有空间，数据也会被回收，而不是放入更大的slab中……在特定的slab中，Session最老的用户将会掉线。用户将会开始随机掉线，而最糟糕的是，你很可能甚至都不会注意到它;当然不光是新用户登录进来会替换掉老用户，即便是新数据，也有可能替换掉老用户登录数据 
  
 *如何冲定义一个会话管理器*  
 > 如果需要在数据库中或者以其他方式存储会话数据， 需要使用 session_set_save_handler() 函数来创建一系列用户级存储函数。 PHP 5.4.0 之后，你可以使用 SessionHandlerInterface 类 或者通过继承 SessionHandler 类来扩展内置的管理器， 从而达到自定义会话保存机制的目的。

 > 函数 session_set_save_handler() 的参数即为在会话生命周期内要调用的一组回调函数： open， read， write 以及 close。 还有一些回调函数被用来完成垃圾清理：destroy 用来删除会话， gc 用来进行周期性的垃圾收集。

 > 因此，会话保存管理器对于 PHP 而言是必需的。 默认情况下会使用内置的文件会话保存管理器。 可以通过 session_set_save_handler() 函数来设置自定义会话保存管理器。 一些 PHP 扩展也提供了内置的会话管理器，例如：sqlite， memcache 以及 memcached， 可以通过配置项 session.save_handler 来使用它们。

 > 会话开始的时候，PHP 会调用 open 管理器，然后再调用 read 回调函数来读取内容，该回调函数返回已经经过编码的字符串。 然后 PHP 会将这个字符串解码，并且产生一个数组对象，然后保存至 $_SESSION 超级全局变量。

 > 当 PHP 关闭的时候（或者调用了 session_write_close() 之后）， PHP 会对 $_SESSION 中的数据进行编码， 然后和会话 ID 一起传送给 write 回调函数。 write 回调函数调用完毕之后，PHP 内部将调用 close 回调函数。

 > 销毁会话时，PHP 会调用 destroy 回调函数。

 > 根据会话生命周期时间的设置，PHP 会不时地调用 gc 回调函数。 该函数会从持久化存储中删除超时的会话数据。 超时是指会话最后一次访问时间距离当前时间超过了 $lifetime 所指定的值。
 
 > 摘自[自定义会话管理器](http://php.net/manual/zh/session.customhandler.php)
 
5. **Session 数据是什么时候被删除的**   
先看看官方手册中的说明：
> session.gc_maxlifetime 指定过了多少秒之后数据就会被视为"垃圾"并被清除。 垃圾搜集可能会在 session 启动的时候开始（ 取决于 session.gc_probability 和 session.gc_divisor）。 session.gc_probability 与 session.gc_divisor 合起来用来管理 gc（garbage collection 垃圾回收）进程启动的概率。此概率用 gc_probability/gc_divisor 计算得来。例如 1/100 意味着在每个请求中有 1% 的概率启动 gc 进程。session.gc_probability 默认为 1，session.gc_divisor 默认为 100。

 再看看两段手册的引用：
 > 如果使用默认的基于文件的会话处理器，则文件系统必须保持跟踪访问时间（atime）。Windows FAT 文件系统不行，因此如果必须使用 FAT 文件系统或者其他不能跟踪 atime 的文件系统，那就不得不想别的办法来处理会话数据的垃圾回收。自 PHP 4.2.3 起用 mtime（修改时间）来代替了 atime。因此对于不能跟踪 atime 的文件系统也没问题了。

 > GC 的运行时机并不是精准的，带有一定的或然性，所以这个设置项并不能确保旧的会话数据被删除。某些会话存储处理模块不使用此设置项。
 
 假如 gc_probability/gc_divisor 设置得比较大，或者网站的请求量比较大，那么 GC 进程启动就会比较频繁。
还有，GC 进程启动后都需要遍历 Session 文件列表，对比文件的修改时间和服务端的当前时间，判断文件是否过期而决定是否删除文件。
这也是不应该使用 PHP 自带的 files 型 Session 处理器的原因。而 Redis 或 Memcached 天生就支持 key/value 过期机制的，用于作为会话处理器很合适。或者自己实现一个基于文件的处理器，当根据 session_id 获取对应的单个 Session 文件时判断文件是否过期。

6. **注意事项**
 * 为什么重启浏览器后 Session 数据就取不到了？  

 > session.cookie_lifetime 以秒数指定了发送到浏览器的 cookie 的生命周期。值为 0 表示"直到关闭浏览器"。默认为 0。
 
 其实，并不是 Session 数据被删除（也有可能是，概率比较小）。只是关闭浏览器时，保存 session_id 的 Cookie 没有了;同理，浏览器 Cookie 被手动清除或者其他软件清除也会造成这个结果。
 
 * 为什么浏览器开着，我很久没有操作就被登出了?  

  这个是称为“防呆”，为了保护用户账户安全的。  
  是因为这个功能的实现可能和 Session 的删除机制有关（之所以说是可能，是因为这个功能不一定要借住 Session 实现，用 Cookie 也同样可以实现）。说简单一点，就是长时间没有操作，服务端的 Session 文件过期被删除了。

 






 



