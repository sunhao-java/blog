title: spring3.0.5的工具类
tags: [Java, spring]
date: 2016-03-18
categories: 后端
---

> spring给我们提供了很多的工具类, 应该在我们的日常工作中很好的利用起来. 它可以大大的减轻我们的平时编写代码的长度. 因我们只想用spring的工具类,而不想把一个大大的spring工程给引入进来. 下面是我从spring3.0.5里抽取出来的工具类.

<!-- more -->

# 内置的resouce类型

    UrlResource
    ClassPathResource
    FileSystemResource
    ServletContextResource
    InputStreamResource
    ByteArrayResource
    EncodedResource 也就是Resource加上encoding, 可以认为是有编码的资源
    VfsResource (在jboss里经常用到, 相应还有 工具类 VfsUtils)
    org.springframework.util.xml.ResourceUtils  用于处理表达资源字符串前缀描述资源的工具. 如: classpath:.
    有 getURL, getFile, isFileURL, isJarURL, extractJarFileURL



# 工具类

	org.springframework.core.annotation.AnnotationUtils 处理注解
	org.springframework.core.io.support.PathMatchingResourcePatternResolver 用于处理 ant 匹配风格(com/*.jsp, com/**/*.jsp),找出所有的资源, 结合上面的resource的概念一起使用,对于遍历文件很有用. 具体请详细查看javadoc
	org.springframework.core.io.support.PropertiesLoaderUtils 加载Properties资源工具类,和Resource结合
	org.springframework.core.BridgeMethodResolver 桥接方法分析器.关于桥接方法请参考: http://java.sun.com/docs/books/jls/third_edition/html/expressions.html#15.12.4.5
	org.springframework.core.GenericTypeResolver 范型分析器, 在用于对范型方法, 参数分析.
	org.springframework.core.NestedExceptionUtils



# xml工具

    org.springframework.util.xml.AbstractStaxContentHandler
    org.springframework.util.xml.AbstractStaxXMLReader
    org.springframework.util.xml.AbstractXMLReader
    org.springframework.util.xml.AbstractXMLStreamReader
    org.springframework.util.xml.DomUtils
    org.springframework.util.xml.SimpleNamespaceContext
    org.springframework.util.xml.SimpleSaxErrorHandler
    org.springframework.util.xml.SimpleTransformErrorListener
    org.springframework.util.xml.StaxUtils
    org.springframework.util.xml.TransformerUtils

# 其它工具集

    org.springframework.util.xml.AntPathMatcher ant风格的处理
    org.springframework.util.xml.AntPathStringMatcher
    org.springframework.util.xml.Assert 断言,在我们的参数判断时应该经常用
    org.springframework.util.xml.CachingMapDecorator
    org.springframework.util.xml.ClassUtils 用于Class的处理
    org.springframework.util.xml.CollectionUtils 用于处理集合的工具
    org.springframework.util.xml.CommonsLogWriter
    org.springframework.util.xml.CompositeIterator
    org.springframework.util.xml.ConcurrencyThrottleSupport
    org.springframework.util.xml.CustomizableThreadCreator
    org.springframework.util.xml.DefaultPropertiesPersister
    org.springframework.util.xml.DigestUtils 摘要处理, 这里有用于md5处理信息的
    org.springframework.util.xml.FileCopyUtils 文件的拷贝处理, 结合Resource的概念一起来处理, 真的是很方便
    org.springframework.util.xml.FileSystemUtils
    org.springframework.util.xml.LinkedCaseInsensitiveMap key值不区分大小写的LinkedMap
    org.springframework.util.xml.LinkedMultiValueMap 一个key可以存放多个值的LinkedMap
    org.springframework.util.xml.Log4jConfigurer 一个log4j的启动加载指定配制文件的工具类
    org.springframework.util.xml.NumberUtils 处理数字的工具类, 有parseNumber 可以把字符串处理成我们指定的数字格式, 还支持format格式, convertNumberToTargetClass 可以实现Number类型的转化.
    org.springframework.util.xml.ObjectUtils 有很多处理null object的方法. 如nullSafeHashCode, nullSafeEquals, isArray, containsElement, addObjectToArray, 等有用的方法
    org.springframework.util.xml.PatternMatchUtils spring里用于处理简单的匹配. 如 Spring's typical xxx*, *xxx and *xxx* pattern styles
    org.springframework.util.xml.PropertyPlaceholderHelper 用于处理占位符的替换
    org.springframework.util.xml.ReflectionUtils 反映常用工具方法. 有 findField, setField, getField, findMethod, invokeMethod等有用的方法
    org.springframework.util.xml.SerializationUtils 用于java的序列化与反序列化. serialize与deserialize方法
    org.springframework.util.xml.StopWatch 一个很好的用于记录执行时间的工具类, 且可以用于任务分阶段的测试时间. 最后支持一个很好看的打印格式. 这个类应该经常用
    org.springframework.util.xml.StringUtils
    org.springframework.util.xml.SystemPropertyUtils
    org.springframework.util.xml.TypeUtils 用于类型相容的判断. isAssignable
    org.springframework.util.xml.WeakReferenceMonitor 弱引用的监控

# 和web相关的工具

    org.springframework.web.util.CookieGenerator
    org.springframework.web.util.HtmlCharacterEntityDecoder
    org.springframework.web.util.HtmlCharacterEntityReferences
    org.springframework.web.util.HtmlUtils
    org.springframework.web.util.HttpUrlTemplate 这个类用于用字符串模板构建url, 它会自动处理url里的汉字及其它相关的编码. 在读取别人提供的url资源时, 应该经常用String url = "http://localhost/myapp/{name}/{id}"
    org.springframework.web.util.JavaScriptUtils
    org.springframework.web.util.Log4jConfigListener 用listener的方式来配制log4j在web环境下的初始化
    org.springframework.web.util.UriTemplate
    org.springframework.web.util.UriUtils 处理uri里特殊字符的编码
    org.springframework.web.util.WebUtils