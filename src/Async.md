## Send & Sync
如何理解Send和Sync？
其实就是考虑什么类型可以在线程之间安全传递。安全传递就是能不能Send
- 如果是T类型，那就是Send
- 如果是&T引用类型，那么T就是Sync