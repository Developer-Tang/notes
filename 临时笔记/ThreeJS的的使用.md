## 场景

### Scene-场景

#### 属性

- `background : Object` 背景
    - Color
    - Texture
    - CubeTexture
- `backgroundBlurriness : Float` 背景模糊度
    - 0-1
- `environment : Texture` 场景通用环境贴图
- `fog:Fog` 雾
    - Fog
- `isScene : Boolean` 是否为场景
- `overrideMaterial : Material` 场景通用材质

### Fog-雾

#### 属性

- `isFog : Boolean` 是否为雾
- `name : String` 对象名称
- `color : Color` 雾的颜色
- `near : Float` 雾的最小距离
    - 默认1
- `far : Float` 雾的最大距离
    - 默认1000

### FogExp2-雾【指数升级】

#### 属性

- `isFogExp2 : Boolean` 是否为雾
- `name : String` 对象名称
- `color : Color` 雾的颜色
- `density : Float` 雾的密度增长速度
    - 默认0.00025

## 相机

### PerspectiveCamera-透视相机

#### 构造器

- `PerspectiveCamera( fov : Number, aspect : Number, near : Number, far : Number )`
    - fov 摄像机视锥体垂直视野角度
    - aspect 摄像机视锥体长宽比
    - near 摄像机视锥体近端面
    - far 摄像机视锥体远端面

#### 属性

### ArrayCamera-摄像机阵列

#### 属性
