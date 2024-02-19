# Visual-SLAM-using-ORBSLAM2
ORBSLAM2 is the second generation of the ORB visual slam program. It uses an ORB feature detector and descriptor (which is a FAST feature detector and BRIEF feature descriptor). There is a new version called ORBSLAM3 which was released in December 2021. We decided to use ORBSLAM2 since there was more documentation available, and since ORBSLAM3's main contribution was visual-inertial SLAM, which was not implemented for this project. For both datasets, we extracted camera calibration parameters from the provided rosbag and used the ONLINE_RECTIFICATION method (for stereo).

## Dataset
We implemented ORBSLAM2 on two data sets, the morning_stereo_rgb_ir_lidar_gps data set provided by the professor, and our own monocular data set recorded on the NUANCE Lincoln autonomous car. For the morning data set, we used the stereo setup for visual SLAM and compared it to the GPS data. For our own data set, we used only one monocular camera to do visual SLAM.


## Issues
<img width="467" alt="image" src="https://github.com/PanchalM19/Visual-SLAM-using-ORBSLAM2/assets/115374409/e0df721a-a2cc-42d1-b86a-3fff0224e363">

There were a few cases where the ORB_SLAM2 library failed and gave silent errors, where no error messages were produced yet the program continued to run without detecting features nor relocalizing. At this point in time the mapping module and keyframe selection stopped as well. There were a few cases in particular that caused issues for the algorithm: high rate turns, long stretches of straight road, and dynamic obstacles. When testing ORB-SLAM2 at faster turns, the keyframes were skewed instead of being rotated, indicating a skid in the car. For slower turns, ORBSLAM2 algorithm was able to localize and map the turn well, as shown in Figure 1. To improve performance, we increased the initial and minimum FAST thresholds, which increase the number of features that the FAST algorithm tries to detect at each grid cell of the image. We also increased the total number of features that the detector tries to extract per image.

<img width="582" alt="image" src="https://github.com/PanchalM19/Visual-SLAM-using-ORBSLAM2/assets/115374409/65cafd7e-6c5a-4ee0-87f0-a718b468676e">

Above figure shows the features that are extracted during a fast right turn. As the vehicle is turning, the previously tracked features are moving to the left of the image, as is expected with true rotation around the yaw axis. When the features are skewed highly left in the image and the vehicle continues to turn, the previously tracked features quickly move out of frame, causing very few features (if at all) to remain in the image after the vehicle completes the turn. This results in the silent error previously mentioned, and the program fails. Additionally, during acceleration and turns the constant velocity model, that is used for feature tracking, is violated. Finally, if the turn is very high rate then there is likely some blurring happening in the image which causes features to be less clear and/or have a lower confidence.

Another artificial workaround for this was replaying the rosbag from the beginning without re-initializing the algorithm (mentioned in the Robustness section). Mapping would then continue and ORBSLAM2 was able to plot map points and keyframes more accurately around the turn, since there were already features present. This also helped in continuing the mapping in instances when ORB-SLAM2 failed in mapping the first time.

### Monocular dataset issues - Loop Closure
In the case of the monocular camera, we faced challenges achieving loop closure due to issues such as z-drift, incorrect feature detection, and dynamic objects being mistaken for stationary features. With the stereo camera dataset, although good turns were obtained, loop closure remained elusive.
<img width="577" alt="image" src="https://github.com/PanchalM19/Visual-SLAM-using-ORBSLAM2/assets/115374409/e0bf20ac-ae9c-47cf-b881-bad0f706cda1">
<img width="577" alt="image" src="https://github.com/PanchalM19/Visual-SLAM-using-ORBSLAM2/assets/115374409/3c6c8397-4891-4772-a06c-5c6de29b4f16">

### Stereo dataset issues
For the stereo dataset, we observed some issues when the NUANCE car was moving along a long, straight road. It is likely because the ORBSLAM2 algorithm is not completely scale invariant. When features far away become closer as the car approaches, the features may not be tracked perfectly. This effect was mitigated by increasing the scale factor (from 1.2 to 1.5) and number of scale levels (from 8 to 15), which increases robustness with respect to scale.
<img width="546" alt="image" src="https://github.com/PanchalM19/Visual-SLAM-using-ORBSLAM2/assets/115374409/a273d90f-9b2a-4151-951a-71d411840717">

### Robustness Issues
ORBSLAM2 exhibited non-deterministic behavior, producing different maps/trajectories or failing altogether when run with the same parameters on a given dataset. We observed that rerunning the rosbag without restarting the algorithm often yielded better performance, though this relocalization was artificially induced.
<img width="375" alt="image" src="https://github.com/PanchalM19/Visual-SLAM-using-ORBSLAM2/assets/115374409/18064b9b-2b8d-4194-8a09-d2b426da8b9a">

### GPS Truth
The trajectory from the ORBSLAM2 run was exported in the TUM format and compared to the GPS data, which served as the ground truth. However, despite being one of the best runs, significant drift and scale issues were observed. Scaling the ORBSLAM2 trajectory by a factor of 50 was necessary for alignment, and loop closure was not achieved.
<img width="445" alt="image" src="https://github.com/PanchalM19/Visual-SLAM-using-ORBSLAM2/assets/115374409/dc9142f5-319f-4f30-957e-3f0415beb540">

## Areas for Improvement
Future work could involve analyzing algorithm failures and relative errors between runs to quantify determinism and determine error bounds. Algorithmic improvements might include incorporating machine learning modules to identify dynamic objects and limit feature extraction around them, as well as enhancing image contrast using filters like Clahe.
