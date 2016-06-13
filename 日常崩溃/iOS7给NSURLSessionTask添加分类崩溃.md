#概述

明天遇到一个BUG，不过只是在iOS7.0上出现的bug。问题是这样的：给NSURLSessionTask添加了分类来增加一个属性，即operation对象（弱引用），在iOS7以上给task.operation赋值时，都没有问题，但是在iOS7.0上必闪退。

闪退打印的提示信息是：

```
[__NSCFUploadTask setOperation:] unrecognized selector ...
```

很怪的bug吧！起初我怀疑是因为我直接在当前Operation文件中添加NSURLSessionTask分类导致查找不到，于是我尝试了将Operation分类改成单独的文件，结果还是一样！

#目前未找到原因

如果大家有遇到过此类问题，还请告知。最终我不得不去掉NSURLSessionTask分类，改成以task对象来判断任务，然后更新对应的数据。

当然，这里是做视频上传的任务的封装，本想提高访问效率，使用添加分类的方式可以在代理中直接取到对象然后更新数据即可。但是，现在在iOS7上不可行，于是不得不去掉这种方式，改成遍历operationQueue中的operations来判断公开的task对象是否与代理返回的task一致，然后更新数据。这样做效率自然要下降，每次回调都需要遍历一遍查找对象。

#小结

目前只发现在iOS7某手机上出现此问题，不知道其他iOS7手机是否都会出现，但是测试机出现了这样的问题，那就是p1级的bug，问题很严重的。

如果大家有遇到过并有足够的时间去研究探索，还请将解决办法分享给大家！谢谢啦！


