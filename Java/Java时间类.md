[toc]

# 1. 引言
经常用Java相关的事件类比如Date,Timestamp，做一总结以便记录和查看
# 2. Date类
## 2.1 初始化类
java.util 包提供了 Date 类来封装当前的日期和时间。 Date 类提供两个构造函数来实例化 Date 对象。
第一个构造函数使用当前日期和时间来初始化对象。
```
Date( )
```
第二个构造函数接收一个参数，该参数是从1970年1月1日起的毫秒数。
```
Date(long millisec)
```
## 2.2 常用方法
```
//若当调用此方法的Date对象在指定日期之后返回true,否则返回false
boolean after(Date date)

//若当调用此方法的Date对象在指定日期之前返回true,否则返回false
boolean before(Date date)

//比较当调用此方法的Date对象和指定日期。两者相等时候返回0。调用对象在指定日期之前则返回负数。调用对象在指定日期之后则返回正数。
int compareTo(Date date)

//当调用此方法的Date对象和指定日期相等时候返回true,否则返回false
boolean equals(Object date)

//返回自 1970 年 1 月 1 日 00:00:00 GMT 以来此 Date 对象表示的毫秒数。
long getTime()

//返回此对象的哈希码值
int hashCode()

//用自1970年1月1日00:00:00 GMT以后time毫秒数设置时间和日期
void setTime(long time)

//把此Date对象转换为以下形式的String： dow mon dd hh:mm:ss zzz yyyy 其中： dow 是一周中的某一天 (Sun, Mon, Tue, Wed, Thu, Fri, Sat)
String toString()
```
# 3. 格式化日期
## 3.1 使用SimpleDateFormat格式化日期
SimpleDateFormat是一个以语言环境敏感的方式来格式化和分析日期的类。SimpleDateFormat允许你选择任何用户自定义日期时间格式来运行。
```
public class DateDemo {
   public static void main(String args[]) {
      Date dNow = new Date( );
      SimpleDateFormat ft = new SimpleDateFormat ("yyyy-MM-dd hh:mm:ss");
 
      System.out.println("当前时间为: " + ft.format(dNow));
   }
}
```
### 3.1.1 SimpleDateFormat日期和时间的格式化编码
时间模式字符串用来指定时间格式。在此模式中，所有的 ASCII 字母被保留为模式字母，定义如下：
<table class="reference">
	<tbody>
		<tr>
			<th style="width:66px;">
				<strong>字母</strong></th>
			<th style="width:151px;">
				<strong>描述</strong></th>
			<th style="width:128px;">
				<strong>示例</strong></th>
		</tr>
		<tr>
			<td style="width:66px;">
				G</td>
			<td style="width:151px;">
				纪元标记</td>
			<td style="width:128px;">
				AD</td>
		</tr>
		<tr>
			<td style="width:66px;">
				y</td>
			<td style="width:151px;">
				四位年份</td>
			<td style="width:128px;">
				2001</td>
		</tr>
		<tr>
			<td style="width:66px;">
				M</td>
			<td style="width:151px;">
				月份</td>
			<td style="width:128px;">
				July or 07</td>
		</tr>
		<tr>
			<td style="width:66px;">
				d</td>
			<td style="width:151px;">
				一个月的日期</td>
			<td style="width:128px;">
				10</td>
		</tr>
		<tr>
			<td style="width:66px;">
				h</td>
			<td style="width:151px;">
				&nbsp;A.M./P.M. (1~12)格式小时</td>
			<td style="width:128px;">
				12</td>
		</tr>
		<tr>
			<td style="width:66px;">
				H</td>
			<td style="width:151px;">
				一天中的小时 (0~23)</td>
			<td style="width:128px;">
				22</td>
		</tr>
		<tr>
			<td style="width:66px;">
				m</td>
			<td style="width:151px;">
				分钟数</td>
			<td style="width:128px;">
				30</td>
		</tr>
		<tr>
			<td style="width:66px;">
				s</td>
			<td style="width:151px;">
				秒数</td>
			<td style="width:128px;">
				55</td>
		</tr>
		<tr>
			<td style="width:66px;">
				S</td>
			<td style="width:151px;">
				毫秒数</td>
			<td style="width:128px;">
				234</td>
		</tr>
		<tr>
			<td style="width:66px;">
				E</td>
			<td style="width:151px;">
				星期几</td>
			<td style="width:128px;">
				Tuesday</td>
		</tr>
		<tr>
			<td style="width:66px;">
				D</td>
			<td style="width:151px;">
				一年中的日子</td>
			<td style="width:128px;">
				360</td>
		</tr>
		<tr>
			<td style="width:66px;">
				F</td>
			<td style="width:151px;">
				一个月中第几周的周几</td>
			<td style="width:128px;">
				2 (second Wed. in July)</td>
		</tr>
		<tr>
			<td style="width:66px;">
				w</td>
			<td style="width:151px;">
				一年中第几周</td>
			<td style="width:128px;">
				40</td>
		</tr>
		<tr>
			<td style="width:66px;">
				W</td>
			<td style="width:151px;">
				一个月中第几周</td>
			<td style="width:128px;">
				1</td>
		</tr>
		<tr>
			<td style="width:66px;">
				a</td>
			<td style="width:151px;">
				A.M./P.M. 标记</td>
			<td style="width:128px;">
				PM</td>
		</tr>
		<tr>
			<td style="width:66px;">
				k</td>
			<td style="width:151px;">
				一天中的小时(1~24)</td>
			<td style="width:128px;">
				24</td>
		</tr>
		<tr>
			<td style="width:66px;">
				K</td>
			<td style="width:151px;">
				&nbsp;A.M./P.M. (0~11)格式小时</td>
			<td style="width:128px;">
				10</td>
		</tr>
		<tr>
			<td style="width:66px;">
				z</td>
			<td style="width:151px;">
				时区</td>
			<td style="width:128px;">
				Eastern Standard Time</td>
		</tr>
		<tr>
			<td style="width:66px;">
				'</td>
			<td style="width:151px;">
				文字定界符</td>
			<td style="width:128px;">
				Delimiter</td>
		</tr>
		<tr>
			<td style="width:66px;">
				"</td>
			<td style="width:151px;">
				单引号</td>
			<td style="width:128px;">
				`</td>
		</tr>
	</tbody>
</table>

### 3.1.2 SimpleDataFormat解析字符串为时间
SimpleDateFormat类有一些附加的方法，特别是parse()，它试图按照给定的SimpleDateFormat对象的格式化存储来解析字符串。
```
public class DateDemo {
 
   public static void main(String args[]) {
      SimpleDateFormat ft = new SimpleDateFormat ("yyyy-MM-dd"); 
 
      String input = args.length == 0 ? "1818-11-11" : args[0]; 
 
      System.out.print(input + " Parses as "); 
 
      Date t; 
 
      try { 
          t = ft.parse(input); 
          System.out.println(t); 
      } catch (ParseException e) { 
          System.out.println("Unparseable using " + ft); 
      }
   }
}
```

## 3.2 使用printf格式化日期
printf 方法可以很轻松地格式化时间和日期。它以 %t开头并且以下面表格中的一个字母结尾。
<table class="reference">
<tbody>
<tr>
<th>
<p>转&nbsp; 换&nbsp; 符</p>
</th>
<th>
<p>说&nbsp;&nbsp;&nbsp; 明</p>
</th>
<th>
<p>示&nbsp;&nbsp;&nbsp; 例</p>
</th>
</tr>
<tr>
<td>
<p>c</p>
</td>
<td>
<p>包括全部日期和时间信息</p>
</td>
<td>
<p>星期六 十月 27 14:21:20 CST 2007</p>
</td>
</tr>
<tr>
<td>
<p>F</p>
</td>
<td>
<p>"年-月-日"格式</p>
</td>
<td>
<p>2007-10-27</p>
</td>
</tr>
<tr>
<td>
<p>D</p>
</td>
<td>
<p>"月/日/年"格式</p>
</td>
<td>
<p>10/27/07</p>
</td>
</tr>
<tr>
<td>
<p>r</p>
</td>
<td>
<p>"HH:MM:SS PM"格式（12时制）</p>
</td>
<td>
<p>02:25:51 下午</p>
</td>
</tr>
<tr>
<td>
<p>T</p>
</td>
<td>
<p>"HH:MM:SS"格式（24时制）</p>
</td>
<td>
<p>14:28:16</p>
</td>
</tr>
<tr>
<td>
<p>R</p>
</td>
<td>
<p>"HH:MM"格式（24时制）</p>
</td>
<td>
<p>14:28</p>
</td>
</tr>
</tbody>
</table>

## 3.3 测量时间
下面的一个例子表明如何测量时间间隔（以毫秒为单位）：
```
public class DiffDemo {
   public static void main(String args[]) {
      try {
         long start = System.currentTimeMillis( );
         System.out.println(new Date( ) + "\n");
         Thread.sleep(5*60*10);
         System.out.println(new Date( ) + "\n");
         long end = System.currentTimeMillis( );
         long diff = end - start;
         System.out.println("Difference is : " + diff);
      } catch (Exception e) {
         System.out.println("Got an exception!");
      }
   }
}

 /**
    * 运行结果如下:
    * Thu May 09 13:56:40 CST 2019
    *
    * Thu May 09 13:56:43 CST 2019
    *
    * Difference is : 3021
    */
```
# 4. Calendar日历类
Calendar类的功能要比Date类强大很多，Calendar类是一个抽象类，创建对象的过程对程序员来说是透明的，只需要使用getInstance方法创建即可。
```
//默认是当前日期
Calendar c = Calendar.getInstance();
```
## 4.1 Calendar字段类型
<table class="reference">
	<tbody>
<tr><th>常量</th><th>描述</th></tr>		
<tr><td>Calendar.YEAR</td><td>年份</td></tr>
<tr><td>Calendar.MONTH</td><td>月份</td></tr>
<tr><td>Calendar.DATE</td><td>日期</td></tr>
<tr><td>Calendar.DAY_OF_MONTH</td><td>日期，和上面的字段意义完全相同</td></tr>
<tr><td>Calendar.HOUR</td><td>12小时制的小时</td></tr>
<tr><td>Calendar.HOUR_OF_DAY</td><td>24小时制的小时</td></tr>
<tr><td>Calendar.MINUTE</td><td>分钟</td></tr>
<tr><td>Calendar.SECOND</td><td>秒</td></tr>
<tr><td>Calendar.DAY_OF_WEEK</td><td>星期几</td>
 </tr>
	</tbody>
</table>

## 4.2 Calendar的简单操作
```
//获得Calendar实例
Calendar c1 = Calendar.getInstance();

//设置Calendar实例的年，月，日
public final void set(int year,int month,int date)
//把Calendar对象c1的年月日分别设这为：2009、6、12
c1.set(2009, 6 - 1, 12);

//设定某个字段，例如日期的值
public void set(int field,int value)
//把c1对象代表的日期设置为10号，其它所有的数值会被重新计算，其他字段设置以此类推
c1.set(Calendar.DATE,10);

//把c1对象的日期加上10，也就是c1也就表示为10天后的日期，其它所有的数值会被重新计算,其他字段设置以此类推
c1.add(Calendar.DATE, 10);
```
## 4.3 GregorianCalendar类
Calendar类实现了公历日历，GregorianCalendar是Calendar类的一个具体实现,Calendar的getInstance（）方法返回一个默认用当前的语言环境和时区初始化的GregorianCalendar对象。GregorianCalendar定义了两个字段：AD和BC。这是代表公历定义的两个时代。
### 4.3.1 GregorianCalendar构造方法
<table class="reference">
	<tbody>
		<tr>
			<td style="width:47px;">
				<strong>序号</strong></td>
			<td style="width:529px;">
				<strong>构造函数和说明</strong></td>
		</tr>
		<tr>
			<td style="width:47px;">
				1</td>
			<td style="width:529px;">
				<strong>GregorianCalendar() </strong><br>
				在具有默认语言环境的默认时区内使用当前时间构造一个默认的 GregorianCalendar。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				2</td>
			<td style="width:529px;">
				<strong>GregorianCalendar(int year, int month, int date) </strong><br>
				在具有默认语言环境的默认时区内构造一个带有给定日期设置的 GregorianCalendar</td>
		</tr>
		<tr>
			<td style="width:47px;">
				3</td>
			<td style="width:529px;">
				<strong>GregorianCalendar(int year, int month, int date, int hour, int minute) </strong><br>
				为具有默认语言环境的默认时区构造一个具有给定日期和时间设置的 GregorianCalendar。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				4</td>
			<td style="width:529px;">
				<strong>GregorianCalendar(int year, int month, int date, int hour, int minute, int second) </strong><br>
				&nbsp; 为具有默认语言环境的默认时区构造一个具有给定日期和时间设置的 GregorianCalendar。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				5</td>
			<td style="width:529px;">
				<strong>GregorianCalendar(Locale aLocale) </strong><br>
				在具有给定语言环境的默认时区内构造一个基于当前时间的 GregorianCalendar。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				6</td>
			<td style="width:529px;">
				<strong>GregorianCalendar(TimeZone zone) </strong><br>
				在具有默认语言环境的给定时区内构造一个基于当前时间的 GregorianCalendar。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				7</td>
			<td style="width:529px;">
				<strong>GregorianCalendar(TimeZone zone, Locale aLocale) </strong><br>
				&nbsp;在具有给定语言环境的给定时区内构造一个基于当前时间的 GregorianCalendar。</td>
		</tr>
	</tbody>
</table>

### 4.3.2 GregorianCalendar常用方法
<table class="reference">
	<tbody>
		<tr>
			<td style="width:47px;">
				<strong>序号</strong></td>
			<td style="width:529px;">
				<strong>方法和说明</strong></td>
		</tr>
		<tr>
			<td style="width:47px;">
				1</td>
			<td style="width:529px;">
				<strong>void add(int field, int amount) </strong><br>
				根据日历规则，将指定的（有符号的）时间量添加到给定的日历字段中。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				2</td>
			<td style="width:529px;">
				<strong>protected void computeFields() </strong><br>
				转换UTC毫秒值为时间域值</td>
		</tr>
		<tr>
			<td style="width:47px;">
				3</td>
			<td style="width:529px;">
				<strong>protected void computeTime() </strong><br>
				覆盖Calendar ，转换时间域值为UTC毫秒值</td>
		</tr>
		<tr>
			<td style="width:47px;">
				4</td>
			<td style="width:529px;">
				<strong>boolean equals(Object obj) </strong><br>
				比较此 GregorianCalendar 与指定的 Object。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				5</td>
			<td style="width:529px;">
				<strong>int get(int field) </strong><br>
				获取指定字段的时间值</td>
		</tr>
		<tr>
			<td style="width:47px;">
				6</td>
			<td style="width:529px;">
				<strong>int getActualMaximum(int field) </strong><br>
				返回当前日期，给定字段的最大值</td>
		</tr>
		<tr>
			<td style="width:47px;">
				7</td>
			<td style="width:529px;">
				<strong>int getActualMinimum(int field) </strong><br>
				返回当前日期，给定字段的最小值</td>
		</tr>
		<tr>
			<td style="width:47px;">
				8</td>
			<td style="width:529px;">
				<strong>int getGreatestMinimum(int field) </strong><br>
				&nbsp;返回此 GregorianCalendar 实例给定日历字段的最高的最小值。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				9</td>
			<td style="width:529px;">
				<strong>Date getGregorianChange() </strong><br>
				获得格里高利历的更改日期。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				10</td>
			<td style="width:529px;">
				<strong>int getLeastMaximum(int field) </strong><br>
				返回此 GregorianCalendar 实例给定日历字段的最低的最大值</td>
		</tr>
		<tr>
			<td style="width:47px;">
				11</td>
			<td style="width:529px;">
				<strong>int getMaximum(int field) </strong><br>
				返回此 GregorianCalendar 实例的给定日历字段的最大值。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				12</td>
			<td style="width:529px;">
				<strong>Date getTime()</strong><br>
				获取日历当前时间。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				13</td>
			<td style="width:529px;">
				<strong>long getTimeInMillis() </strong><br>
				获取用长整型表示的日历的当前时间</td>
		</tr>
		<tr>
			<td style="width:47px;">
				14</td>
			<td style="width:529px;">
				<strong>TimeZone getTimeZone() </strong><br>
				获取时区。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				15</td>
			<td style="width:529px;">
				<strong>int getMinimum(int field) </strong><br>
				返回给定字段的最小值。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				16</td>
			<td style="width:529px;">
				<strong>int hashCode() </strong><br>
				重写hashCode.</td>
		</tr>
		<tr>
			<td style="width:47px;">
				17</td>
			<td style="width:529px;">
				<strong>boolean isLeapYear(int year)</strong><br>
				确定给定的年份是否为闰年。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				18</td>
			<td style="width:529px;">
				<strong>void roll(int field, boolean up) </strong><br>
				在给定的时间字段上添加或减去（上/下）单个时间单元，不更改更大的字段。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				19</td>
			<td style="width:529px;">
				<strong>void set(int field, int value) </strong><br>
				用给定的值设置时间字段。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				20</td>
			<td style="width:529px;">
				<strong>void set(int year, int month, int date) </strong><br>
				设置年、月、日的值。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				21</td>
			<td style="width:529px;">
				<strong>void set(int year, int month, int date, int hour, int minute) </strong><br>
				设置年、月、日、小时、分钟的值。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				22</td>
			<td style="width:529px;">
				<strong>void set(int year, int month, int date, int hour, int minute, int second) </strong><br>
				设置年、月、日、小时、分钟、秒的值。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				23</td>
			<td style="width:529px;">
				<strong>void setGregorianChange(Date date) </strong><br>
				设置 GregorianCalendar 的更改日期。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				24</td>
			<td style="width:529px;">
				<strong>void setTime(Date date) </strong><br>
				用给定的日期设置Calendar的当前时间。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				25</td>
			<td style="width:529px;">
				<strong>void setTimeInMillis(long millis) </strong><br>
				用给定的long型毫秒数设置Calendar的当前时间。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				26</td>
			<td style="width:529px;">
				<strong>void setTimeZone(TimeZone value) </strong><br>
				用给定时区值设置当前时区。</td>
		</tr>
		<tr>
			<td style="width:47px;">
				27</td>
			<td style="width:529px;">
				<strong>String toString() </strong><br>
				返回代表日历的字符串。</td>
		</tr>
	</tbody>
</table>


参考：https://www.runoob.com/java/java-date-time.html
