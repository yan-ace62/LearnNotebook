# sync.Pool对象
Pool是标准库sync中的一个可以存放临时对象并且线程安全的数据容器,但是其数据内容可能在任何时候被移除。

## Pool定义

    type Pool struct {
        New func() interface{} // 当Pool中没有值时候,New函数能够生成特定值.
    }

    // 如果Pool中有多个值，任意选择一个值返回，并从Pool移除该值
    func (p *Pool) Get() interface{}

    // 将值添加到Pool中
    func (p *Pool) Put(interface{})

