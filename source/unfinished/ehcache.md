### ehcache
1. 依赖
```
     	<dependency>
			<groupId>net.sf.ehcache</groupId>
			<artifactId>ehcache</artifactId>
			<version>2.8.2</version>
		</dependency>
```
![](https://upload-images.jianshu.io/upload_images/1049928-5be87852a2c8e88f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/425)
2. spring配置CacheManager
```
	<bean id="cacheManagerFactory"
		class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
		<property name="configLocation" value="classpath:ehcache.xml" />
	</bean>
	<bean id="commonCacheManager"
		class="org.springframework.cache.ehcache.EhCacheCacheManager">
		<property name="cacheManager" ref="cacheManagerFactory" />
	</bean>
```
3. ehcache配置(ehcache.xml)
```
<ehcache name="ehCacheDataDictCache" updateCheck="false"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd">

		<diskStore path="java.io.tmpdir/dataDict" />

		<defaultCache maxEntriesLocalHeap="100" eternal="false"
			timeToIdleSeconds="120" timeToLiveSeconds="120" overflowToDisk="true"
			maxEntriesLocalDisk="100" diskPersistent="false"
			diskExpiryThreadIntervalSeconds="120" memoryStoreEvictionPolicy="LRU" />
		<cache name="dataDictCache" maxEntriesLocalHeap="100" eternal="true"
			overflowToDisk="true" diskPersistent="false"
			diskExpiryThreadIntervalSeconds="120" memoryStoreEvictionPolicy="LRU">
			<cacheEventListenerFactory
				class="net.sf.ehcache.distribution.RMICacheReplicatorFactory" />
		</cache>

		<!-- 配置说明： 负载均衡web部署几台rmiUrls就要配置几个，多个用|隔开；格式为：//ip:port/dataDictCache-->
		<cacheManagerPeerProviderFactory
			class="net.sf.ehcache.distribution.RMICacheManagerPeerProviderFactory"
			properties="peerDiscovery=manual,rmiUrls=//192.168.54.168:11001/dataDictCache|//192.168.54.90:11002/dataDictCache" />
		<!-- 配置说明： port就是当前web的端口与rmiUrls中的port保持一直，建议用11001开始累加-->
		<cacheManagerPeerListenerFactory
			class="net.sf.ehcache.distribution.RMICacheManagerPeerListenerFactory"
			properties="port=11001" />
	</ehcache>
```
- diskStore:当内存缓存中对象数量超过maxElementsInMemory时,将缓存对象写到磁盘缓存中(需对象实现序列化接口)。
- cachemanager限制大小
```
<ehcache ...  
   maxBytesLocalHeap="500M" maxBytesLocalOffHeap="2G" maxBytesLocalDisk="50G">  
</ehcache> 
```
**maxBytesLocalHeap**是用来限制缓存所能使用的堆内存的最大字节数的，默认为0不限制
**maxBytesLocalOffHeap**是用来限制缓存所能使用的非堆内存的最大字节数，默认为0不限制
**maxBytesLocalDisk**是用来限制缓存所能使用的磁盘的最大字节数的，默认为0不限制
另外cache级别上，还可以设置
**maxEntriesLocalHeap**是用来限制当前缓存在堆内存上所能保存的最大元素数量的
**maxEntriesLocalDisk**是用来限制在磁盘上所能保存的元素的最大数量的。
- **eternal 缓存中对象是否永久有效,即是否永驻内存**,true时将忽略timeToIdleSeconds和timeToLiveSeconds。
- timeToIdleSeconds缓存数据在失效前的允许闲置时间(单位:秒),默认0无限制，即访问这个cache中元素的最大间隔时间,若超过这个时间没有访问此Cache中的某个元素,那么此元素将被从Cache中清除。
- timeToLiveSeconds缓存数据在失效前的允许存活时间(单位:秒),默认0无限制，也就是说从创建开始计时,当超过这个时间时,此元素将从Cache中清除。
- overflowToDisk内存不足时,是否启用磁盘缓存(即内存中对象数量达到maxElementsInMemory时,Ehcache会将对象写到磁盘中，path指定的目录下)
- diskPersistent是否持久化磁盘缓存,当这个属性的值为true时,系统在初始化时会在磁盘中查找文件名为cache名称,后缀名为index的文件，这个文件中存放了已经持久化在磁盘中的cache的index,找到后会把cache加载到内存，要想把cache真正持久化到磁盘,写程序时注意执行net.sf.ehcache.Cache.put(Element element)后要调用flush()方法。
- diskExpiryThreadIntervalSeconds：磁盘缓存的清理线程运行间隔,默认120秒。
- memoryStoreEvictionPolicy：内存存储与释放策略，共有三种策略,分别为LRU(最近最少使用)、LFU(最常用的)、FIFO(先进先出)。

### 2.10.x使用
//创建缓存管理器
CacheManager cacheManager = CacheManager.create("./src/main/resources/ehcache.xml");
//获取缓存对象
Cache cache = cacheManager.getCache("myCache");
//添加缓存条目
Element e1 = new Element(1L,"Hello world!");
Element e2 = new Element(2L,new User("assad","Guangzhou",20));
cache.put(e1);
cache.put(e2);
//获取缓存条目
Element e3 = cache.get(1L);
String str = (String)e3.getObjectValue();
Element e4 = cache.get(2L);
User user = (User)e4.getObjectValue();
//刷新缓存
cache.flush();
//关闭缓存管理器
cacheManager.shutdown();
[3.5使用](http://www.ehcache.org/documentation/)

#### RMI方式缓存集群/配置分布式缓存
[](https://www.cnblogs.com/hoojo/archive/2012/07/19/2599534.html)