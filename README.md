# data-preparation

this doc is aiming at describing the process of dealing with vast volume of raw data preparation for the downstream analytics.

## raw data pull from source
for customers from mainland China, Baidu Net Disk is highly probable to used to store the raw data. As a result, the first step in terms of data preparation is to pull the raw data that is stored on Baudi Net Disk to aws S3.

network route is designed as below to achieve an optimized route to transfer these data.
![image](https://user-images.githubusercontent.com/97269758/160224355-aadf9cbb-71e7-4b8e-8803-30b974e48e43.png)

It is essential to download these data from Baidu Netdisk manually to the windows EC2. After that, can use aws cli installed on windows EC2 to unzip the file in batch and push them to S3 bucket.

