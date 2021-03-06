# Quickstart Guide to Re-Training Models on AWS

This is a guide to getting AWS up and running so that we may re-train Tensorflow's models to work with the gathered traffic light data for Udacity's Capstone Integration Project.


This is based off of other Udacity student's TL Detection projects. You may find their projects at this location:
https://github.com/alex-lechner/Traffic-Light-Classification
https://github.com/swirlingsand/deeper-traffic-lights


-------------
### Getting your Term 3 AWS Credits from Udacity

As a reminder, you get another $100 credit for enrolling in term 3 in Udacity's SDC-ND. I assume you already have an AWS account, if not, go through the walkthrough in Term 1.

To get the credit, fill out the form located [here](https://www.awseducate.com/PromotionSignup?pcode=400HZJ).

### Setting up your AWS Instance

To set up an AWS spot instance do the following steps:
1. [Login to your Amazon AWS Account][aws login]
2. Navigate to ``EC2`` -> ``Instances`` -> ``Spot Requests`` -> ``Request Spot Instances``
3. Under ``AMI`` click on ``Search for AMI``, type ``udacity-carnd-advanced-deep-learning`` in the search field, choose ``Community AMIs`` from the drop-down and select the AMI (**This AMI is only available in US Regions so make sure you request a spot instance from there!**)
4. Delete the default instance type, click on ``Select`` and select the ``g2.xlarge`` instance
5. **IMPORTANT** Uncheck the ``Delete`` checkbox under ``EBS Volumes`` so your progress is not deleted when the instance get's terminated. (**CAEd: LESSON LEARNED: I didn't do this and lost a whole day's worth of trained data.**)
6. Set ``Security Groups`` to ``default``
7. Select your key pair under ``Key pair name`` (if you don't have one create a new key pair) **SAVE THIS PEM file and distribute it if you want to log into your instance from different computers**
8. At the very bottom set ``Request valid until`` to about **24 - 48 hours** and set ``Terminate instances at expiration`` as checked (You don't have to do this but keep in mind to receive a very large bill from AWS if you forget to terminate your spot instance because the default value for termination is set to 1 year.)
9. Click ``Launch``, wait until the instance is created and then connect to your instance via ssh

### Logging into your Instance


### Setting up your environment




---------------

```
cd tensorflow/models/research
protoc object_detection/protos/*.proto --python_out=.
export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
cd ../../..

```

```
python train.py --logtostderr \
      --train_dir=./models/ssd_inception_v2_coco_11_06_2017/ \
      --pipline_config_path=./config/ssd_inception_v2_coco.config
```
------------------------
## Commands to upload TF record files to aws

```
scp -i ~/capstone.pem \
  ~/Development/TL_model_make/data/bosch/bosch_train.record  \
  ubuntu@ec2-52-53-215-45.us-west-1.compute.amazonaws.com:~/data

scp -i ~/capstone.pem \
  ~/Development/TL_model_make/data/bosch/bosch_train_14labels.record \
  ubuntu@ec2-52-53-215-45.us-west-1.compute.amazonaws.com:~/data

scp -i ~/capstone.pem \
  ~/Development/TL_model_make/data/dataset-sdcnd-capstone/sim_data.record \
  ubuntu@ec2-52-53-215-45.us-west-1.compute.amazonaws.com:~/data

scp -i ~/capstone.pem \
  ~/Development/TL_model_make/data/dataset-sdcnd-capstone/real_data.record \
  ubuntu@ec2-52-53-215-45.us-west-1.compute.amazonaws.com:~/data

scp -i ~/capstone.pem \
    ~/Development/TL_model_make/data/udacity_label_map.pbtxt \
    ubuntu@ec2-52-53-215-45.us-west-1.compute.amazonaws.com:~/labels

scp -i ~/capstone.pem \
      ~/Development/TL_model_make/data/bosch_14label_map.pbtxt \
      ubuntu@ec2-52-53-215-45.us-west-1.compute.amazonaws.com:~/labels

scp -i ~/capstone.pem \
            ~/Development/TL_model_make/data/coco_label_map.pbtxt \
            ubuntu@ec2-52-53-215-45.us-west-1.compute.amazonaws.com:~/labels

scp -i ~/capstone.pem \
        ~/Development/TL_model_make/config/ssd_inception_v2_coco.config \
        ubuntu@ec2-52-53-215-45.us-west-1.compute.amazonaws.com:~/config/         


scp -i ~/capstone.pem \
    ubuntu@ec2-52-53-215-45.us-west-1.compute.amazonaws.com:~/.bashrc \
       ~/Development/TL_model_make\AWS_bash.txt
```
-----------------------
## labelImg
This is a tool to annotate imagery for the classifier. Tool may be found here:
https://github.com/tzutalin/labelImg

First you will need to check out the git repository


Then you will need to go to that repository directory


Then use the following command to create or run a docker container with the labelImg program
```
sudo docker run -it \
--user $(id -u) \
-e DISPLAY=unix$DISPLAY \
--workdir=$(pwd) \
--volume="/home/$USER:/home/$USER" \
--volume="/etc/group:/etc/group:ro" \
--volume="/etc/passwd:/etc/passwd:ro" \
--volume="/etc/shadow:/etc/shadow:ro" \
--volume="/etc/sudoers.d:/etc/sudoers.d:ro" \
-v /tmp/.X11-unix:/tmp/.X11-unix \
tzutalin/py2qt4
```

Then use the following command to create the program (you can skip this if you already done so)
```
make qt4py2
```

The call the program
```
./labelImg.py
```
----------------------------
## Bosch Datasets
Note that the bosch data set is in parts. You will need to put the partial zip files in a common directory and fuse them together.

Link to datasets:
https://hci.iwr.uni-heidelberg.de/node/6132/download/4230201a07fab2fb9acdcee4f2dd9cd8

**NOTE**: This link will be accessible until Wed, 07/04/2018 - 07:54. If you need
access after the link expires, don't hesitate to revisit the download page on
https://hci.iwr.uni-heidelberg.de/

**Training Data**
```
cd /home/cheryl/Development/TL_data/bosch/train
cat dataset_train_rgb.zip.* > dataset_train_rgb.zip
```
**Testing Data**
```
cd /home/cheryl/Development/TL_data/bosch/test
cat dataset_test_rgb.zip.* > dataset_test_rgb.zip
```


**convert dataset to TF Record file**
Bosch Training Set
```
python create_tf_record.py \
 --data_dir=/home/cheryl/Development/TL_model_make/data/bosch/train/train.yaml \
  --output_path=/home/cheryl/Development/TL_model_make/data/bosch/bosch_train_14labels.record \
   --label_map_path=/home/cheryl/Development/TL_model_make/data/bosch_label_map.pbtxt
```
Bosch Testing Set
```
python create_tf_record.py \
 --data_dir=/home/cheryl/Development/TL_model_make/data/bosch/test/test.yaml \
  --output_path=/home/cheryl/Development/TL_model_make/data/bosch/bosch_test_14labels.record \
   --label_map_path=/home/cheryl/Development/TL_model_make/data/bosch_label_map.pbtxt
```
-----
--------------
ProtoBuf 3.4
https://github.com/google/protobuf/releases/tag/v3.4.0
