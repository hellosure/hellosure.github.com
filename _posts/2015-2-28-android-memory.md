---
layout: post
title: Android内存监测
category: Android
tags: [Android,内存]
---

                try{
                    Log.i("Memory","try");
                    int i = 0;
                    while(true){
                        Log.i("Memory","try "+i);
                        byte[] bytes = new byte[1024 * 1024 * i];
                        i++;
                    }
                }catch (OutOfMemoryError e){
                    logAvailableMem();
                }

1024 * 1024就是1M

java的内存管理，如果在循环体中每次创建1M，那么一直不会OOM，因为指向的是同一块1M的内存。只能像上面代码这样每次增量开辟新的内存空间。

Error是可以catch的，只不过一般程序中不去catch error，因为没有意义。

    private void logAvailableMem(){
        MemoryInfo mi = new MemoryInfo();
        ActivityManager activityManager = (ActivityManager) getSystemService(ACTIVITY_SERVICE);
        activityManager.getMemoryInfo(mi);
        long availableMegs = mi.availMem / 1048576L;

        Log.i("Memory", "" + availableMegs);
    }

<http://stackoverflow.com/questions/3170691/how-to-get-current-memory-usage-in-android>

打印出来的就是M数。

－EOF-
