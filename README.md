# 网址：
1. [EasyMocap](https://chingswy.github.io/easymocap-public-doc/install/install.html)
2. [Neural_body](https://github.com/zju3dv/neuralbody)
3. [Animatable_nerf](https://github.com/zju3dv/animatable_nerf)
# 前提：
- .26 服务器运行EasyMocap(/home/zhaozisong/EasyMocap/)时：
```
conda activate fenerf   #确保进入fenerf环境运行
export PYOPENGL_PLATFORM=osmesa
```
- 运行Animatable_nerf时：
 ```
conda activate animatable_nerf
```
# 步骤：
## 预估图片关键点信息 `(已经完成)`
[参考网址](https://chingswy.github.io/easymocap-public-doc/quickstart/keypoints.html#mediapipe)
推荐用Mediapipe:
```
#Run the detection of full body:
python3 apps/preprocess/extract_keypoints.py ${data} --mode mp-holistic
```
## EasyMocap预估SMPL参数 `(已经完成)`
1. 先用设置data路径
```
data=/home/zhaozisong/EasyMocap/data/new/
```
2. 然后运行
```
python3 apps/demo/mocap.py ${data} --work internet
```
得到`output-smpl-3d\`文件夹下的全部文件。
## 脚本提取预估的SMPL参数获取params和vertives
1. 用[脚本](https://github.com/zju3dv/neuralbody/blob/master/zju_smpl/easymocap_to_neuralbody.py)处理得到的`output-smpl-3d/smpl`文件夹下的.json文件，获取params和vertices
   ```
   #generate params and vertices
   python easymocap_to_neuralbody.py --input_dir {data_dir} --type vertices
   ```

    问题:
- 目前存在的问题是 脚本得到的params和vertices维度和最终需要的对应不上。
- 肉眼对比与numpy.shape和可运行的数据集完全一致
- params需要修改脚本的代码自己保存
- ```怀疑是代码脚本代码的问题```
## 脚本获取annots.npy
1. 用[脚本](https://github.com/zju3dv/neuralbody/blob/master/zju_smpl/easymocap_to_neuralbody.py)获取```annots.npy```
    ```
    #generate annots.npy
    python easymocap_to_neuralbody.py --input_dir {data_dir} --type annots
    ```
## 脚本获取lbs文件夹下的相关文件
1. 用[脚本a](https://github.com/zju3dv/animatable_nerf/blob/master/tools/custom_dataset/prepare_blend_weights.py)和[脚本b](https://github.com/zju3dv/animatable_nerf/blob/master/tools/custom_dataset/prepare_lbs_meta.py)来获取和lbs相关的.npy文件

      问题：

      - 也是因为params和vertices的问题处理不了
      - 需要的维度为 23x3x3
      - 输入的为   24x3x1


## 结束
---
