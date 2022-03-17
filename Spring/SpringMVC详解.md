## 简介

> Spring的模型-视图-控制器（MVC）框架是围绕 **DispatcherServlet** 来设计的，这个servlet会把请求分发给各个处理器，并支持可配置的处理器映射、视图渲染、本地化、时区与主题渲染以及文件上传等

## MVC流程

- 客户端请求提交到DispatchServlet(前端控制器)
- DispatchServlet查询HandlerMapping(处理器映射器)，找到对应的handler(处理器)
- DispatchServlet通过HandlerAdapter(处理器适配器)调用对应的Handler
- 执行Handler逻辑，返回ModelAndView(模型数据和视图)至DispatchServlet
- DispatchServlet将ModelAndView传给ViewResolver(视图解析器)
- ViewResolver解析后返回对应的View(视图)
- DispatchServlet将Model(模型数据)渲染至View中
- DispatchServlet将最终结果响应客户端