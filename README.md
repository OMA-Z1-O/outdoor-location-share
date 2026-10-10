---
AIGC:
    Label: "1"
    ContentProducer: 001191440300708461136T1XGW3
    ProduceID: a9e35891d65172a196593882cfa7f8e6_c0c40ed6c40211f1820a525400638852
    ReservedCode1: s3Ll2xt+Y2fc0lAKZZsp7wnW+Jgee6z6vZZhPcKZgMPA11bckFfk3fqhcwoOHqGvZO6zYD5s8opy4+217zWYmVNFpRmrAESJR38TT2qxagYs0ggsxvKmTWZqKQ0kWKehGeYMJxvwYtn9kMs1NPMdimTJa9tq2VEI7xGFiot9+Lew2B3Wu8dKdmQpYss=
    ContentPropagator: 001191440300708461136T1XGW3
    PropagateID: a9e35891d65172a196593882cfa7f8e6_c0c40ed6c40211f1820a525400638852
    ReservedCode2: s3Ll2xt+Y2fc0lAKZZsp7wnW+Jgee6z6vZZhPcKZgMPA11bckFfk3fqhcwoOHqGvZO6zYD5s8opy4+217zWYmVNFpRmrAESJR38TT2qxagYs0ggsxvKmTWZqKQ0kWKehGeYMJxvwYtn9kMs1NPMdimTJa9tq2VEI7xGFiot9+Lew2B3Wu8dKdmQpYss=
---

# 户外位置共享 · AR 指向（H5）

一个纯 Web 实现的户外实时位置共享应用：**发送端**通过 GPS + 指南针 + 拍照定点计算目标位置并推送，**接收端**打开摄像头画面，用高亮标记叠加指示目标方向（AR 效果）。

## 文件结构

| 文件 | 说明 |
|---|---|
| `index.html` | 单页应用（发送端 + 接收端两个角色） |
| `server.js` | Node.js 服务器：托管页面 + WebSocket 转发目标位置 |
| `README.md` | 本说明 |

## 快速开始

```bash
# 1. 安装依赖（仅需要 ws）
npm install ws

# 2. 启动服务器（默认端口 8080）
node server.js

# 3. 局域网访问
#    电脑 IP 假设为 192.168.1.100
#    手机浏览器打开：http://192.168.1.100:8080
```

自定义端口：`PORT=9000 node server.js`

## 使用流程

### 发送端（定点拍照，计算目标）
1. 打开页面，默认进入"发送端"；确认 WebSocket 地址已填好，点**连接**（状态变绿）。
2. 点 **获取我的位置**，等待 GPS 定位成功。
3. 点 **启动指南针**，授权后手机顶部朝向会实时显示。
4. 点 **启动摄像头预览**，把摄像头画面中心对准远处目标（建筑/同伴集合点等），点 **📸 拍照**。
5. 点 **记录当前观测点**（记为 A）。然后**移动 10~30 米**（尽量让两次观测与目标形成明显夹角），再次对准同一目标，点 **拍照 + 记录当前观测点**（记为 B）。
6. 点 **计算目标位置（三角交汇）**，页面给出目标经纬度和估算距离。
   - 若无法移动，可切换 **单点+距离模式**：输入到目标的直线距离，直接推算目标坐标。
7. 点 **🚀 推送给同伴**，目标位置实时转发给所有接收端。

### 接收端（AR 标注）
1. 打开同一页面，切到 **接收端** 标签，连接服务器。
2. 点 **打开摄像头 AR 视图**，授权摄像头；屏幕出现实时画面。
3. 收到目标位置后，画面中出现**高亮黄绿色圆环标记**：转动手机让标记移向屏幕中央即对准目标方向；目标偏离视野时，屏幕边缘出现闪烁箭头指示转动方向。
4. 底部刻度条实时显示目标方位角与相对角。

## 定位计算原理

- **三角交汇**：两个不同观测点的 (GPS坐标 + 指南针方位角) 形成两条方位线，在局部平面坐标系中求交点即目标位置；距离为交汇点到观测点的直线距离。
- **单点+距离**：由观测点坐标 + 方位角 + 用户输入距离，按球面推算公式直接计算目标经纬度。

## 关键技术点

| 能力 | API |
|---|---|
| 指南针方向 | `DeviceOrientationEvent`（iOS 用 `webkitCompassHeading`；Android 用 `alpha` 近似，需水平放置并画“8”校准） |
| 定位 | `navigator.geolocation`（高精度模式） |
| 摄像头 | `getUserMedia({ video: { facingMode: 'environment' } })` |
| 实时推送 | WebSocket（服务器转发 `target-update` 给所有接收端） |

## 重要注意事项

1. **HTTPS 要求**：`getUserMedia` / `geolocation` / iOS 指南针授权均要求**安全上下文**（`localhost` 除外）。局域网内访问 IP 属于非安全上下文，调试方法：
   - Chrome / Edge：启动参数加 `--unsafely-treat-insecure-origin-as-secure="http://192.168.1.100:8080"`（手机 Chrome 需开 `chrome://flags` 的 Insecure origins treated as secure，填入该地址）；
   - 或者用 `mkcert` / `ngrok` / 云服务器部署 HTTPS。
2. **iOS 指南针**：首次点击"启动指南针"会弹出系统授权框，必须允许；`webkitCompassHeading` 是已校正的磁航向，直接使用。
3. **Android 指南针精度**：`alpha` 并非磁北航向，误差较大，仅作近似；更精确需磁力计+加速度计融合，可后续扩展。
4. **三角交汇误差**：GPS 精度（约 5~15m）+ 指南针误差（±3~10°）会放大目标误差，两次观测点间距越大、夹角越接近 90° 越准；目标距离越远误差越大。
5. **距离仅为估算**：画面上的"距离约 X 米"是三角交汇/输入距离的估算值，非激光测距。
6. 指南针数值表示**手机顶部朝向**，使用时请将手机顶部对准目标。
7. 建议两台以上设备同时测试；单机可开两个浏览器标签页分别充当发送端/接收端验证链路。

## 可扩展方向

- 磁力计+加速度计融合计算 Android 真磁北航向
- 摄像头俯仰角（`DeviceOrientationEvent` 的 beta/gamma）驱动标记垂直偏移
- 多接收端群组、目标锁定与跟随、历史轨迹
- 接入地图底图校验目标位置
*（内容由AI生成，仅供参考）*
