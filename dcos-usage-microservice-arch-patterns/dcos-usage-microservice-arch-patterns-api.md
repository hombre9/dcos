## API管理

### API文档自动化

当前互联网特别是移动互联网，设备与平台之间的交互的基础是服务API接口。以API驱动的开发是团队之间最常用的协作方式，而作为交互的基石，API的准确性，完整性和及时性是影响开发效率的关键。基于API驱动的开发模式具体到实际应用受到团队、人员、规模和管理流程等众多因素的影响。在敏捷开发实践中，一种理想的情况是开发人员在代码中描述API的详细信息，这些信息是整个开发跟踪的基准。借助于文档生成工具，自动提取API信息并生成文档。当代码产生变更时，通过CI/CD流程自动更新发布的API文档，让开发人员始终基于最新的接口进行开发，避免因沟通和跟踪问题导致开发效率低下甚至引入BUG。

微服务的API接口根据不同的业务需求有不同的技术实现选择，当前常用的有HTTP/REST，gRPC，Thrift等。

#### REST API

通过[Swagger](http://swagger.io/)，[ReDoc](https://github.com/Rebilly/ReDoc)等工具，可以为REST接口自动生成API接口描述，并依此生成清晰的API文档。Swagger是一个规范和完整的框架，用于生成、描述、调用和可视化**RESTful**风格的 Web 服务。ReDoc是一款根据OpenAPI/Swagger生成的API接口描述文件生成API参考文档的工具。
