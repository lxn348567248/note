# ClassLoder的体系
## 1.BootstrapClassLoader
>是用于加载此System.getProperty("sun.boot.class.path")路径的类，即加载系统内置的类如rt.jar,character.jar等
## 2.ExtClassLoader
>是用于加载此System.getProperty("java.ext.dirs")路径的类，即加载系统jre\lib\ext目录下的类
## 3.AppClassLoader
>是用于加载此System.getProperty("java.class.path")路径的类，即我们常说的classpath下的类
# 类的双亲委派
>Tip:为什么要有双亲委派这个机制,主要是为安全，当在父加载以后，子加载无法加载。

## 类加载的代码
java.lang.ClassLoader
```
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException {
          synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name); // 1. 从缓存中查找
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false); //父加载器加载
                    } else {
                        c = findBootstrapClassOrNull(name); //根加载器
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
    }
```

## 加载的顺序
>findLoadedClass()方法加载（查看自己是加载过，类似缓存）-> 判断是否有父加载器，
->parent.loadClass(name, false) 父加载器(递归加载)
->findBootstrapClassOrNull() 根加载器

# 每个加载器的父加载器
## 类加载器初始的代码
sun.misc.Launcher
```
     public Launcher() {
        Launcher.ExtClassLoader var1;
        try {
            var1 = Launcher.ExtClassLoader.getExtClassLoader();
        } catch (IOException var10) {
            throw new InternalError("Could not create extension class loader", var10);
        }

        try {
            this.loader = Launcher.AppClassLoader.getAppClassLoader(var1); //1.这个地方可以看到AppClassLoader 的父加载器是ExtClassLoader
        } catch (IOException var9) {
            throw new InternalError("Could not create application class loader", var9);
        }

        Thread.currentThread().setContextClassLoader(this.loader);
        String var2 = System.getProperty("java.security.manager");
        if (var2 != null) {
            SecurityManager var3 = null;
            if (!"".equals(var2) && !"default".equals(var2)) {
                try {
                    var3 = (SecurityManager)this.loader.loadClass(var2).newInstance();
                } catch (IllegalAccessException var5) {
                } catch (InstantiationException var6) {
                } catch (ClassNotFoundException var7) {
                } catch (ClassCastException var8) {
                }
            } else {
                var3 = new SecurityManager();
            }

            if (var3 == null) {
                throw new InternalError("Could not create SecurityManager: " + var2);
            }

            System.setSecurityManager(var3);
        }

    }

     public ExtClassLoader(File[] var1) throws IOException {
            super(getExtURLs(var1), (ClassLoader)null, Launcher.factory);//2.ExtClassLoader 的parent 是null
            SharedSecrets.getJavaNetAccess().getURLClassPath(this).initLookupCache(this);
        }

```


## 每个加载器的父加载器可知 AppClassLoader -> ExtClassLoader -> null(BootStrapClassLoader)。
如果要验证可以通过ClassLoader.getParent()方法自己去测试一下(下面是伪代码)。
>while(classLoader!=null){<br/>
    System.out.println(classLoader);<br/>
    classLoader  = classLoader.getParent();<br/>
    }<br/>



