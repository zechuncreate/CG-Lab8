杜秋扬202411180016

# SMPL 人体线性混合蒙皮 (Linear Blend Skinning, LBS) 手写复现

本仓库包含计算机图形学（Computer Graphics）课程关于人体三维重建与运动学流水线（Kinematic Pipeline）的实验项目。本实验深入解析并手动手写复现了著名的 **SMPL (Skinned Multi-Person Linear Model)** 模型的核心前向计算流水线，消除了对高层 API 前向传播的依赖，且手写计算结果与官方结果达到 **100% 绝对对齐（零误差）**。

---

## 📊 实验原理与四阶段流水线 (Pipeline)

SMPL 模型将人体表示为基础模板网格 $\bar{\mathbf{T}}$，并通过输入体型参数 $\vec{\beta}$ 和姿态参数 $\vec{\theta}$ 来驱动网格形变。本实验完整复现了其核心的四个数学形变阶段：

1. **阶段 A：基础模板与蒙皮权重 (Template Mesh & LBS Weights)**
   加载拥有 $V=6890$ 个顶点和 $F=13776$ 个三角面片的中性人体模板网格 $\bar{\mathbf{T}}$，并提取大小为 $[V, 24]$ 的骨骼蒙皮权重矩阵 $\mathcal{W}$（每个顶点受到 24 个骨骼关节的主导影响力）。
2. **阶段 B：形状混合与关节回归 (Shape Blend & Joint Regression)**
   注入 10 维的体型参数 $\vec{\beta}$，通过体型混合形状矩阵（Shape Blend Shapes）计算由于高矮胖瘦产生的顶点形变位移，生成校正后的网格 $\mathbf{T}_S$。随后，利用预设的线性回归矩阵 $J$ 从形变顶点中精准回归出 24 个骨骼关节的初始 3D 坐标。
3. **阶段 C：姿态校正混合形状 (Pose Blend Shapes)**
   输入姿态旋转参数 $\vec{\theta}$。由于关节在旋转时会发生拉伸或挤压（如手肘弯曲），为了消除传统 LBS 的“糖纸包装”和“关节塌陷”缺陷，通过 Pose Blend Shapes 计算出肌肉位移补偿量，生成最终准备进行刚体变换的顶点 $\mathbf{T}_P$。
4. **阶段 D：线性混合蒙皮 (Linear Blend Skinning, LBS)**
   应用正向运动学（Forward Kinematics），沿着人体骨骼层级树（Kinematic Tree）自底向上计算 24 个关节的全局刚体变换矩阵 $\mathbf{A}$。最后，结合蒙皮权重矩阵 $\mathcal{W}$ 对每个顶点进行加权线性矩阵变换，输出最终动态的人体 3D 动作网格。

---

## 📂 项目目录结构



```text
practice8/
├── main.py                          # 实验核心补全与运行脚本
├── SMPL_NEUTRAL.pkl                 # SMPL 官方中性人体模型数据文件
├── README.md                        # 项目说明文档 (本文件)
└── outputs/                         # 运行后自动生成的实验成果图目录
    ├── stage_a_template_weights.png # 阶段 A：模板网格与指定单关节权重热力图
    ├── stage_b_shaped_joints.png    # 阶段 B：体型混合后的网格与回归关节
    ├── stage_c_pose_offsets.png     # 阶段 C：肌肉拉伸位移模长热力图
    ├── stage_d_lbs_result.png       # 阶段 D：线性混合蒙皮最终动作结果网格
    ├── all_joint_weights.png        # 24个全关节主导权重的色彩斑斓分布图
    ├── comparison_grid.png          # 2x2 四阶段流水线高清对比大图
    └── summary.txt                  # 包含绝对误差数据、可直接提交的摘要
