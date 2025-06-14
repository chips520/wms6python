# WMSv2 Python 方案

## 1. 技术栈
- **框架**：Python 3.11。
- **HTTP API**：FastAPI。
- **数据库**：SQLAlchemy，操作 SQLite。
- **GUI**：PyQt5，集成 `QSystemTrayIcon` 用于系统托盘。
- **开机自启动**：启动文件夹创建快捷方式。
- **部署**：PyInstaller，生成单 `.exe` 文件。
- **日志**：Python `logging`。

## 2. 数据库设计
- **`trays` 表**：
  - `tray_id` (String, 主键)：料盘 ID（如 "AGV1"、"TS01"）。
  - `description` (String, 可空)：描述。
  - `max_slots` (Int, 非空)：最大槽位数（如 50、60）。
  - `created_at` (DateTime, 默认当前时间)。
  - `updated_at` (DateTime, 默认当前时间，更新时刷新)。
- **`material_locations` 表**：
  - `id` (Int, 主键，自增)：记录 ID。
  - `tray_id` (String, 非空，外键)：料盘 ID。
  - `slot_index` (Int, 非空)：槽位索引（1 到 `max_slots`）。
  - `item_id` (String, 可空)：样品 ID（空为 `""`，禁用为 `"-99"`）。
  - `tray_code` (String, 可空)：料盘编号。
  - `timestamp` (DateTime, 默认当前时间，更新时刷新)。
  - `process_info` (String, 可空)：工序/任务 ID。
  - **约束**：唯一键 (`tray_id`, `slot_index`)。
  - **索引**：`tray_id`, `slot_index`, `item_id`。
- **数据库**：SQLite，存储在运行目录（`wmsv2.db`）。

## 3. 实现方式
- **数据库**：
  - 使用 SQLAlchemy ORM 定义模型，配置外键和唯一约束。
  - 动态生成数据库路径，首次运行创建 `wmsv2.db`。
- **HTTP API**：
  - FastAPI 提供 RESTful 端点，运行于 `http://localhost:8080`。
  - 端点包括：创建料盘、查询空槽位、放置样品、清空槽位、批量更新、批量清空、查询样品位置。
  - 使用 Pydantic 验证请求，错误返回 JSON 格式（包含错误码和消息）。
- **GUI**：
  - PyQt5 实现主界面，包含服务控制（启动/停止按钮）、查询输入框、结果表格。
  - QTableWidget 显示查询结果，支持按料盘 ID 或样品 ID 过滤。
  - 系统托盘（QSystemTrayIcon）支持最小化、右键菜单（显示、启动/停止服务、退出）。
- **开机自启动**：
  - 使用 `pywin32` 在启动文件夹创建 `.lnk` 快捷方式，指向 `.exe`。
- **GUI 扩展**：
  - **批量操作**：QTableWidget 支持多选行，添加“批量更新”和“批量清空”按钮。批量更新弹出对话框，输入样品 ID、料盘编号、工序信息；批量清空需确认。
  - **禁用槽位**：QTableWidget 右键菜单提供“禁用槽位”和“启用槽位”选项，禁用槽位标记为 `item_id = "-99"`，表格行高亮为红色（使用 QSS 样式）。
- **日志**：
  - 使用 Python `logging` 记录服务启停、数据库操作、错误信息，存储到 `wmsv2.log`。

## 4. 文档

### 4.1. API 文档
- **格式**：Markdown。
- **内容**：
  - 概述：描述 API 功能，运行地址（`http://localhost:8080`）。
  - 端点列表：每个端点包含路径、方法、请求参数、响应格式、错误码。
  - 示例：创建料盘、查询空槽位、放置样品、清空槽位、批量更新、批量清空、查询样品位置。
  - 错误格式：JSON 包含 `error`（错误码）和 `message`（描述）。

### 4.2. 用户手册
- **格式**：Markdown。
- **内容**：
  - 安装：复制 `.exe`、图标和数据库文件到指定目录，运行 `.exe`。
  - 系统要求：Windows 10，4GB RAM，500MB 磁盘空间。
  - 操作指南：
    - 启动/停止服务：按钮控制 HTTP API。
    - 查询：输入料盘 ID 或样品 ID，表格显示结果。
    - 批量操作：多选槽位，更新或清空信息。
    - 禁用槽位：右键菜单标记禁用，高亮显示。
    - 系统托盘：最小化、控制服务、退出。
  - 故障排除：托盘图标缺失、端口占用、数据库错误。

## 5. 打包与部署
- **打包**：
  - 使用 PyInstaller 命令，生成单 `.exe`，包含图标文件（`icon.ico`）。
  - 可选使用 UPX 压缩减小文件大小。
- **部署**：
  - 复制 `wmsv2.exe`、`icon.ico`、`wmsv2.db` 到 `C:\WMSv2`。
  - 运行 `.exe`，勾选“开机自启”创建快捷方式。
  - 测试托盘、GUI、API，重启验证自动启动。
- **文件大小**：约 50-100 MB。

## 6. 优缺点
- **优点**：
  - 开发速度快：Python 语法简单，FastAPI 和 PyQt5 易用。
  - GUI 简单：PyQt5 托盘和界面开发直观。
  - 文件较小：打包后 `.exe` 比 .NET 小。
  - 社区生态：Python 库丰富，调试方便。
  - GUI 扩展：批量操作和禁用槽位提升用户体验。
- **缺点**：
  - 性能稍逊：FastAPI 不如 ASP.NET Core，Python 解释执行慢。
  - 类型安全：动态类型易引入运行时错误。
  - Windows 集成：需 `pywin32` 支持启动文件夹，略逊于 .NET。