# Basic Codebase

This folder contains some basic files for studying 3D adversarial attack and defense in point cloud. We implement training and testing of victim models, several baseline attack and defense methods. This codebase is highly extendable. You can easily add more victim model architectures, attacks and defenses based on it.

## Preliminary

### Data Storage

In order to simplify the process of data loading, I convert all the data (e.g. point cloud, GT label, target label) into a NumPy npz file. For example, for the ModelNet40 npz file ```data/MN40_random_2048.npz```, it contains the following data:

- **train/test_pc**: point clouds for training and testing, shape [num_data, num_points, 3]
- **train/test_label**: ground-truth labels for training and testing data
- **target_label**: pre-assigned target labels for targeted attack

### Configurations

Some commonly-used settings are hard-coded in ```config.py```. For example, the pre-trained weights of the victim models, the batch size used in model evaluation and attacks. **Please change this file if you want to use your own settings.**

## Requirements

### Data Preparation

Please download the point cloud data used for training, testing and attacking [here](https://drive.google.com/file/d/1o47ZvVcNvwBGv55xibEFw6KAOLaRF1IF/view?usp=sharing). Uncompress it to ```data/```.

- ```MN40_random_2048.npz``` is randomly sampled ModelNet40 data, which has 2048 points per point cloud
- ```attack_data.npz``` is the data used in all the attacks, which contains only test data and each point cloud has 1024 points, the pre-assigned target label is also in it

### Pre-trained Victim Models

We provided the pre-trained weights for the victim models used in our experiments. Download from [here](https://drive.google.com/file/d/1n9bRWyjPWSMyQktodCP2fnKpmzDFEf3e/view?usp=sharing) and uncompress them into ```pretrain/```. You can also train your victim models by yourself and put them into the folder.

**Note that**, if you want to use your own model/weight, please modify the variable called 'BEST_WEIGHTS' in ```config.py```.

## Usage

We briefly introduce the commands for different tasks here. Please refer to ```command.txt``` for more details about the parameters in each commands.

### Training Victim Models

We provide four types victim models for the experiments, **PointNet, PointNet++, DGCNN and PointConv**, where *Single Scale Grouping (SSG)* is adopted for PointNet++ and PointConv.

#### Train on Clean Data

Train a model on the randomly sampled ModelNet40 dataset:

```shell
python baselines/train.py --model={$MODEL} --num_points=1024
```

This will train the model for 200 epochs using the Adam optimizer with learning rate starting from 1e-3 and cosine decay to 1e-5.

The pre-trained weights we provided using this command achieve slightly lower accuracy than reported in their paper. This is because we do not use any tricks (e.g. label smoothing in the official TensorFlow implementation of [DGCNN](https://github.com/WangYueFt/dgcnn/blob/master/tensorflow/models/dgcnn.py#L105)), and we train/test on randomly sampled point clouds while some official implementations are train/tested on Farthest Point Sampling (FPS) sampled point clouds.


## Attacks


We implement **Perturb**, **Add Point**, **kNN**, variants of **FGM**, **Drop** , **GeoA3**, **SIA**, **L3A**, **AOF** and **AdvPc** attack. The attack scripts are in ```attack_scripts/``` folder. Except for Drop method that cannot targetedly attack the victim model, we perform targeted attack according to pre-assigned and fixed target labels.

### Perturb
```shell
python baselines/attack_scripts/targeted_perturb_attack.py --model={$MODEL} --dataset={$DATASET} --batch_size={$batchsize}
```
### Add
```shell
python baselines/attack_scripts/targeted_add_attack.py --model={$MODEL} --dataset={$DATASET} --batch_size={$batchsize}
```

### KNN
```shell
python baselines/attack_scripts/targeted_knn_attack.py --model={$MODEL} --dataset={$DATASET} --batch_size={$batch_size}
```

### FGM
```shell
python baselines/attack_scripts/targeted_fgm_attack.py --model={$MODEL} --attack_type=FGM/IFGM/MIFGM/PGD --dataset={$DATASET} --batch_size={$batch_size}
```
### Drop
```shell
python baselines/attack_scripts/untargeted_drop_attack.py --model={$MODEL} --num_drop=100/200 --dataset={$DATASET} --batch_size={$batch_size}
```
### GEOA3
```shell
python baselines/attack_scripts/GAO.py --model={$MODEL} --batch_size={$batch_size}
```
### ADVPC
```shell
python baselines/attack_scripts/untargeted_advpc_attack.py --data_root=path/to/ori/data.npz --attack_data_root=path/to/attacked/data --model={$MODEL} --batch_size={$batch_size}
```

### SIA
```shell
python baselines/attack_scripts/SIA.py  --surrogate_model={$MODEL} --target_model={$MODEL}
```
### L3A
```shell
python baselines/attack_scripts/L3A_attack.py
```
### AOF
```shell
python baselines/attack_scripts/AOF.py -data_root=path/to/ori/data.npz --model={$MODEL} --batch_size={$batch_size}
```

### Defenses

We implement **SRS**, **SOR** and **DUP-Net** defense. To apply a defense, simply run

```shell
python baselines/defend_npz.py --data_root=path/to/adv_data.npz --defense=srs/sor/dup/''
```
Additionally, we also provide other defense methods, such as **IF-defense**, **adversarial training** and **LPF**, simply run 
The defense result will still be a NumPy npz file saved in the same directory as the adv_data.

#### IF-defense
```shell
cd ConvONet
python opt_defense.py --sample_npoint=1024 --train=False --rep_weight=500.0 -data_root=path/to/adv_data.npz
```

#### Adversarial-training 
```shell
python AT/adversarial_training.py --ori_data=path/to/ori_data.npz --def_data=path/to/defense_data.npz  --model={$MODEL} --batch_size={$Batchsize}
```
### Evaluating Victim Models

You can test the attacks/defenses performance using ```inference.py```:

```shell
python baselines/inference.py --model={$MODEL} --dataset={$DATASET} --data_root=path/to/test_data.npz
```

Please refer to ```command.txt``` for details about different parameters.

