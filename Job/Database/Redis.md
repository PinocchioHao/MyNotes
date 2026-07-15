### ✅ 1. Redis支持哪些常见的数据类型？

**What data types does Redis support?**

**EN：** Redis supports common types like **String**, **List**, **Hash**, **Set**, and **Sorted Set (ZSet)**.  
It also supports some special types like **Bitmap**, **HyperLogLog**, and **Geo**.

**CN：** Redis 支持常见的数据类型包括 **字符串、列表、哈希、集合、有序集合**。  
也支持一些特殊类型，比如 **位图、HyperLogLog 和地理位置类型**。

---

### ✅ 2. Redis 为什么这么快？

**Why is Redis so fast?**

**EN：** Redis is fast because it is **in-memory**, uses **efficient data structures**, and runs on a **single-threaded** event loop with **I/O multiplexing**.  
Also, it uses **compact encoding** and **optimized access patterns**.

**CN：** Redis 快是因为它是 **基于内存** 的，用了高效的数据结构，并且是 **单线程 + 多路复用** 模型。  
还用了 **紧凑编码** 和优化的访问方式。

---

### ✅ 3. 如何防止缓存穿透？

**How do you prevent cache penetration?**

**EN：** We can **cache empty results**, use **Bloom filters**, or check the **request parameters** before querying the cache.

**CN：** 可以 **缓存空结果**，或者用 **布隆过滤器** 来提前判断。也可以在请求前先验证参数。

---

### ✅ 4. 如何避免缓存雪崩？

**How to prevent cache avalanche?**

**EN：** Set **different expiration times**, use **hot backup**, or add a **fallback mechanism** to protect the database.

**CN：** 通过设置 **不同的过期时间**、做 **热点数据备份** 或加上 **降级机制**，来保护数据库。

---

### ✅ 5. 如何避免缓存击穿？

**How do you prevent cache breakdown?**

**EN：** Use **distributed locks**, or let **only one thread** rebuild the cache, while others wait or use **old data**.

**CN：** 可以使用 **分布式锁**，只让一个线程去回源，其他线程等或者读旧数据。

---

### ✅ 6. Redis过期策略有哪些？

**What are the expiration strategies in Redis?**

**EN：** Redis uses **lazy deletion**, **scheduled deletion**, and **passive expiration** during reads.

**CN：** Redis 会用 **惰性删除**、**定时清理** 和 **访问时顺带判断是否过期** 的方式。

---

### ✅ 7. Redis 是线程安全的吗？

**Is Redis thread-safe?**

**EN：** Yes, because Redis uses **a single-threaded model** for command execution, which avoids race conditions.

**CN：** 是的，因为 Redis 是 **单线程执行命令**，避免了线程竞争。

---

### ✅ 8. 你在项目中是怎么用 Redis 的？

**How do you use Redis in your project?**

**EN：** We use Redis for **configuration caching** and **idempotency control**.  
It helps reduce database pressure and prevent repeated submissions.

**CN：** 我们主要用 Redis 做了 **配置信息缓存** 和 **幂等控制**，能减少数据库压力，避免重复提交。

---

### ✅ 9. Redis 如何实现分布式锁？

**How do you implement a distributed lock in Redis?**

**EN：** Use **SET key value NX EX**. Set if not exists, with expiration.  
To release the lock safely, check if value matches before deleting.

**CN：** 用 **SET key value NX EX**，设置不存在才写入并加过期时间。  
释放时要先判断 value 是否匹配再删除，避免误删。

---

### ✅ 10. Redis 是怎么持久化的？

**How does Redis persist data?**

**EN：** Redis supports **RDB snapshots** and **AOF (Append Only File)**.  
RDB is faster to load, AOF is safer for data recovery.

**CN：** Redis 支持 **RDB 快照** 和 **AOF 追加日志**。  
RDB 启动快，AOF 数据恢复更安全。



### ✅ Q6. How do you prevent cache penetration?（如何避免缓存穿透？）

**EN:**  
We can use a **null cache**. When a query returns no result from DB, we still **cache a placeholder**, like a null or empty value, with a short TTL.  
This avoids hitting DB again and again for the same invalid request.

**CN:**  
可以用**空值缓存**，查询数据库结果为空时，也缓存一个占位值（比如 null），加一个短的过期时间，这样下次就不会再次查库了。

---

### ✅ Q7. What is cache avalanche and how do you handle it?（什么是缓存雪崩，怎么处理？）

**EN:**  
Cache avalanche means **lots of keys expire at the same time**, and DB gets too many requests suddenly.  
We can avoid it by setting **random expiration times** or using **mutex locks**.

**CN:**  
缓存雪崩指的是**大量缓存同时失效**导致数据库被打爆。  
可以通过设置**过期时间的随机值**，或者加**互斥锁**来防止。

---

### ✅ Q8. What is cache breakdown?（缓存击穿是啥？）

**EN:**  
It happens when a **hot key expires**, and many requests hit the DB before it's cached again.  
We can solve this with a **mutex lock**, or use **logical expiration** with background refresh.

**CN:**  
缓存击穿是某个**高频 key 过期**，大量请求同时打到数据库。  
可以用**互斥锁**或**逻辑过期 + 后台刷新**解决。

---

### ✅ Q9. How do you keep cache and DB consistent?（如何保证缓存和数据库一致性？）

**EN:**  
One way is **write-through**, where we write to DB and update cache at the same time.  
Another is **delayed double delete** – delete cache before and after updating DB.

**CN:**  
可以用**写穿模式**，同时写数据库和更新缓存。  
也可以用**延迟双删**：先删缓存，改完数据库后延迟一段时间再删一次。

---

### ✅ Q10. Redis 是线程安全的吗？（Is Redis thread-safe?）

**EN:**  
Yes, because it uses a **single-threaded model** for command execution.  
So commands are processed one at a time, avoiding race conditions.

**CN:**  
是的，Redis 是**单线程执行命令**的，所以天然线程安全，没有竞争问题。



### "What’s the difference between Redis and a database?"（Redis 和数据库的区别？）

这个来一个简要回答