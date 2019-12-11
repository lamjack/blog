title: system-structure
tags:

### PO(Persistant Object) 持久对象

通常为数据库模型（表）和对象（类）之间的映射，即一张表对应一个 PO 对象，PO 对象中不应存在对数据库的操作。

### BO(Business Object) 业务对象

通常为一个封装的复杂对象，由 PO 组成，调用 DAO 完成业务处理。

### VO(View Object) 视图对象

与 DTO 类似，通常用于展示层，用来封装客户端所需要的数据，经典使用场景是 MVC。

### DO(Domain Object) 领域对象

### DAO(Date Access Object) 数据访问对象

主要负责持久层的操作，用于访问数据库，为业务提供数据接口。

### DTO(Data Transfer Object) 数据传输对象

通常用于提供数据接口，如 Web API、Web Service 等。简单说就是如果 PO 是一个具有多个属性的对象，但是不想完全把 PO 的内容暴露给外界，这时候可以使用 DTO。