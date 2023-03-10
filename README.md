<!-- <style>
H1{color:Blue !important;}
H2{color:DarkOrange !important;}
p{color:Black !important;}
</style> -->

CS 7641 Team 1

## Instruction-Tuned-Clinical-Notes-Scoring - Final Project Report
Shreyas Verma, Saksham Arora, Mehul Soni, Samaksh Gulati, Sindhu Raghuram Panyam

[Gantt Chart](https://docs.google.com/spreadsheets/d/127ndniyh15_XrMLIP_JnUK5plsJSzKy0/edit?usp=sharing&ouid=105287803295740692275&rtpof=true&sd=true)

[Proposal Video](https://drive.google.com/file/d/1Rn7oqu6s77t66SFFsS9U225rkypubJZz/view?usp=sharing)

## Introduction
Recently introduced instruction-paradigm empowers non-expert users to leverage NLP resources by defining a new task in natural language. Instruction-tuned models have the potential to significantly outperform task-specific language models (without instruction), but their effect has not yet been systematically studied in the healthcare domain. Also, Large Language models have been trained on a general English corpus and may not perform well on domain-specific domains which have a high proportion of rare words.

Our problem statement and methodology is  primarily driven by the advances in NLP that have created significant impact in healthcare for both patients and physicians.
With these guidelines, we narrowed down to a problem statement related to reducing the burden of conducting the United States Medical Licensing Examination taken by medical professionals each year. 

One phase of this Examination, namely Step 2 Clinical Skills involves test-takers interacting with Standardized Patients (people trained to portray specific clinical cases) and writing  patient notes. Trained physician raters later scored patient notes with rubrics that outlined each case’s important concepts (referred to as features).


[
<img width="813" alt="Screen Shot 2022-10-07 at 12 33 48 AM" src="https://user-images.githubusercontent.com/65465058/194675093-ab31dfe7-240f-4d50-9e4b-aa249b7c9d61.png">
](url)



Many techniques have been employed to tackle this problem statement. One such paper<sup>[6]</sup> tackles the task of domain specific passage matching by using a rule-based self-supervision objective for training Transformer language models. Upon extensive literature review, we realize that the following methodologies are interesting avenues that can be explored for our task :

1. <b>Effect of domain specific pretraining - </b>Large Language models, trained on general English corpus, might lead to an under-representation of domain-specific term embeddings in the corpus. Research in tweaking the pretraining phase focuses on task-adaptive pretraining of models like BERT using static word embeddings (fastText) in conjunction with different regularization techniques<sup>[3]</sup>, or framing subtasks<sup>[4]</sup> to explicitly learn the rare embeddings. Research in the fine-tuning phase has worked towards explaining rare words with paraphrases on the basis of prompt-based tuning techniques<sup>[5]</sup>. The former, albeit computationally expensive, is seemingly better - encouraging its use in representations of rare biomedical terminologies. 

2. <b>Instruction based fine tuning - </b>Recent works in prompting techniques<sup>[1][2]</sup> have explored multi-task instruction-based fine tuning. They demonstrate that instruction tuned models have a great ability to generalize and can perform arduous generative tasks with ease. Using that idea as an inspiration we propose instruction based fine-tuning. 

The dataset presents a corpus of 43,985 clinical patient notes (PNs) written by 35,156 examinees during the high-stakes USMLE® Step 2 Clinical Skills examination.



## Problem Definition

USMLE® Step 2 Clinical Skills examination is one of the most prestigious and rigorous medical licensure exams for medical professionals. The exam required test-takers to interact with Standardized Patients (people trained to portray specific clinical cases) and write a patient note. Trained physician raters later scored patient notes with rubrics that outlined each case’s important concepts (referred to as features). 

The goal of the project is to extract substrings from the patient notes that are semantically closer to the feature at hand using an instruction-based learning of a pre-trained language model. From a project point of view, our goal is to study the instructional learning task ( Instruction+1 static example) on Large Language Models of different sizes and compare results to identify optimal performance for our problem statement.


## Data Definition

The database presents a corpus of 43,985 clinical patient notes (PNs) written by 35,156 examinees during the high-stakes USMLE® Step 2 Clinical Skills examination. 
In medical education, students are often assessed through encounters with standardised patients - people trained to portray simulated scenarios called clinical cases. For each such encounter, the student is expected to perform a history and physical examination, determine differential diagnoses, and then document their findings in a PN.  

During this exam, each test taker sees a Standardised Patient, a person trained to portray a clinical case. After interacting with the patient, the test taker documents the relevant facts of the encounter in a patient note. Each patient note is scored by a trained physician who looks for the presence of certain key concepts or features relevant to the case as described in a rubric.

Clinical Case: The scenario (e.g., symptoms, complaints, concerns) the Standardised Patient presents to the test taker (medical student, resident or physician). Ten clinical cases are represented in this dataset.
Patient Note: Text detailing important information related by the patient during the encounter (physical exam and interview).
Feature: A clinically relevant concept. A rubric describes the key concepts relevant to each case.


## Data Processing
We used the following data preprocessing techniques
1. Created Input text using Patient Notes and Feature
2. Appended an Example to pass in along with an instruction (See Below)  
3. Added “No text found” for instances where the feature was not found in the Patient Note
4. We randomly split the dataset (14,300 Notes) into train and test splits as follows:
    80% - Training Set (11,400 Notes)
    20% - Testing Set (2,860 Notes)



<img src="https://user-images.githubusercontent.com/65465058/201565889-20929121-f57c-4774-8b36-7fbadf7ef5bf.png" width="500">

## Methods

When working on a specific domain, it is ideal to pre-train the LLM on rare words first and then perform the domain-specific task. At the outset, we began scoping different LLM models. From the plethora of options that HuggingFace provides with, we played around with the inferencing APIs of different models like BERT,bioBERT and T5-variants. We realise the diverse variants of T5-based models offer specific LLMs that are tuned for instruction-based tasks and are thus bound to perform marginally better than their counterparts. Thus, for this project, we plan to test two variants of the Tk-INstruct model - 
1. [Tk-Instruct Small](https://huggingface.co/allenai/tk-instruct-small-def-pos) - this is the smallest T5-based Instruction tuned model. The t5-small LLM has been tuned with an instructional dataset consisting of a definition and 2 positive examples to create this fine-tuned LLM version. The model consists of 60 million parameters and we chose to start with fine-tuning this first and then moving on to a bigger LLM for comparative analysis and benchmarking purposes.
2. [Tk-Instruct Base](https://huggingface.co/allenai/tk-instruct-base-def-pos) - Similar to the above model, the only difference is that this model has been trained on t5-base model which is a 220 million parameter model.

Our initial plan was to  pre-train the instruction-tuned [Tk-Instruct LLM](https://huggingface.co/allenai/tk-instruct-3b-def-pos-neg)<sup>[2]</sup> so that it learns the rare biomedical terms. The Masked Language Modelling (MLM) technique underlying pre-training a large language model is a semi-supervised approach that would help the model learn the domain-specific vocabulary. 

<img src="https://user-images.githubusercontent.com/65465058/194685081-51c1b248-27c9-4441-9b89-8eb3ee671fa3.png" width="500">

Unfortunately, due to compute resource constraints, we are unable to proceed with this task. However, we do fine-tune it using Instructional prompts composed of a definition and a positive example. As the LLM is itself a base T5 model tuned specifically for inteructional tasks, we hypothesise that passing simple elaborate instructions should work well for our use case even without the pretraining step. Also, since the output is an extraction task and not a generation task, the model should be able to extract relevant output phrases from the patient notes even without learing the domain-specific medical vocabulary.

We have thus trained the T5 model with ~13000 instances. The parameters we have used in the model are as follows:

<img src="https://user-images.githubusercontent.com/65465058/201564479-de72c6e0-15b1-4227-a0fc-c4726d074fde.png" width="500">


As stated in the data processing section, we have currently used a static example for all training inputs. In addition to the outputs of the model, we can extract the weights of the final layer that have been updated while training which can be reused.


### 2. Unsupervised Analysis - Topic Modeling

#### Background and Objective:
Medical Texts often contain a large amount of information pertaining to patient history, dated information, diagnosis and treatment records which are of high importance in decision making for doctors and other stakeholders in the healthcare ecosystem.

In the supervised learning approach discussed above, we deal with the patient history notes and automatically extract features using large language models. However, we can also often gain some useful insights from such data while solving the main objective which we focus on in this part of the project. We leverage an unsupervised technique based on Latent Dirichlet allocation-based Topic modeling to analyze and compare the content of clinical notes.


#### Approach

Topic Modeling is one way to perform EDA on textual data and derive relevant insights. The main purpose of this unsupervised statistical modeling algorithm is to understand the topics in input data and those topics help to analyze the context of the articles or documents. Additionally, this course of action will make use of the topics generated to assist with labeling the data in the subsequent steps of processing similar documents. Although Topic Modeling is not relevant to our task, it would be of key importance for downstream tasks such as clustering patients with similar health problems, semi-supervised classification tasks,etc.

Our focus in achieving this task is to use LDA or Latent Dirichlet Allocation. Latent Dirichlet Allocation is an unsupervised approach that identifies unobserved groups from a set of observations and helps us understand the presence of similar occurrences in the data. Treating the documents as a bag of words, the model assumes that the documents (statements) are composed of words that aid the process of topic recognition. Analogous documents are then mapped to the inventory  of topics chosen by creating an association between each word in the document and the various topics. The objective of executing this step is to leverage the imaginary topics in substantially capturing the words existing in the documents.


We followed the steps below to analyze our clinical notes corpus:

**1. Exploratory Data Analysis:**

Exploratory Data Analysis (EDA) refers to the process of performing initial explorations on data in order to uncover patterns, detect anomalies, test hypotheses and to check postulates using evaluation statistics and visualizations.

To begin with, we require a word cloud (a visualization of the information present in the data). The model uses textual data to achieve this. As a result of this, the popularity of prominent words and phrases are made evident where the sizes and weights of the words are positively associated with their popularities. It also helps us get a sense of the extent to which data pre-processing is required.

<img src="https://user-images.githubusercontent.com/114270661/205794571-45939bd8-6242-4c53-acfe-08ec46a5a21b.png" width="500">

The results obtained imply  higher measures of popularity associated with words such as “sexually active” and “chest pain.” We can also observe that the popularities of “ROS negative” and “drug use” are lesser in comparison with the words mentioned earlier. In our particular example, we know that majority of the patients that we are dealing with suffere from problems like chest pain, and are sexually active (reflecting a younger demographic).

**2. Data Preprocessing**

For any modeling exercise, it is important for the textual data to be clean and of relevant input type to get the required results. Hence,we performed some pre-processing steps on the clinical notes before proceeding with the LDA topic model.

These steps involved tokenization and removal of stop words from the patient history notes. This was followed by creating a dictionary of tokens from the data and a corpus of words in the text, using which we also calculated the term document frequency.

Gensim creates a unique id for each word in the document. Its mapping of word_id and word_frequency. Below is an example of how we assign the frequency to the unique id of each word in every patient note along with the corresponding words mapped to the IDs in this example:

\[(0, 'abdominal'), (1, 'abusing'), (2, 'active'), (3, 'adderall'), (4, 'adderrall'), (5, 'ago'), (6, 'attcak'), (7, 'beating'), (8, 'bowel'), (9, 'breath'), (10, 'changes'), (11, 'chest'), (12, 'chills'), (13, 'college'), (14, 'conciousness'), (15, 'condoms'), (16, 'dad'), (17, 'date'), (18, 'days'), (19, 'denies'), (20, 'diaphoresis'), (21, 'disease'), (22, 'drinks'), (23, 'endorses'), (24, 'episode'), (25, 'episodes'), (26, 'fatigue'), (27, 'felt'), (28, 'fevers'), (29, 'fhx')]

\[(0, 1), (1, 1), (2, 1), (3, 1), (4, 2), (5, 1), (6, 1), (7, 1), (8, 1), (9, 1), (10, 3), (11, 2), (12, 1), (13, 1), (14, 1), (15, 1), (16, 1), (17, 1), (18, 1), (19, 2), (20, 1), (21, 1), (22, 1), (23, 3), (24, 1), (25, 1), (26, 1), (27, 1), (28, 1), (29, 1)]

**3. LDA Model**

Once the textual data is transformed into a format that will serve as an input for training the LDA model, we can begin by tokenizing the text and removing stopwords. Tokenization is the process of breaking a stream of textual data into words, terms, sentences, symbols, or some other meaningful elements called tokens and  stop words are a set of commonly used words in any language. NLP uses stop words to value significant words and turn a blind eye to insignificant words. 

Once the tokenized object is converted into a corpus and a dictionary, we can proceed with the model training. The objective of topic modeling is to infer insights from the various topics entailed in data. In order to achieve this, the number of topics are allocated corresponding values. 

We tried to come up with the optimal values of the model hyperparameters to be able to capture the topic probabilities in the documents. We chose the number of topics as 7, with a chunk size of 50 (number of documents in each pass) and passes as 15.

In this case, each topic is a combination of keywords, and each keyword contributes a certain weightage to the topic.


#### Visualisation and Insights

Having obtained the trained model, we can visualize our findings to aid the process of interpretation -

We can see the first 5 topics that we obtained for the trained model:

<img src="https://user-images.githubusercontent.com/65465058/206098711-84f4b3a7-0ee6-423e-b1d9-6a79d44ff139.png" width="800">

The coefficients with keywords above correspond to the weightage contributed by the word to that topic.

<img src="https://user-images.githubusercontent.com/114270661/205795018-638d17e2-a70e-4d19-bf38-d1ab7c60dc74.png" width="800">

In order to better understand and interpret the individual topics, we can manually select each topic to view its top most frequent and/or “relevant” terms. This can help in attempting to assign a human interpretable name or “meaning” to each topic. Additionally, exploring the Intertopic Distance Plot can help you learn about how topics relate to each other, including potential higher-level structure between groups of topics.

For instance -

<img width="800" alt="LDA Results 4" src="https://user-images.githubusercontent.com/114270661/205815039-db3ae315-a86d-4df8-aa14-bc7b04e41265.png">

The results obtained in the illustrated plot seems to suggest that patients who use marijuana and tylenol tend to expeirnce head aches upong bending over. This suggests that these features can contribute to providing meaningful information when used in tandem with each other.

In this way, topic modeling can serve as a means of extracting meaningful information from textual data and support the insights derived from additional analysis obtained from using supervised models. 



## Evaluation:

### Large Language Model Evaluation:

As our task can be framed as a span detection task - where we extract a span of words from the patient note as output - we decided to use 2 primary metrics for our task. The Exact Match metric measures the percentage of predictions that match any one of the ground truth answers exactly. The F1 score metric is a loser metric that measures the average overlap between the prediction and ground truth answer. These scores are calculated on every prompt-response pair. Overall EM and F1 scores are computed for a model by averaging over the individual example scores. 

**Exact Match**:
For each prompt-response pair, the value of the Exact Match metric will be 1 when the characters of the model’s prediction and that of the ground truth match exactly. This metric is comparatively more conservative; if a match is not found, the value is assigned to be 0. A difference in one character would result in a score of 0.

**F1 Score**:
This metric is utilized when equal weightage is assigned to precision and recall. In this case, an attempt is made to find a match between the words in the prediction and the ground truth. 

We will be using a modified version of F1-score. This is because in our case we value recall more than precision. That is, it is important that in the output that the model gives us, the relevant feature text is present. However, we would not want to penalize the model as much for providing us with text in addition to the exact relevant phrase. Thus we will be using a Weighted F1-Score with greater emphasis on Recall rather than precision.

Having examined the resulting metrics, we get -

<img src="https://user-images.githubusercontent.com/65465058/205745072-bc75dd79-3d66-4054-9c60-2a8c1a43b9f8.jpeg" width="500">

From these results, it can be observed that a respectable F1 score has been obtained. This implies that our predictions are quite close to some of the True Answers. The results also suggest that there is an exact match found between the predictions and the ground truth to a certain extent. However, we can explore the potential for enhancing these scores by increasing the complexity of the model (using Tk-instruct base).

### Topic Model Evaluation:

In order to select the optimal number of topics, we evaluate our model on different values of our parameters using **Perplexity** and **Coherence** scores.


Iterating through the different range of topic values, we plot the perplexity scores and coherence scores, then find the sweet spot between the two metrics. The perplexity score is a measure of how surprised a model is when it comes to new data. The coherence score directly translates to human interpretability. The higher the coherence score, the better whereas we require a low perplexity score. 


We use the 'C_v' measure for coherence which is based on a sliding window, one-set segmentation of the top words and an indirect confirmation measure that uses normalized pointwise mutual information (NPMI) and the cosine similarity.



<img src="https://user-images.githubusercontent.com/65465058/206097312-7409559e-c46e-4f27-b402-9bb20aba9c7e.png" width="500">

<img src="https://user-images.githubusercontent.com/65465058/206097540-7ff907dd-fd4b-447c-96e1-d5ce80e998d2.png" width="500">


Through the above plots, we see that the perplexity scores have a slow decline with the number of topics and there is a sudden decline after 15 topics. The coherence plot shows an overall decline with few spikes. The selection for the number of topics as **7** is based on the value of coherence score of **0.595** and a perplexity score of **-7.38**. We desire a higher value of coherence rather than giving more focus to the perplexity score. The current scores indicate that our model is doing a fine job of identifying keywords and topics across the notes. However, there is the scope of improvement in these metrics which we can achieve using advanced data pre-processing techniques and also we can explore advanced unsupervised approaches to deal with clinical notes.

## Potential Results and Discussions:
As we aim to identify semantically similar phrases from the patient notes that match the medically termed features, we plan to use a micro-averaged F1 score and Exact Match score to evaluate the overlap of the predicted phrase spans with the span of the labels. As can be seen in the table above, the TK-instruct base model being a bigger model with 220 million parameters is better able to extract relevant phrases on mild fine-tuning when compared to the TK-instruct small model.

## References
1. Mihir Parmar, Swaroop Mishra, Mirali Purohit, Man Luo, Murad Mohammad, and Chitta Baral. 2022. In-BoXBART: Get Instructions into Biomedical Multi-Task Learning. In Findings of the Association for Computational Linguistics: NAACL 2022, pages 112–128, Seattle, United States. Association for Computational Linguistics.
2. Wang, Y., Mishra, S., Alipoormolabashi, P., Kordi, Y., Mirzaei, A., Arunkumar, A., Ashok, A., Dhanasekaran, A.S., Naik, A., Stap, D. and Pathak, E., 2022. Benchmarking generalization via in-context instructions on 1,600+ language tasks. arXiv preprint arXiv:2204.07705.
3. Kosuke Nishida, Kyosuke Nishida, and Sen Yoshida. 2021. Task-adaptive Pre-training of Language Models with Word Embedding Regularization. In Findings of the Association for Computational Linguistics: ACL-IJCNLP 2021, pages 4546–4553, Online. Association for Computational Linguistics.
4. Schick, T., & Schütze, H. (2020). Rare Words: A Major Problem for Contextualized Embeddings and How to Fix it by Attentive Mimicking. Proceedings of the AAAI Conference on Artificial Intelligence, 34(05), 8766-8774. https://doi.org/10.1609/aaai.v34i05.6403
5. Wang H, Liu C, Xi N, Zhao S, Ju M, Zhang S, Zhang Z, Zheng Y, Qin B, Liu T. Prompt Combines Paraphrase: Teaching Pre-trained Models to Understand Rare Biomedical Words. arXiv preprint arXiv:2209.06453. 2022 Sep 14.
6. Grundmann, Paul, Sebastian Arnold, and Alexander Löser. "Self-supervised answer retrieval on clinical notes." arXiv preprint arXiv:2108.00775 (2021).

