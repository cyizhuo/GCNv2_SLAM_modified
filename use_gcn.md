
# use gcn

## 1. 下载

- <https://github.com/jiexiong2016/GCNv2_SLAM/tree/master>

## 2. 编译安装

### 2.1. sophus

- sudo apt install ros-melodic-sophus

### 2.2. pytorch

```bash
git clone --recursive -b v1.0.1 https://github.com/pytorch/pytorch
cd pytorch && mkdir build && cd build
python ../tools/build_libtorch.py
```

- `git clone` 需要耐心等待，即使用了 vpn 好几分钟才会有动静
- 先给整个 pytorch 权限 +x ， `sudo chmod -R +x pytorch/`
- `python ../tools/build_libtorch.py` 需要 vpn

```bash
File "./pytorch/aten/src/ATen/cwrap_parser.py", line 18, in parse
    declaration = yaml.load('\n'.join(declaration_lines))
TypeError: load() missing 1 required positional argument: 'Loader'
```

- `pyyaml` 版本问题，安装 `4.2b4` 版本或参考 <https://blog.csdn.net/qq_44824148/article/details/122337056>
  - python==3.9
  - pyyaml==4.2b4
  - 或者参考  <https://blog.csdn.net/qq_44824148/article/details/122337056>

### 2.3 gcn_slam

- 删除所有 `CMakeLists.txt` 里的 `-march=native`

- 修改 `build.sh`

```bash
mkdir build
cd build
# cmake .. -DCMAKE_BUILD_TYPE=Release -DTORCH_PATH=/home/t/Workspace/deps/pytorch/torch/share/cmake/Torch
cmake .. -DCMAKE_BUILD_TYPE=Release -DTORCH_PATH=./pytorch/torch/lib/tmp_install/share/cmake/Torch
make -j4
```

- Eigen 问题： Target links to target "Eigen3::Eigen" but the target was not found.
  - 注释掉原 cmakelists.txt 的 `# LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)` ， 让 cmake 自己找 eigen 库就行了

- 添加位姿话题发布：
  - set 并 include SOPHUS_INCLUDE_DIRS
  - find 并 link catkin

## 3、运行

- `run.sh` 中提到的 `associations.txt` 就是 `orb_slam3/Examples/RGB-D/associations/fr3_office.txt`
- `associations.txt` 路径错误的话， `rgbd_gcn.cc` `LoadImages()` 会没反应，已修改代码

```bash
FULL_RESOLUTION=1 GCN_PATH=gcn2_640x480_cpu.pt ./rgbd_gcn ../Vocabulary/GCNvoc.bin TUM3.yaml ./data_slam/tum/freiburg3/rgbd_dataset_freiburg3_long_office_household ./data_slam/tum/freiburg3/rgbd_dataset_freiburg3_long_office_household/associations.txt

GCN_PATH=gcn2_320x240_cpu.pt ./rgbd_gcn ../Vocabulary/GCNvoc.bin TUM3_small.yaml ./data_slam/tum/freiburg3/rgbd_dataset_freiburg3_long_office_household ./data_slam/tum/freiburg3/rgbd_dataset_freiburg3_long_office_household/associations.txt

GCN_PATH=gcn2_tiny_320x240_cpu.pt ./rgbd_gcn ../Vocabulary/GCNvoc.bin TUM3_small.yaml ./data_slam/tum/freiburg3/rgbd_dataset_freiburg3_long_office_household ./data_slam/tum/freiburg3/rgbd_dataset_freiburg3_long_office_household/associations.txt

NN_ONLY=1 USE_ORB=1 ./rgbd_gcn ../Vocabulary/ORBvoc.bin TUM3_small.yaml ./data_slam/tum/freiburg3/rgbd_dataset_freiburg3_long_office_household ./data_slam/tum/freiburg3/rgbd_dataset_freiburg3_long_office_household/associations.txt
```

```bash
terminate called after throwing an instance of 'c10::Error'
  what():  Cannot initialize CUDA without ATen_cuda library.
```

- cuda 相关报错需要改 gcn 代码和模型为 cpu 版本： <https://blog.csdn.net/qq_35942419/article/details/120556046> ， 模型中的 `model.json` 中也要把 `cuda:0` 替换成 `cpu`，然后可以成功加载模型运行

## 4、fps 测试

| 硬件 | 数据 | 模型 | fps |
| - | - | - | - |
| 笔记本 wsl | rgbd | gcn_full | 0.1 |
| 笔记本 wsl | rgbd | gcn | 1 |
| 笔记本 wsl | rgbd | gcn_tiny | 2-3 |
| 笔记本 wsl | rgbd | orb | 15 |
| 工控机 | rgbd | gcn_full | 0.7 |
| 工控机 | rgbd | gcn | 2 |
| 工控机 | rgbd | gcn_tiny | 3-4 |
| 工控机 | rgbd | orb | 19 |
