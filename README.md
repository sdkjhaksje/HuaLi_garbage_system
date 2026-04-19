# HuaLi_garbage_system

> 本项目为中国大学生计算机设计大赛的参赛作品，面向智慧校园与智慧社区场景，聚焦垃圾桶满溢、散落垃圾、火焰与烟雾的智能识别与预警。

[![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?logo=fastapi)](https://fastapi.tiangolo.com/)
[![Ultralytics](https://img.shields.io/badge/Ultralytics-YOLO-FF9F00)](https://github.com/ultralytics/ultralytics)
[![ONNX](https://img.shields.io/badge/ONNX_Runtime-1.20+-grey?logo=onnx)](https://onnxruntime.ai/)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.8+-green?logo=opencv)](https://opencv.org/)
[![Celery](https://img.shields.io/badge/Celery-5.4+-37814A?logo=celery)](https://docs.celeryq.dev/)
[![Redis](https://img.shields.io/badge/Redis-5.2+-DC382D?logo=redis)](https://redis.io/)
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.0+-D71F00?logo=sqlalchemy)](https://www.sqlalchemy.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## 项目简介

本项目是一个融合了目标检测、Web 可视化和异步视频处理的智能垃圾监测系统。代码中已经实现了以下核心能力：

- 垃圾桶、垃圾满溢、散落垃圾、火焰、烟雾共 5 类目标识别
- 图片上传检测与 Base64 图像检测
- 视频上传异步处理与结果视频回传
- 告警记录持久化、统计分析与历史查询
- ONNX Runtime 优先推理，找不到 ONNX 时自动回退到 Ultralytics YOLO `.pt` 模型
- Redis + Celery 异步任务队列，若 Redis 不可用则自动退回本地后台线程处理视频任务

当前推荐使用的是 `FastAPI` 版本入口：`app/main.py`。仓库中保留了一个较早期的 `Flask` 原型文件 `app/app.py`，但不是当前主入口。

## 功能对应代码

### 1. Web 系统

- `app/main.py`：FastAPI 应用入口
- `app/api/routes.py`：图片、Base64、视频、告警、统计等 API
- `app/api/pages.py`：页面路由
- `app/templates/`：前端页面模板

### 2. 推理与告警

- `app/services/detection_service.py`：多模型检测、告警冷却、场景分析、绘框输出
- `app/services/inference.py`：ONNX Runtime / Ultralytics 双后端推理
- `app/services/video_service.py`：视频逐帧处理与结果导出
- `app/services/record_service.py`：检测记录、告警记录、统计数据写入数据库

### 3. 训练与数据处理

- `train_garbage.py` / `train_yolo.py`：垃圾相关模型训练
- `train_fire_smoke.py`：火焰与烟雾模型训练
- `export_onnx.py`：将 `.pt` 权重导出为 `.onnx`
- `convert_coco.py`、`convert_coco2yolo_separate.py`：数据集格式转换
- `merge_datasets.py`、`create_smoke_dataset.py`：火焰/烟雾数据整理

## 项目结构

```text
HuaLi_garbage_system/
├─ app/
│  ├─ api/                 # API 与页面路由
│  ├─ models/              # 训练权重与 ONNX 模型
│  ├─ services/            # 推理、视频处理、记录服务
│  ├─ templates/           # 页面模板
│  ├─ uploads/             # 告警截图与视频输出
│  └─ main.py              # FastAPI 主入口
├─ dataset/                # 垃圾数据集
├─ dataset_fire/           # 火焰数据集
├─ dataset_smoke_5images_new/  # 烟雾数据集
├─ requirements.txt
├─ export_onnx.py
├─ train_garbage.py
├─ train_yolo.py
└─ train_fire_smoke.py
```

## 从克隆到使用

### 1. 克隆项目

```bash
git clone https://github.com/Nyzeep/HuaLi_garbage_system.git
cd HuaLi_garbage_system
```

### 2. 创建虚拟环境

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
```

Linux / macOS:

```bash
source .venv/bin/activate
```

### 3. 安装依赖

```bash
pip install -r requirements.txt
```

### 4. 可选：配置环境变量

项目会自动读取根目录下的 `.env` 文件；如果你不提供，也会使用代码中的默认配置直接启动。

示例：

```env
APP_NAME=垃圾分类检测系统
APP_VERSION=2.0.0
DEBUG=true
DATABASE_URL=sqlite:///garbage_system.db
REDIS_URL=redis://localhost:6379/0
VIDEO_DEFAULT_SKIP_FRAMES=3
```

### 5. 准备模型

仓库中的 `app/models/` 已经包含了项目使用过的权重与部分 ONNX 文件，通常可以直接运行。

系统默认会优先尝试加载以下模型：

- 垃圾检测：`app/models/garbege.onnx` 或 `app/models/garbege.pt`
- 火焰检测：`app/models/only_fire.onnx` 或 `app/models/only_fire.pt`
- 烟雾检测：`app/models/fire_smoke.onnx` 或 `app/models/fire_smoke.pt`


### 6. 启动 FastAPI 服务

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

启动后可访问：

- 首页：`http://127.0.0.1:8000/`
- 检测页：`http://127.0.0.1:8000/detection`
- 视频页：`http://127.0.0.1:8000/video`
- 告警页：`http://127.0.0.1:8000/alerts`
- 统计页：`http://127.0.0.1:8000/statistics`
- 数据接口文档：`http://127.0.0.1:8000/docs`

### 7. 可选：启动 Redis 与 Celery

项目的视频检测接口支持异步任务。如果你已经安装 Redis，推荐再启动 Celery Worker：

```bash
celery -A app.celery_app.celery_app worker --loglevel=info --pool=solo
```

默认 Redis 地址为：

```env
redis://localhost:6379/0
```

即使没有 Redis，项目也会退回到本地线程处理视频任务，只是更适合演示或轻量测试。

## 常用接口

### 图片检测

```http
POST /api/detect/image
```

表单字段：

- `file`：上传图片文件

### Base64 图像检测

```http
POST /api/detect/base64
```

请求体示例：

```json
{
  "image": "data:image/jpeg;base64,..."
}
```

### 视频检测

```http
POST /api/detect/video
```

表单字段：

- `file`：上传视频
- `skip_frames`：跳帧数，默认 `3`

提交后可通过下面的接口轮询任务状态：

```http
GET /api/tasks/{task_id}
```

### 告警与统计

```http
GET /api/alerts
GET /api/alerts/{record_uid}/image
GET /api/statistics
GET /api/status
GET /api/classes
```

## 训练与导出

### 1. 垃圾相关模型训练

```bash
python train_garbage.py --mode train
```

或：

```bash
python train_yolo.py --mode train
```

验证：

```bash
python train_garbage.py --mode val
```

### 2. 火焰与烟雾模型训练

```bash
python train_fire_smoke.py
```

### 3. 导出 ONNX

```bash
python export_onnx.py
```

导出后可将生成的 `.onnx` 文件放入 `app/models/`，供系统优先加载。

## 数据与存储说明

- 默认数据库为根目录下的 `garbage_system.db`
- 告警截图默认保存到 `app/uploads/alerts/`
- 视频结果默认保存到 `app/uploads/videos/`
- 数据表会在服务启动时自动创建

## 使用建议与注意事项

### 1. 推荐入口

日常演示、接口联调、页面展示时，优先使用：

```bash
uvicorn app.main:app --reload
```

### 2. 关于旧版 Flask 文件

仓库中存在 `app/app.py`，这是较早保留的 Flask 版本原型。由于当前 `requirements.txt` 已经围绕 FastAPI 版本整理，默认不建议再将它作为主启动方式。

### 3. 训练脚本中的绝对路径

部分训练与数据处理脚本中写有绝对路径，例如：

- `D:/garbage_system/...`
- `/root/workspace/...`

如果你在其他机器上复现，请先按本地环境修改这些路径，再运行训练或数据转换脚本。

### 4. 模型文件名存在历史命名

仓库中保留了 `garbege.pt` 这一历史命名文件名，代码当前就是按这个名字加载模型，因此直接使用时不要随意改名，除非你同步修改了配置文件。

## 许可证

本项目采用 [MIT License](LICENSE) 开源，适合学习、课程设计、竞赛展示与二次开发参考。
