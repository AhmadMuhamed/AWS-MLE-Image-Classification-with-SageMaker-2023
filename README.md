# Image Classification using AWS SageMaker

In this project a pretrained RESNET50 is used and finetuned to fit CIFAR10 Dataset. As in hpo.py script, in net function, it was found that 
the number of features extracted are 2048, so three fully connected layers were added. One to reduce features to 1000, the second to reduce them to 100, and the third to reduce them to 10, which is the number of total classes. In the Dataset subsection, a detailed explanation for the dataset is provided.

HyperParameter tuner was used to tune the learning rate and epochs number. The best model was chosen to be trained using an estimator object where SageMkaer debugging and profiling was used to to track some events like vanishing gradient, overfitting, overtraining,poor weight initialization , checking if loss is not decreaing, plus some checks on CPU and GPU utilization during training.

This model is deployed  then to an endpoint and tested against some datapoints.

## Project Set Up and Installation

Cloning the GitRepo of the project https://github.com/udacity/CD0387-deep-learning-topics-within-computer-vision-nlp-project-starter.git, You will find it containing one Jupyter notebook train_and_deploy.ipynb and two python files, the first is for hyper parameter tuning hpo.py and the other is used to train the best model including profiling and debugging train_model.py.

It's important to make sure that the kernel used for the notebook is conda_amazonei_pytorch_latest_p37 to support pytorch libraries and modules.

Install smdebug using !pip install smdebug.

## Dataset
The used dataset is CIFAR10. The CIFAR-10 dataset consists of 60000 32x32 colour images in 10 classes, with 6000 images per class.
CIFAR10 dataset is used with 50000 training datapoints and 1000 datapoints.
Data labels are: airplane,automobile,bird,cat,deer,dog,frog,horse,ship, and truck

The dataset is divided into five training batches and one test batch, each with 10000 images. The test batch contains exactly 1000 randomly-selected images from each class. The training batches contain the remaining images in random order, but some training batches may contain more images from one class than another. Between them, the training batches contain exactly 5000 images from each class.

## Model used

ResNet50 was used for the project since it's a CNN used for Image Recognition that consists of (48 convolutional layers, one MaxPool layer, and one average pool layer). This architecture specifically is used since it, like ResNet34, supports more convolution layers without running into vanishing gradient problem because of using a skip-over connections, hince the name residual. What makes it different than ResNet34 is that it uses a bottleneck design for the building block which reduces the number of parameters and matrix multiplications. This enables much faster training of each layer. The attached article is a gentrle intro. about ResNet50.

### Access
Upload the data to an S3 bucket through the AWS Gateway so that SageMaker has access to the data. 

## Hyperparameter Tuning
Hyper parameters used were the learning rate that was allowed to have a continous value from 0.001 to 0.1 and the number of epochs that has catigorical values of 5,10,15,20. Four training jobs were perfprmed and completed successfukky with the following hyper parameters.



![HPO Successful 1](ScreenShots/HPO%20Successful%20.PNG)
![HPO Successful 2](ScreenShots/HPO%20Successful%204.PNG)

- First training job: lr = 0.03550200513396233, epochs=5 , Test LOss= 0.03319

![Tj1 logs](ScreenShots/t1logs.PNG)
![TJ1 Monitor](ScreenShots/t1%20monitor.PNG)


- Seconed training job: lr = 0.002520061454874928 , epochs=10,  Test LOss= 0.002199

![Tj2 logs](ScreenShots/t2logs.PNG)
![TJ2 Monitor](ScreenShots/t2%20monitor.PNG)


- Third training job: lr = 0.06752170730722379 , epochs=15, Test LOss= 0.09899

![Tj3 logs](ScreenShots/t3logs.PNG)
![TJ3 Monitor](ScreenShots/t3%20monitor.PNG)

 
- Fourth training job: lr = 0.003937592032800326 , epochs=20, Test LOss= 0.002499

![Tj4 logs](ScreenShots/t4logs.PNG)
![TJ4 Monitor](ScreenShots/t4%20monitor.PNG)


The best training job in terms of lowest test loss was the seconed training job where lr= 0.002520061454874928 and Number of epochs was 10.


## Debugging and Profiling

To perform Profiling and debugging in SageMaker, First I had to install smdebug inside the ipynb and set up debugging and profiling rules and hooks that will be used. I had to configure the debugger and profiler and add their configuration alongside the rules to the estimator object. In the training script, I initialized the hook and assigned the model to it, then inside the train and test function, I also set the ir modes to training and evaluation respectively.

After training job is finished, Profiler report was fetched from s3 pucket to the local directory and displayed inside the notebook for visualizing the results.

### Results

The insights that drew my attention inside the profiling report 
1 - First the CPU utilization. 

2 - As seen under Framework metrics summary, the most expensive operator on the CPUs was "aten::batch_norm" with 19 % consumption. Under the Rules summary, we can see how the model performed regarding the specified rules.For example, During my training job, the StepOutlier rule was the most frequently triggered. It processed 15715 datapoints and was triggered 93 times with average step duration of 0.19 seconed dor Evaluation phase.

3 - As seen from System usage statistics and rules summary, GPUs weren't used.

4 - As seen in Dataloading analysis, there was 4 CPU available cores, but one parallel processor was used for data loading. So, it's recommended to increase the number of dataloader workers. It's also recommended to use pinned memory that enables fast data transfer to CUDA-enabled GPUs.

5 - Other events like I/O bottleneck, CPU bottlenecks  and Batch Size were't tiggered.

## Model Deployment

First, to instanciate/create the endpoint, type and number of instances need to be specified.
Seconed, Data should be prepared in a way that the model can accept, in this nodel, it accepts tensors.
Third, A predictor can  be created and output size should be 10 indicating the 10 classes. 

Two Images from the training dataset were selected and passed to the endpoint. The first is of A "leptodactylus_pentadactylus" Which is a sort of frog and has a label of 6 and the output of the predictor showed that the 6th element has the highest softmax value. The seconed element name is camion which is a type of trucks that holds label 9 and the output of the predictor gives maximum softmax value for the ninth element in the output array. 
![Deployed Endpoint1](ScreenShots/Deployed%20Endpoint1.png)
![Deployed Endpoint2](ScreenShots/Deployed%20Endpoint2.PNG)
![EndpointLogs](ScreenShots/EndpointLogs.PNG)

     
     
## Standout Suggestions
This section is for further tasks in the future.

