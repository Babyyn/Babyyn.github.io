# 使用Calendar获取月份

```java
Calendar cd = Calendar.getInstance();
int day = cd.get(Calendar.DAY_OF_YEAR);
int month = cd.get(Calendar.MONTH);
int year =cd.get(Calendar.YEAR);

System.out.println(day);
System.out.println(month);
System.out.println(year);
// 获取英文简写月分: Sep
System.out.println(new SimpleDateFormat("MMM").format(cd.getTime()));
// 获取英文全称月分: September
System.out.println(new SimpleDateFormat("MMMM").format(cd.getTime()));
// 获取英文全称月分: September
System.out.println(cd.getDisplayName(Calendar.MONTH, Calendar.LONG, Locale.getDefault()));
// 指定国家: 中国
System.out.println(cd.getDisplayName(Calendar.MONTH, Calendar.LONG, Locale.CHINA));
```



[参考链接](// https://stackoverflow.com/questions/14832151/how-to-get-month-name-from-calendar)

