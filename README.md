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

3. 用[脚本a](https://github.com/zju3dv/animatable_nerf/blob/master/tools/custom_dataset/prepare_blend_weights.py)和[脚本b](https://github.com/zju3dv/animatable_nerf/blob/master/tools/custom_dataset/prepare_lbs_meta.py)来获取和lbs相关的.npy文件

      问题：

      - 也是因为params和vertices的问题处理不了
      - 需要的维度为 23x3x3
      - 得到的为    24x3x1


## 结束
---
# 问题：










---

### Maintenance mode

I've lost interest/time in developing this further, other things have taken priority for some time now. However, all is not lost. I will be willing to review/comment on any issues/PRs but will not complete any fixes or feature requests myself. 

---

#### Purpose
Dynamically identify the suggested number of clusters in a data-set
using the gap statistic.

---

### Full example available in a notebook [HERE](Example.ipynb)

---
#### Install:  
Bleeding edge:
```commandline
pip install git+git://github.com/milesgranger/gap_statistic.git
```

PyPi:  
```commandline
pip install --upgrade gap-stat
```

With Rust extension:
```commandline
pip install --upgrade gap-stat[rust]
```


---
#### Uninstall:
```commandline
pip uninstall gap-stat
```

---

### Methodology:

This package provides several methods to assist in choosing the optimal number of clusters for a given dataset, based on the Gap method presented in ["Estimating the number of clusters in a data set via the gap statistic"](https://statweb.stanford.edu/~gwalther/gap) (Tibshirani et al.).

The methods implemented can cluster a given dataset using a range of provided k values, and provide you with statistics that can help in choosing the right number of clusters for your dataset. Three possible methods are:

  - Taking the `k` maximizing the Gap value, which is calculated for each `k`. This, however, might not always be possible, as for many datasets this value is monotonically increasing or decreasing.
  - Taking the smallest `k` such that Gap(k) >= Gap(k+1) - s(k+1). This is the method suggested in Tibshirani et al. (consult the paper for details). The measure `diff = Gap(k) - Gap(k+1) + s(k+1)` is calculated for each `k`; the parallel here, then, is to take the smallest `k` for which `diff` is positive. Note that in some cases this can be true for the entire range of `k`.
  - Taking the `k` maximizing the Gap\* value, an alternative measure suggested in ["A comparison of Gap statistic definitions with and
with-out logarithm function"](https://core.ac.uk/download/pdf/12172514.pdf) by Mohajer, Englmeier and Schmid. The authors claim this measure avoids the over-estimation of the number of clusters from which the original Gap statistics suffers, and can also suggest an optimal value for k for cases in which Gap cannot. They do warn, however, that the original Gap statistic performs better than Gap\* in the case of overlapped clusters, due to its tendency to overestimate the number of clusters.

Note that none of the above methods is guaranteed to find an optimal value for `k`, and that they often contradict one another. Rather, they can provide more information on which to base your choice of `k`, which should take numerous other factors into account.

---

### Use:

First, construct an `OptimalK` object. Optional intialization parameters are:

  - `n_jobs` - Splits computation into this number of parallel jobs. Requires choosing a parallel backend.
  - `parallel_backend` - Possible values are `joblib`, `rust` or `multiprocessing` for the built-in Python backend. If `parallel_backend == 'rust'` it will use all cores.
  - `clusterer` - Takes a custom clusterer function to be used when clustering. See the example notebook for more details.
  - `clusterer_kwargs` - Any keyword arguments to be forwarded to the custom clusterer function on each call.

An example intialization:
```python
optimalK = OptimalK(n_jobs=4, parallel_backend='joblib')
```


After the object is created, it can be called like a function, and provided with a dataset for which the optimal K is found and returned. Parameters are:

  - `X` - A pandas dataframe or numpy array of data points of shape `(n_samples, n_features)`.
  - `n_refs` - The number of random reference data sets to use as inertia reference to actual data. Optional.
  - `cluster_array` - A 1-dimensional iterable of integers; each representing `n_clusters` to try on the data. Optional.

For example:
```python
import numpy as np
n_clusters = optimalK(X, cluster_array=np.arange(1, 15))
```

After performing the search procedure, a DataFrame of gap values and other usefull statistics for  each passed cluster count is now available as the `gap_df` attributre of the `OptimalK` object:

```python
optimalK.gap_df.head()
```

The columns of the dataframe are:

  - `n_clusters` - The number of clusters for which the statistics in this row were calculated.
  - `gap_value` - The Gap value for this `n`.
  - `gap*` - The Gap\* value for this `n`.
  - `ref_dispersion_std` - The standard deviation of the reference distributions for this `n`.
  - `sk` - The standard error of the Gap statistic for this `n`.
  - `sk*` - The standard error of the Gap\* statistic for this `n`.
  - `diff` - The diff value for this `n` (see the methodology section for details).
  - `diff*` - The diff\* value for this `n` (corresponding to the diff value for Gap\*).


Additionally, the relation between the above measures and the number of clusters can be plotted by calling the `OptimalK.plot_results()` method (meant to be used inside a Jupyter Notebook or a similar IPython-based notebook), which prints four plots:

  - A plot of the Gap value versus n, the number of clusters.
  - A plot of diff versus n.
  - A plot of the Gap\* value versus n, the number of clusters.
  - A plot of the diff\* value versus n.

---
