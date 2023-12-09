# Serialization

这里面最核心的类就是 FArchive



### FStructuredArchiveFormatter



<img src="../../.gitbook/assets/file.excalidraw (3).svg" alt="" class="gitbook-drawing">

### FArchiveProxy

其实自己一直不理解这些代理是什么意思，但是看源码的话可以很快的清楚里面的内容。父类FArchiveProxy只是套了一层的空壳子，里面的任何操作还是调用构造时候传件去的Archive去处理。

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

