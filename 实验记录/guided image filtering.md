# **6.9-6.15**周报

### 最近主要在尝试MR引导PET重建，改善PET成像质量

首先固定PET配准MRI，利用MRI引导PET重建，使用引导图像滤波[Guided image filtering, GIF]([Guided Image Filtering | IEEE Journals & Magazine | IEEE Xplore](https://ieeexplore.ieee.org/abstract/document/6319316/))，结果如下：

![image-20250612113203902](C:\code\knowhow-transfer\md_pics\image-20250612113203902.png)

![image-20250612142316229](C:\code\knowhow-transfer\md_pics\image-20250612142316229.png)



以上方法有几个问题：

1. GIF计算方式
   $$
   q=G(p, I)=a_kI+b_k
   $$

   - 计算后 $q$ 与 $I$ 接近，该计算方式是对PET图像SUV值的一种破坏，尽管肉眼看上去有解剖结构的对比度，但对SUV值有影响，进而影响诊断过程
   - GIF强制PET向MRI靠近，PET本身的结构没有被保留，例如眼球部分，
   - 半径 $r$ 控制平滑程度，$\epsilon$ 影响几乎没有，因为 $COV(PET, MRI)$ 巨大

2. 临床上阅片关心SUV绝对值还是不同区域之间的相对值，输出图像必须要具有临床意义

3. MR引导PET重建的方法做出的结果似乎只能定性不能定量分析，因为没有标签，没有一个评判标准，不知道得出的图像是好还是不好

针对以上问题作出的改进：

> **残差引导滤波**：使用 MR 引导预测出 PET 的“结构差异成分”，再将这部分加回到 PET 原始图像中。

$$
q=p+G(I, p-\bar p)
$$

- `p` 是原始 PET；

- `I` 是 MR 引导图；

- `p̄` 是 `p` 的模糊版（可用 box 或 Gaussian）；

- `G(I, r)` 是 Guided Filter，作用在残差 `r = p - p̄` 上；

- `q` 是输出图像，保留了 PET 的 SUV（来自 `p`），同时结构清晰（由 `G` 贡献）。

> **结果**：

![image-20250612190009786](C:\code\knowhow-transfer\md_pics\image-20250612190009786.png)

尽管提供残差有了保护边效果，但是还是太像MR了，而且强度分布还是受到了破坏，因为初始的PET强度和MR相差就很大，且残差项有可能出现无意义的负数。其实本质还是因为线性模型没有办法很好的刻画PET和MR之间的联系，GIF最大的用途还是用于同一图像的滤波去噪





**所以GIF这种线性模型可以不用考虑了，考虑一些利用迭代展开、字典学习的方式，以期通过非线性的的方式刻画两种模态之间的关系**

- **正则项**
  $$
  \min_{q}\ \underbrace{\|q-p\|^{2}}_{\text{保持}\operatorname{SUV}\text{强度}}+\lambda\underset{\text{去噪}/\text{边缘保留}}{\underbrace{TV(q)}}+\mu\underset{\text{结构引导}: \ \text{边缘与}\operatorname{MR}\text{一致}}{\underbrace{\|\nabla q-\nabla I\|^{2}}}
  \\
  TV(q)=\sum_{x,y,z}^{} \left | \frac{\partial q}{\partial x}  \right | +\left | \frac{\partial q}{\partial y}  \right |+\left | \frac{\partial q}{\partial z}  \right |
  $$

  ```python
  import torch
  import torch.nn.functional as F
  import numpy as np
  import random
  
  def set_seed(seed=343):
      torch.manual_seed(seed)
      torch.cuda.manual_seed_all(seed)
      np.random.seed(seed)
      random.seed(seed)
      torch.backends.cudnn.deterministic = True
      torch.backends.cudnn.benchmark = False
  
  def total_variation_loss_3d(img): # 前向差分 (x(i+1) - x(i)) 
      dx = torch.abs(img[1:, :, :] - img[:-1, :, :])
      dy = torch.abs(img[:, 1:, :] - img[:, :-1, :])
      dz = torch.abs(img[:, :, 1:] - img[:, :, :-1])
      return dx.mean() + dy.mean() + dz.mean() # 不关心方向，只关心强度
  
  def compute_gradients_3d(img): # 中心差分 (x(i+1) - x(i-1))/2
      gx = img[2:, 1:-1, 1:-1] - img[:-2, 1:-1, 1:-1] # 其他方向保留中心区域，防止数组越界
      gy = img[1:-1, 2:, 1:-1] - img[1:-1, :-2, 1:-1]
      gz = img[1:-1, 1:-1, 2:] - img[1:-1, 1:-1, :-2]
      return gx / 2, gy / 2, gz / 2 # 结构对齐，不能合并
  
  def guided_pet_filtering_3d(pet_data, mr_data, lam, mu, steps=100, lr=1e-2):
      set_seed(343)
      device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
      pet_data = pet_data.float().to(device)
      mr_data = mr_data.float().to(device)
  
      # 归一化
      pet_data = (pet_data - pet_data.min()) / (pet_data.max() - pet_data.min() + 1e-8)
      mr_data = (mr_data - mr_data.min()) / (mr_data.max() - mr_data.min() + 1e-8)
  
      q = pet_data.clone().detach().requires_grad_(True).to(device)
  
      optimizer = torch.optim.Adam([q], lr=lr)
      last_loss = 0
      cnt = 0
      for i in range(steps):
          optimizer.zero_grad()
  
          loss_data = F.mse_loss(q, pet_data) # 第一项
          loss_tv = total_variation_loss_3d(q) # 第二项
  
          grad_qx, grad_qy, grad_qz = compute_gradients_3d(q)
          grad_Ix, grad_Iy, grad_Iz = compute_gradients_3d(mr_data) # 第三项，分别计算mri和pet
          loss_struct = F.mse_loss(grad_qx, grad_Ix) + F.mse_loss(grad_qy, grad_Iy) + F.mse_loss(grad_qz, grad_Iz)
          loss = loss_data + lam * loss_tv + mu * loss_struct
          loss.backward()
          optimizer.step()
          # early stop
          if i % 10 == 0:
              if loss == last_loss:
                  cnt = cnt + 1
              else:
                  cnt = 0
              last_loss = loss
              print(f"Step {i:03d} | Loss: {loss.item():.6f}")
              lr = lr / 1.5 
          if cnt == 10:
              print(f"\n Done:" f"Step {i:03d}")
              return q
  
      return q
  
  
  
  ```
  
  
  
  结果：似乎好很多，既有保变效果，又保存了PET的一些关键结构，然而这是固定PET配准MRI，MRI的结构信息是被压缩了的，可视化结果可能稍弱，当然参数要调一下
  
  ![image-20250613170006262](C:\code\knowhow-transfer\md_pics\image-20250613170006262.png)



​		如果固定MRI配准PET，分辨率肉眼可见会更高一点，结果如下：

![image-20250613173848854](C:\code\knowhow-transfer\md_pics\image-20250613173848854.png)



- **联合双边滤波**
  $$
  q(x)=\frac{1}{W(x)}\sum_{y\in\Omega}G_s(\|x-y\|)\cdot G_r(\|I(x)-I(y)\|)\cdot p(y)
  $$

  - $x(i,j)$：中心像素；$y(k,l)$：领域像素；邻域 $\Omega$ 半径自选

  - $G_s=exp(-\frac {d^2}{2\sigma_s^2})$：空间高斯核（由MR的坐标距离决定滤波范围）

  - $G_r=exp(-\frac {\nabla I}{2\sigma_s^2})$：强度高斯核（由PET的强度差异决定滤波权重）

  - $W(x)=\sum_{y\in\Omega}G_s(\|x-y\|)\cdot G_r(\|I(x)-I(y)\|)$：归一化因子
  
  - 可将将 $G_r$的强度计算替换为PET与MR的联合相似性
    $$
  G_r=\exp\left(-\frac{(p(x)-p(y))^2+\alpha(I(x)-I(y))^2}{2\sigma_r^2}\right)
    $$
  
  ```python
  def joint_bilateral_filter_torch(mr, pet, radius, sigma_s, sigma_r, alpha):
      device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
      
      pet = (pet - pet.min()) / (pet.max() - pet.min() + 1e-8)
      mr = (mr - mr.min()) / (mr.max() - mr.min() + 1e-8)
      pet = pet.to(device)  # shape: [H, W, D]
      mr = mr.to(device)
  
      grid_range = torch.arange(-radius, radius + 1, device=device)
      z, y, x = torch.meshgrid(grid_range, grid_range, grid_range, indexing='ij')
      G_s = torch.exp(-(x**2 + y**2 + z**2) / (2 * sigma_s**2))  # [K,K,K]
  
      padding = radius
      pet_p = F.pad(pet.unsqueeze(0).unsqueeze(0), (padding,)*6, mode='replicate').squeeze(0).squeeze(0)  # [H+2r,W+2r,D+2r]
      mr_p = F.pad(mr.unsqueeze(0).unsqueeze(0), (padding,)*6, mode='replicate').squeeze(0).squeeze(0)
  
      q = torch.zeros_like(pet)
  
      weight_sum = torch.zeros_like(pet)
      for dz in range(-radius, radius+1):
          for dy in range(-radius, radius+1):
              for dx in range(-radius, radius+1):
                  
                  shifted_pet = pet_p[ radius+dy:-radius+dy or None, radius+dx:-radius+dx or None, radius+dz:-radius+dz or None]
                  shifted_mr = mr_p[radius+dy:-radius+dy or None, radius+dx:-radius+dx or None, radius+dz:-radius+dz or None]
  
                  diff_pet = (shifted_pet - pet)**2
                  diff_mr = (shifted_mr - mr)**2
                  G_r = torch.exp(-(diff_pet + alpha * diff_mr) / (2 * sigma_r**2))
  
                  weight = G_r * G_s[dy+radius,dx+radius,dz+radius]
  
                  q += weight * shifted_pet
                  weight_sum = weight if (dx==dy==dz==0) else weight_sum + weight
  
      q /= weight_sum
      return q
  ```
  
  
  
  轻信了GPT，此方法不太行，代码实现的时候本质上还是线性变换模型，只适合去噪，结果如下：
  
  ![image-20250614170406906](C:\code\knowhow-transfer\md_pics\image-20250614170406906.png)
  
  

- **字典学习表示**

  - PET天然可以稀疏表示，可以利用字典学习的思想

  - 联合字典训练：从配对中提取Patch对$\{p_i,I_i\}$，学习联合字典$D=[D_p;D_I]$，使得$p_i\approx D_p\alpha_i$和$I_i\approx D_I\alpha_i$共享稀疏系数$\alpha_i$
  
  - 图像重建，用MR图像稀疏编码得到$\alpha_i$，再通过$D_p$重建PET：
    $$
  q_i=D_p\cdot\arg\min_\alpha\|I_i-D_I\alpha\|^2+\|\alpha\|_1
    $$
  
  **结果还在跑，暂时没有**



### 可以深入地方

1. 提供一套范式，比如用以上方式造PET-MR“合成”后的标签，再用深度学习进行网络监督，用在低剂量的场景，如下几篇文章，有类似范式：
   - [Pseudo-MRI-Guided PET Image Reconstruction Method Based on a Diffusion Probabilistic Model]([Pseudo-MRI-Guided PET Image Reconstruction Method Based on a Diffusion Probabilistic Model | IEEE Journals & Magazine | IEEE Xplore](https://ieeexplore.ieee.org/document/10843797))
   - [Score-Based Generative Models for PET Image Reconstruction]([MELBA – Score-Based Generative Models for PET Image Reconstruction](https://www.melba-journal.org/papers/2024:001.html))
   - [PET image denoising based on denoising diffusion probabilistic model]([PET image denoising based on denoising diffusion probabilistic model | European Journal of Nuclear Medicine and Molecular Imaging](https://link.springer.com/article/10.1007/s00259-023-06417-8))

2. PET-MR pair数量稀少和自动配准的问题，可以用GAN或diffusion生成，搭配ITK配准，建立一套完整的pipeline

3. 双域如何实施？

4. 如何设计定量指标，定量分析成像效果，调研发现目前没有人这样设计过，因为”合成“出来的图像没有标签

   - 例如合成后的图和原PET的suv分布相比偏差有多大？这个可以作为一个定量指标（那原始分布可不可以作为标签？）

   - ...

