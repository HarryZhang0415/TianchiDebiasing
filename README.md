# KDD Cup 2020 Challenges for Modern E-Commerce Platform: Debiasing

This is the competition repo. Team members are: 张涵，袁逸超，万道奇

## 1. Background

This track focuses on the [fariness of exposure](http://www.ec.tuwien.ac.at/~dimitris/research/recsys-fairness.html), i.e., how to recommend items that are rarely exposed in the past, to combat the Matthew effect frequently encountered in a recommender system. In particular, performing bias reduction when training on the click data is crucial for the success of this task. Just like there is a gap between the logged click data and the actual online environment in a modern recommender system, there will be a gap between the training data and the test data, mostly with respect to the trends and the items' popularity.

* The winning solutions need to perform well on the items that are historically rarely exposed.
* The training data and the test data are collected across many periods and includes even a massive sales campaign. Performing bias reduction is unavoidable for reliable prediction due to the shifts in trends.
* We provide multimodal features of the items as well as a few (anonymized) key user features, to help the participants explore solutions that are resilient to the data bias and capable of handling the under-explored items well.

## 2. Dataset

### 2.1 Data Download

The dataset download application link will be available for contestants after the registration for the competition. The dataset is composed of training and test set. The participants need to construct their own validation set.

The files are in CSV format, with UTF-8 encoding. The columns of the CSV files can be:

- item_id：the unique identifier of the item
- txt_vec：the item's text feature, which is a 128-dimensional real-valued vector produced by a pre-trained model
- img_vec：the item's image feature, which is a 128-dimensional real-valued vector produced by a pre-trained model
- user_id：the unique identifier of the user
- time：timestamp when the click event happens, i.e.,（unix_timestamp - random_number_1）/ random_number_2
- user_age_level：the age group to which the user belongs
- user_gender：the gender of the user, which can be empty
- user_city_level：the tier to which the user's city belongs

### 2.2 Files

The data is collected from more than ten days and includes a sales campaign. It involves more than 1 million clicks, 100k items, and 30k users. The total size of the dataset is around 500MB.

The training data：underexpose_train.zip

- It includes a file named underexpose_item_feat.csv, the columns of which are: item_id, txt_vec, img_vec
- It includes another file named underexpose_user_feat.csv, the columns of which are: user_id, user_age_level, user_gender, user_city_level
- It includes ten encrypted files whose names are in the form of underexpose_train_click-T.zip. Here T=0,1,2,…,9 means we are in phase T of the competition. We will release the password for underexpose_train_click-T.zip in the forum when the competition goes into phase T. The content of underexpose_train_click-T.zip is underexpose_train_click-T.csv, whose columns are: user_id, item_id, time

The test data：underexpose_test.zip

- It includes ten encrypted files whose names are in the form of underexpose_test_click-T.zip. Here T=0,1,2,…,9 means we are in phase T of the competition. We will release the password for underexpose_train_click-T.zip in the forum when the competition goes into phase T. The content of underexpose_test_click-T.zip is underexpose_test_click-T.csv and underexpose_test_qtime-T.csv.
- The columns of underexpose_test_click-T.csv are: user_id, item_id, time
- The columns of underexpose_test_qtime-T.csv are: user_id, query_time
  - Here query_time is the timestamp when the user clicks the next item. The task of this competition is to predict the next item clicked by each user that appeared in underexpose_test_qtime-T.csv. In particular, the participants need to recommend fifty items for each user. The participants will receive a positive score if any one of the fifty recommended items matches the ground-truth click.
  - We ensure that the ground-truth next item is in underexpose_item_feat.csv. However, there can zero observed clicks in the training data, albeit not very likely.

## 3. Submission

The participants need to submit their prediction for underexpose_test_qtime-0,1,2,…,T.csv when the competition is in phase T.

- Filename of the submission：underexpose_submit-T.csv
- The submitted file should be a CSV file with 51 columns. There is no need to include the headers, i.e., the names of the columns. The 51 columns of the submitted file should be:
  - user_id, item_id_01, item_id_02, …, item_50
  - Here item_id_01, item_id_02, …, item_id_50 is the fifty items recommended for user_id. The ordering of these fifty items matters. Please place the items that are deemed most likely to be clicked by the user in the front. In other words, item_id_01 should be the most likely one.
  - We have ensured that each user_id will not appear in more than one phase. Therefore, you don't need to specify which phase each row of your submission is for.
- Be sure that you have read the official script for evalution, which is posted in the forum.

## 4. Evaluation

For this competition, we use NDCG@50 to measure the quality of the recommendation list.

- We will compute two metrics: NDCG@50-full and NDCG@50-rare.
  - NDCG@50-full is computed on the whole test set, i.e., all test cases in underexpose_test_qtime-T.csv.
  - NDCG@50-rare is computed on half of the test cases in underexpose_test_qtime-T.csv. The selected half includes cases whose next items to be predicted are less explored than the other half in the past training sets, i.e., underexpose_train_click-0.zip, underexpose_train_click-1.zip, …, underexpose_train_click-T.zip.
- Phase T=0,1,2,…,6 are for development. The final rank of the participants will be computed based on T=7,8,9.
  - NDCG@50-full of the winning teams need to be among the top 10% while achieving the best NDCG@50-rare among the qualified teams.

## 5. Code Standard

The contestants should submit the trained model, and the prediction program that can generate prediction results for **testing dataset B (i.e., phase 7,8,9)**. Please compress all the files in a `zip` format package, and the files in the package should be organized as follows:

### 5.1 Data folder `data/`

Data files are not required to be submitted by contestants. We will put all the datasets used in the competition (the filename is consistent with the official website) to the `data/` folder. **Note**: the `data/` folder will be emptied at the beginning and only contains the raw dataset.

An example is as follows:

```markdown
|-- data
	|-- underexpose_train
		|-- underexpose_user_feat.csv
		|-- underexpose_item_feat.csv
		|-- underexpose_train_click-0.csv
		|-- underexpose_train_click-1.csv
		|-- ...
		|-- underexpose_train_click-9.csv
	|-- underexpose_test
		|-- underexpose_test_click-0
			|-- underexpose_test_qtime-0.csv
			|-- underexpose_test_click-0.csv
		|-- underexpose_test_click-1
			|-- underexpose_test_qtime-1.csv
			|-- underexpose_test_click-1.csv
		|-- ...
		|-- underexpose_test_click-9
			|-- underexpose_test_qtime-9.csv
			|-- underexpose_test_click-9.csv
```

### 5.2 User data folder `user_data/`

The trained model or other intermediate files generated during prediction should be placed in this folder. The file names or subfolder names in `user_data/` is unconstrained.

An example is as follows:

```markdown
|-- user_data
	|-- model_data
		|-- model.dat
	|-- tmp_data
		|-- tmp.dat
```

### 5.3 Prediction result output folder `prediction_result/`

The produced prediction results by the submitted code should be placed in this folder, named as `result.csv`, which should reproduce your submission for phase 9.
Please keep the format of `result.csv` the same as required in the competition. **Note**: the `prediction_result/` folder will be emptied at the beginning.

An example is as follows:

```markdown
|-- prediction_result
	|-- result.csv
```

### 5.4 Code folder `code/`

- Please make sure that prediction results of **testing dataset B** can be produced by the submitted code, and all the source code used in the prediction step should be included in the submitted files.
- All dependencies (**Operating system, MATLAB/Python version, Python packages, TensorFlow, PyTorch, MXNet version, etc.**) must be demonstrated in README file.
- If GPU is used, please specify CUDA version, CUDNN version, etc. in README file.
- If deep neural network is used, please provide network configuration to ensure that the prediction results can be produced.
- If there are files that need to be compiled, please provide a script to compile.
- Please provide `main.py` or `main.sh` as the entrance of prediction program, make sure that the final results can be obtained and saved in the aforementioned `prediction_result/result.csv` file.
- Please use relative path when reading data, for example, `../data/XX`.

### 5.5 Appendix

An example of the submitted files.

```
project
	|--README.md
	|--data
	|--user_data
		|-- model_data
			|-- model.dat
	|--prediction_result
	|--code
		|-- main.py or main.sh
```

