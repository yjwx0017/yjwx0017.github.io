title: UE 资产命名规范
categories: 日志
tags: [虚幻引擎,UE]
date: 2024-01-13 10:28:00
---
[https://github.com/Allar/ue5-style-guide/][1]

## Most Common

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Level / Map             |            |            |
| Level (Persistent)      |            | _P         |
| Level (Audio)           |            | _Audio     |
| Level (Lighting)        |            | _Lighting  |
| Level (Geometry)        |            | _Geo       |
| Level (Gameplay)        |            | _Gameplay  |
| Blueprint               | BP_        |            |
| Material                | M_         |            |
| Static Mesh             | S_         |            |
| Skeletal Mesh           | SK_        |            |
| Texture                 | T_         | _?         |
| Particle System         | PS_        |            |
| Widget Blueprint        | WBP_       |            |

## 动画

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Aim Offset              | AO_        |            |
| Aim Offset 1D           | AO_        |            |
| Animation Blueprint     | ABP_       |            |
| Animation Composite     | AC_        |            |
| Animation Montage       | AM_        |            |
| Animation Sequence      | A_         |            |
| Blend Space             | BS_        |            |
| Blend Space 1D          | BS_        |            |
| Level Sequence          | LS_        |            |
| Morph Target            | MT_        |            |
| Paper Flipbook          | PFB_       |            |
| Rig                     | Rig_       |            |
| Skeletal Mesh           | SK_        |            |
| Skeleton                | SKEL_      |            |

<!--more-->

## AI

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| AI Controller           | AIC_       |            |
| Behavior Tree           | BT_        |            |
| Blackboard              | BB_        |            |
| Decorator               | BTDecorator_ |          |
| Service                 | BTService_ |            |
| Task                    | BTTask_    |            |
| Environment Query       | EQS_       |            |
| EnvQueryContext         | EQS_       | Context    |

## 蓝图

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Blueprint               | BP_        |            |
| Blueprint Component     | BP_        | Component  |
| Blueprint Function Library | BPFL_   |            |
| Blueprint Interface     | BPI_       |            |
| Blueprint Macro Library | BPML_      |            |
| Enumeration             | E          |            |
| Structure               | F or S     |            |
| Tutorial Blueprint      | TBP_       |            |
| Widget Blueprint        | WBP_       |            |

## 材质

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Material                      | M_         |            |
| Material (Post Process)       | PP_        |            |
| Material Function             | MF_        |            |
| Material Instance             | MI_        |            |
| Material Parameter Collection | MPC_       |            |
| Subsurface Profile            | SP_        |            |
| Physical Materials            | PM_        |            |
| Decal                         | M_, MI_    | _Decal     |

## 纹理

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Texture                 | T_         |            |
| Texture (Diffuse/Albedo/Base Color)| T_ | _D      |
| Texture (Normal)        | T_         | _N         |
| Texture (Roughness)     | T_         | _R         |
| Texture (Alpha/Opacity) | T_         | _A         |
| Texture (Ambient Occlusion) | T_     | _O         |
| Texture (Bump)          | T_         | _B         |
| Texture (Emissive)      | T_         | _E         |
| Texture (Mask)          | T_         | _M         |
| Texture (Specular)      | T_         | _S         |
| Texture (Metallic)      | T_         | _M         |
| Texture (Packed)        | T_         | _*         |
| Texture Cube            | TC_        |            |
| Media Texture           | MT_        |            |
| Render Target           | RT_        |            |
| Cube Render Target      | RTC_       |            |
| Texture Light Profile   | TLP        |            |

## 其他

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Animated Vector Field      | VFA_       |            |
| Camera Anim                | CA_        |            |
| Color Curve                | Curve_     | _Color     |
| Curve Table                | Curve_     | _Table     |
| Data Asset                 | *_         |            |
| Data Table                 | DT_        |            |
| Float Curve                | Curve_     | _Float     |
| Foliage Type               | FT_        |            |
| Force Feedback Effect      | FFE_       |            |
| Landscape Grass Type       | LG_        |            |
| Landscape Layer            | LL_        |            |
| Matinee Data               | Matinee_   |            |
| Media Player               | MP_        |            |
| File Media Source          | FMS_       |            |
| Object Library             | OL_        |            |
| Redirector                 |            |            |
| Sprite Sheet               | SS_        |            |
| Static Vector Field        | VF_        |            |
| Substance Graph Instance   | SGI_       |            |
| Substance Instance Factory | SIF_       |            |
| Touch Interface Setup      | TI_        |            |
| Vector Curve               | Curve_     | _Vector    |

## Paper 2D

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Paper Flipbook          | PFB_       |            |
| Sprite                  | SPR_       |            |
| Sprite Atlas Group      | SPRG_      |            |
| Tile Map                | TM_        |            |
| Tile Set                | TS_        |            |

## 物理

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Physical Material       | PM_        |            |
| Physics Asset           | PHYS_      |            |
| Destructible Mesh       | DM_        |            |

## 声音

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Dialogue Voice          | DV_        |            |
| Dialogue Wave           | DW_        |            |
| Media Sound Wave        | MSW_       |            |
| Reverb Effect           | Reverb_    |            |
| Sound Attenuation       | ATT_       |            |
| Sound Class             |            |            |
| Sound Concurrency       |            | _SC        |
| Sound Cue               | A_         | _Cue       |
| Sound Mix               | Mix_       |            |
| Sound Wave              | A_         |            |

## UI

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Font                    | Font_      |            |
| Slate Brush             | Brush_     |            |
| Slate Widget Style      | Style_     |            |
| Widget Blueprint        | WBP_       |            |

## Effects

| 资产类型 | 前缀 | 后缀 |
| --- | --- | --- |
| Particle System         | PS_        |            |
| Material (Post Process) | PP_        |            |


  [1]: https://github.com/Allar/ue5-style-guide/