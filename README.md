Serverless Face Recognition
---

### Description:

Serverless compute platforms such as Amazon Web Services (AWS) lambda were
intended to be used for web microservices and to asynchronously handle events
generated by AWS services (S3, DynamoDB, etc.). However, AWS allows users to
upload arbitrary linux binaries along with their lambda functions; and these
binaries can executed during a lambda invocation! We take advantage of this
*feature* and run a full-blown deep neural network based face recognition tool
on AWS lambda which can be used to detect and recognize people in jpeg images.

See our [blog post](BLOG.md) for a detailed description of the system and the
machine learning methods that underpin this work. And if this project sparks new
ideas or influences your research, please cite us (see below)!

### Prerequisites:

1. You must **have an AWS account** to run this code. Create an account [here](http://aws.amazon.com) if you do not have one already.
2. You must also **create an AWS access key** to use with awscli (a commmandline tool for managing your AWS account). To do this follow the instructions on [this page](http://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html).

### Setup:

```bash
# install pip and virtualenv if necessary
$ curl -O https://raw.github.com/pypa/pip/master/contrib/get-pip.py
$ python get-pip.py
$ pip install virtualenv

# clone the repo and install awscli and boto3
$ git clone https://github.com/excamera/serverless-face-recognition.git
$ cd serverless-face-recognition
$ virtualenv .venv
$ source .venv/bin/activate
$ pip install -r requirements.txt # installs awscli and boto3

# setup the awscli (create an access key and enter details below)
$ aws configure
AWS Access Key ID []: ****************ANWQ
AWS Secret Access Key []: ****************JawS
Default region name []: us-east-1
Default output format [None]:

# download the binary blobs necessary for the face recognizer
$ ./blobs/get_blobs.sh
downloading... (lots of output)

# deploy the face recognition lambdas to your account
$ ./install_lambdas.sh
creating bucket
uploading dependencies to s3
upload: blobs/deps.zip to s3://9a480eb7-be5b-4e15-81fe-4de41323907e/deps.zip
upload: blobs/root-495M-2017-02-06.tar.gz to s3://9a480eb7-be5b-4e15-81fe-4de41323907e/root-495M-2017-02-06.tar.gz
upload: blobs/lfw_face_vectors.csv.gz to s3://9a480eb7-be5b-4e15-81fe-4de41323907e/lfw_face_vectors.csv.gz
adding 'lambda-executor' role
adding 'prepare-face-recognizer' lambda
adding 'recognize-face' lambda
Done!
```

```bash
# remove the deployed lambdas when you are done
$ ./uninstall_lambdas.sh
removing s3 bucket
delete: s3://9a480eb7-be5b-4e15-81fe-4de41323907e/lfw_face_vectors.csv.gz
delete: s3://9a480eb7-be5b-4e15-81fe-4de41323907e/deps.zip
delete: s3://9a480eb7-be5b-4e15-81fe-4de41323907e/root-495M-2017-02-06.tar.gz
removing 'lambda-executor' role
removing 'prepare-face-recognizer' lambda
removing 'recognize-face' lambda
Done!
```

### Usage (code located in `scripts` directory):

```bash
# generating a model for a face
$ ./scripts/train_face_recognizer.py --help
description:
    returns the augmented feature vectors for the face in IMAGE.csv.
    Assume exactly one face in the image.

# using a model to recognize a face in an image
$ ./scripts/recognize_face.py --help
usage: ./recognize_face.py FACEVECTORS.csv IMAGE.jpg
description:
    returns `true` or `false` if the face used to generate
    FACEVECTORS.csv is present in IMAGE.csv.
```

### Simple example (code located in `scripts` directory):

To see how things work let's jump into a simple example. The following snippet
will generate a model to recognize [John Emmons'](http://johnemmons.com) face
then use the model to check if John's face is in a photo.


#### Model generation


Training image              |
:---------------------------:
![](blog/pics/john0.jpg) |

```bash
# generate a model for John's face
$ cd scripts
$ ./train_face_recognizer.py pics/john0.jpg > john.model.csv
reading input file
connecting to AWS lambda
waiting for remote lambda worker to finish
reading result from remote lambda worker
```

#### Recognition

Training image              | Test image 
:--------------------------:|:-------------------------:
![](blog/pics/john0.jpg) | ![](blog/pics/john1.jpg)

```bash
# use the model to see if John's face is present in an image
$ ./recognize_face.py john.model.csv pics/john1.jpg
reading input files
connecting to AWS lambda
waiting for remote lambda worker to finish
reading result from remote lambda worker
True
```

Training image              | Test image 
:--------------------------:|:-------------------------:
![](blog/pics/john0.jpg) | ![](blog/pics/john2.jpg)

```bash
$ ./recognize_face.py john.model.csv pics/john2.jpg
reading input files
connecting to AWS lambda
waiting for remote lambda worker to finish
reading result from remote lambda worker
True
```
Training image              | Test image 
:--------------------------:|:-------------------------:
![](blog/pics/john0.jpg) | ![](blog/pics/jim-carrey.jpg)

```bash
$ ./recognize_face.py john.model.csv pics/jim-carrey.jpg
reading input files
connecting to AWS lambda
waiting for remote lambda worker to finish
reading result from remote lambda worker
False
```

### About Us:

The main author of this demo is [John Emmons](http://johnemmons.com); John is a
computer science PhD student at Stanford University whose research is at the
intersection of computer systems and machine learning (primarily computer
vision).

If you like this work and want to learn more about it, feel free to drop John a
line a [mail@johnemmons.com](mailto:mail@johnemmons.com). And if this work
inspires you to do your own related work, please cite our research group! (see
below)

### Citation:

```
@inproceedings {201559,
  author = {Sadjad Fouladi and Riad S. Wahby and Brennan Shacklett and Karthikeyan Vasuki Balasubramaniam and William Zeng and Rahul Bhalerao and Anirudh Sivaraman and George Porter and Keith Winstein},
  title = {Encoding, Fast and Slow: Low-Latency Video Processing Using Thousands of Tiny Threads},
  booktitle = {14th USENIX Symposium on Networked Systems Design and Implementation (NSDI 17)},
  year = {2017},
  isbn = {978-1-931971-37-9},
  address = {Boston, MA},
  pages = {363--376},
  url = {https://www.usenix.org/conference/nsdi17/technical-sessions/presentation/fouladi},
  publisher = {USENIX Association},
 }
 ```