### Controller相关注解
在 RESTful 规范中，**GET 请求原则上不应该有 Body**（虽然技术上能实现，但很多中间件、代理服务器会直接过滤掉 GET 的 Body）。

#### 方案 A：Spring MVC 的自动绑定（推荐用于 GET）

Spring 其实支持直接把 URL 参数映射到一个 POJO 对象里，**不需要加任何注解**（或者加 `@ModelAttribute`，但不加也行）。

- **Controller 写法：**
    
```java
@GetMapping("/search")
public ApiResponse<Page<TaskVO>> search(TaskQueryDTO query, Pageable pageable) {
    // Spring 会自动把 ?title=abc&priority=1 映射到 query 对象的属性中
    return ApiResponse.success(taskService.complexSearch(query, pageable));
}
```

- **优点**：符合 GET 获取资源的语义，方便 URL 分享和书写。


少量参数可以使用@RequestParam；
get请求的参数直接就会在url链接后面通过?分隔

前端POST请求都按照JSON传输（即使表单也要按照raw的json传输，一般只有文件才会使用form-data），后端@RequestBody接收JSON请求体


---

### REST 常见规则及设计
**路径命名规则：Base路径留给简单CRUD，使用REST API的谓词就能表面接口作用，不必加后缀；复杂业务请求使用POST+后缀设计接口。**


在真正的 RESTful 设计中，我们通过 HTTP 方法来表达**意图**。

|**谓词**|**语义 (Intent)**|**是否幂等**|**例子 (URI)**|
|---|---|---|---|
|**GET**|获取资源（单条或列表）|是|`GET /tasks`, `GET /tasks/1`|
|**POST**|**新建**资源，或执行**非幂等**的复杂动作|否|`POST /tasks`|
|**PUT**|**完全替换**资源（客户端提供完整对象）|是|`PUT /tasks/1`|
|**PATCH**|**局部更新**资源（只传修改的字段）|否|`PATCH /tasks/1`|
|**DELETE**|**移除**资源|是|`DELETE /tasks/1`|

#### 逻辑删除（Soft Delete）用什么？

**结论：依然建议用 `DELETE`。**

- **理由**：REST API 的设计应该**屏蔽底层实现细节**。从客户端（前端）的角度来看，它的意图就是“删除这个任务”。至于后端是把数据从硬盘抹掉，还是把 `is_deleted` 改成 `1`，那是**数据库的实现逻辑**。
    
- **语义映射**：`DELETE` 方法代表“使该资源不可见/不可用”。
    
- **特例**：如果你需要专门记录“谁在什么时间禁用了它”，且这是一个可以被“激活/反激活”的状态切换，你可以用 `PATCH /tasks/1` 更新 `status` 字段。但 90% 的场景，`DELETE` 是最标准的。
    

#### 复杂业务逻辑：后缀（Suffix）到底加不加？

你提到的“复杂接口包含多个系统交互”确实是 REST 的难点。**“完全不加后缀”确实太理想化了，工业界并不会死脑筋。**

当你的操作超出了简单的 CRUD（增删改查），我们通常使用 **“动作（Action）”模式**。

#### 业界推荐的命名方式：

- **审核任务**：`POST /api/tasks/{id}/approve`
    
- **重发通知**：`POST /api/tasks/{id}/resend-notification`
    
- **同步外部数据**：`POST /api/tasks/sync`
    
- **批量操作**：`POST /api/tasks/batch-delete`
    

**原则：**

1. **资源根路径**（`/tasks`）和 **特定资源路径**（`/tasks/1`）尽量保持干净，只留给标准的 CRUD。
    
2. **特定的业务动作**，可以在路径末尾加动词，但**一律使用 POST**（因为这类动作通常不是幂等的，且涉及状态变更）。
    

#### 4. 复杂业务 Controller 示例

这是一个融合了“理想”与“现实”的工业级 Controller 模板：

```java
@RestController
@RequestMapping("/api/tasks")
@RequiredArgsConstructor
public class TaskController {

    private final TaskService taskService;

    // --- 标准 CRUD 部分 (不加后缀) ---

    @GetMapping
    public ApiResponse<Page<TaskVO>> list(TaskQueryDTO query, Pageable pageable) {
        return ApiResponse.success(taskService.search(query, pageable));
    }

    @PostMapping
    public ApiResponse<TaskVO> create(@Valid @RequestBody TaskDTO dto) {
        return ApiResponse.success(taskService.create(dto));
    }

    @DeleteMapping("/{id}")
    public ApiResponse<Void> delete(@PathVariable Long id) {
        taskService.softDelete(id); // 后续调用逻辑删除
        return ApiResponse.success(null);
    }

    // --- 复杂业务逻辑部分 (加后缀，且明确动词) ---

    /**
     * 场景：复杂的任务指派，涉及通知第三方系统、更新多张表
     */
    @PostMapping("/{id}/assign")
    public ApiResponse<Void> assignTask(@PathVariable Long id, @RequestBody Long userId) {
        taskService.assign(id, userId);
        return ApiResponse.success(null);
    }

    /**
     * 场景：批量处理。不建议在 DELETE 链接里带一串 ID，而是通过 POST Body 传 List
     */
    @PostMapping("/batch-archive")
    public ApiResponse<Integer> batchArchive(@RequestBody List<Long> ids) {
        int count = taskService.archiveMany(ids);
        return ApiResponse.success(count);
    }
}
```

### 总结建议：

1. **脚手架里不要出现 `saveTask`**：这是最容易被贴上“不专业”标签的行为。
    
2. **保持 CRUD 纯净**：`GET`, `POST`, `PUT`, `DELETE` 对应资源根路径。
    
3. **复杂操作大胆加后缀**：但路径要像 `/tasks/{id}/action` 这样写，而不是 `/tasks/actionTask`。


---


### 常见返回码值

#### 业务码 (Business Code) vs. HTTP 状态码

这是一个经典的面试题：有了 HTTP 状态码（200, 404, 500），为什么还要在 JSON 里定义 `code`？

- **HTTP 状态码**：是给**网络基础设施**（浏览器、Nginx、网关）看的。它表示请求在传输层和协议层的结果。
    
- **业务状态码 (`ApiResponse.code`)**：是给**业务逻辑**看的。例如，同样是 400 (Bad Request)，业务码可以细分为：`40001` (余额不足)、`40002` (库存不足)。
    

**最佳实践建议：** 在脚手架里，**外层的 HTTP 状态码**尽量保持标准（如 200, 400, 500），而**内层的 `code`** 用于**细化业务场景**。


#### 常见返回码组织建议（取舍）

如果你觉得维护两套码太麻烦，业界还有一种“**简化版**”做法：

- **全站 HTTP 永远返回 200**：不管后端出什么错，Header 永远是 200。所有的错误全靠 JSON 里的 `code`（如 400, 500, 2001）来区分。
    
- **澳洲/欧美评价**：这种做法在一些国内公司很流行（为了方便前端统一处理），但在澳洲的 **Enterprise 项目**里会被认为“**不符合标准（Anti-pattern）**”。因为这会导致监控系统（如 Prometheus/CloudWatch）无法通过 HTTP 状态码来统计接口的故障率。


#### 常见业务返回码的组织逻辑

业界通常会对错误码进行**分段**，方便一眼看出是哪个模块出的错：

| **错误码范围**     | **含义**         | **例子**                                 |
| ------------- | -------------- | -------------------------------------- |
| **200**       | 纯成功            | `SUCCESS`                              |
| **1000-1999** | 通用参数/权限类       | `PARAM_ERROR`, `TOKEN_EXPIRED`         |
| **2000-2999** | 用户模块相关         | `USER_LOCKED`, `WRONG_PASSWORD`        |
| **3000-3999** | 业务模块 (Task) 相关 | `TASK_NOT_FOUND`, `TASK_LIMIT_REACHED` |
| **5000+**     | 系统/中间件/第三方接口错误 | `DATABASE_ERROR`, `MAIL_SEND_FAILED`   |


---


