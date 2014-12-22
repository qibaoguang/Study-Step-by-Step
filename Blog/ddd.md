领域驱动设计
============
###胖模型 VS 贫血模型（有误，有空重新整理）
贫血模型：又称失血模型，是指领域对象里只有get和set方法，或者包含少量的CRUD方法，所有的业务逻辑都不包含在内而是放在Business Logic层。

优点是系统的层次结构清楚，各层之间单向依赖，Client->(Business Facade)->Business Logic->Data Access。当然Business Logic是依赖Domain Object的。似乎现在流行的架构就是这样，当然层次还可以细分。

该模型的缺点是不够面向对象，领域对象只是作为保存状态或者传递状态使用，所以就说只有数据没有行为的对象不是真正的对象。在Business Logic里面处理所有的业务逻辑，在POEAA(企业应用架构模式)一书中被称为Transaction Script模式。

胖模型：又称充血模型，层次结构和上面的差不多，不过大多业务逻辑和持久化放在Domain Object里面，Business Logic只是简单封装部分业务逻辑以及控制事务、权限等，这样层次结构就变成Client->（Business Facade)->Business Logic->Domain Object->Data Access。

优点是面向对象，Business Logic符合单一职责，不像在贫血模型里面那样包含所有的业务逻辑太过沉重。

缺点是如何划分业务逻辑，什么样的逻辑应该放在Domain Object中，什么样的业务逻辑应该放在Business Logic中，这是很含糊的。即使划分好了业务逻辑，由于分散在Business Logic和Domain Object层中，不能更好的分模块开发。熟悉业务逻辑的开发人员需要渗透到Domain Logic中去，而在Domian Logic又包含了持久化，对于开发者来说这十分混乱。其次，因为Business Logic要控制事务并且为上层提供一个统一的服务调用入口点，它就必须把在Domain Logic里实现的业务逻辑全部重新包装一遍，完全属于重复劳动。

贫血模型结构清晰，便于分模块开发，但贫血的领域模型不够优雅，不够OO。充血模型的领域对象由于依赖于数据访问层，如果没有透明持久化支持，Domain Object的业务逻辑脱离Data Access层就无法进行单元测试，还好Java世界已有Hibernate等ORM框架，加上Spring的动态注入，实现充血模型还是能够做到的。

### See also 
* [失血，贫血，充血，胀血模型的解释](http://www.oschina.net/question/54100_10400)






