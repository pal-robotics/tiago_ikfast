# tiago_ikfast

A version of the arm URDF simplified for ikfast usage.

To get to this version I did:

## Get the latest version of the robot model
    cd ~/ikfast_ws/src/
    git clone https://github.com/pal-robotics/tiago_robot
    cd ..
    catkin_make
    source devel/setup.bash

In the folder `tiago_description/robot` we can find `tiago_steel.urdf.xacro` (for example, any will do). We remove everything that is not the arm and call it `only_arm.urdf.xacro`.

First we generate a URDF of it:

    rosrun xacro xacro only_arm.urdf.xacro > only_arm.urdf

We need to simplify it without gazebo nor tranmission tags.

So we'll do (`sed` didn't execute the regular expression as expected, so this did the trick):

    cat only_arm.urdf | python -c "import sys,re;sys.stdout.write(re.sub('<gazebo(.|\n)*?<\/gazebo>*', '', sys.stdin.read()))" > only_arm.urdf.nogazebo 

And then:

    cat only_arm.urdf.nogazebo | python -c "import sys,re;sys.stdout.write(re.sub('<transmission(.|\n)*?<\/transmission>*', '', sys.stdin.read()))" > only_arm.urdf.clean

Finally we need to add `torso_lift_link` as it's the first link of the arm.

    <link name="torso_lift_link" />
Is added just after the line:

    <robot name="tiago" xmlns:xacro="http://ros.org/wiki/xacro">

## Create .DAE for the arm
As there is [this bug #45](https://github.com/ros/robot_model/issues/45) still unresolved, we'll need to do one more trick to our model:

    sed 's/radius="0.005/radius="0.01/' only_arm.urdf.clean > only_arm.urdf.final

    rosrun collada_urdf urdf_to_collada only_arm.urdf.final arm.dae

We can check if the model makes any sense with `meshlab`

    sudo apt-get install meshlab
    meshlab arm.dae

![arm.dae image](https://raw.githubusercontent.com/pal-robotics/tiago_ikfast/master/arm.dae.png)
Beautiful. Haha.

## Last tunning
Taken from [Baxter tutorial](http://sdk.rethinkrobotics.com/wiki/Custom_IKFast_for_your_Baxter):
The resulting DAE also needs some tuning - specifically the rotational portions of the description. Specifically, we will need to round values logically (ie. 119.9999999999999 to 120.0) This will be addressed in the next step.

We need:

    sudo apt-get install ros-indigo-moveit-ikfast

Then we can:

    rosrun moveit_ikfast round_collada_numbers.py arm.dae arm.rounded.dae 5

