## Motivation

Try to explore the influence of adding rate code and visit history dist'n info to pipeline v0 model with pointwise/pairwise/listwise algorithm.


## Definition

### New attributes

a. 'srch_rate_type' -- regroup ~3k Marriott rate code to 7 major groups

| Rate Type Regroup | Orig rate code | 
|-----------|---| 
| MM | 'MMF','MMP', 'MM4' | 
| MW | 'MRW','MW1' |
| AAA | AAA |
| GOV | GOV |
| S9R | S9R |
| STANDARD | NULL |
| OTHER | all other codes except the above |

b. user past visit tier dist'n -- We grab previous 1yr visit history for the 1wk users used for training and the dist'n is calculated by tier and it's stored like below

| pgm_freq_member_key | usr_vst_%_longer | usr_vst_%_luxury | usr_vst_%_other | usr_vst_%_premium | usr_vst_%_select |
|-----------|---|------|-----|---|-----|

c. final training/testing data orgnized as below

|    | search_id | srch_position | marsha | click_marsha | click_pts | srch_rate_grp | srch_len_stay | srch_book_wndw | srch_num_rooms | usr_stays_%_longer | … | usr_vst_%_luxury | … |
|----|-----------|---------------|--------|--------------|-----------|---------------|---------------|----------------|----------------|--------------------|---|------------------|---|
| 0  | 0         | 1             | FFATS  | FFATS        | 2         | MW            | 2             | 176            | 1              |                    |   |                  |   |
| 1  | 1         | 1             | ORFOC  |              |           | MW            | 2             | 176            | 1              |                    |   |                  |   |
| 2  | 1         | 2             | ORFVA  |              |           | MW            | 2             | 176            | 1              |                    |   |                  |   |
| 3  | 1         | 3             | ORFBO  | ORFBO        | 1         | MW            | 2             | 176            | 1              |                    |   |                  |   |
| 4  | 1         | 4             | ORFVS  |              |           | MW            | 2             | 176            | 1              |                    |   |                  |   |
| 5  | 1         | 5             | ORFFP  |              |           | MW            | 2             | 176            | 1              |                    |   |                  |   |
| 6  | 2         | 1             | WASHW  |              |           | MW            | 2             | 43             | 1              |                    |   |                  |   |
| 7  | 2         | 2             | WASWA  | WASWA        | 1         | MW            | 2             | 43             | 1              |                    |   |                  |   |
| 8  | 2         | 3             | WASNH  |              |           | MW            | 2             | 43             | 1              |                    |   |                  |   |
| 9  | 2         | 4             | WASMO  |              |           | MW            | 2             | 43             | 1              |                    |   |                  |   |
| 10 | 2         | 5             | WASMH  |              |           | MW            | 2             | 43             | 1              |                    |   |                  |   |
| 11 | 2         | 6             | WASGN  | WASGN        | 2         | MW            | 2             | 43             | 1              |                    |   |                  |   |
| …  |           |               |        |              |           |               |               |                |                |                    |   |                  |   |
| …  |           |               |        |              |           |               |               |                |                |                    |   |                  |   |

d. Attribute details

| attr level | details |
|----|----|
| user | stay dist'n, vist dist'n |
| hotel | marsha, m_adr_3mo, m_adr_9mo, m_satisfaction , m_avg_booking_window, m_avg_length_stay, m_redemption_category, m_ctr |
| search | srch_rate_grp, srch_rate_grp, srch_dayofwk, srch_len_stay, srch_book_wndw, srch_num_rooms, srch_x, srch_y, srch_z |

### Algorithms

#### [Introduction](]https://medium.com/@nikhilbd/pointwise-vs-pairwise-vs-listwise-learning-to-rank-80a8fe8fadfd)

At a high-level, pointwise, pairwise and listwise approaches differ in how many documents you consider at a time in your loss function when training your model.

##### Pointwise approaches

Pointwise approaches look at ```a single document``` at a time in the loss function. They essentially take a single document and train a classifier / regressor on it to predict how relevant it is for the current query. The final ranking is achieved by simply sorting the result list by these document scores. For pointwise approaches, the score for each document is independent of the other documents that are in the result list for the query.
All the standard regression and classification algorithms can be directly used for pointwise learning to rank.

##### Pairwise approaches

Pairwise approaches look at ```a pair of documents``` at a time in the loss function. 
Given a pair of documents, they try and come up with the optimal ordering for that pair and compare it to the ground truth. 
The goal for the ranker is to ```minimize the number of inversions in ranking``` i.e. cases where the pair of results are in the wrong order relative to the ground truth.
Pairwise approaches work better in practice than pointwise approaches because predicting relative order is closer to the nature of ranking than predicting class label or relevance score. Some of the most popular Learning to Rank algorithms like ```RankNet, LambdaRank and LambdaMART```are pairwise approaches.

##### Listwise approaches

Listwise approaches directly look at the ```entire list``` of documents and try to come up with the optimal ordering for it. 
There are 2 main sub-techniques for doing listwise Learning to Rank:

1). Direct optimization of IR measures such as NDCG. E.g. SoftRank, AdaRank
2). Minimize a loss function that is defined based on understanding the unique properties of the kind of ranking you are trying to achieve. E.g. ListNet, ListMLE
Listwise approaches can get fairly complex compared to pointwise or pairwise approaches.

#### Reference

[Learning to Rank: From Pairwise Approach to Listwise Approach](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-2007-40.pdf)

[From RankNet to LambdaRank to LambdaMART: An Overview](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/MSR-TR-2010-82.pdf)

[SoftRank: Optimising Non-Smooth Rank Metrics - Microsoft Research](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.114.4495&rep=rep1&type=pdf)

[AdaRank on Letor](https://www.microsoft.com/en-us/research/project/letor-learning-rank-information-retrieval/)


## Steps for model training and testing

### 1. Pull digital data

[pull_weekly_data.ipynb](https://git.marriott.com/CXAnalytics/Search-Result-Opt/blob/master/dc_test/src/pull_weekly_data.ipynb)

### 2. Pull hotel attribute data

[get_hotelattr_staydist_data.ipynb](https://git.marriott.com/CXAnalytics/Search-Result-Opt/blob/master/dc_test/src/get_hotelattr_staydist_data.ipynb)

### 3. Get visit history data

[get_vst_hist_data.ipynb](https://git.marriott.com/CXAnalytics/Search-Result-Opt/blob/master/dc_test/src/get_vst_hist_data.ipynb)

### 4. Combine and lean data

[clean_data.ipynb](https://git.marriott.com/CXAnalytics/Search-Result-Opt/blob/master/dc_test/src/clean_data.ipynb)

### 5. Train the model

with time holdout split 
[train_model_timeholdout.ipynb](https://git.marriott.com/CXAnalytics/Search-Result-Opt/blob/master/dc_test/src/train_model_timeholdout.ipynb)

### 6. Prediction on new data

[pred_on_newdata.ipynb](https://git.marriott.com/CXAnalytics/Search-Result-Opt/blob/master/dc_test/src/pred_on_newdata.ipynb)

## Model testing metrics

We used raw and simulated NDCG/lift to evaluate the model. [Notebooks for testing](https://git.marriott.com/CXAnalytics/Search-Result-Opt/blob/master/dc_test/src/model_test)

Explaination:
Raw: for each search_id, we got the predicted marsha list by sorting the corresponding prediction probability descendingly

Simulation: for each search_id, we kept the top 3 marsha from the original search result, and removed any marsha matched with those 3 in the predicted sorted marsha list,  then concatenate the top3 marsha with the remained marsha in the predicted sorted marsha list

For example (one search_id):

| Marsha list                              | position 1 | position 2 | position 3 | position 4 | position 5 | position 6 | position 7 |
|------------------------------------------|------------|------------|------------|------------|------------|------------|------------|
| Original (groundtruth)                   | B          | D          | L          | E          | K          | N          | G          |
| Raw predicted(ordered by pred prob desc) | E          | D          | G          | B          | N          | K          | L          |
| Simulated(removed matched top3 with GT)  | B          | D          | L          | E          | G          | N          | K          |
 
Then we calculated the relevance score for the raw predicted marsha list and simulated marsha list respectively by contrasting to the original marsha list to get NDCG and lift.


### 1. Compare different attribute combination

Pipeline v0 attributes defined as bsln_attr

cat_feats = ['marsha', 'srch_dayofwk','m_tier']

num_feats = ['srch_position', 'srch_len_stay', 'srch_book_wndw', 'srch_num_rooms', 'srch_x', 'srch_y',  'srch_z', 'm_adr_3mo', 'm_adr_9mo', 'm_satisfaction', 'm_avg_booking_window', 'm_avg_length_stay', 'm_redemption_category', 'm_ctr', 'usr_stays_%_longer', 'usr_stays_%_luxury', 'usr_stays_%_other', 'usr_stays_%_premium', 'usr_stays_%_select']

bsln_attr = cat_feats+num_feats

Based on this, we tested rate group and tier dist'n of 1yr prior visit history.

#### a. Treat each visit as binary

| Notebook                              | Full list          |                |              | Simulation         |                |              | Attributes |             |               |                 | Algo          | Training Sample Size | Validation Sample Size |
|---------------------------------------|--------------------|----------------|--------------|--------------------|----------------|--------------|------------|-------------|---------------|-----------------|---------------|----------------------|------------------------|
|                                       | Raw_Baseline _NDCG | Raw_Model_NDCG | Raw_Lift(%)* | Sim_Baseline _NDCG | Sim_Model_NDCG | Sim_Lift(%)* | bsln_attr  | stay_dist'n | srch_rate_grp | vst_hist_dist'n |               |                      |                        |
| train_model_timeholdout_pipeline_v0   | 0.529281           | 0.555155       | 4.889        | 0.244596           | 0.264569       | 8.166        | x          | x           |               |                 | XGB Pointwise | 398,643              | 78,190                 |
| search_opt_pipeline_v0-stay_ratetype  | 0.529281           | 0.557892       | 5.406        | 0.244596           | 0.267059       | 9.184        | x          | x           | x             |                 | XGB Pointwise | 398,643              | 78,190                 |
| train_model_timeholdout_vst_hist_only | 0.529281           | 0.558669       | 5.552        | 0.245569           | 0.266479       | 8.515        | x          |             | x             | x               | XGB Pointwise | 398,643              | 78,190                 |
| search_opt_pipeline_v0_vst_hist_stay  | 0.529281           | 0.557760       | 5.381        | 0.244596           | 0.267758       | 9.469        | x          | x           | x             | x               | XGB Pointwise | 398,643              | 78,190                 |


#### b. Consider each pgvw of a visit

| Notebook                               | Full list          |                |              | Simulation         |                |              | Attributes |             |               |                 | Algo          | Training Sample Size | Validation Sample Size |
|----------------------------------------|--------------------|----------------|--------------|--------------------|----------------|--------------|------------|-------------|---------------|-----------------|---------------|----------------------|------------------------|
|                                        | Raw_Baseline _NDCG | Raw_Model_NDCG | Raw_Lift(%)* | Sim_Baseline _NDCG | Sim_Model_NDCG | Sim_Lift(%)* | bsln_attr  | stay_dist'n | srch_rate_grp | vst_hist_dist'n |               |                      |                        |
| train_model_timeholdout_pipeline_v0    | 0.529281           | 0.555155       | 4.889        | 0.244596           | 0.264569       | 8.166        | x          | x           |               |                 | XGB Pointwise | 398,643              | 78,190                 |
| train_model_timeholdout_pgvw_hist_only | 0.529281           | 0.558732       | 5.564        | 0.244596           | 0.266657       | 9.019        | x          |             | x             | x               | XGB Pointwise | 398,643              | 78,190                 |
| train_model_timeholdout_pgvw_hist_stay | 0.529281           | 0.560262       | 5.853        | 0.244596           | 0.268141       | 9.626        | x          | x           | x             | x               | XGB Pointwise | 398,643              | 78,190                 |


### 2. Compare different algorithms (XGB pointwise/pairweise/listwise)

1wk training data (20200226-20200303)

attr: bsln_attr+rate_type_grp+vis_hist 

#### a. Treat each visit as binary

| Notebook                                   | Full list     |            |         | Simulation    |            |         | Algo          | Training Sample Size | Validation Sample Size |
|--------------------------------------------|---------------|------------|---------|---------------|------------|---------|---------------|----------------------|------------------------|
|                                            | Baseline NDCG | Model NDCG | Lift(%) | Baseline NDCG | Model NDCG | Lift(%) |               |                      |                        |
| search_opt_pipeline_v0_vst_hist_stay-new       | 0.529281      | 0.557760   | 5.381   | 0.244596      | 0.267758   | 9.469   | XGB Pointwise | 398643              | 78190                 |
| main_xgb_ID13_rategrp_vsthist-pairwise-new | 0.529281      | 0.557460   | 5.324   | 0.244596      | 0.268773   | 9.884   | XGB Pairwise  | 398643              | 78190                 |
| main_xgb_ID13_rategrp_vsthist-listwise-new | 0.529281      | 0.558526   | 5.525   | 0.244596      | 0.268734   | 9.869   | XGB Listwise  | 398643              | 78190                 |


#### b. Consider each pgvw of a visit

| Notebook                                                 | Full list     |            |         | Simulation    |            |         | Algo          | Training Sample Size | Validation Sample Size |
|----------------------------------------------------------|---------------|------------|---------|---------------|------------|---------|---------------|----------------------|------------------------|
|                                                          | Baseline NDCG | Model NDCG | Lift(%) | Baseline NDCG | Model NDCG | Lift(%) |               |                      |                        |
| train_model_timeholdout_pgvw_hist_stay                   | 0.529281      | 0.560262   | 5.853   | 0.244596      | 0.268141   | 9.626   | XGB Pointwise | 398,643              | 78,190                 |
| main_xgb_ID13_rategrp_vsthist_pgvw-pairwise-listwise-new | 0.529281      | 0.558140   | 5.452   | 0.244596      | 0.269363   | 10.126  | XGB Pairwise  | 398,643              | 78,190                 |
| main_xgb_ID13_rategrp_vsthist_pgvw-pairwise-listwise-new | 0.529281      | 0.558389   | 5.500   | 0.244596      | 0.268571   | 9.802   | XGB Listwise  | 398,643              | 78,190                 |

### 3. Compare different size of training data (prediction power)

#### a. Trained on 1wk data (20200226-20200302) with XGB pointwise with all attributes

| Date_range | Usage       | Num(search_id) | Raw_Baseline _NDCG | Raw_Model_NDCG | Raw_Lift(%)* | Sim_Baseline _NDCG | Sim_Model_NDCG | Sim_Lift(%)* |
|------------|-------------|----------------|--------------------|----------------|--------------|--------------------|----------------|--------------|
| 0226-0302  | trainingset | 446737        | 0.52928            | 0.55776        | 5.381        | 0.244596           | 0.267758       | 9.469        |
| 0303-0310  | testset     | 444872        | 0.522344           | 0.550355       | 5.363        | 0.246672           | 0.269579       | 9.286        |
| 0311-0317  | testset     | 272849        | 0.515567           | 0.544843       | 5.678        | 0.250254           | 0.274396       | 9.647        |
| 0303-0317  | testset     | 717721        | 0.519705         | 0.5482086      | 5.485        | 0.248066         | 0.271454    | 9.428        |

#### b. Trained on 2Day (20200301-20200302) data with XGB pointwise with all attributes

Validated on 20200303, Test on 20200311-20200317

| Date_range        | Usage        | Num(search_id)           | Num(obs)                    | Raw_Baseline _NDCG | Raw_Model_NDCG | Raw_Lift(%) | Sim_Baseline _NDCG | Sim_Model_NDCG | Sim_Lift(%) |
|-------------------|--------------|--------------------------|-----------------------------|--------------------|----------------|-------------|--------------------|----------------|-------------|
| 20200301-20200303 | trainset(*)  | train/valid=133040/73487 | train/valid=3957988/2206280 | 0.529281           | 0.55392        | 4.655       | 0.244596           | 0.264277       | 8.046       |
| 20200311-20200317 | testset(*)   | 272849                  | 8012045                     | 0.515567           | 0.539723       | 4.685       | 0.250254           | 0.270028       | 7.902       |
|                   |              |                          |                             |                    |                |             |                    |                |             |
| 20200301-20200303 | trainset(**) | train/valid=133040/73487 | train/valid=3957988/2206280 | 0.529281           | 0.551068       | 4.116       | 0.244596           | 0.262496       | 7.318       |
| 20200311-20200317 | testset(**)  | 272849                  | 8012045                     | 0.515567           | 0.535691       | 3.903       | 0.250254           | 0.266239       | 6.388       |


*early_stopping_round=20, colsample_bytree= 1,  subsample=1

**early_stopping_round=20, colsample_bytree=0.8 ,  subsample=0.6


## Results

### Common important features from various model testing

srch_book_wndw, srch_position, usr_stay%, usr_vst%, srch_len_stay, srch _y/x/z, MW, MM

### Model perfomance on new data

Model trained using 1wk user data with XGB pointwise method performs consistent on training and testing on a new week or two weeks combined;

Model trained using 2day user data with XGB pointwise method performs worse on a new week testing data than traingset.


## ToDo

### Try embedding of marsha code instead of one hot encoding

### Try NN
