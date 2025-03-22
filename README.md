# DeepFlexTab

This repository holds the code for the paper

**DeepFlexTab: Deep flexible-universal paradigm for free tabular data**

All the materials released in this library can **ONLY** be used for **RESEARCH** purposes and not for commercial use.

The authors' institution (School of Biomedical Engineering, Shanghai Jiao Tong University) preserve the copyright and all legal rights of these codes.

# Author List

Rui Guo, Xinlu Tang, and Xiaohua Qian

# Abstract

Free tabular data, a fundamental information medium for decision-making, faces a critical bottleneck for flexible analysis in real-world open scenarios. Although recently popularized pre-trained foundation models that leverage external knowledge for cross-task universality have been extended to tabular data, they overlooked its unique task-sensitive nature, remaining constrained by the flexibility bottleneck. The challenge lies in the gap between unavoidable non-uniform attributes and the uniformity assumption underlying idealized research. Inspired by human cognitive evolution, we propose DeepFlexTab, a deep flexible-universal paradigm for directly analyzing free tabular data. DeepFlexTab extracts invariant universal knowledge across diverse free-attribute patterns through meta-learning while capturing dynamic inter-sample relationships through graph neural networks, mirroring the expertise progression from junior to senior specialist. Extensive experiments demonstrated the overwhelming universality of DeepFlexTab across extreme conditions common in real world, including highly free tables and diverse (even unseen) free-attribute patterns, especially outperforming a tabular foundation model and large language models. Overall, DeepFlexTab represents a profound transformation from rigidly enforcing unification towards directly embracing non-uniformity to mimic human flexibility, establishing a new paradigm for free tabular data analysis in real-world open scenarios.

# Requirements

Our code is mainly based on **Python 3.6** and **PyTorch 1.2.0**. The corresponding environment may be created via conda as follows:

```shell
conda env create -f ./environment/torch1.2.0_env.yaml
conda activate torch1.2.0_env
```

# Raw Data

Raw data contain the following two datasets.

1. TADPOLE
   
   Details of TADPOLE challenge data are available at https://tadpole.grand-challenge.org. The processed data can be downloaded from the official website of ADNI at http://adni.loni.usc.edu/. ADNI requires a user to register and request data with justification of data use.
   
   Details:
   
   - Log in ADNI database
   
   - Access the webpage: https://ida.loni.usc.edu/pages/access/studyData.jsp?categoryId=43&subCategoryId=94
   
   - Click on "Tadpole Challenge Data" button and get a package named "tadpole_challenge_201911210.zip"
   
   - Unzip it and copy the file named `TADPOLE_D1_D2.csv` to `./raw_data/TADPOLE`

2. UCI
   
   Eight UCI datasets are publicly available in the UCI Machine Learning Repository at https://archive.ics.uci.edu/ml/datasets/ without registration or request. 
   
   The source websites of these datasets are listed in Supplementary Table 8.
   
   Below are the folder names corresponding to these datasets.
   
   - BC -- Breast Cancer
   
   - CC -- Cervical Cancer
   
   - CK -- Chronic Kidney Disease
   
   - HC -- Horse Colic
   
   - HD -- heart-disease
   
   - HP -- Hepatitis
   
   - HS -- hcc-survival
   
   - PI -- pima-indians-diabetes

# Example

Use the first naturally free tabular TADPOLE dataset as an example to demonstrate the usage of our code.

```shell
conda activate torch1.2.0_env
cd ./DeepFlexTab/
bash example_nat1.sh
```

# Data processing

1. Simulated datasets from TADPOLE
   
   These datasets have been generated and saved in the package `./data/simu`.
   
   ```shell
   # generate data csv for experiments from raw database
   cd ./preprocessing/
   python data_generate_TADPOLE.py --data_root ./../data/ --data_name simu
   
   # get 1 split for 5-CV
   python data_process_simu.py --data_root ./../data/ --lost_num 0 --rand 1
   python data_process_simu.py --data_root ./../data/ --mode MCAR --lost_num 663 --p_inc 0.05 --rand 1
   
   # repeat 10 times (10 splits with 5-CV)
   cd ./reproducibility/
   python data_process_batch.py --data_root ./../data/ --data_name full --job 5
   python data_process_batch.py --data_root ./../data/ --data_name MCAR --job 5
   ```

2. Naturally incomplete datasets from TADPOLE
   
   ```shell
   # generate data csv for experiments from raw database
   cd ./preprocessing/
   python data_generate_TADPOLE.py --data_root ./../data/ --data_name nat1
   python data_generate_TADPOLE.py --data_root ./../data/ --data_name nat2
   
   # get missing patterns and statistics
   python refine_miss.py --data_root ./../data/ --data_name nat1
   python refine_miss.py --data_root ./../data/ --data_name nat2
   
   # get 1 split for 5-CV
   python data_process_nat.py --data_root ./../data/ --data_name nat1 --rand 1
   python data_process_nat.py --data_root ./../data/ --data_name natme nat2 --job 5
   
   # repeat 10 times (10 splits with 5-CV)
   cd ./reproducibility/
   python data_process_batch.py --data_root ./../data/ --data_name nat1 --job 5
   python data_process_batch.py --data_root ./../data/ --data_name nat2 --job 5
   ```

3. Naturally incomplete datasets from UCI
   
   ```shell
   # generate data csv for experiments from raw database
   cd ./preprocessing/
   python data_generate_UCI.py --data_root ./../data/ --data_name BC
   
   # get 1 split for 5-CV
   python data_process_uci.py --data_root ./../data/ --data_name BC --rand 1
   
   # repeat 10 times (10 splits with 5-CV)
   cd ./reproducibility/
   python data_process_batch.py --data_root ./../data/ --data_name BC--job 5
   ```

# Running code

1. Simulated datasets from TADPOLE
   
   ```shell
   conda activate torch1.2.0_env
   # 1 fold
   cd ./DeepFlexTab/
   python main_simu.py --data_root ./../data/ --save_dir ./../results/ \
       --data_name MCAR --lost_num 663 --p_inc=0.05 \
       --rand 1 --fold 0 \
       --test_N_way 5 --train_N_way 5 --batch_size 20 --batch_size_test 20 \
       --dec_lr 500 --lr 0.01 --lambda1 0.05
   
   # repeat 10 times (10 splits with 5-CV)
   cd ./reproducibility/
   python Ours_batch.py --data_root ./../data/ --save_dir ./../results/ \
       --data_name MCAR --lost_name L663p0.05 --job 2
   ```

2. Naturally incomplete datasets from TADPOLE
   
   ```shell
   conda activate torch1.2.0_env
   # 1 fold
   cd ./DeepFlexTab
   python main_nat.py --data_root ./../data/ --save_dir ./../results/ \
       --data_name nat1 \
       --rand 1 --fold 0 \
       --test_N_way 5 --train_N_way 5 --batch_size 20 --batch_size_test 20 \
       --dec_lr 500 --lr 0.001 --lambda1 0.05
   
   # repeat 10 times (10 splits with 5-CV)
   cd ./../reproducibility/
   python Ours_batch.py --data_root ./../data/ --save_dir ./../results/ --data_name nat1 --job 2
   ```

3. Naturally incomplete datasets from UCI
   
   ```shell
   conda activate torch1.2.0_env
   # 1 fold
   cd ./DeepFlexTab/
   python main_uci.py --data_root ./../data/ --save_dir ./../results/ \
       --data_name BC \
       --rand 1 --fold 0 \
       --test_N_way 1 --train_N_way 1 --batch_size 20 --batch_size_test 20 \
       --dec_lr 500 --lr 0.0005 --lambda1 0.001
   
   # repeat 10 times (10 splits with 5-CV)
   cd ./reproducibility/
   python Ours_batch.py --data_root ./../data/ --save_dir ./../results/ \
       --data_name BC --job 4
   ```

# Contact

For any question, feel free to contact

> Xiaohua Qian: [xiaohua.qian@sjtu.edu.cn](mailto:xiaohua.qian@sjtu.edu.cn)
