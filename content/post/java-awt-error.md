---
title: "OpenJDK Awt Error"
date: 2020-03-25T19:58:39+09:00
categories:
- java
tags:
- java error
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---
JAVA 프로그램을 실해하려는데 `Assistive Technology not found`라는 에러를 만났다. 
<!--more-->
## 탐색
Swing의 JFrame을 사용하는 코드다.  
Stack trace는 이렇다.  
```
Exception in thread "main" java.awt.AWTError: Assistive Technology not found: org.GNOME.Accessibility.AtkWrapper
	at java.awt.Toolkit.loadAssistiveTechnologies(Toolkit.java:807)
	at java.awt.Toolkit.getDefaultToolkit(Toolkit.java:886)
	at java.awt.Window.getToolkit(Window.java:1358)
	at java.awt.Window.init(Window.java:506)
	at java.awt.Window.<init>(Window.java:537)
	at java.awt.Frame.<init>(Frame.java:420)
	at java.awt.Frame.<init>(Frame.java:385)
	at javax.swing.JFrame.<init>(JFrame.java:189)
	at CMWinClient.<init>(CMWinClient.java:77)
	at CMWinClient.main(CMWinClient.java:4525)
```

Stack trace를 뒤져본다.  
아래는 `java.awt.Toolkit`의 코드 (intellij 디컴파일)
```java
    private static void loadAssistiveTechnologies() {
        if (atNames != null) {
            ClassLoader var0 = ClassLoader.getSystemClassLoader();
            StringTokenizer var1 = new StringTokenizer(atNames, " ,");

            while(var1.hasMoreTokens()) {
                String var2 = var1.nextToken();

                try {
                    Class var3;
                    if (var0 != null) {
                        var3 = var0.loadClass(var2);
                    } else {
                        var3 = Class.forName(var2);
                    }

                    var3.newInstance();
                } catch (ClassNotFoundException var4) {
                    throw new AWTError("Assistive Technology not found: " + var2);
                } catch (InstantiationException var5) {
                    throw new AWTError("Could not instantiate Assistive Technology: " + var2);
                } catch (IllegalAccessException var6) {
                    throw new AWTError("Could not access Assistive Technology: " + var2);
                } catch (Exception var7) {
                    throw new AWTError("Error trying to install Assistive Technology: " + var2 + " " + var7);
                }
            }
        }

    }
```
음... `atNames`라는게 뭘까 하니  
```java
   private static void initAssistiveTechnologies() {
        final String var0 = File.separator;
        final Properties var1 = new Properties();
        atNames = (String)AccessController.doPrivileged(new PrivilegedAction<String>() {
            public String run() {
                File var1x;
                FileInputStream var2;
                try {
                    var1x = new File(System.getProperty("user.home") + var0 + ".accessibility.properties");
                    var2 = new FileInputStream(var1x);
                    var1.load(var2);
                    var2.close();
                } catch (Exception var4) {
                }

                if (var1.size() == 0) {
                    try {
                        var1x = new File(System.getProperty("java.home") + var0 + "lib" + var0 + "accessibility.properties");
                        var2 = new FileInputStream(var1x);
                        var1.load(var2);
                        var2.close();
                    } catch (Exception var3) {
                    }
                }

                String var5 = System.getProperty("javax.accessibility.screen_magnifier_present");
                if (var5 == null) {
                    var5 = var1.getProperty("screen_magnifier_present", (String)null);
                    if (var5 != null) {
                        System.setProperty("javax.accessibility.screen_magnifier_present", var5);
                    }
                }

                String var6 = System.getProperty("javax.accessibility.assistive_technologies");
                if (var6 == null) {
                    var6 = var1.getProperty("assistive_technologies", (String)null);
                    if (var6 != null) {
                        System.setProperty("javax.accessibility.assistive_technologies", var6);
                    }
                }

                return var6;
            }
        });
    }
```
그래서 `accessibility.properties` 이 친구를 찾아보자.
```sh
$ cat /etc/java-8-openjdk/accessibility.properties 
#
# The following line specifies the assistive technology classes 
# that should be loaded into the Java VM when the AWT is initailized.
# Specify multiple classes by separating them with commas.
# Note: the line below cannot end the file (there must be at
# a minimum a blank line following it).
#
assistive_technologies=org.GNOME.Accessibility.AtkWrapper
```
`assistive_technologies`라는 세팅이 되어 있을때 `org.GNOME.Accessibility.AtkWrapper`라는 클래스를 찾아보게 되어있다.  

## 해결
그래서 검색을 해봤더니 아래와 같이 하라고 한다.  
```sh
$ sudo vim /etc/java-8-openjdk/accessibility.properties 
#assistive_technologies=org.GNOME.Accessibility.AtkWrapper
```
되긴 되는데  

## 결론
awt에선 `accessibility.properties`를 왜 찾는걸까.  
`org.GNOME.Accessibility.AtkWrapper`얘는 왜 없는걸까.  
OpenJDK에만 없는건가?  
이거 버그 아닌가?  
https://bugs.openjdk.java.net/browse/JDK-8061622  
버그 맞네.  

## 참고
https://blog.softhints.com/java-11-and-ubuntu-assistive-technology-not-found-awterror/