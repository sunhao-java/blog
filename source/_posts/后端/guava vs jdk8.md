title: guava vs jdk8
tags: [Java,JDK8,guava,函数式编程]
date: 2015-06-14
categories: 后端
---

#Guava
Guava是Google公司开源的一个实用工具库，对Java类库进行了多方面的增强。比如说，对函数式编程的支持，新的集合类（Multimap等），Cache支持，等等。在Java8之前，Guava和Java之间的关系，可以表示成下面这幅图：
![](http://img.blog.csdn.net/20140923165231890?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenhob28=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<!-- more -->
但是随着Java8的发布，Guava和Java的关系发生了一些改变。Guava提供的很多功能，被内置在了Java8里，如下图所示：
![](http://img.blog.csdn.net/20140923134859281?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenhob28=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 源码： [Github](https://github.com/google/guava  "Github") [Google Code](https://code.google.com/p/guava-libraries/ "Google Code" )
- API： [API](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/index.html  "API")
- Users' Guide：[Users' Guide](https://code.google.com/p/guava-libraries/wiki/GuavaExplained  "Users' Guide")

1. Joiner用来拼接n个字符串，下面是一个例子：
    - 使用guava

            public static String useGuava(List strList) {
                return Joiner.on(",").skipNulls().join(strList);
            }

    - 使用jdk8

            public static String useJdk8(List strList) {
                //lambda表达式
                return strList.stream().filter(m -> m != null).collect(Collectors.joining(","));
            }

    - 使用jdk7以及jdk7之前的版本

            public static String useJdk7(List strList) {
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < strList.size(); i++) {
                    sb.append(strList.get(i));
                    if (i + 1 != strList.size()) {
                        sb.append(",");
                    }
                }
                return sb.toString();
            }

2. Guava提供了Ordering类以方便我们创建Comparator
    - 使用guava

            public static List useGuava(List players) {
                Ordering ordering = Ordering.natural().nullsFirst().onResultOf(new Function() {
                    @Override
                    public Integer apply(Player foo) {
                        return foo.getOrder();
                    }
                });

                return ordering.sortedCopy(players);
            }

    - 使用jdk8

            public static List useJdk8(List players) {
                Comparator cmp = Comparator.comparing(Player::getOrder, Comparator.nullsFirst(Comparator.naturalOrder()));

                players.sort(cmp);
                return players;
            }

    - 使用jdk7以及jdk7之前的版本

            public static List useJdk7(List players) {
                players.sort(new Comparator() {
                    @Override
                    public int compare(Player o1, Player o2) {
                        Integer order1 = o1.getOrder();
                        Integer order2 = o2.getOrder();

                        if (order1 > order2) {
                            return 1;
                        } else if (order1 == order2) {
                            return 0;
                        } else {
                            return -1;
                        }
                    }
                });

                return players;
            }

3. 对null的处理
   - guava中对null的处理`Optional`
      - null是模棱两可的，会引起令人困惑的错误，有些时候它让人很不舒服。很多Guava工具类用快速失败拒绝null值，而不是盲目地接受。
      - `Optional`是一个`abstract`类，两个实现类`Present`、`Absent`
          - `Present`: Implementation of an Optional containing a reference.
          - `Absent`: Implementation of an Optional not containing a reference.

       eg:

	           // 创建一个非空对象，类型为integer
	           Optional possible = Optional.of(6);
	           // Returns true if this holder contains a (non-null) instance.
	           System.out.println("possible isPresent: " + possible.isPresent());
	           System.out.println("possible value: " + possible.get());

	           // 创建一个空对象
	           Optional absentOpt = Optional.absent();
	           // 取值，会报错
	           System.out.println("absentOpt value: " + absentOpt.get());

	           // If nullableReference is non-null, returns an Optional instance containing that reference; otherwise returns Optional.absent.
	           // fromNullable表示如果创建一个非空对象，则返回Present
	           Optional<Integer> NoNullableOpt  = Optional.fromNullable(10);
	           // 创建空对象，则返回Absent
	           Optional<Integer> NullableOpt = Optional.fromNullable(null);

       其他详细请查阅API - [Guava Optional API](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html "Guava Optional API")
   - Guava使用`Preconditions`对null的预先判断
      - 类似我们使用StringUtils等工具类
      - 但是不同的是，StringUtils等工具类是判断是否为空并得到结果，而Guava是使用判断的结果，如果为空，则直接抛异常。
      eg.

		          // givenName是传入的参数
		          String name = Preconditions.checkNotNull(givenName);
		          // com.google.common.base.Preconditions.checkNotNull方法重载了三次
		          public static <T> T checkNotNull(T reference)
		          public static <T> T checkNotNull(T reference, Object errorMessage)
		          public static <T> T checkNotNull(T reference, String errorMessageTemplate, Object... errorMessageArgs)

		          // 而jdk8中的java.util.Objects也提供了三个类似的方法
		          public static <T> T requireNonNull(T obj) // @since 1.7
		          public static <T> T requireNonNull(T obj, String message) // @since 1.7
		          public static <T> T requireNonNull(T obj, Supplier<String> messageSupplier) // @since 1.8

   - jdk8对null的处理
      jdk8新加入类java.util.Optional，使用方法与Guava相似。关于jdk8中的Optional可参考[迟到的Optional](http://www.ithome.com.tw/voice/91013  "迟到的Optional")。Optional 被定义为一个简单的容器，其值可能是null或者不是null。在Java 8之前一般某个函数应该返回非空对象但是偶尔却可能返回了null，而在Java 8中，不推荐你返回null而是返回Optional。
   eg.

			   // 不直接返回需要的类型String，而是返回Optional包含的String
			   Optional<String> optional = Optional.of("bam");
			   //  true if there is a value present, otherwise false
			   optional.isPresent();           // true
			   //  取值，如果为空，则抛出异常NoSuchElementException
			   optional.get();                 // "bam"
			   // 如果为空则返回传入的值，否则返回其值
			   optional.orElse("fallback");    // "bam"
			   // 结合lambda表达式
			   optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"

   其他详细请查阅API - [JDK8 Optional API](http://docs.oracle.com/javase/8/docs/api/java/util/Optional.html "JDK8 Optional API")

4. 对于集合的处理
    - 创建集合
        - 普通做法
            eg.

	            List<Player> players = new ArrayList<Player>();
	            players.add(new Player("bowen",20));
	            players.add(new Player("bob", 20));
	            players.add(new Player("Katy", 18));
	            players.add(new Player("Logon", 24));

        - 在Guava中：
            eg.

	            List<Player>  players = Lists.newArrayList(new Player("bowen", 20), new Player("bob", 20), new Player("Katy", 18), new Player("Logon", 24));

        - Java中：
            eg.

	            List<Player> players = new ArrayList<Player>() { {
	                add(new Player("bowen", 20));
	                add(new Player("bob", 20));
	                add(new Player("Katy", 18));
	                add(new Player("Logon", 24));
	            }};

    - 操作集合
        要求：按年龄将上述人员分组
        - 通常的做法（jdk8之前的版本）

		        public void mapListJava7(List<Player> players) {
		            Map<Integer, List<Player>> groups = new HashMap<>();
		            for (Player player : players) {
		                List<Player> group = groups.get(player.getAge());
		                if (group == null) {
		                    group = new ArrayList<>();
		                    groups.put(player.getAge(), group);
		                }
		                group.add(player);
		            }
		        }

        - 在guava中，提供了`Multimap `

		        public void guavaMultimap(List<Player> players) {
		            Multimap<Integer, Player> groups = ArrayListMultimap.create();
		            for (Player player : players) {
		                groups.put(player.getAge(), player);
		            }
		        }

        - 在jdk8中，给Map添加了一个新的方法`getOrDefault()`，即使不用lambda表达式，代码也清洁许多：

		        // 非lambda
		        public void mapListJava8(List<Player> players) {
		            Map<Integer, List<Player>> groups = new HashMap<>();
		            for (Player player : players) {
		                List<Player> group = groups.getOrDefault(player.getAge(), new ArrayList<>());
		                group.add(player);
		                groups.put(player.getAge(), group);
		            }
		        }

		        // 使用lambda表达式
		        public void groupingByJava8(List<Player> players) {
		            Map<Integer, List<Player>> groups = players.stream().collect(Collectors.groupingBy(Player::getAge));
		        }
