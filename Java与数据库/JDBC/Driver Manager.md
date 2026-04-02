
# DriverManager 的核心职责

- **注册管理 (Registration)**：当数据库厂商（如 MySQL, Oracle）提供的驱动程序加载到 JVM 时，它们会自动向 `DriverManager` “报到”登记。
    
- **建立连接 (Connection)**：当你需要连接数据库时，你把数据库地址（URL）交给 `DriverManager`，它会从登记簿里找出一个能听懂这个地址的驱动，帮你握手建立连接。