---
title: Java 8 其他重要新特性概览
date: 2025-10-15 11:36:33
category: 后端
tags: 新特性
---

Java 8 除了 Lambda、Stream、Optional 等核心特性外，还引入了许多其他重要的改进和新功能。这些特性虽然不像前者那样耀眼，但在实际开发中同样发挥着重要作用。

1. 内置 Base64 编码解码
   在 Java 8 之前，Base64 编码解码需要使用第三方库或手动实现。Java 8 将这一常用功能集成到了标准库中。

java
import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class Base64Demo {
public static void main(String[] args) {
// 原始数据
String originalText = "Java 8 Base64 编码解码演示";
System.out.println("原始文本: " + originalText);

        // 1. 基本编码解码
        System.out.println("\n=== 基本Base64 ===");
        String basicEncoded = Base64.getEncoder()
                .encodeToString(originalText.getBytes(StandardCharsets.UTF_8));
        System.out.println("编码后: " + basicEncoded);
        
        String basicDecoded = new String(
                Base64.getDecoder().decode(basicEncoded),
                StandardCharsets.UTF_8
        );
        System.out.println("解码后: " + basicDecoded);
        
        // 2. URL安全的编码解码（不包含 + / 字符）
        System.out.println("\n=== URL安全Base64 ===");
        String url = "https://example.com/search?q=java+8+features";
        String urlEncoded = Base64.getUrlEncoder()
                .encodeToString(url.getBytes(StandardCharsets.UTF_8));
        System.out.println("URL编码后: " + urlEncoded);
        
        // 3. MIME格式编码（每76字符换行）
        System.out.println("\n=== MIME格式Base64 ===");
        String longText = "这是一个很长的文本".repeat(10);
        String mimeEncoded = Base64.getMimeEncoder()
                .encodeToString(longText.getBytes(StandardCharsets.UTF_8));
        System.out.println("MIME编码（前100字符）:\n" + 
                mimeEncoded.substring(0, Math.min(100, mimeEncoded.length())));
        
        // 4. 流式处理大文件
        System.out.println("\n=== 流式Base64处理 ===");
        byte[] largeData = generateLargeData();
        
        // 编码
        byte[] encodedBytes = Base64.getEncoder().encode(largeData);
        System.out.println("原始大小: " + largeData.length + " bytes");
        System.out.println("编码后大小: " + encodedBytes.length + " bytes");
        System.out.println("大小增加: " + 
                String.format("%.1f%%", (encodedBytes.length * 100.0 / largeData.length) - 100));
        
        // 5. 包装流处理
        processWithWrappedStreams();
    }
    
    static byte[] generateLargeData() {
        // 生成1MB的测试数据
        byte[] data = new byte[1024 * 1024];
        for (int i = 0; i < data.length; i++) {
            data[i] = (byte) (i % 256);
        }
        return data;
    }
    
    static void processWithWrappedStreams() {
        System.out.println("\n=== 包装流处理 ===");
        
        String text = "流式Base64编码解码";
        
        try {
            // 编码包装
            java.io.ByteArrayOutputStream baos = new java.io.ByteArrayOutputStream();
            java.io.OutputStream base64Encoder = Base64.getEncoder().wrap(baos);
            base64Encoder.write(text.getBytes(StandardCharsets.UTF_8));
            base64Encoder.close();
            
            String encoded = baos.toString(StandardCharsets.UTF_8.name());
            System.out.println("包装流编码: " + encoded);
            
            // 解码包装
            java.io.ByteArrayInputStream bais = 
                    new java.io.ByteArrayInputStream(encoded.getBytes(StandardCharsets.UTF_8));
            java.io.InputStream base64Decoder = Base64.getDecoder().wrap(bais);
            
            byte[] buffer = new byte[1024];
            int bytesRead = base64Decoder.read(buffer);
            String decoded = new String(buffer, 0, bytesRead, StandardCharsets.UTF_8);
            System.out.println("包装流解码: " + decoded);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}

2. 新的日期时间 API (JSR 310)
   Java 8 引入了全新的日期时间 API，解决了旧的 java.util.Date 和 java.util.Calendar 类的诸多问题。

java
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjusters;
import java.util.Locale;

public class DateTimeAPIDemo {
public static void main(String[] args) {
System.out.println("=== Java 8 新日期时间API ===\n");

        // 1. 时钟（替代 System.currentTimeMillis()）
        System.out.println("1. 时钟系统:");
        Clock utcClock = Clock.systemUTC();
        Clock systemClock = Clock.systemDefaultZone();
        
        System.out.println("UTC时间: " + utcClock.instant());
        System.out.println("系统时间: " + systemClock.instant());
        System.out.println("UTC毫秒: " + utcClock.millis());
        System.out.println("系统毫秒: " + systemClock.millis());
        
        // 2. 本地日期
        System.out.println("\n2. 本地日期:");
        LocalDate today = LocalDate.now();
        LocalDate birthDate = LocalDate.of(1990, Month.JUNE, 15);
        LocalDate nextWeek = today.plusWeeks(1);
        
        System.out.println("今天: " + today);
        System.out.println("生日: " + birthDate);
        System.out.println("下周: " + nextWeek);
        System.out.println("是否闰年: " + today.isLeapYear());
        System.out.println("星期几: " + today.getDayOfWeek());
        System.out.println("月份: " + today.getMonth());
        System.out.println("年龄: " + birthDate.until(today).getYears() + " 岁");
        
        // 3. 本地时间
        System.out.println("\n3. 本地时间:");
        LocalTime now = LocalTime.now();
        LocalTime meetingTime = LocalTime.of(14, 30);
        LocalTime endTime = meetingTime.plusHours(2).plusMinutes(15);
        
        System.out.println("现在时间: " + now);
        System.out.println("会议时间: " + meetingTime);
        System.out.println("结束时间: " + endTime);
        System.out.println("是否在会议前: " + now.isBefore(meetingTime));
        
        // 4. 本地日期时间
        System.out.println("\n4. 本地日期时间:");
        LocalDateTime currentDateTime = LocalDateTime.now();
        LocalDateTime eventDateTime = LocalDateTime.of(2024, 12, 25, 20, 0);
        
        System.out.println("当前日期时间: " + currentDateTime);
        System.out.println("事件日期时间: " + eventDateTime);
        System.out.println("距离事件还有: " + 
                ChronoUnit.DAYS.between(currentDateTime.toLocalDate(), eventDateTime.toLocalDate()) + " 天");
        
        // 5. 时区处理
        System.out.println("\n5. 时区处理:");
        ZonedDateTime beijingTime = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
        ZonedDateTime newYorkTime = beijingTime.withZoneSameInstant(ZoneId.of("America/New_York"));
        ZonedDateTime londonTime = beijingTime.withZoneSameInstant(ZoneId.of("Europe/London"));
        
        System.out.println("北京时间: " + beijingTime);
        System.out.println("纽约时间: " + newYorkTime);
        System.out.println("伦敦时间: " + londonTime);
        
        // 6. 时间间隔和持续时间
        System.out.println("\n6. 时间间隔和持续时间:");
        LocalDate startDate = LocalDate.of(2024, 1, 1);
        LocalDate endDate = LocalDate.of(2024, 12, 31);
        
        Period period = Period.between(startDate, endDate);
        System.out.println("期间: " + period.getYears() + "年 " + 
                period.getMonths() + "月 " + period.getDays() + "日");
        
        LocalTime startTime = LocalTime.of(9, 0);
        LocalTime finishTime = LocalTime.of(17, 30);
        
        Duration duration = Duration.between(startTime, finishTime);
        System.out.println("工作时长: " + duration.toHours() + " 小时 " + 
                duration.toMinutesPart() + " 分钟");
        
        // 7. 格式化与解析
        System.out.println("\n7. 格式化与解析:");
        DateTimeFormatter formatter = DateTimeFormatter
                .ofPattern("yyyy年MM月dd日 HH:mm:ss")
                .withLocale(Locale.CHINA);
        
        String formatted = currentDateTime.format(formatter);
        System.out.println("格式化: " + formatted);
        
        LocalDateTime parsed = LocalDateTime.parse("2024年05月20日 14:30:00", formatter);
        System.out.println("解析后: " + parsed);
        
        // 8. 时间调整器
        System.out.println("\n8. 时间调整器:");
        LocalDate firstDayOfMonth = today.with(TemporalAdjusters.firstDayOfMonth());
        LocalDate lastDayOfMonth = today.with(TemporalAdjusters.lastDayOfMonth());
        LocalDate nextMonday = today.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
        LocalDate lastInMonth = today.with(TemporalAdjusters.lastInMonth(DayOfWeek.FRIDAY));
        
        System.out.println("本月第一天: " + firstDayOfMonth);
        System.out.println("本月最后一天: " + lastDayOfMonth);
        System.out.println("下个周一: " + nextMonday);
        System.out.println("本月最后一个周五: " + lastInMonth);
        
        // 9. 时间运算
        System.out.println("\n9. 时间运算:");
        System.out.println("10天后的日期: " + today.plusDays(10));
        System.out.println("3个月前: " + today.minusMonths(3));
        System.out.println("2年后的同一天: " + today.plusYears(2));
        
        // 10. 比较和判断
        System.out.println("\n10. 比较和判断:");
        LocalDate holiday = LocalDate.of(2024, 10, 1);
        System.out.println("今天是否在假期前: " + today.isBefore(holiday));
        System.out.println("今天是否在假期后: " + today.isAfter(holiday));
        System.out.println("今天是否是假期: " + today.equals(holiday));
    }

}

3. Nashorn JavaScript 引擎
   Java 8 引入了 Nashorn JavaScript 引擎，替代了旧的 Rhino 引擎，提供了更好的性能和与 Java 的互操作性。

java
import javax.script.*;

public class NashornDemo {
public static void main(String[] args) throws ScriptException {
System.out.println("=== Nashorn JavaScript 引擎 ===\n");

        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("nashorn");
        
        if (engine == null) {
            System.out.println("Nashorn 引擎不可用");
            return;
        }
        
        System.out.println("引擎名称: " + engine.getClass().getName());
        
        // 1. 基本JavaScript执行
        System.out.println("\n1. 基本JavaScript执行:");
        Object result = engine.eval("'Hello from JavaScript!'.toUpperCase()");
        System.out.println("结果: " + result);
        
        // 2. 变量和计算
        System.out.println("\n2. 变量和计算:");
        engine.eval("var x = 10; var y = 20;");
        Object sum = engine.eval("x + y");
        System.out.println("10 + 20 = " + sum);
        
        // 3. 函数定义和调用
        System.out.println("\n3. 函数定义和调用:");
        engine.eval("function factorial(n) { " +
                    "    if (n <= 1) return 1; " +
                    "    return n * factorial(n - 1); " +
                    "}");
        Object factorialResult = engine.eval("factorial(5)");
        System.out.println("5! = " + factorialResult);
        
        // 4. Java与JavaScript互操作
        System.out.println("\n4. Java与JavaScript互操作:");
        
        // 将Java对象暴露给JavaScript
        engine.put("javaList", java.util.Arrays.asList("Apple", "Banana", "Cherry"));
        engine.eval("print('Java列表: ' + javaList);");
        engine.eval("print('列表大小: ' + javaList.size());");
        
        // 在JavaScript中调用Java方法
        engine.eval("var ArrayList = Java.type('java.util.ArrayList');");
        engine.eval("var jsList = new ArrayList();");
        engine.eval("jsList.add('JavaScript添加的元素');");
        engine.eval("jsList.addAll(javaList);");
        engine.eval("print('混合列表: ' + jsList);");
        
        // 5. 调用JavaScript函数从Java
        System.out.println("\n5. 从Java调用JavaScript函数:");
        engine.eval("function greet(name) { " +
                    "    return 'Hello, ' + name + '!'; " +
                    "}");
        
        Invocable invocable = (Invocable) engine;
        Object greeting = invocable.invokeFunction("greet", "Java开发者");
        System.out.println(greeting);
        
        // 6. 实现Java接口
        System.out.println("\n6. JavaScript实现Java接口:");
        engine.eval("var Runnable = Java.type('java.lang.Runnable');");
        engine.eval("var r = new Runnable() { " +
                    "    run: function() { " +
                    "        print('来自JavaScript的run()方法'); " +
                    "    } " +
                    "};");
        
        Object jsRunnable = engine.get("r");
        Thread thread = new Thread((Runnable) jsRunnable);
        thread.start();
        
        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // 7. 性能测试：计算斐波那契数列
        System.out.println("\n7. 性能测试:");
        long start = System.nanoTime();
        engine.eval("function fib(n) { " +
                    "    if (n <= 1) return n; " +
                    "    return fib(n-1) + fib(n-2); " +
                    "} " +
                    "fib(30);");
        long jsTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        fibJava(30);
        long javaTime = System.nanoTime() - start;
        
        System.out.printf("JavaScript: %d ns%n", jsTime);
        System.out.printf("Java:       %d ns%n", javaTime);
        System.out.printf("性能比率: %.1f%n", (double)jsTime / javaTime);
        
        // 8. 错误处理
        System.out.println("\n8. JavaScript错误处理:");
        try {
            engine.eval("undefinedFunction();");
        } catch (ScriptException e) {
            System.out.println("捕获JavaScript错误: " + e.getMessage());
        }
        
        // 9. 脚本编译（提高性能）
        System.out.println("\n9. 脚本编译:");
        if (engine instanceof Compilable) {
            Compilable compilable = (Compilable) engine;
            String script = "function multiply(a, b) { return a * b; }";
            CompiledScript compiled = compilable.compile(script);
            
            // 多次执行编译后的脚本
            Bindings bindings = engine.createBindings();
            bindings.put("a", 7);
            bindings.put("b", 8);
            
            Object compiledResult = compiled.eval(bindings);
            System.out.println("编译执行结果: 7 * 8 = " + compiledResult);
        }
    }
    
    static int fibJava(int n) {
        if (n <= 1) return n;
        return fibJava(n-1) + fibJava(n-2);
    }

}

4. 并行数组操作
   Java 8 为数组提供了并行操作支持，可以充分利用多核处理器的优势。

java
import java.util.Arrays;
import java.util.concurrent.ThreadLocalRandom;

public class ParallelArraysDemo {
public static void main(String[] args) {
System.out.println("=== 并行数组操作 ===\n");

        // 1. 生成测试数据
        int size = 10_000_000;
        long[] array = new long[size];
        
        System.out.println("生成 " + size + " 个随机数...");
        Arrays.parallelSetAll(array, i -> ThreadLocalRandom.current().nextLong(1000));
        
        // 2. 顺序操作 vs 并行操作
        System.out.println("\n1. 数组填充性能对比:");
        
        // 顺序填充
        long start = System.currentTimeMillis();
        long[] sequential = new long[size];
        for (int i = 0; i < size; i++) {
            sequential[i] = i;
        }
        long sequentialTime = System.currentTimeMillis() - start;
        
        // 并行填充
        start = System.currentTimeMillis();
        long[] parallel = new long[size];
        Arrays.parallelSetAll(parallel, i -> i);
        long parallelTime = System.currentTimeMillis() - start;
        
        System.out.printf("顺序填充: %d ms%n", sequentialTime);
        System.out.printf("并行填充: %d ms%n", parallelTime);
        System.out.printf("加速比: %.1fx%n", (double)sequentialTime / parallelTime);
        
        // 3. 并行排序
        System.out.println("\n2. 并行排序:");
        long[] toSort = array.clone();
        
        start = System.currentTimeMillis();
        Arrays.sort(toSort);  // 传统排序
        long sortTime = System.currentTimeMillis() - start;
        
        toSort = array.clone();
        start = System.currentTimeMillis();
        Arrays.parallelSort(toSort);  // 并行排序
        long parallelSortTime = System.currentTimeMillis() - start;
        
        System.out.printf("Arrays.sort():      %d ms%n", sortTime);
        System.out.printf("Arrays.parallelSort(): %d ms%n", parallelSortTime);
        System.out.printf("加速比: %.1fx%n", (double)sortTime / parallelSortTime);
        
        // 验证排序正确性
        boolean sortedCorrectly = true;
        for (int i = 1; i < toSort.length; i++) {
            if (toSort[i] < toSort[i-1]) {
                sortedCorrectly = false;
                break;
            }
        }
        System.out.println("排序验证: " + (sortedCorrectly ? "正确" : "错误"));
        
        // 4. 并行前缀计算
        System.out.println("\n3. 并行前缀计算:");
        int[] numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        System.out.println("原始数组: " + Arrays.toString(numbers));
        
        // 并行计算前缀和
        Arrays.parallelPrefix(numbers, (left, right) -> left + right);
        System.out.println("前缀和:   " + Arrays.toString(numbers));
        
        // 重置并计算前缀积
        numbers = new int[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        Arrays.parallelPrefix(numbers, (left, right) -> left * right);
        System.out.println("前缀积:   " + Arrays.toString(numbers));
        
        // 5. 复杂并行操作
        System.out.println("\n4. 复杂并行操作:");
        double[] values = new double[100];
        Arrays.parallelSetAll(values, i -> Math.sin(i * 0.1) * Math.cos(i * 0.05));
        
        // 并行处理：先平方，然后累加
        Arrays.parallelPrefix(values, (a, b) -> a + b * b);
        
        System.out.println("前10个处理后的值:");
        for (int i = 0; i < 10; i++) {
            System.out.printf("  [%2d] = %.6f%n", i, values[i]);
        }
        
        // 6. 性能优化建议
        System.out.println("\n5. 性能优化建议:");
        int[] smallArray = new int[100];
        int[] largeArray = new int[10_000_000];
        
        // 小数组：顺序操作更快
        start = System.nanoTime();
        Arrays.sort(smallArray);
        long smallSortTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        Arrays.parallelSort(smallArray);
        long smallParallelTime = System.nanoTime() - start;
        
        // 大数组：并行操作更快
        Arrays.parallelSetAll(largeArray, i -> ThreadLocalRandom.current().nextInt());
        int[] largeCopy = largeArray.clone();
        
        start = System.nanoTime();
        Arrays.sort(largeArray);
        long largeSortTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        Arrays.parallelSort(largeCopy);
        long largeParallelTime = System.nanoTime() - start;
        
        System.out.println("小数组(100元素):");
        System.out.printf("  顺序排序: %d ns%n", smallSortTime);
        System.out.printf("  并行排序: %d ns%n", smallParallelTime);
        System.out.printf("  建议: %s%n", smallSortTime < smallParallelTime ? "使用顺序排序" : "使用并行排序");
        
        System.out.println("\n大数组(10,000,000元素):");
        System.out.printf("  顺序排序: %d ms%n", largeSortTime / 1_000_000);
        System.out.printf("  并行排序: %d ms%n", largeParallelTime / 1_000_000);
        System.out.printf("  建议: %s%n", largeSortTime < largeParallelTime ? "使用顺序排序" : "使用并行排序");
    }

}

5. 其他重要改进
   5.1 方法参数反射
   Java 8 提供了获取方法参数名称的能力（需要编译时添加 -parameters 参数）。

java
import java.lang.reflect.Method;
import java.lang.reflect.Parameter;

public class ParameterNamesDemo {
public static void main(String[] args) throws Exception {
System.out.println("=== 方法参数名称反射 ===\n");

        Method method = Calculator.class.getMethod("add", int.class, int.class);
        
        System.out.println("方法: " + method.getName());
        System.out.println("参数数量: " + method.getParameterCount());
        
        // 获取参数信息（需要编译时添加 -parameters 参数）
        Parameter[] parameters = method.getParameters();
        
        for (int i = 0; i < parameters.length; i++) {
            Parameter param = parameters[i];
            System.out.printf("  参数 %d:%n", i + 1);
            System.out.printf("    类型: %s%n", param.getType().getSimpleName());
            
            // 如果编译时添加了 -parameters，可以获取参数名
            if (param.isNamePresent()) {
                System.out.printf("    名称: %s%n", param.getName());
            } else {
                System.out.println("    名称: [未保留参数名，编译时请添加 -parameters 参数]");
            }
        }
        
        // 实际使用
        Calculator calc = new Calculator();
        int result = calc.add(10, 20);
        System.out.println("\n计算结果: 10 + 20 = " + result);
    }
    
    static class Calculator {
        // 编译时添加 -parameters 参数以保留参数名
        public int add(int firstNumber, int secondNumber) {
            return firstNumber + secondNumber;
        }
    }

}
5.2 类型注解和重复注解增强
java
import java.lang.annotation.*;
import java.util.Arrays;

public class TypeAnnotationsDemo {
public static void main(String[] args) throws Exception {
System.out.println("=== 类型注解和重复注解 ===\n");

        // 1. 类型注解
        processNullableField();
        
        // 2. 重复注解处理
        processRepeatedAnnotations();
        
        // 3. 在泛型中使用类型注解
        processGenericAnnotations();
    }
    
    static void processNullableField() throws Exception {
        System.out.println("1. 类型注解示例:");
        
        // 获取字段的类型注解
        var field = User.class.getDeclaredField("email");
        var annotations = field.getAnnotatedType().getAnnotations();
        
        System.out.println("字段: " + field.getName());
        System.out.println("类型注解数量: " + annotations.length);
        
        for (Annotation ann : annotations) {
            if (ann instanceof Nullable) {
                System.out.println("  该字段允许为null");
            }
        }
    }
    
    static void processRepeatedAnnotations() {
        System.out.println("\n2. 重复注解处理:");
        
        // 获取类上的重复注解
        var authors = Book.class.getAnnotationsByType(Author.class);
        System.out.println("书籍作者数量: " + authors.length);
        
        for (Author author : authors) {
            System.out.printf("  作者: %s (%s)%n", 
                    author.name(), author.role());
        }
    }
    
    static void processGenericAnnotations() {
        System.out.println("\n3. 泛型中的类型注解:");
        
        // 模拟处理带注解的泛型
        Repository<User> repo = new UserRepository();
        User user = repo.findById(1L);
        
        if (user != null) {
            System.out.println("找到用户: " + user.name);
        }
    }
    
    // 类型注解定义
    @Target({ElementType.TYPE_USE, ElementType.TYPE_PARAMETER})
    @Retention(RetentionPolicy.RUNTIME)
    @interface Nullable {
        String value() default "";
    }
    
    // 重复注解定义
    @Repeatable(Authors.class)
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.TYPE)
    @interface Author {
        String name();
        String role() default "作者";
    }
    
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.TYPE)
    @interface Authors {
        Author[] value();
    }
    
    // 示例类
    static class User {
        @Nullable String email;  // 类型注解
        String name;
    }
    
    @Author(name = "张三", role = "主编")
    @Author(name = "李四", role = "技术审核")
    static class Book {
        // 重复注解
    }
    
    // 泛型接口
    interface Repository<T> {
        @Nullable  // 类型注解
        T findById(Long id);
    }
    
    static class UserRepository implements Repository<User> {
        @Override
        public User findById(Long id) {
            // 模拟数据库查找
            if (id == 1L) {
                User user = new User();
                user.name = "测试用户";
                return user;
            }
            return null;  // 允许返回null
        }
    }

}

6. JVM 改进：元空间替代永久代
   Java 8 在 JVM 层面也有重要改进，最显著的是使用元空间（Metaspace）替代永久代（PermGen）。

java
public class MetaspaceDemo {
public static void main(String[] args) {
System.out.println("=== JVM 元空间 (Metaspace) ===\n");

        // 1. 查看当前JVM的元空间信息
        System.out.println("1. JVM 内存区域:");
        Runtime runtime = Runtime.getRuntime();
        
        System.out.printf("最大内存:   %,d MB%n", runtime.maxMemory() / 1024 / 1024);
        System.out.printf("总内存:     %,d MB%n", runtime.totalMemory() / 1024 / 1024);
        System.out.printf("空闲内存:   %,d MB%n", runtime.freeMemory() / 1024 / 1024);
        System.out.printf("已使用内存: %,d MB%n", 
                (runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024);
        
        // 2. 动态类加载演示
        System.out.println("\n2. 动态类加载测试:");
        
        try {
            // 创建自定义类加载器
            CustomClassLoader loader = new CustomClassLoader();
            
            // 动态生成并加载类
            for (int i = 0; i < 1000; i++) {
                String className = "DynamicClass" + i;
                String classCode = generateClassCode(className);
                Class<?> clazz = loader.defineClass(className, classCode);
                
                if (i % 100 == 0) {
                    System.out.printf("已加载 %,d 个类，空闲内存: %,d MB%n", 
                            i + 1, runtime.freeMemory() / 1024 / 1024);
                }
            }
            
        } catch (Exception e) {
            System.out.println("加载类时出错: " + e.getMessage());
            e.printStackTrace();
        }
        
        // 3. 元空间监控建议
        System.out.println("\n3. 元空间监控和调优建议:");
        System.out.println("监控命令:");
        System.out.println("  jstat -gcmetacapacity <pid>");
        System.out.println("  jcmd <pid> VM.metaspace");
        
        System.out.println("\n常用JVM参数:");
        System.out.println("  -XX:MetaspaceSize=初始大小");
        System.out.println("  -XX:MaxMetaspaceSize=最大大小");
        System.out.println("  -XX:+UseCompressedClassPointers (默认启用)");
        System.out.println("  -XX:+UseCompressedOops (默认启用)");
        
        System.out.println("\n永久代 vs 元空间:");
        System.out.println("  永久代: 位于堆内存，固定大小，容易OOM");
        System.out.println("  元空间: 使用本地内存，自动扩展，减少OOM风险");
    }
    
    static String generateClassCode(String className) {
        return "public class " + className + " {\n" +
               "    private int id;\n" +
               "    private String name;\n" +
               "    \n" +
               "    public " + className + "(int id, String name) {\n" +
               "        this.id = id;\n" +
               "        this.name = name;\n" +
               "    }\n" +
               "    \n" +
               "    public int getId() { return id; }\n" +
               "    public String getName() { return name; }\n" +
               "    \n" +
               "    @Override\n" +
               "    public String toString() {\n" +
               "        return \"" + className + "{\" + \"id=\" + id + \", name='\" + name + \"'}\";\n" +
               "    }\n" +
               "}\n";
    }
    
    // 自定义类加载器
    static class CustomClassLoader extends ClassLoader {
        Class<?> defineClass(String name, String code) throws Exception {
            // 简单模拟：实际应该编译字节码
            // 这里只是演示，不实际编译
            return Object.class; // 返回假类
        }
    }

}
总结
Java 8 的这些"其他"新特性虽然不如 Lambda 和 Stream 那样引人注目，但它们在各自领域都提供了重要的改进：

Base64 支持：标准化的编码解码，告别第三方库依赖

新日期时间 API：解决了旧 API 的设计缺陷，更安全、更易用

Nashorn JavaScript 引擎：更好的性能和 Java 互操作性

并行数组操作：充分利用多核处理器的数组处理能力

方法参数反射：增强了反射 API 的能力

JVM 元空间：解决了永久代的内存问题

这些特性共同构成了 Java 8 的完整生态系统，使得 Java 平台更加现代化、功能更全面。在实际开发中，根据具体需求选择合适的特性，可以显著提高开发效率和代码质量。

记住，虽然这些特性都很实用，但也要注意：

了解每个特性的适用场景和限制

注意性能影响，特别是在大规模数据处理时

保持代码的可读性和维护性，不要过度使用复杂特性

Java 8 是一个里程碑式的版本，全面掌握其特性对于现代 Java 开发至关重要。