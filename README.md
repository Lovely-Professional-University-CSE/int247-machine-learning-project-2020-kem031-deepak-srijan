Project By->


Name: Deepak Yadav      
Roll No: A33      
Registration No: 11716024

&

Name: Srijan Singh      
Roll No: B45      
Registration No: 11709122


Human Activity Detection for Surveillance Video Compression



Abstract

This is an implementation of the paper: https://www.leadingindia.ai/downloads/projects/VP/vp_8.pdf
With the increase in the number of anti-social activities that have been taking place, security has been given utmost importance lately. Many organizations have installed CCTVs for constant monitoring of people and their interactions. For a developed country with a population of 64 million, every person is captured by a camera ~ 30 times a day. A lot of video is generated and stored for certain time duration (India: 30 days). A 704x576 resolution image recorded at 25fps will generate roughly 20GB per day. Since constant monitoring of data by humans to judge if the events are abnormal is a near impossible task as it requires a workforce and their constant attention. This creates a need to automate the same. Also, there is a need to show in which frame and which parts of it contain the unusual activity which aid the faster judgment of that unusual activity being abnormal. The method involves generating motion influence map for frames to represent the interactions that are captured in a frame. The main characteristic feature of the proposed motion influence map is that it effectively depicts the motion characteristics of the movement speed, movement direction, and size of the objects and their interactions within a frame sequence. It further extracts frames of high motion influence values and compares with the testing frames to automatically detect global and local unusual activities.
Dataset:	http://www.svcl.ucsd.edu/projects/anomaly/
To detect the abnormal/unusual human activities in a video



Requirements

Python openCV 3 for python numpy 1.7 FFmpeg



Implementation

The code is divided into 5 modules, optflowofblocks, motioninfluencegenerator, createmegablocks, training and testing. In this section, a method for representing motion characteristics is described for the detection and localization of unusual activities within a crowded scene. Here, we should note that we considered two types of unusual activities:
Local and Global. Local unusual activities occur within a relatively small area. Different motion patterns may appear in a portion of the frame, such as the unique appearance of nonhuman objects or the fast movement of a person when most of the other pedestrians are walking slowly. Global unusual activities occur across the frame, for example, when every pedestrian within a scene starts to run suddenly to escape from the scene.
1) Data Input and Pre-Processing The video file is given as an input to the system, which is subjected to pre processing. A video is treated as sequence of images called frames and these frames are processed sequentially. An RGB frame is first converted to gray scale. A gray scaled image consists of only the intensity information of the image rather than the apparent colors. RGB vector is 3 dimensional (as it consists of values of colors red, green and blue) whereas gray scaled vector is one dimensional.
2) Optical Flow After the pre-processing step, for each frame in the video, optical-flow is computed for each pixel in a frame using FarneBack algorithm. Optical flow is the pattern of apparent motion of objects, surfaces, and edges in a visual scene caused by the relative motion between an observer and the scene. Figure 5.4 shows the optical-flow of each pixel in the image of a ball moving upwards. Optical-flow is a vector of the form (r , θ), where, r represents the magnitude of each pixel and θ represents the direction in which the each pixel has moved relative to the corresponding pixel in the previous frames. The calcOpticalFlowFarneback( ) function in openCV computes a dense optical flow using the Gunnar Farneback’s algorithm .
3) Optical-Flow of blocks
3.1) Dividing a frame into blocks After computing the optical flows for every pixel within a frame, we partition the frame into M by N uniform blocks without loss of generality, where the blocks can be indexed by {B1, B2, . . . , BMN}. Figure 5.5 shows a frame of size 240 x 320 divided into 48 blocks where each block is of the size 20 x 20.
3.2) Calculating Optical-Flow of each block After dividing the frames into blocks, we compute optical-flow of each block by computing the average of optical-flows of all the pixels constituting a block. Figure 5.6 gives the formula for calculating the optical-flow of a block. Here, bi denotes an optical flow of the i th block, J is the number of pixels in a block, and f ji denotes an optical flow of the j th pixel in the i th block. Optical-flow of a block is a vector (r , θ) which represents how much each block has moved and in which direction compared to the corresponding block in the previous frames.
4) Motion Influence Map The movement direction of a pedestrian within a crowd can be influenced by various factors, such as obstacles along the path, nearby pedestrians, and moving carts. We call this interaction characteristic as the motion influence. We assume that the blocks under influence on which a moving object can affect are determined by two factors:
1.	the motion direction
2.	the motion speed. The faster an object moves, the more neighboring blocks that are under the influence of the object. Neighboring blocks have a higher influence than distant blocks.
5) Algorithm for creating a motion influence map INPUT: B ← motion vector set, S ← block size, K ← a set of blocks in a frame  OUTPUT: H ← motion influence map  H j (j   K) is set to zero at the beginning of each frame  ∈ for all i   K do  Td = bi × S;  Fi/2 = bi + ? 2; −Fi/2 = bi − ? 2 ;  for all j   K do  if i = j then  Calculate the Euclidean distance D(i, j) between bi and bj  if D(i, j) < Td then  Calculate the angle ?ij between bi and bj  if − Fi 2 < ?ij < Fi 2 then  H j ( bi) = H j ( bi) + exp− D(i,j)bi  end if  end if  end if  end for  end for
6) Feature Extraction In the motion influence map, a block in which an unusual activity occurs, along with its neighboring blocks, has unique motion influence vectors. Furthermore, since an activity is captured by multiple consecutive frames, we extract a feature vector from a cuboid defined by n × n blocks over the most recent t number of frames.
6.1) Creating Megablocks Frames are partitioned into non-overlapping mega blocks, each of which is a combination of multiple motion influence blocks. The Motion Influence value of a Megablock is the summation of motion influence values of all the smaller blocks constituting a larger block.
6.2) Extracting Features After the recent ‘t’ number of frames are divided into Megablocks, for each megablock, an 8 × t-dimensional concatenated feature vector is extracted across all the frames. For example, we take mega block (1,1) of all the frames ('t' number of frames) and concatenate their feature vectors, to create a concatenated feature vector for block (1,1).
7) Clustering For each mega block, we perform clustering using the spatio-temporal features and set the centers as codewords. That is, for the (i , j)th mega block, we have K codewords, {w(i, j ) k }K k=1. Here, we should note that in our training stage, we use only video clips of normal activities. Therefore, the codewords of a mega block model the patterns of usual activities that can occur in the respective area.
8) Testing phase Now that we have generated the codewords for normal activities, it is time to test the generated model with a test dataset which contains unusual activities.
8.1) Minimum Distance Matrix In the testing state, after extracting the spatio-temporal feature vectors for all mega blocks, we construct a minimum distance matrix E over the mega blocks, in which the value of an element is defined by the minimum Euclidean distance between a feature vector of the current test frame and the codewords in the corresponding mega block.
8.2) Frame level detection of unusual activities in a minimum-distance matrix, the smaller the value of an element, the less likely an unusual activity is to occur in the respective block. On the other hand, we can say that there are unusual activities in t consecutive frames if a higher value exists in the minimum-distance matrix. Therefore, we find the highest value in the minimum-distance matrix as the frame representative feature value. If the highest value of the minimum distance matrix is larger than the threshold, we classify the current frame as unusual.
8.3) Pixel level detection of unusual activities Once a frame is detected as unusual, we compare the value of the minimum distance matrix of each megablock with the threshold value. If the value is larger than the threshold, we classify that block as unusual. Figure 5.19 shows an example of pixel level unusual activity detection.








References

[1]	Human Activity Detection for Surveillance Video Compression - https://www.leadingindia.ai/downloads/projects/VP/vp_8.pdf
[2]	Human Detection and Tracking in Video Surveillance System - https://www.vocal.com/video/human-detection-and-tracking-in-video-surveillance-system
[3]	Dong-Gyu Lee, Heung-Il Suk, Sung-Kee Park and Seong-Whan Lee “Motion Influence Map for Unusual Human Activity Detection and Localization in Crowded Scenes” IEEE transactions on circuits and systems for video technology, vol. 25, no. 10, October 2015
[4]	Data set – http://mha.cs.umn.edu/Movies/Crowd-Activity-All.avi
[5]	Data set - http://www.svcl.ucsd.edu/projects/anomaly
[6] 	T. Xiang and S. Gong, “Video behavior profiling for anomaly detection,” IEEE Trans. Pattern Anal. Mach. Intell., vol. 30, no. 5, pp. 893–908, May 2008.
[7]	F. Jiang, J. Yuan, S. A. Tsaftaris, and A. K. Katsaggelos, “Anomalous video event detection using spatiotemporal context,” Comput. Vis. Image Understand., vol. 115, no. 3, pp. 323–333, 2011.
[8]	B. D. Lucas and T. Kanade, “An iterative image registration technique with an application to stereo vision,” in Proc. 7th Int. Joint Conf. Artif. Intell., San Francisco, CA, USA, Aug. 1981, pp. 674–679.
[9]	OpenCV Python documentation at http://docs.opencv.org/3.0-beta/doc/py_tutorials/py_tutorials.html 
[10]	OpenCV references at  http://opencv-python-tutroals.readthedocs.io/en/latest
