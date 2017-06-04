## Project: Search and Sample Return

#### Medium Article:  https://medium.com/@christiansafka/udacity-robotics-nanodegree-project-1-search-and-sample-return-5bdd2d82788f
---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./output/train_images.png
[image2]: ./output/warped_train_images.png
[image3]: ./output/color_threshold_train_images.png
[image4]: ./output/rover_centric_train_images.png
[image5]: ./output/test_mapping.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
I selected two images from training data I recorded, one for testing the color filter on ground, and the other for a rock sample.
![One open area, and one picture with a rock sample][image1]

Here are the above images after the perspective transform to top-down view
![Warped images][image2]

I modified the color_threshold function to include a rgb_thresh_max parameter, to set maximum values for the RGB thresholds in addition to the minimums.  Here is the output on the same training images
![Color thresholded images][image3]

And into rover-centric coordinates, with the mean angle drawn
![Rover-centric coordinates][image4]

#### 2. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 


![test_mapping.mp4 in ./output folder][image5]
### Autonomous Navigation and Mapping


##### Perception
 The first change was to again add the rgb_thresh_max to the color_thresh function.  In my perception_step function, I set the Rover.vision_image red channel to rock_sample color threshhold output, green channel for walls, and blue channel for navigable terrain. I then converted all three of the threshholded outputs into rover-centric coordinates.  The navigable terrain and wall coordinates were transformed to world coordinates, and applied as follows:
 
 Blue channel of worldmap receives += 255 for all navigable terrain and -= 255 for walls

 Red channel of worldmap receives += 255 for all walls and -= 255 for navigable terrain
 
 I found that this method gave me the highest fidelity.  As for the rover-centric rock sample coordinates, they were used to check if there is a sample in the image, and determine the angle the rover needs to turn to get to that sample.  If a sample was in sight, the rovers mode was set to 'rock_visible', a custom mode I handle in the decision_step.  Further, if no rock sample is present, the navigable terrain rover-centric coordinates are converted to polar coordinates, and set to the rovers nav_dists and nav_angles variables.

##### Decision
In the decision_step function, I added a conditional to check if the rover is currently picking up a sample, and if so, to stop any movement and set the mode to 'stop'.  Next I added the check if a sample is near the rover.  If there is, then send the pickup sample signal to the rover, increment the samples_found count, stop moving, and set the mode to 'stop'.  
The last conditional I added was a check if the rover mode was 'rock_visible'.  If it was, then max out the throttle and steer toward the rock, regardless if there is not enough navigable terrain to continue much farther.  This was done because the rock samples are close to the walls, and the normal 'forward' mode tells the robot to stop and turn away from walls.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

My simulator was running at ~30 FPS with 1280x768 resolution and 'Beautiful' graphics quality.

My approach is working for the most part, and is able to map a good percentage of the environment it covers with ~85 % fidelity. I wasn't able to test the simulator for longer than a couple minutes due to a memory leak in the simulator. My rover can also successfully see and pickup rock samples, but it does not yet return them to its original position.  I could have included that the 'easy' way, waiting for the robot to randomly enter its starting position, but I would like to implement it in a more correct way in the future.  

A few drawbacks of my approach include that there is no intelligent algorithm for exploration of the map, so the rover will cover the same spots several times, and miss some spots where the mean angle does not take it.  There is also nothing to prevent the rover from driving in a circle for a while when following the mean angle of navigable terrain.  The last caveat of my rover is that in the case of getting stuck, there is no 'get unstuck' routine.

I would like to revisit this project in the future, and try some end to end deep learning for this rover.  I would also like to fix my current approach in the near future by making my rover a wall crawler.  This project was a lot of fun!
